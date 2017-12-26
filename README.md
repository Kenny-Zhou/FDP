# FDP
Hybrid Recommendation based Securities Lending System.

This is based on Apache PredictionIO to recommend Securities to user.

PredictionIO setup:

1. PredictionIO-0.12.0-incubating.tar.gz
untar it and add "vendors" dir in it, un-tar following dependencies into vendors 
spark-2.1.1-bin-hadoop2.6.tgz
hbase-1.2.6-bin.tar.gz
elasticsearch-5.5.2.tar.gz

2. update conf/pio-env.sh
PIO_STORAGE_REPOSITORIES_METADATA_NAME=pio_meta
PIO_STORAGE_REPOSITORIES_METADATA_SOURCE=ELASTICSEARCH

PIO_STORAGE_REPOSITORIES_EVENTDATA_NAME=pio_event
PIO_STORAGE_REPOSITORIES_EVENTDATA_SOURCE=HBASE

PIO_STORAGE_REPOSITORIES_MODELDATA_NAME=pio_model
PIO_STORAGE_REPOSITORIES_MODELDATA_SOURCE=LOCALFS

un-comment out associated conf, update hbase to version hbase-1.2.6

3. update hbase add basic configuration
Edit PredictionIO-0.12.0-incubating/vendors/hbase-1.2.6/conf/hbase-site.xml

<configuration>
  <property>
      <name>hbase.rootdir</name>
      <value>file:///opt/pfApps/PredictionIO-0.12.0-incubating/vendors/hbase-1.2.6/data</value>
  </property>
  <property>
      <name>hbase.zookeeper.property.dataDir</name>
      <value>/opt/pfApps/PredictionIO-0.12.0-incubating/vendors/hbase-1.2.6/zookeeper</value>
  </property>
</configuration>


Edit PredictionIO-0.12.0-incubating/vendors/hbase-1.2.6/conf/hbase-env.sh to set JAVA_HOME
export JAVA_HOME=/opt/pfApps/jdk/java/1.8.0_121l64/jre

In addition, you must set your environment variable JAVA_HOME. For example, in /home/abc/.bashrc add the following line:

export JAVA_HOME=/opt/pfApps/jdk/java/1.8.0_121l64

--update elasticsearch/bin/elasticsearch to use same JAVA_HOME

4. start it
PredictionIO-0.12.0-incubating/bin/pio-start-all

5 check status
PredictionIO-0.12.0-incubating/bin/pio status
should see: [INFO] [Management$] Your system is all ready to go.
or
$ jps -l
15344 org.apache.hadoop.hbase.master.HMaster
15409 org.apache.predictionio.tools.console.Console
15256 org.elasticsearch.bootstrap.Elasticsearch
15469 sun.tools.jps.Jps



