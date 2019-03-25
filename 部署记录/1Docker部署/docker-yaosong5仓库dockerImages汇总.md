#  docker images汇总

引用的是有ssh服务的Docker镜像**kinogmt/centos-ssh:6.7**，生成容器os

## centos

1.0 ssh配置了javahome

## centosbase

1.0 包含centos ssh mysql创建了hive库，hue库，更改了/JAVA_HOME

## bigdatabase

1.0包含除elk的hue的镜像都在bigdatabase1.0中 

​	slave，zk，hive，hbase，kafka测试一下目前都用的这个

2.0将编译过hue4的镜像保存为yaosong5/bigdatabase:2.0  maker install header的改为

## bigdata:

1.0包含hadoop

2.0是包含了hadoop spark hive hue node header等文件夹的镜像 ，但需要source 或者env一下，在使用中需要source

## elk

es满足不了，必须要单独镜像，所以就回退，只是编译执行hue即可

```
docker run -itd --net=br --name bb --hostname bb yaosong5/bigdatabase:1.0 &> /dev/null
```

创建时将data/es目录下的basedir 镜像到es

可用通过外部挂载文件夹来解决问题

## hue

hue直接引用已有镜像也可以

```
docker run -it  --net=br --name hue -v /Users/yaosong/Yao/DockerSource/hueSource/hue4/desktop/conf/hue.ini:/hue/desktop/conf/pseudo-distributed.ini --hostname hue   gethue/hue:latest bash
运行命令
./build/env/bin/hue runserver_plus 0.0.0.0:8888
```



## flinkonyarn

```
需要检查
```



## hbasezk

```
可能是同时需要zk和hbase的时候，目前用的bigdatabase1.0
```



如有需要引用镜像的：

```shell
docker search yaosong5
```

即可列出所有我制作的镜像

选择对应镜像进行引用



# 区别

那么详细的docker容器exec为：
master： bigdatabase2.0     datavol
slave01:  bigdatabase1.0    datavol
slave02:  bigdatabase1.0    datavol

elk1 : elk1.0    bigdatabase1.0
elk2 : elk1.0
elk3 : elk1.0

hbase1: bigdatabase1.0  hbasedatavol
hbase2: bigdatabase1.0  hbasedatavol
hbase3: bigdatabase1.0  hbasedatavol

hive: bigdatabase1.0  hivedatavol


hue:  gethue/hue:lates  只是把hue4目录下的配置文件替换了容器的配置文件

kakfa： bigdatabase1.0  
kafka无挂载： bigdata1.0

zk: bigatabase1.0  zkdatavol


那么现在要弄清楚bigdata 和 bigdatabase的区别：需要依赖外部文件。bigdata镜像内有所有的文件

bigdata1.0 与 bigdata2.0区别：bigdata1.0只有hadoop，2.0中有hadoop、spark、hive、hue等等等
bigdatabase1.0 与 bigdatabase2.0的区别：
都需要创建容器才能得到区别：

# 这是啥

 Manager Addresses:
  10.0.2.15:2377

现在做游戏日志采集的部分

游戏日志文件放在share/data/es里面的

现在需要做一个挂载 将machine的/Users/yaosong/Yao/share/data/es/挂载到容器的/下（这个需要在容器创建的时候做，所以需要重新做，header也需要重新启动，异常问题是因为原始有data导致了node不对，所以注册不到elk2）

这儿有个问题，flume据我的了解是监控的文件级别的，文件内数据新增是无法检测到的









