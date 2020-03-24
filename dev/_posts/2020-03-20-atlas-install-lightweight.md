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

- ** 20200320 build가 안됨. **
- ** 일단 repo가 변경 된 듯, 추가해서 해결 **
- ** https://issues.apache.org/jira/browse/ATLAS-3671 이슈는 남겼는데 답변이 있을 지 의문**

