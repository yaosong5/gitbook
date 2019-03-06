```bash
#java
export JAVA_HOME=/usr/java/jdk
export PATH=$JAVA_HOME:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
#hadoop
export HADOOP_HOME=/usr/hadoop
export HADOOP_CONFIG_HOME=$HADOOP_HOME/etc/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
#spark
export SCALA_HOME=/usr/scala/
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
SPARK_MASTER_IP=master
SPARK_LOCAL_DIRS=/usr/spark
SPARK_DRIVER_MEMORY=1G
export SPARK_HOME=/usr/spark
export PATH=$SPARK_HOME/bin:$PATH
export PATH=$SPARK_HOME/sbin:$PATH
#hue
MAVEN_HOME=/usr/maven
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
ANT_HOME=/usr/ant
PATH=$JAVA_HOME/bin:$ANT_HOME/bin:$PATH
export ANT_HOME PATH
HUE_HOME=/usr/hue4
#zk hbase
export ZK_HOME=/usr/zk
export HBASE_HOME=/usr/hbase
export PATH=$HBASE_HOME/bin:$PATH
export PATH=$ZK_HOME/bin:$PATH
#kafka
export KAFKA_HOME=/usr/kafka
#elk
export ES_HOME=/usr/es
export PATH=$ES_HOME/bin:$PATH
export KIBANA_HOME=/usr/kibana
export PATH=$KIBANA_HOME/bin:$PATH
export LOGSTASH_HOME=/usr/logstash
export PATH=$LOGSTASH_HOME/bin:$PATH
export NODE_HOME=/usr/node
export PATH=$NODE_HOME/bin:$PATH
export NODE_PATH=$NODE_HOME/lib/node_modules
#flink
export FLINK_HOME=/usr/flink
export PATH=$PATH:$FLINK_HOME/bin
```

