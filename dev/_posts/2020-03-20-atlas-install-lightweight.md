# atlas 2.0.0 설치 with docker(lightweight)

- base linux image : openjdk-8-jdk-alpine
- 기존에 했던 이미지가 2.85G 로 너무 커서 경량화 시킴.
- as-is 
  - process는 docker 위에 source 파일을 받아 build
  - mvn, atlas 등 사이즈가 너무 큼
- to-be
  - build는 local에서 
  - build된 binary directory를 copy


## base images
```
FROM java:openjdk-8-jdk-alpine
MAINTAINER aiden@lezhin.net

ARG ATLAS_VERSION=2.0.0

ENV PATH $PATH:/usr/lib/altas/bin

WORKDIR /usr/lib

RUN set -euxo pipefail && \
    apk add --no-cache bash python && \
    addgroup -S atlas && \
    adduser -S atlas -G atlas && \
    mkdir -p /usr/lib/atlas && \
    chown -R "atlas:atlas" /usr/lib/atlas

COPY --chown=atlas:atlas apache-atlas-2.0.0 /usr/lib/atlas

EXPOSE 21000
USER atlas:atlas
CMD ["/bin/bash", "-c", "/usr/lib/atlas/bin/atlas_start.py"]

```

- **20200320 build가 안됨.**
- **일단 repo가 변경 된 듯, 추가해서 해결**
- **https://issues.apache.org/jira/browse/ATLAS-3671 이슈는 남겼는데 답변이 있을 지 의문(오래 전 부터 답변이 없음. 포기한 프로젝트인가..)**


## dockerfile
```
FROM java:openjdk-8-jdk-alpine
MAINTAINER aiden@lezhin.net

ARG ATLAS_VERSION=2.0.0

ENV PATH $PATH:/usr/lib/atlas

WORKDIR /usr/lib

RUN set -euxo pipefail && \
    apk add --no-cache bash python

COPY  apache-atlas-2.0.0 /usr/lib/atlas
COPY  apache-hive-3.1.2-bin /usr/lib/hive
COPY  hive-env.sh /usr/lib/hive/conf
COPY  hive-site.xml /usr/lib/hive/conf
COPY  apache-atlas-2.0.0/conf/atlas-application.properties /usr/lib/hive/conf
COPY  hadoop-3.1.2 /usr/lib/hadoop

EXPOSE 21000
ENV SOLR_VAR_DIR=/usr/lib/atlas/solr
ENV MANAGE_LOCAL_HBASE=true
ENV MANAGE_LOCAL_SOLR=true
ENV MANAGE_EMBEDDED_CASSANDRA=false
ENV MANAGE_LOCAL_ELASTICSEARCH=false
ENV ATLAS_HOME_DIR=/usr/lib/atlas
ENV ATLAS_CONF=/usr/lib/atlas/conf
ENV ATLAS_LOG_DIR=/usr/lib/atlas/logs
ENV HIVE_HOME=/usr/lib/hive
ENV HADOOP_HOME=/usr/lib/hadoop

CMD ["/bin/bash", "-c", "/usr/lib/atlas/bin/atlas_start.py; tail -fF /usr/lib/atlas/logs/application.log"]
```

- **issue**
	- build 안 됨. pom.xml repo 추가
	- atlas_config.py, atlas_start.py 소스 수정 후 start 
	- embedded 된 solr, hbase를 사용하려다 보니 atlas로 띄우면 권한 문제가 발생함. 해서 root로
	- hadoop binary와 hbase 가 안 맞음. hbase 시작 할 때 에러 뿜뿜
```
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
```
	- 
