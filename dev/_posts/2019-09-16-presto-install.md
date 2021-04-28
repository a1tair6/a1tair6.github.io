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

### docker-presto.sh
```sh
#!/bin/bash
echo "ip host-nn-001-01" >> /etc/hosts
echo "ip host-nn-001-02" >> /etc/hosts
echo "ip host-dn-001-01" >> /etc/hosts
echo "ip host-dn-001-02" >> /etc/hosts
echo "ip host-dn-001-03" >> /etc/hosts
cp /etc/presto/*.properties $PRESTO_CONF_DIR
cp /etc/presto/*.config $PRESTO_CONF_DIR
cp /etc/presto/hive.properties $PRESTO_CONF_DIR/catalog
cp /etc/presto/core-site.xml $PRESTO_CONF_DIR
cp /etc/presto/hdfs-site.xml $PRESTO_CONF_DIR
launcher run
```

### hive.properties
```
connector.name=hive-hadoop2
hive.metastore.uri=thrift://host-nn-001-02:9083
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
```sh
#!/bin/bash
curl --silent presto-qxqmc:8080/v1/node | tr "," "\n" | grep --silent $(hostname -i)
```
