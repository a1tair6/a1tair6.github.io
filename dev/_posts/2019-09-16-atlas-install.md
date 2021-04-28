# atlas 2.0.0 설치 with docker
- base linux image : alpine

## base images
```
FROM java:openjdk-8-jdk-alpine
MAINTAINER aiden

RUN apk update && \
    apk add wget && \
    apk add bash

# setting
RUN adduser -D atlas && \
    mkdir /home/atlas/installed && \
    mkdir /home/atlas/eco_service && \
    mkdir -p /home/atlas/installed/hadoop && \
    chown atlas.atlas -R /home/atlas/installed/hadoop && \
    chown -R atlas.atlas /home/atlas/installed /home/hadoop/eco_service && \
    su atlas && \
    cd /home/atlas/installed

WORKDIR /home/atlas/installed
```

- ** binary 를 build 하기 위해 docker 안에서 명령 실행 **
- ** docker command로 해봤지만 size가 너무 커짐 **


## docker run
```
docker run -it {images_id} bash
```

## maven, hadoop, hive, atlas binary download

```sh
wget http://xenia.sote.hu/ftp/mirrors/www.apache.org/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz && \
tar -xvf apache-maven-3.3.9-bin.tar.gz && \
wget http://apache.mirror.cdnetworks.com/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz && \
tar -xvf hadoop-3.1.2.tar.gz && \
rm hadoop-3.1.2.tar.gz
wget http://apache.mirror.cdnetworks.com/atlas/2.0.0/apache-atlas-2.0.0-sources.tar.gz && \
tar -xvf apache-atlas-2.0.0-sources.tar.gz && \
rm apache-atlas-2.0.0-sources.tar.gz
wget http://apache.mirror.cdnetworks.com/hive/hive-3.1.1/apache-hive-3.1.1-bin.tar.gz && \
tar -xvf apache-hive-3.1.1-bin.tar.gz && \
rm apache-hive-3.1.1-bin.tar.gz
```

## hadoop, hive, atlas soft link
```sh
ln -s /home/atlas/installed/hadoop-3.1.2 /home/atlas/eco_service/hadoop
ln -s /home/atlas/installed/hive-3.1.1 /home/atlas/eco_service/hive
ln -s /home/atlas/installed/apache-atlas-sources-2.0.0 /home/atlas/eco_service/atlas

export JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
export MAVEN_HOME=/home/atlas/installed/apache-maven-3.3.9
export HIVE_HOME=/home/atlas/eco_service/hive
export ATLAS_HOME=/home/atlas/eco_service/atlas
export HADOOP_HOME=/home/atlas/eco_service/hadoop
export LD_LIBRARY_PATH="$HADOOP_HOME/lib/native"
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HIVE_HOME/bin:$SPARK_HOME/bin:$LD_LIBRARY_PATH:$OOZIE_HOME/bin:$MAVEN_HOME/bin
```

## atlas build
```sh
cd /home/atlas/installed/apache-atlas-sources-2.0.0 && \
mvn clean -DskipTests package -Pdist,embedded-hbase-solr
cd /home/atlas/installed/apache-atlas-sources-2.0.0/distro/target && \
cp apache-atlas-2.0.0-server.tar.gz /home/atlas/installed && \
cd /home/atlas/installed/ && \
mv apache-atlas-sources-2.0.0 apache-atlas-sources-2.0.0-build && \
tar -xvf apache-atlas-2.0.0-server.tar.gzz && \
rm apache-atlas-2.0.0-server.tar.gz && \
cd /home/atlas/eco_service/apache-atlas-2.0.0
```


## docker images 생성
```sh
docker commit -m "initial atlas" -a "user_id" {container_id} {tag_name}
```

## docker images push
```sh
docker push {tag_name}
```


- ** 여기까지 기본 images **
- ** 설정파일 보안상 image version 나눔 **

## hadoop, atlas, hive config setting
```
FROM {docker url}/dw-util-atlas:0.1
MAINTAINER aiden

USER atlas
COPY --chown=atlas:atlas hive-site.xml /home/atlas/eco_service/hive/conf/
COPY --chown=atlas:atlas hive-env.sh /home/atlas/eco_service/hive/conf/
COPY --chown=atlas:atlas yarn-site.xml /home/atlas/eco_service/hadoop/etc/hadoop/
COPY --chown=atlas:atlas hdfs-site.xml /home/atlas/eco_service/hadoop/etc/hadoop/
COPY --chown=atlas:atlas core-site.xml /home/atlas/eco_service/hadoop/etc/hadoop/
COPY --chown=atlas:atlas mapred-site.xml /home/atlas/eco_service/hadoop/etc/hadoop/
COPY --chown=atlas:atlas hadoop-env.sh /home/atlas/eco_service/hadoop/etc/hadoop/
COPY --chown=atlas:atlas hbase-site.xml.template /home/atlas/eco_service/atlas/conf/hbase/

#chmod
RUN chmod -R 755 /home/atlas/eco_service/hive/conf/hive-site.xml
RUN chmod -R 755 /home/atlas/eco_service/hive/conf/hive-env.sh
RUN chmod -R 755 /home/atlas/eco_service/hadoop/etc/hadoop/yarn-site.xml
RUN chmod -R 755 /home/atlas/eco_service/hadoop/etc/hadoop/hdfs-site.xml
RUN chmod -R 755 /home/atlas/eco_service/hadoop/etc/hadoop/core-site.xml
RUN chmod -R 755 /home/atlas/eco_service/hadoop/etc/hadoop/mapred-site.xml
RUN chmod -R 755 /home/atlas/eco_service/hadoop/etc/hadoop/hadoop-env.sh

# Atlas web port
EXPOSE 21000

USER atlas
RUN mkdir /home/atlas/eco_service/atlas/logs && touch /home/atlas/eco_service/atlas/logs/application.log
RUN cp /home/atlas/eco_service/atlas/conf/atlas-application.properties /home/atlas/eco_service/hive/conf
ENV ATLAS_HOME /home/atlas/eco_service/atlas
ENV PATH $ATLAS_HOME/bin:$PATH
RUN echo "export JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk" >> /home/atlas/eco_service/atlas/conf/atlas-env.sh
RUN echo "export ATLAS_CONF=/home/atlas/eco_service/atlas/conf" >> /home/atlas/eco_service/atlas/conf/atlas-env.sh
WORKDIR /home/atlas/eco_service/atlas

CMD ["python2.7", "/home/atlas/eco_service/atlas/bin/atlas_start.py"]
CMD ["/bin/bash", "-c", "tail -f /home/atlas/eco_service/atlas/logs/application.log"]
```


## hadoop-env.sh
```sh
export JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
export HADOOP_HOME=/home/atlas/eco_service/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export LD_LIBRARY_PATH=${HADOOP_HOME}/lib/native
export HADOOP_COMMON_HOME=${HADOOP_HOME}/share/hadoop/common
export HADOOP_HDFS_HOME=${HADOOP_HOME}/share/hadoop/hdfs
export HADOOP_MAPRED_HOME=${HADOOP_HOME}/share/hadoop/mapreduce
export HADOOP_YARN_HOME=${HADOOP_HOME}/share/hadoop/yarn
export YARN_HOME=${HADOOP_HOME}/share/hadoop/yarn
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${HADOOP_CONF_DIR}:${HADOOP_COMMON_HOME}/*:${HADOOP_COMMON_HOME}/lib/*:${HADOOP_HDFS_HOME}/*:${HADOOP_HDFS_HOME}/lib/*:${HADOOP_MAPRED_HOME}/*:${HADOOP_MAPRED_HOME}/lib/*:${HADOOP_YARN_HOME}/*:${HAOOP_YARN_HOME}/lib/*:$HADOOP_HOME/share/hadoop/tools/lib/*
```

## Apache Atlas Hook & Bridge for Apache Hive
- import-hive.sh 실행