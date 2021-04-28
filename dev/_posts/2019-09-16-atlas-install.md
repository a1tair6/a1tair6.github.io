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
```
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
```
docker commit -m "initial atlas" -a "user_id" {container_id} {tag_name}
```

## docker images push
```
docker push {tag_name}
```


- ** 여기까지 기본 images **
- ** 설정파일 보안상 image version 나눔 **

## hadoop, atlas, hive config setting
```
FROM docker.lezhin.com/dw-util-atlas:0.1
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


<details>
  <summary>hive-site.xml</summary>
  <p>

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?><!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
--><configuration>
  <!-- WARNING!!! This file is auto generated for documentation purposes ONLY! -->
  <!-- WARNING!!! Any changes you make to this file will be ignored by Hive.   -->
  <!-- WARNING!!! You must make your changes in hive-site.xml instead.         -->
  <!-- Hive Execution Parameters -->
    <!--property>
        <name>hive.metastore.local</name>
        <value>false</value>
    </property-->

    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://{host}:{port}/{database_nm}?createDatabaseIfNotExist=true&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>{db_user}</value>
    </property>

    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>{db_password}</value>
    </property>

    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://host-nn-001-02:9083</value>
    </property>

    <property>
         <name>hive.metastore.connect.retries</name>
         <value>30</value>
    </property>

     <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>host-nn-001-02</value>
    </property>

     <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
    </property>

     <property>
        <name>hive.server2.thrift.min.worker.threads</name>
        <value>5</value>
    </property>

     <property>
        <name>hive.server2.thrift.max.worker.threads</name>
        <value>500</value>
            </property>

    <property>
        <name>hive.server2.enable.doAs</name>
        <value>false</value>
    </property>

    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
        <description>
        Local or HDFS directory where Hive keeps table contents.
        </description>
    </property>

    <!--property>
        <name>hive.server2.authentication</name>
        <value>NONE</value>
    </property-->

    <property>
        <name>hive.support.concurrency</name>
        <value>false</value>
    </property>


    <!--property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>true</value>
    </property-->


    <property>
        <name>hive.server2.webui.host</name>
        <value>host-nn-001-02</value>
    </property>

    <property>
        <name>hive.server2.webui.port</name>
        <value>10002</value>
    </property>

    <property>
        <name>hive.execution.engine</name>
        <value>tez</value>
    </property>

    <!--property>
        <name>hive.server2.tez.initialize.default.sessions</name>
        <value>true</value>
    </property-->

    <!--property>
        <name>hive.tez.container.size</name>
        <value>1024</value>
    </property>

    <property>
        <name>hive.tez.java.opts</name>
        <value>-Xmx816m</value>
    </property-->

  <property>
    <name>hive.vectorized.execution.enabled</name>
    <value>true</value>
    <description>
      This flag should be set to true to enable vectorized mode of query execution.
      The default value is true to reflect that our most expected Hive deployment will be using vectorization.
    </description>
  </property>
  <property>
    <name>hive.vectorized.execution.reduce.enabled</name>
        <value>true</value>
    <description>
      This flag should be set to true to enable vectorized mode of the reduce-side of query execution.
      The default value is true.
    </description>
  </property>
  <property>
    <name>hive.vectorized.execution.reduce.groupby.enabled</name>
    <value>true</value>
    <description>
      This flag should be set to true to enable vectorized mode of the reduce-side GROUP BY query execution.
      The default value is true.
    </description>
  </property>
  <property>
    <name>hive.vectorized.execution.mapjoin.native.enabled</name>
    <value>true</value>
    <description>
      This flag should be set to true to enable native (i.e. non-pass through) vectorization
      of queries using MapJoin.
      The default value is true.
    </description>
  </property>

  <property>
    <name>hive.cbo.enable</name>
    <value>true</value>
    <description>Flag to control enabling Cost Based Optimizations using Calcite framework.</description>
  </property>

  <property>
    <name>hive.compute.query.using.stats</name>
    <value>true</value>
    <description>
      When set to true Hive will answer a few queries like count(1) purely using stats
      stored in metastore. For basic stats collection turn on the config hive.stats.autogather to true.
      For more advanced stats collection need to run analyze table queries.
    </description>
  </property>
<property>
  <name>fs.s3a.access.key</name>
  <value>{access_key}</value>
  <description>AWS access key ID. Omit for Role-based authentication.</description>
</property>
<property>
  <name>fs.s3a.secret.key</name>
  <value>{secret_key}</value>
  <description>AWS secret key. Omit for Role-based authentication.</description>
</property>

 <property>
      <name>hive.exec.post.hooks</name>
      <value>org.apache.atlas.hive.hook.HiveHook</value>
    </property>
    <property>
      <name>atlas.cluster.name</name>
      <value>primary</value>
    </property>

</configuration>
```
</p>
</details>


## hive-env.sh
```
export HIVE_AUX_JARS_PATH=/home/atlas/eco_service/atlas/hook/hive
```

<details>
  <summary>yarn-site.xml</summary>
  <p>

```xml
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle,spark_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
        <value>org.apache.spark.network.yarn.YarnShuffleService</value>
    </property>
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>/usr/local/hadoop/data/yarn/nm-local-dir</value>
    </property>
    <property>
        <name>yarn.resourcemanager.fs.state-store.uri</name>
        <value>/usr/local/hadoop/data/yarn/system/rmstore</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>rm-cluster</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
<property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>host-nn-001-02</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>host-nn-001-01</value>
    </property>
    <property>
        <name>hadoop.zk.address</name>
        <value>host-nn-001-01:2181,host-nn-001-02:2181,host-dn-001-01:2181</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.client.failover-proxy-provider</name>
        <value>org.apache.hadoop.yarn.client.ConfiguredRMFailoverProxyProvider</value>
    </property>
    <property>
        <name>yarn.resourcemanager.store.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>
    <property>
            <name>yarn.resourcemanager.ha.automatic-failover.zk-base-path</name>
        <value>/yarn-leader-election</value>
    </property>
    <property>
        <name>yarn.resourcemanager.recovery.enabled</name>
        <value>true</value>
    </property>

    <property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>host-nn-001-02:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>
        <value>host-nn-001-02:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>host-nn-001-02:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
<value>host-nn-001-02:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address.rm1</name>
        <value>host-nn-001-02:8033</value>
    </property>

    <property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>host-nn-001-01:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>host-nn-001-01:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>host-nn-001-01:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>host-nn-001-01:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address.rm2</name>
        <value>host-nn-001-01:8033</value>
    </property>

    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>8096</value>
    </property>

    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>8</value>
    </property>
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>1024</value>
    </property>

    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>2048</value>
    </property>

    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
            </property>

    <property>
        <name>yarn.resourcemanager.nodes.include-path</name>
        <value>/home/hadoop/eco_service/hadoop/etc/hadoop/nodes_include</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services.spark_shuffle.classpath</name>
        <value>/home/hadoop/eco_service/spark/yarn/*</value>
    </property>

    <property>
        <name>yarn.log.server.url</name>
        <value>http://host-nn-001-01:19888/jobhistory/logs</value>
    </property>

    <property>
        <name>yarn.application.classpath</name>
        <value>$HADOOP_CONF_DIR,
               $HADOOP_COMMON_HOME/share/hadoop/common/*,$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,
               $HADOOP_HDFS_HOME/share/hadoop/hdfs/*,$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*,
               $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*,
               $HADOOP_YARN_HOME/share/hadoop/yarn/*,$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*</value>
    </property>

    <!--property>
        <name>yarn.application.classpath</name>
        <value>$HADOOP_CONF_DIR,
               $HADOOP_COMMON_HOME/*,$HADOOP_COMMON_HOME/lib/*,
               $HADOOP_HDFS_HOME/*,$HADOOP_HDFS_HOME/lib/*,
               $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*,
               $HADOOP_YARN_HOME/*,$HADOOP_YARN_HOME/lib/*</value>
    </property-->

    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```
</p>
</details>

<details>
  <summary>hdfs-site.xml</summary>
  <p>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>

  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>

  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/usr/local/hadoop/data/namenode</value>
  </property>

  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/usr/local/hadoop/data/datanode</value>
  </property>

  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/usr/local/hadoop/data/dfs/journalnode</value>
  </property>

  <property>
    <name>dfs.nameservices</name>
    <value>host-cluster</value>
  </property>

  <property>
    <name>dfs.ha.namenodes.host-cluster</name>
    <value>nn1,nn2</value>
  </property>

  <property>
    <name>dfs.namenode.rpc-address.host-cluster.nn1</name>
    <value>host-nn-001-01:8020</value>
  </property>

  <property>
    <name>dfs.namenode.rpc-address.host-cluster.nn2</name>
    <value>host-nn-001-02:8020</value>
  </property>

  <property>
    <name>dfs.namenode.http-address.host-cluster.nn1</name>
    <value>host-nn-001-01:50070</value>
  </property>

  <property>
    <name>dfs.namenode.http-address.host-cluster.nn2</name>
    <value>host-nn-001-02:50070</value>
  </property>

  <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://host-nn-001-01:8485;host-nn-001-02:8485;host-dn-001-01:8485/host-cluster</value>
  </property>
    <property>
    <name>dfs.client.failover.proxy.provider.host-cluster</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>

  <property>
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
  </property>

  <property>
    <name>dfs.ha.automatic-failover.enabled</name>
    <value>true</value>
  </property>

  <property>
    <name>fs.s3a.access.key</name>
      <value>{access_key}</value>
    <description>AWS access key ID. Omit for Role-based authentication.</description>
  </property>

  <property>
    <name>fs.s3a.secret.key</name>
    <value>{secret_key}</value>
    <description>AWS secret key. Omit for Role-based authentication.</description>
  </property>

  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>

  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property>
</configuration>
```
</p>
</details>

<details>
  <summary>core-site.xml</summary>
  <p>

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
       <name>fs.defaultFS</name>
        <value>hdfs://host-cluster</value>
    </property>

    <property>
        <name>ha.zookeeper.quorum</name>
        <value>host-nn-001-01:2181,host-nn-001-02:2181,host-dn-001-01:2181</value>
    </property>

    <property>
        <name>fs.trash.interval</name>
        <value>14400</value>
    </property>

    <property>
        <name>hadoop.proxyuser.hadoop.hosts</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.hadoop.groups</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.hive.hosts</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.hive.groups</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.oozie.hosts</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.oozie.groups</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.hue.hosts</name>
        <value>*</value>
    </property>

    <property>
        <name>hadoop.proxyuser.hue.groups</name>
        <value>*</value>
    </property>

</configuration>
```
</p>
</details>


<details>
  <summary>mapred-site.xml</summary>
  <p>

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>

    <property>
        <name>mapreduce.job.ubertask.enable</name>
        <value>true</value>
    </property>

    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/home/hadoop/eco_service/hadoop</value>
    </property>

    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/home/hadoop/eco_service/hadoop</value>
    </property>

    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/home/hadoop/eco_service/hadoop</value>
    </property>

    <property>
        <name>yarn.app.mapreduce.am.resource.mb</name>
        <value>1024</value>
    </property>

    <property>
        <name>mapreduce.map.memory.mb</name>
        <value>2048</value>
    </property>

    <property>
        <name>mapreduce.reduce.memory.mb</name>
        <value>2048</value>
    </property>

    <property>
        <name>mapreduce.map.java.opts.max.heap</name>
        <value>1638</value>
    </property>

    <property>
        <name>mapreduce.reduce.java.opts.max.heap</name>
        <value>1638</value>
    </property>

    <property>
        <name>yarn.nodemanager.vmem-check-enable</name>
        <value>false</value>
    </property>
        <property>
        <name>yarn.nodemanager.vmem-pmem-ratio</name>
        <value>4</value>
    </property>


    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>host-nn-001-01:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>host-nn-001-01:19888</value>
    </property>

    <!--property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property-->

    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_CONF_DIR,
               $HADOOP_COMMON_HOME/share/hadoop/common/*,$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,
               $HADOOP_HDFS_HOME/share/hadoop/hdfs/*,$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*,
               $HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*,
               $HADOOP_YARN_HOME/share/hadoop/yarn/*,$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*</value>
    </property>


</configuration>
```
</p>
</details>



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
```

```
