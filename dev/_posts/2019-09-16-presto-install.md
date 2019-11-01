# presto 설치
- rancher 의 apps 사용
- rancher - resources - configmaps 설정

## setting coordinator

### config.properties
```
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8080
query.max-memory=4GB
query.max-memory-per-node=1GB
discovery-server.enabled=true
discovery.uri=http://presto-qxqmc:8080
```

### core-site.xml
```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://lz-edw-cluster</value>
</property>

<property>
<name>ha.zookeeper.quorum</name>
<value>lz-edw-nn-001-01:2181,lz-edw-nn-001-02:2181,lz-edw-dn-001-01:2181</value>
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

<property>
<name>hadoop.proxyuser.admin.hosts</name>
<value>*</value>
</property>

<property>
<name>hadoop.proxyuser.admin.groups</name>
<value>*</value>
</property>


</configuration>
```

### docker-presto.sh
```
#!/bin/bash
echo "10.40.11.128 lz-edw-nn-001-01" >> /etc/hosts
echo "10.40.12.187 lz-edw-nn-001-02" >> /etc/hosts
echo "10.40.11.136 lz-edw-dn-001-01" >> /etc/hosts
echo "10.40.12.236 lz-edw-dn-001-02" >> /etc/hosts
echo "10.40.12.173 lz-edw-dn-001-03" >> /etc/hosts
cp /etc/presto/*.properties $PRESTO_CONF_DIR
cp /etc/presto/*.config $PRESTO_CONF_DIR
cp /etc/presto/hive.properties $PRESTO_CONF_DIR/catalog
cp /etc/presto/core-site.xml $PRESTO_CONF_DIR
cp /etc/presto/hdfs-site.xml $PRESTO_CONF_DIR
launcher run
```

### hdfs-site.xml
```
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
<value>lz-edw-cluster</value>
</property>

<property>
<name>dfs.ha.namenodes.lz-edw-cluster</name>
<value>nn1,nn2</value>
</property>

<property>
<name>dfs.namenode.rpc-address.lz-edw-cluster.nn1</name>
<value>lz-edw-nn-001-01:8020</value>
</property>

<property>
<name>dfs.namenode.rpc-address.lz-edw-cluster.nn2</name>
<value>lz-edw-nn-001-02:8020</value>
</property>

<property>
<name>dfs.namenode.http-address.lz-edw-cluster.nn1</name>
<value>lz-edw-nn-001-01:50070</value>
</property>

<property>
<name>dfs.namenode.http-address.lz-edw-cluster.nn2</name>
<value>lz-edw-nn-001-02:50070</value>
</property>

<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://lz-edw-nn-001-01:8485;lz-edw-nn-001-02:8485;lz-edw-dn-001-01:8485/lz-edw-cluster</value>
</property>
<property>
<name>dfs.client.failover.proxy.provider.lz-edw-cluster</name>
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

### hive.properties
```
connector.name=hive-hadoop2
hive.metastore.uri=thrift://lz-edw-nn-001-02:9083
hive.config.resources=/presto/etc/core-site.xml,/presto/etc/hdfs-site.xml
```

### jvm.config
```
-server
-Xmx8G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
```

### log.properties
```
com.facebook.presto=INFO
```

### node.properties
```
node.environment=production
node.data-dir=/presto/etc/data
```

## setting worker
- coordinator의 config.properties, health_check.sh 외 동일

### config.properties
```
coordinator=false
http-server.http.port=8080
query.max-memory=4GB
query.max-memory-per-node=1GB
discovery.uri=http://presto-qxqmc:8080
```

### health_check.sh
```
#!/bin/bash
curl --silent presto-qxqmc:8080/v1/node | tr "," "\n" | grep --silent $(hostname -i)
```
