# hue 4.5.0 소개 & 설치

== Hue is an open source SQL Workbench for Data Warehouses ==

- 말 그대로 Data Warehouse를 위한 SQL Workbench 오픈 소스입니다.
- 주로 oozie property와 workflow를 ui로 생성 해주는 이점이 있습니다.
- 다양한 설치 방법이 있지만 hue docker images 사용합니다.

- hue docker url : [https://github.com/cloudera/hue/tree/master/tools/docker](https://github.com/cloudera/hue/tree/master/tools/docker)

## dockerfile
```
FROM gethue/hue:4.5.0
MAINTAINER aiden

# copy hue config
COPY hue.ini /usr/share/hue/desktop/conf
COPY hue.ini /usr/share/hue/desktop/conf/z-defaults.ini

# Hue web port
# EXPOSE 8888

WORKDIR /usr/share/hue

# hadoop, spark, hive, livy, solr 설치
# livy, solr 등 나눠서 설치 하는 것을 권고 합니다.
RUN apt-get install wget
RUN apt-get update
RUN apt-get -y install openjdk-8-jre
RUN wget "http://apache.tt.co.kr/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz" && \
tar -xvf "hadoop-3.1.2.tar.gz" && \
rm -f "hadoop-3.1.2.tar.gz" && \
ln -sv "hadoop-3.1.2" hadoop
RUN wget "http://archive.apache.org/dist/spark/spark-2.4.2/spark-2.4.2-bin-hadoop2.7.tgz" && \
tar -xvf "spark-2.4.2-bin-hadoop2.7.tgz" && \
rm -f "spark-2.4.2-bin-hadoop2.7.tgz" && \
ln -sv "spark-2.4.2-bin-hadoop2.7" spark
RUN wget "http://apache.tt.co.kr/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz" && \
tar -xvf "apache-hive-3.1.2-bin.tar.gz" && \
rm -f "apache-hive-3.1.2-bin.tar.gz" && \
ln -sv "apache-hive-3.1.2-bin" hive
RUN apt-get install unzip && \
wget "http://apache.mirror.cdnetworks.com/incubator/livy/0.6.0-incubating/apache-livy-0.6.0-incubating-bin.zip" && \
unzip "apache-livy-0.6.0-incubating-bin.zip" && \
rm -f "apache-livy-0.6.0-incubating-bin.zip" && \
ln -sv "apache-livy-0.6.0-incubating-bin" livy
RUN wget "http://apache.tt.co.kr/lucene/solr/8.2.0/solr-8.2.0.tgz" && \
tar -xvf "solr-8.2.0.tgz" && \
rm -f "solr-8.2.0.tgz" && \
ln -sv "solr-8.2.0" solr

# copy etc config
COPY yarn-site.xml /usr/share/hue/hadoop/etc/hadoop
COPY hdfs-site.xml /usr/share/hue/hadoop/etc/hadoop
COPY core-site.xml /usr/share/hue/hadoop/etc/hadoop
COPY mapred-site.xml /usr/share/hue/hadoop/etc/hadoop
COPY hadoop-env.sh /usr/share/hue/hadoop/etc/hadoop
COPY livy.conf /usr/share/hue/livy/conf
COPY spark-defaults.conf /usr/share/hue/spark/conf
COPY spark-env.sh /usr/share/hue/spark/conf
COPY hive-site.xml /usr/share/hue/hive/conf
COPY hive-site.xml /usr/share/hue/spark/conf

ENV HADOOP_HOME /usr/share/hue/hadoop
ENV SPARK_HOME /usr/share/hue/spark
ENV HIVE_HOME /usr/share/hue/hive
ENV PATH $SPARK_HOME/bin:$PATH

# EXPOSE 8998

# CMD ["/bin/bash/", "livy/bin/livy-server", "start"]

WORKDIR /usr/share/hue
COPY --chown=root:root start_script.sh /usr/share/hue
USER root
EXPOSE 8888
ENTRYPOINT ["bash", "start_script.sh"]
```


## hadoop-env.sh
```sh
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export HADOOP_HOME=/usr/share/hue/hadoop
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export LD_LIBRARY_PATH=${HADOOP_HOME}/lib/native
export HADOOP_COMMON_HOME=${HADOOP_HOME}/share/hadoop/common
export HADOOP_HDFS_HOME=${HADOOP_HOME}/share/hadoop/hdfs
export HADOOP_MAPRED_HOME=${HADOOP_HOME}/share/hadoop/mapreduce
export HADOOP_YARN_HOME=${HADOOP_HOME}/share/hadoop/yarn
export YARN_HOME=${HADOOP_HOME}/share/hadoop/yarn
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${HADOOP_CONF_DIR}:${HADOOP_COMMON_HOME}/*:${HADOOP_COMMON_HOME}/lib/*:${HADOOP_HDFS_HOME}/*:${HADOOP_HDFS_HOME}/lib/*:${HADOOP_MAPRED_HOME}/*:${HADOOP_MAPRED_HOME}/lib/*:${HADOOP_YARN_HOME}/*:${HAOOP_YARN_HOME}/lib/*:$HADOOP_HOME/share/hadoop/tools/lib/*
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/usr/share/hue/hadoop/lib/native"
```


## hive-env.sh
```sh
export HADOOP_HEAPSIZE=4096
#
# Larger heap size may be required when running queries over large number of files or partitions.
# By default hive shell scripts use a heap size of 256 (MB).  Larger heap size would also be
# appropriate for hive server.


# Set HADOOP_HOME to point to a specific hadoop install directory
HADOOP_HOME=${HADOOP_HOME}

# Hive Configuration Directory can be controlled by:
export HIVE_CONF_DIR=${HIVE_HOME}/conf
```


<details>
  <summary>start_script.sh</summary>
  

```sh
#!/bin/bash

# Start the solr process
./solr/bin/solr start -force
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start solr: $status"
  exit $status
fi

# Start the livy process
./livy/bin/livy-server start -force
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start livy: $status"
  exit $status
fi

# Start the hue process
./startup.sh
status=$?
if [ $status -ne 0 ]; then
  echo "Failed to start hue: $status"
  exit $status
fi


# Naive check runs checks once a minute to see if either of the processes exited.
# This illustrates part of the heavy lifting you need to do if you want to run
# more than one service in a container. The container exits with an error
# if it detects that either of the processes has exited.
# Otherwise it loops forever, waking up every 60 seconds

while sleep 60; do
  ps aux |grep solr |grep -q -v grep
  PROCESS_1_STATUS=$?
  ps aux |grep livy |grep -q -v grep
  PROCESS_2_STATUS=$?
  ps aux |grep hue |grep -q -v grep
  PROCESS_3_STATUS=$?
  # If the greps above find anything, they exit with 0 status
  # If they are not both 0, then something is wrong
  if [ $PROCESS_1_STATUS -ne 0 -o $PROCESS_2_STATUS -ne 0 -o $PROCESS_3_STATUS -ne 0 ]; then
    echo "One of the processes has already exited."
    exit 1
  fi
done
```

</details>