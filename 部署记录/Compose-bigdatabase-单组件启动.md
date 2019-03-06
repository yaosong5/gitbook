[TOC]
#docker 操作命令：


实时查看 docker 容器日志

```docker logs -f -t --tail 100  kanbigdata_namenode_1```

docker logs -f -t --tail 100  namenode

## docker容器挂载

    前目录为宿主机目录，后目录为容器目录

    compose文件中：
    volumes:
        - /Users/yaosong/Yao/share/hadoop/dfs/name:/root/hadoop/dfs/name
    shell命令：
        -v : docker run -it -v /test:/soft centos /bin/bash

引用  [基于 docker 的大数据架构](https://www.kancloud.cn/huangzhenyou/shuoming/545497)



## dockerfile
自己更改，引用的包为基础centos镜像


```Shell
# hadoop镜像封装
FROM openjdk:8u131-jre-alpine
MAINTAINER <yaosong5>
# 设置一些系统环境变量（字符集和时区）
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV TZ=Asia/Shanghai

# 安装一些必需的软件包，同步时区 【标记*1】
RUN apk --update add wget bash tzdata \
    && cp /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone

# 设置HADOOP相关的环境变量，这里我们使用2.8.4版本
ENV HADOOP_VERSION 2.8.4
ENV HADOOP_PACKAGE hadoop-${HADOOP_VERSION}
ENV HADOOP_HOME=/usr/hadoop
ENV PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin

# Hadoop安装
RUN wget http://www.apache.org/dist/hadoop/common/${HADOOP_PACKAGE}/${HADOOP_PACKAGE}.tar.gz && \
    tar -xzvf $HADOOP_PACKAGE.tar.gz && \
    mv $HADOOP_PACKAGE /usr/hadoop && \
    # 删除安装包，减少镜像大小
    rm $HADOOP_PACKAGE.tar.gz && \
    # 创建一系列HDFS需要的文件目录
    mkdir -p /root/hadoop/tmp && \
    mkdir -p /root/hadoop/var && \
    mkdir -p /root/hadoop/dfs/name && \
    mkdir -p /root/hadoop/dfs/data && \
    mkdir $HADOOP_HOME/logs && \
    # 格式化NameNode元数据
    $HADOOP_HOME/bin/hdfs namenode -format  -Ddfs.namenode.name.dir=/root/hadoop/dfs/name -format

WORKDIR $HADOOP_HOME
```


**构建Dockerfile 构建镜像**
`docker build -f Dockerfile -t hadoop:v1 .`
`docker build -f Dockerfile -t hadoop:v2 .`

`docker build -f Dockerfile -t yaosong5/bigadata:3.0 .`

## 单独启动namenode
原来是 **/root/hadoop/dfs/name**
在格式化中为 **/tmp/hadoop-root/dfs/name**

```
hadoop:v1 hdfs namenode \
hadoop:v1 hdfs namenode -Ddfs.namenode.name.dir=/root/hadoop/dfs/name -format -force \
猜想测试
hadoop:v1 hdfs namenode -Ddfs.namenode.name.dir=/usr/hadoop/namenode -format -force \
yaosong5/hadoop:1.0 hdfs  namenode \
```


```
docker run -d --name namenode \
--net=br -h namenode \
 --volumes-from  datavol \
yaosong5/bigadatabase:1.0   hdfs  namenode \
-Dfs.defaultFS=hdfs://0.0.0.0:9000 \
-Ddfs.namenode.name.dir=/usr/hadoop/namenode \
-Ddfs.replication=2 \
-Ddfs.http.address=0.0.0.0:50070 \
-Ddfs.namenode.datanode.registration.ip-hostname-check=false \
-Ddfs.permissions.enabled=false \
-Ddfs.namenode.safemode.threshold-pct=0
```





执行sh -c 多条命令

```
docker run -d --name namenode \
--net=br -h namenode \
-v /Users/yaosong/Yao/share/hadoop/dfs/namenode:/usr/hadoop/namenode \
yaosong5/hadoop:1.0 sh  -c "hdfs namenode -Ddfs.namenode.name.dir=/usr/hadoop/namenode -format -force  &&   hdfs  namenode
-Dfs.defaultFS=hdfs://0.0.0.0:9000
-Ddfs.namenode.name.dir=/usr/hadoop/namenode
-Ddfs.replication=3
-Ddfs.http.address=0.0.0.0:50070
-Ddfs.permissions.enabled=false
-Ddfs.namenode.datanode.registration.ip-hostname-check=false
-Ddfs.permissions.enabled=false
-Ddfs.namenode.safemode.threshold-pct=0
"
```



##  启动datanode

```shell
docker run -d --name datanode \
 --volumes-from  datavol --net=br \
-v /Users/yaosong/Yao/share/hadoop/dfs/datanode:/usr/hadoop/datanode \
--link namenode \
yaosong5/hadoop:1.0 hdfs datanode   \
-fs hdfs://namenode:9000 \
-Ddfs.datanode.data.dir=/usr/hadoop/datanode \
-Ddfs.permissions.enabled=false
```





## 启动ResourceManager 服务



```shell
docker run -d --name resourcemanager \
--net=br  -h resourcemanager \
yaosong5/hadoop:1.0 yarn resourcemanager \
-Dfs.defaultFS=hdfs://namenode:9000 \
-Dmapreduce.framework.name=yarn \
-Dyarn.resourcemanager.hostname=resourcemanager \
-Dyarn.resourcemanager.address=resourcemanager:8032 \
-Dyarn.resourcemanager.scheduler.address=resourcemanager:8030 \
-Dyarn.resourcemanager.resource-tracker.address=resourcemanager:8031 \
-Dyarn.resourcemanager.webapp.address=0.0.0.0:8088
```

需要在yarn-site.xml加上配置

```xml
<name>yarn.resourcemanager.address</name>
<value>master:8032</value>
</property>
<property>
  <name>yarn.resourcemanager.scheduler.address</name>
  <value>master:8030</value>
</property>
<property>
  <name>yarn.resourcemanager.resource-tracker.address</name>
  <value>master:8031</value>
</property>
```



##  启动nodemanager

```shell
docker run -d --name nodemanager \
--net=br --link resourcemanager \
yaosong5/hadoop:1.0 yarn nodemanager \
-Dfs.defaultFS=hdfs://namenode:9000 \
-Dmapreduce.framework.name=yarn \
-Dyarn.resourcemanager.hostname=resourcemanager \
-Dyarn.nodemanager.aux-services=mapreduce_shuffle \
-Dyarn.nodemanager.resource.memory-mb=2048 \
-Dyarn.nodemanager.resource.cpu-vcores=1 \
-Dyarn.nodemanager.vmem-check-enabled=false
```









## 执行多条命令
```shell
sh -c " "
"-Dyarn.resourcemanager.address=resourcemanager:8032",
"-Dyarn.resourcemanager.scheduler.address=resourcemanager:8030",
"-Dyarn.resourcemanager.resource-tracker.address=resourcemanager:831",
```





```shell
docker stop namenode
docker rm  namenode
docker stop datanode
docker rm  datanode
docker stop  resourcemanager
docker rm  resourcemanager
docker stop nodemanager
docker rm nodemanager

docker stop datanode1
docker rm  datanode1
docker stop datanode2
docker rm  datanode2
```





## docker-compose




```yml
version: '2'
services:
  web:
    image: dockercloud/hello-world
    ports:
      8080
         tworks:
      front-tier
      back-tier
```

Version 1 file format only. In version 2, use network_mode.

网络模式。 使用与 docker client --net参数相同的值。 container：...形式可以接受服务名称，而不是容器名称或 ID。


**一键启动docker-compose.yml编排的所有服务**

`docker-compose -f docker-compose.yml up -d`



`docker-compose -f /Users/yaosong/Yao/docker-compose/bigdatabase.yml up -d`

https://stackoverflow.com/questions/43664866/how-to-check-the-docker-compose-file-version

compose和docker版本对比
http://www.ywnds.com/?p=13097

docker compose 版本 3 语法需要 docker 版本 1.13 和 docker-compose 版本 1.10（请参阅发行说明）。 有关版本兼容性列表和升级说明，请参阅发行说明。

请注意，版本 3 语法是为 docker swarm 模式设计的，它最初是docker stack deploy在 docker 版本 1.13 中支持的。 如果您仍在使用docker-compose自己，则没有太多理由升级到版本 3 语法。

另请参阅撰写文件版本控制页面，该页面描述了不同 yml 版本之间的差异。


docker-machine ssh default 'mkdir -p ~/share && sudo mount -t vboxsf share ~/share'
感受：
通过这种方式，可以直接只需要保证有包，然后参数用命令行的配置即可，每次改动无需更改到镜像，可以很方便的读取,不需要每次臃肿的更改，打包镜像

问题：
未能访问（待解决）
读取本地配置的思路还需要完善
hue的集成及其他的组件的集成
docker还有很多相关的知识还需要进一步的学习
需要将近期的总结发布到博客上
