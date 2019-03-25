## DockerFile代码注意事项
```dockerfile
FROM openjdk:8u131-jre-alpine
# openjdk:8u131-jre-alpine作为基础镜像，体积小，自带jvm环境。
MAINTAINER <eway>
# 切换root用户避免权限问题，utf8字符集，bash解释器。安装一个同步为上海时区的时间
USER root
ENV LANG=C.UTF-8
RUN apk add --no-cache  --update-cache bash
ENV TZ=Asia/Shanghai
RUN apk --update add wget bash tzdata \
    && cp /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone
# 下载解压spark
WORKDIR /usr/local
RUN wget "http://www.apache.org/dist/spark/spark-2.0.2/spark-2.0.2-bin-hadoop2.7.tgz" \
   && tar -zxvf spark-*  \
   && mv spark-2.0.2-bin-hadoop2.7 spark \
   && rm -rf spark-2.0.2-bin-hadoop2.7.tgz

# 配置环境变量、暴露端口
ENV SPARK_HOME=/usr/local/spark
ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk
ENV PATH=${PATH}:${JAVA_HOME}/bin:${SPARK_HOME}/bin

EXPOSE 6066 7077 8080 8081 4044
WORKDIR $SPARK_HOME
CMD ["/bin/bash"]

```

## 单个启动组件

```bash
# 启动 master 容器
docker run -itd --name spark-master --net=br  -h spark-master yaosong5/bigdata:2.0  sh -c " source /etc/profile && spark-class org.apache.spark.deploy.master.Master"

```



# 启动 worker 容器

```bash
docker run -itd  --net=br --link  spark-master:worker01 yaosong5/bigdata:2.0 sh -c " source /etc/profile && spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077 "


docker run -it  --net=br --link  spark-master:worker01 yaosong5/bigdata:2.0 sh -c " source /etc/profile && spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077 && ping namenode "
```
> 注：--link <目标容器名> 该参数的主要意思是本容器需要使用目标容器的服务，所以指定了容器的启动顺序，并且在该容器的 hosts 文件中添加目标容器名的 DNS。

# 启动 historyserver 容器

```Bash
docker run -itd --net=br --name=spark-history --link namenode:namenode --link  spark-master \
-e SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 \
-Dspark.history.retainedApplications=10 \
-Dspark.history.fs.logDirectory=hdfs://namenode:9000/sparkhistory" \
yaosong5/bigdata:2.0   sh -c " source /etc/profile  && ping namenode && spark-class org.apache.spark.deploy.history.HistoryServer"



docker run -itd  --net=br --link  spark-master:worker01 yaosong5/bigdata:2.0 sh -c " source /etc/profile && ping namenode"
docker run -it  --net=br  --link  spark-master   yaosong5/bigdata:2.0 sh -c " source /etc/profile && ping namenode "



docker run -itd --name=history --link  spark-master  \
 --net=br  \
yaosong5/bigdata:2.0  sh -c " source /etc/profile && ping namenode "

docker run -itd  \
 --net=br  --link namenode \
yaosong5/bigdata:2.0  sh -c " source /etc/profile && ping namenode "


-e SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=10 -Dspark.history.fs.logDirectory=hdfs://namenode:9000/sparkhistory" spark-class org.apache.spark.deploy.history.HistoryServer

spark-class org.apache.spark.deploy.history.HistoryServer --conf  spark.history.ui.port=18080 --conf spark.history.retainedApplications=10 --conf spark.history.fs.logDirectory=hdfs://namenode:9000/sparkhistory



/usr/spark/bin/spark-submit --master spark://spark-master:7077 --class org.apache.spark.examples.SparkPi /usr/spark/examples/jars/spark-examples_2.11-2.2.0.jar
```
# 运行spark程序
```Bash
spark-submit --conf spark.eventLog.enabled=true \
--conf spark.eventLog.dir=hdfs://namenode:9000/sparkhistory \
--master spark://namenode:7077 --class org.apache.spark.examples.SparkPi /usr/spark/examples/jars/spark-examples_2.11-2.0.2.jar
```

运行 spark 程序例子
spark-submit  \  #提交spark程序
--conf spark.eventLog.enabled=true \   # 允许spark程序日志的记录，最后在ip:18080能查询到
--conf spark.eventLog.dir=hdfs://namenode:9000/user/spark/history \
# 将spark程序日志放在hadoop的【hdfs】分布式文件系统。也可以挂在在本地。
但分布式文件系统因为分布式的原因他更安全，和能跨主机。    \
--master spark://namenode:7077  \  # spark-submit提交spark程序到--master指定的master节点
--class org.apache.spark.examples.SparkPi  ./examples/jars/spark-examples_2.11-2.0.2.jar




## docker-compose部署服务
```dockerfile
version: "2.0"
  master:
    image: yaosong5/hadoop:3.0
    command: bin/spark-class org.apache.spark.deploy.master.Master -h master
    hostname: spark-master
    container_name: spark-master
    network_mode: "br"
    environment:
      MASTER: spark://spark-master:7077
      SPARK_MASTER_OPTS: "-Dspark.eventLog.dir=hdfs://namenode:9000/user/spark/history"
      SPARK_PUBLIC_DNS: spark-master


  worker:
    image: yaosong5/hadoop:3.0
    command: bin/spark-class org.apache.spark.deploy.worker.Worker spark://spark-master:7077
    hostname: worker
    container_name: worker
    network_mode: "br"
    environment:
      SPARK_WORKER_CORES: 1
      SPARK_WORKER_MEMORY: 1g
      SPARK_WORKER_PORT: 8881
      SPARK_WORKER_WEBUI_PORT: 8081
      SPARK_PUBLIC_DNS: spark-master
    links:
      - spark-master

 historyServer:
    image: yaosong5/hadoop:3.0
    command: spark-class org.apache.spark.deploy.history.HistoryServer
    hostname: historyServer
    container_name: historyServer
    network_mode: "br"
    environment:
      MASTER: spark://spark-master:7077
      SPARK_PUBLIC_DNS: spark-master
      SPARK_HISTORY_OPTS: "-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs://namenode:9000/spark/history"
    links:
      - spark-master
    expose:
      - 18080
```
