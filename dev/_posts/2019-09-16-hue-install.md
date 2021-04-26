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

## hue.ini
```
# Hue configuration file
# ===================================
#
# For complete documentation about the contents of this file, run
#   $ <hue_root>/build/env/bin/hue config_help
#
# All .ini files under the current directory are treated equally.  Their
# contents are merged to form the Hue configuration, which can
# can be viewed on the Hue at
#   http://<hue_host>:<port>/dump_config


###########################################################################
# General configuration for core Desktop features (authentication, etc)
###########################################################################

[desktop]

  # Set this to a random string, the longer the better.
  # This is used for secure hashing in the session store.
  secret_key=
  # Webserver listens on this address and port
  http_host=0.0.0.0
  http_port=8888

  # Time zone name
  time_zone=America/Los_Angeles

  # Enable or disable debug mode.
  django_debug_mode=false
  # Enable or disable backtrace for server error
  http_500_debug_mode=false

  # Webserver runs as this user
  server_user=hue
  server_group=hue

  # This should be the Hue admin and proxy user
  default_user=hue

  # This should be the hadoop cluster admin
  default_hdfs_superuser=hadoop

  # Configuration options for user authentication into the web application
  # ------------------------------------------------------------------------
  [[auth]]

    # Users will automatically be logged out after 'n' seconds of inactivity.
    # A negative number means that idle sessions will not be timed out.
    idle_session_timeout=-1

  # ------------------------------------------------------------------------
  [[database]]
    # Database engine is typically one of:
    # postgresql_psycopg2, mysql, sqlite3 or oracle.
    #
    # Note that for sqlite3, 'name', below is a path to the filename. For other backends, it is the database name
    # Note for Oracle, options={"threaded":true} must be set in order to avoid crashes.
    # Note for Oracle, you can use the Oracle Service Name by setting "host=" and "port=" and then "name=<host>:<port>/<service_name>".
    # Note for MariaDB use the 'mysql' engine.
    engine=mysql
    host={host}
    port={port}
    user={db_user}
    password={db_password}
    # conn_max_age option to make database connection persistent value in seconds
    # https://docs.djangoproject.com/en/1.9/ref/databases/#persistent-connections
    ## conn_max_age=0
    # Execute this script to produce the database password. This will be used when 'password' is not set.
    ## password_script=/path/script
    name=lezhincomics_hue
    ## options={}
    # Database schema, to be used only when public schema is revoked in postgres
    ## schema=public

###########################################################################
# Settings to configure the snippets available in the Notebook
###########################################################################

[notebook]

  ## Show the notebook menu or not
  show_notebooks=true

  ## Flag to enable the bulk submission of queries as a background task through Oozie.
  enable_batch_execute=true

  ## Flag to turn on the Presentation mode of the editor.
  enable_presentation=true

  # One entry for each type of snippet.
  [[interpreters]]
    # Define the name and how to connect and execute the language.
    # http://cloudera.github.io/hue/latest/administrator/configuration/editor/

    # [[[mysql]]]
    #  name = MySQL
    #  interface=sqlalchemy
    #   ## https://docs.sqlalchemy.org/en/latest/dialects/mysql.html
    #   ## options='{"url": "mysql://${USER}:${PASSWORD}@localhost:3306/hue"}'

    [[[hive]]]
      name=Hive
      interface=hiveserver2

    # [[[druid]]]
    #   name = Druid
    #   interface=sqlalchemy
    #   options='{"url": "druid://host:8082/druid/v2/sql/"}'

    [[[sql]]]
      name=SparkSql
      interface=livy

    [[[spark]]]
      name=Scala
      interface=livy

    [[[pyspark]]]
      name=PySpark
      interface=livy

    [[[jar]]]
      name=Spark Submit Jar
      interface=livy-batch

    [[[py]]]
      name=Spark Submit Python
      interface=livy-batch

    [[[text]]]
      name=Text
      interface=text

    [[[java]]]
      name=Java
      interface=oozie

    [[[spark2]]]
      name=Spark
      interface=oozie

    [[[mapreduce]]]
      name=MapReduce
      interface=oozie

    [[[sqoop1]]]
      name=Sqoop1
      interface=oozie

    [[[distcp]]]
      name=Distcp
      interface=oozie

    [[[shell]]]
      name=Shell
      interface=oozie

    # [[[presto]]]
    #   name=Presto SQL
    #   interface=presto
    #   ## Specific options for connecting to the Presto server.
    #   ## The JDBC driver presto-jdbc.jar need to be in the CLASSPATH environment variable.
    #   ## If 'user' and 'password' are omitted, they will be prompted in the UI.
    #   options='{"url": "jdbc:presto://localhost:8080/catalog/schema", "driver": "io.prestosql.jdbc.PrestoDriver", "user": "root", "password": "root"}'


###########################################################################
# Settings to configure your Analytics Dashboards
###########################################################################

[dashboard]

  # Activate the Dashboard link in the menu.
  is_enabled=true

  # Activate the SQL Dashboard (beta).
  ## has_sql_enabled=false

  # Activate the Query Builder (beta).
  ## has_query_builder_enabled=false

  # Activate the static report layout (beta).
  ## has_report_enabled=false

  # Activate the new grid layout system.
  ## use_gridster=true

  # Activate the widget filter and comparison (beta).
  ## has_widget_filter=false

  # Activate the tree widget (to drill down fields as dimensions, alpha).
  ## has_tree_widget=false

  [[engines]]

    #  [[[solr]]]
    #  Requires Solr 6+
    ##  analytics=true
    ##  nesting=false

    #  [[[sql]]]
    ##  analytics=true
    ##  nesting=false


###########################################################################
# Settings to configure your Hadoop cluster.
###########################################################################

[hadoop]

  # Configuration for HDFS NameNode
  # ------------------------------------------------------------------------
  [[hdfs_clusters]]
    # HA support by using HttpFs

    [[[default]]]
      # Enter the filesystem uri
      fs_defaultfs=hdfs://host-nn-001-01:8020

      # NameNode logical name.
      logical_name=

      # Use WebHdfs/HttpFs as the communication mechanism.
      # Domain should be the NameNode or HttpFs host.
      # Default port is 14000 for HttpFs.
      webhdfs_url=http://host-nn-001-01:50070/webhdfs/v1

      # In secure mode (HTTPS), if SSL certificates from YARN Rest APIs
      # have to be verified against certificate authority
      ssl_cert_ca_verify=false

      # Directory of the Hadoop configuration
      hadoop_conf_dir='/usr/share/hue/hadoop/etc/hadoop'

  # Configuration for YARN (MR2)
  # ------------------------------------------------------------------------
  [[yarn_clusters]]

    [[[default]]]
      # Enter the host on which you are running the ResourceManager
      resourcemanager_host=host-nn-001-02

      # The port where the ResourceManager IPC listens on
      resourcemanager_port=8032

      # Whether to submit jobs to this cluster
      submit_to=True

      # Resource Manager logical name (required for HA)
      logical_name=rm-cluster

      # Change this if your YARN cluster is Kerberos-secured
      security_enabled=false

      # URL of the ResourceManager API
      resourcemanager_api_url=http://host-nn-001-02:8088

      # URL of the ProxyServer API
      proxy_api_url=http://host-nn-001-02:8088

      # URL of the HistoryServer API
      history_server_api_url=http://host-001-01:19888

      # URL of the Spark History Server
      spark_history_server_url=http://host-nn-001-02:18080

      # Change this if your Spark History Server is Kerberos-secured
      spark_history_server_security_enabled=false

      # In secure mode (HTTPS), if SSL certificates from YARN Rest APIs
      # have to be verified against certificate authority
      ssl_cert_ca_verify=false


###########################################################################
# Settings to configure Beeswax with Hive
###########################################################################

[beeswax]

  # Host where HiveServer2 is running.
  # If Kerberos security is enabled, use fully-qualified domain name (FQDN).
  hive_server_host=host-nn-001-02

  # Port where HiveServer2 Thrift server runs on.
  hive_server_port=10000

  # Host where Hive Metastore Server (HMS) is running.
  # If Kerberos security is enabled, the fully-qualified domain name (FQDN) is required.
  hive_metastore_host=host-nn-001-02

  # Configure the port the Hive Metastore Server runs on.
  hive_metastore_port=9083

  # Hive configuration directory, where hive-site.xml is located
  hive_conf_dir=/usr/share/hue/hive/conf

  [[ssl]]
    # Path to Certificate Authority certificates.
    ## cacerts=/etc/hue/cacerts.pem

    # Choose whether Hue should validate certificates received from the server.
    ## validate=true


###########################################################################
# Settings to configure Metastore
###########################################################################

[metastore]
  # Flag to turn on the new version of the create table wizard.
  ## enable_new_create_table=true

  # Flag to force all metadata calls (e.g. list tables, table or column details...) to happen via HiveServer2 if available instead of Impala.
  ## force_hs2_metadata=false


###########################################################################
# Settings to configure the Spark application.
###########################################################################

[spark]
  # The Livy Server URL.
  livy_server_url=http://localhost:8998

  # Configure Livy to start in local 'process' mode, or 'yarn' workers.
  livy_server_session_kind=yarn

  # Whether Livy requires client to perform Kerberos authentication.
  security_enabled=false

  # Whether Livy requires client to use csrf protection.
  csrf_enabled=false

  # Host of the Sql Server
  sql_server_host=host-nn-002

  # Port of the Sql Server
  sql_server_port=10001

  # Choose whether Hue should validate certificates received from the server.
  ssl_cert_ca_verify=false


###########################################################################
# Settings to configure the Oozie app
###########################################################################

[oozie]
  # Location on local FS where the examples are stored.
  ## local_data_dir=..../examples

  # Location on local FS where the data for the examples is stored.
  ## sample_data_dir=...thirdparty/sample_data

  # Location on HDFS where the oozie examples and workflows are stored.
  # Parameters are $TIME and $USER, e.g. /user/$USER/hue/workspaces/workflow-$TIME
  remote_data_dir=/user/hue/oozie/workspaces

  # Maximum of Oozie workflows or coodinators to retrieve in one API call.
  ## oozie_jobs_count=100

  # Use Cron format for defining the frequency of a Coordinator instead of the old frequency number/unit.
  ## enable_cron_scheduling=true

  # Flag to enable the saved Editor queries to be dragged and dropped into a workflow.
  ## enable_document_action=true

  # Flag to enable Oozie backend filtering instead of doing it at the page level in Javascript. Requires Oozie 4.3+.
  ## enable_oozie_backend_filtering=true

  # Flag to enable the Impala action.
  ## enable_impala_action=false

  # Flag to enable the Altus action.
  ## enable_altus_action=false


###########################################################################
# Settings to configure the Filebrowser app
###########################################################################

[filebrowser]
  # Location on local filesystem where the uploaded archives are temporary stored.
  archive_upload_tempdir=/tmp

  # Show Download Button for HDFS file browser.
  show_download_button=true

  # Show Upload Button for HDFS file browser.
  show_upload_button=true

  # Flag to enable the extraction of a uploaded archive in HDFS.
  ## enable_extract_uploaded_archive=true

  # Redirect client to WebHdfs or S3 for file download. Note: Turning this on will override notebook/redirect_whitelist for user selected file downloads on WebHdfs & S3.
  ## redirect_download=false

###########################################################################
# Settings to configure Sqoop2
###########################################################################

[sqoop]
  # If the Sqoop2 app is enabled. Sqoop2 project is deprecated. Sqoop1 is recommended.
  ## is_enabled=false

  # Sqoop server URL
  ## server_url=http://localhost:12000/sqoop

  # Path to configuration directory
  ## sqoop_conf_dir=/etc/sqoop2/conf

  # Choose whether Hue should validate certificates received from the server.
  ## ssl_cert_ca_verify=true

  # For autocompletion, fill out the librdbms section.

###########################################################################
# Settings to configure the Data Import Wizard
###########################################################################

[indexer]

  # Filesystem directory containing Solr Morphline indexing libs.
  ## config_indexer_libs_path=/tmp/smart_indexer_lib

  # Filesystem directory containing JDBC libs.
  config_jdbc_libs_path=/user/oozie/share/lib

  # Filesystem directory containing jar libs.
  config_jars_libs_path=/user/oozie/share/lib

  # Flag to turn on the Solr Morphline indexer.
  ## enable_scalable_indexer=true

  # Flag to turn on Sqoop ingest.
  ## enable_sqoop=true

  # Flag to turn on Kafka topic ingest.
  ## enable_kafka=false

###########################################################################
# Settings for the User Admin application
###########################################################################

[useradmin]
  # Default home directory permissions
  ## home_dir_permissions=0755

  # The name of the default user group that users will be a member of
  ## default_user_group=default

  [[password_policy]]
    # Set password policy to all users. The default policy requires password to be at least 8 characters long,
    # and contain both uppercase and lowercase letters, numbers, and special characters.

    ## is_enabled=false
    ## pwd_regex="^(?=.*?[A-Z])(?=(.*[a-z]){1,})(?=(.*[\d]){1,})(?=(.*[\W_]){1,}).{8,}$"
    ## pwd_hint="The password must be at least 8 characters long, and must contain both uppercase and lowercase letters, at least one number, and at least one special character."
    ## pwd_error_message="The password must be at least 8 characters long, and must contain both uppercase and lowercase letters, at least one number, and at least one special character."


###########################################################################
# Settings to configure liboozie
###########################################################################

[liboozie]
  # The URL where the Oozie service runs on. This is required in order for
  # users to submit jobs. Empty value disables the config check.
  oozie_url=http://dev-lz-oozie.oozie:11000/oozie

  # Requires FQDN in oozie_url if enabled
  security_enabled=false

  # Location on HDFS where the workflows/coordinator are deployed when submitted.
  remote_deployement_dir=/user/hue/oozie/deployments


###########################################################################
# Settings for the AWS lib
###########################################################################

[aws]
  [[aws_accounts]]
    # Default AWS account
    [[[default]]]
      # AWS credentials
      access_key_id={access_key}
      secret_access_key={secret_key}

      # Execute this script to produce the AWS access key ID.
      ## access_key_id_script=/path/access_key_id.sh

      # Execute this script to produce the AWS secret access key.
      ## secret_access_key_script=/path/secret_access_key.sh

      # Allow to use either environment variables or
      # EC2 InstanceProfile to retrieve AWS credentials.
      allow_environment_credentials=false

      # AWS region to use, if no region is specified, will attempt to connect to standard s3.amazonaws.com endpoint
      region={region}

      # Endpoint overrides
      #host=s3.{region}.amazonaws.com

      # Proxy address and port
      ## proxy_address=
      ## proxy_port=8080
      ## proxy_user=
      ## proxy_pass=

      # Secure connections are the default, but this can be explicitly overridden:
      #is_secure=true

      # The default calling format uses https://<bucket-name>.s3.amazonaws.com but
      # this may not make sense if DNS is not configured in this way for custom endpoints.
      # e.g. Use boto.s3.connection.OrdinaryCallingFormat for https://s3.amazonaws.com/<bucket-name>
      # calling_format=boto.s3.connection.OrdinaryCallingFormat

      # The time in seconds before a delegate key is expired. Used when filebrowser/redirect_download is used. Default to 4 Hours.
      ## key_expiry=14400

```

## core-size.xml
```
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

## hadoop-env.sh
```
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
```
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

## hive-site.xml.spark
```
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
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://{host}:{port}/{database_name}?createDatabaseIfNotExist=true&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
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
        <name>hive.server2.idle.session.check.operation</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.server2.session.check.interval</name>
        <value>6h</value>
    </property>


    <property>
        <name>hive.server2.idle.operation.timeout</name>
        <value>5d</value>
    </property>


    <property>
        <name>hive.server2.idle.session.timeout</name>
        <value>7d</value>
    </property>

    <property>
        <name>hive.metastore.client.socket.timeout</name>
        <value>1h</value>
        <description>
      Expects a time value with unit (d/day, h/hour, m/min, s/sec, ms/msec, us/usec, ns/nsec), which is sec if not specified.
      MetaStore Client socket timeout in seconds
        </description>
    </property>

  <property>
    <name>hive.vectorized.execution.enabled</name>
    <value>true</value>
    <description>
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
</configuration>
```

## mapreduce-site.xml
```
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

## spark-defaults.conf
```
spark.master                     yarn
spark.submit.deployMode          client
spark.eventLog.enabled           true
spark.yarn.jars                  hdfs://host-cluster/user/oozie/share/lib/lib_20190910061053/spark2/*
spark.eventLog.dir               hdfs://host-cluster/user/spark/eventlog
spark.history.fs.logDirectory    hdfs://host-cluster/user/spark/historylog
spark.serializer                 org.apache.spark.serializer.KryoSerializer
spark.driver.memory              1548m
#spark.executor.memory            10240m
spark.executor.memory            2048m
spark.memory.offHeap.enabled     true
spark.memory.offHeap.size        900m
spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dcom.amazonaws.services.s3.enableV4=true
spark.driver.extraJavaOptions  -XX:+PrintGCDetails -Dcom.amazonaws.services.s3.enableV4=true

#Dynamic allocation on YARN
spark.dynamicAllocation.enabled             true
spark.dynamicAllocation.minExecutors        1
spark.executor.instances                    3
spark.dynamicAllocation.maxExecutors        100
spark.shuffle.service.enabled               true
spark.scheduler.minRegisteredResourcesRatio 0.0

spark.sql.warehouse.dir=hdfs://host-cluster:8020/user/hive/warehouse
spark.yarn.historyServer.address            host-nn-001-02:18080

spark.hadoop.fs.s3a.access.key={access_key}
spark.hadoop.fs.s3a.endpoint=s3.{region}.amazonaws.com
spark.hadoop.fs.s3a.secret.key={secret_key}
spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem
```

## spark-env.sh
```
export SPARK_HOME=${SPARK_HOME:-/usr/share/hue/spark}
export YARN_CONF_DIR=${YARN_CONF_DIR:-${HADOOP_HOME}/etc/hadoop}
export HIVE_SERVER2_THRIFT_PORT=10001

export HADOOP_HOME=${HADOOP_HOME:-/usr/share/hue/hadoop}
export HADOOP_HDFS_HOME=${HADOOP_HDFS_HOME:-${HADOOP_HOME}}
export HADOOP_MAPRED_HOME=${HADOOP_MAPRED_HOME:-${HADOOP_HOME}}
export HADOOP_YARN_HOME=${HADOOP_YARN_HOME:-${HADOOP_HOME}}
export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-/usr/share/hue/hadoop/etc/hadoop}

export STANDALONE_SPARK_MASTER_HOST=`hostname -f`
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8080

export SPARK_WORKER_DIR=${SPARK_WORKER_DIR:-/var/run/spark/work}
export SPARK_WORKER_PORT=7078
export SPARK_WORKER_WEBUI_PORT=18081

export SPARK_WORKER_DIR=${SPARK_WORKER_DIR:-/var/run/spark/work}
export SPARK_LOG_DIR=${SPARK_HOME}/logs

### change the following to specify a real cluster's Master host
export STANDALONE_SPARK_MASTER_HOST=\`hostname\`

SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_CONF_DIR"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_HOME/lib/native"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_HOME/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_HOME/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_HDFS_HOME/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_HDFS_HOME/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_MAPRED_HOME/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_MAPRED_HOME/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_YARN_HOME/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_YARN_HOME/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/etc/hive/conf"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$HADOOP_HOME/share/tools/lib/*"
export PYSPARK_PYTHON=/usr/bin/python3
export PYSPARK_DRIVER_PYTHON=/usr/bin/python3
```

## yarn-site.xml
```
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

## start_script.sh
```
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
