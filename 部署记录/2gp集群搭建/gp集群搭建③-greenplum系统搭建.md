# 更改集群机器主机名

master 1台（架构图中的主节点），Standby 1台（架构图中的从节点），Segment 3台。共5台服务器。

```
     ip       主机名
10.105.201.140     gpmaster
10.105.19.24       gpstandby
10.105.3.145       gpseg1
10.105.20.38       gpseg2
10.105.192.27      gpseg3
```

1. 修改/etc/hosts文件

   sudo vim /etc/hosts

   添加下面内容（5台服务器相同的配置）

    ```bash
   10.105.201.140     gpmaster
   10.105.19.24       gpstandby
   10.105.3.145       gpseg1
   10.105.20.38       gpseg2
   10.105.192.27      gpseg3
    ```

2. 更改主机名

   分别更改主机名

   ```bash
   hostnamectl --static set-hostname gpmaster
   hostnamectl --static set-hostname gpstandby
   hostnamectl --static set-hostname gpseg1
   hostnamectl --static set-hostname gpseg2
   hostnamectl --static set-hostname gpseg3
   ```

   查看效果

   ```bash
   hostnamectl --static
   可以看到设定的机器名
   ```

   其实，你不必重启机器以激活永久主机名修改。上面的命令会立即修改内核主机名。
   注销并重新登入后在命令行提示来观察新的静态主机名



# 系统配置

先在master上做配置，再拷贝集群其他机器

## ①修改系统内核/etc/sysctl.conf文件

（说明：相同的配置先在主节点节点上配置，配置完成后在2.5小节中复制到其它节点上）

vim /etc/sysctl.conf

```Bash
kernel.shmmni = 4096
kernel.shmall = 5000000000
kernel.sem = 250 512000 100 2048
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.ip_forward = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.defalut.arp_filter = 1
net.ipv4.ip_local_port_range = 1025 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
### 测试环境要取消这个，否则oracle启不来 ### 值为1
```

让配置生效

```bash
sysctl -p
```

## ②修改进程数

（说明：相同的配置先在主节点节点上配置，配置完成后在2.5小节中复制到其它节点上）

vim /etc/security/limits.conf 

```
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```

> 此条是参考其他搭建步骤，但是发现不设置也可，遇到错误后，删除该文件后搭建无问题
>
> 我的操作系统是centos 6.3，所以还有一个文件要修改：那就是/etc/security/limits.d/90-nproc.conf /vim /etc/security/limits.d/90-nproc.conf 
> 修改后的内容如下：
>
> ```
> * soft nproc 131072
> * hard nproc 131072
> * soft nofile 1048576
> * hard nofile 1048576
> 此行待确认
> ???root soft nproc unlimited
> ```
>

##  ③修改/etc/selinux/config文件

防火墙的问题：

```
centos6：
关闭防火墙： service iptables stop
关闭开机启动防火墙：chkconfig iptables off
查看防火墙状态 service iptables status

centos7：
启动： systemctl start firewalld
关闭： systemctl stop firewalld
查看状态： systemctl status firewalld 
开机禁用  ： systemctl disable firewalld
开机启用  ： systemctl enable firewalld
firewall-cmd --list-ports   
 
```

关闭SELinux

vim /etc/selinux/config 

```Config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
# targeted - Targeted processes are protected,
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

## ④磁盘预读参数及 deadline算法修改 

```
blockdev --setra 65536 /dev/sda
echo deadline > /sys/block/sda/queue/scheduler
```

## ⑤将1-4的配置文件传送到其他

依次复制到各个子节点

```bash
scp /etc/sysctl.conf gp-sdw1:/etc
scp /etc/selinux/config gp-sdw1:/etc/selinux
#可选
scp /etc/security/limits.d/90-nproc.conf gp-sdw1:/etc/security/limits.d
```



参考：有主备[Greenplum 集群部署](http://www.cnblogs.com/gomysql/p/5812591.html)

参考：无standby[GreenPlum集群搭建安装超详细步骤](https://blog.csdn.net/king13127/article/details/83989704)



# 安装gp

> 此段废弃： 得到greenplum-db-*.*.*.*-build-1-RHEL5-x86_64.bin文件，将其拷贝到/usr/local文件夹下进行安装（因为官网默认在此目录安装，为了不至于后面配置参数之类的太麻烦，我们也在这个目录下安装）
>
> [Greenplum 在Linux下的安装（centOS，RedHat）](https://blog.51cto.com/greatdalin/1722114)

## 1.在Master节点上安装Greenplum DB

安装包是rpm格式的执行rpm安装命令：

```Bash
rpm -ivh greenplum-db-5.15.1-rhel7-x86_64.rpm
```

默认的安装路径是/usr/local，然后需要修改该路径gpadmin操作权限：

```Bash
chown -R gp:gp /usr/local
```

确保所有的服务器安装了ed软件，否则后面初始化集群就会报错。

```bash
yum install ed -y
```

## 2.创建配置集群hostlist文件,打通节点

用于批量安装软件以及后续集群的初始化

### 2.1 创建一个hostlist,包含所有节点主机名：

> 此方法无效：
>
> 由于ssh免密服务设置的端口为36000
>
> 设置环境变量：
>
> ```Bash
> vim /etc/profile
> alias ssh='ssh -p36000 '
> alias scp='scp -P 36000 '
> ```

```
su - gp
mkdir -p /home/gp/conf
vim /home/gp/conf/hostlist
```

输入以下内容：

```
gpmaster
gpstandby
gpseg1
gpseg2
gpseg3
```



### 2.2 创建一个 seg_hosts ，包含所有的Segment Host的主机名：

vim /home/gp/conf/seg_hosts

```
gpseg1
gpseg2
gpseg3
```

### 2.3 配置免密

```bash
su gp
source /usr/local/greenplum-db/greenplum_path.sh 

gpssh-exkeys -f /home/gp/conf/hostlist
#step3时会提示：这里过程中会提示输入各个子节点gp用户密码

```

测试免密

```bash
ssh gpstandby 
```

或者：

```bash
source /usr/local/greenplum-db/greenplum_path.sh 
gpssh -f /home/gp/conf/hostlist
```

### 2.4 批量安装gp

①②只需要选择一个即可

#### ①这是直接调用自带命令来

```bash
cd /home/gp/
source /usr/local/greenplum-db/greenplum_path.sh
gpseginstall -f /home/gp/conf/hostlist -u gp -p greenplum
```



检查是否批量完成

```bash
su gp
source /usr/local/greenplum-db/greenplum_path.sh 
gpssh -f /home/gp/conf/hostlist -e ls -l $GPHOME
```

#### ②或者使用直接拷贝的方式来安装集群

在各个子节点进行文件夹赋权：

```bash
root用户
chown -R gp:gp /usr/local
```


在主节点打包安装包并复制到各个子节点：

```bash
cd /usr/local/
tar -cf gp.tar greenplum-db-5.15.1/
gpscp -f /home/gp/conf/seg_hosts gp.tar =:/usr/local/

#或者用linux自带命令
cd /usr/local/

scp gp.tar gp@gpstandby:/usr/local/
scp gp.tar gp@gpseg1:/usr/local/
scp gp.tar gp@gpseg2:/usr/local/
scp gp.tar gp@gpseg3:/usr/local/
```

ok，如果没有意外，就批量复制成功了，可以去子节点的相应文件夹查看，之后要将tar包解压，现在我们将采用对子节点使用批量解压操作：

source /usr/local/ greenplum-db/greenplum_path.sh

gpssh -f /home/gpadmin/conf/seg_hosts  #统一处理子节点

```Bash

=> cd /usr/local
[gpstandby]
[gpseg1]
[gpseg2]
[gpseg3]
=> tar -xf gp.tar
[gpstandby]
[gpseg1]
[gpseg2]
[gpseg3]
 
#建立软链接
=> ln -s ./greenplum-db-5.15.1 greenplum-db
[gpstandby]
[gpseg1]
[gpseg2]
[gpseg3]
=> ll(可以使用ll查看一下是否已经安装成功)
=>exit(退出)
```

或者用linux自带命令

```bash
#解压，建立软连接
tar -xf gp.tar
ln -s /usr/local/greenplum-db-5.15.1 /usr/local/greenplum-db
```



#### 检查是否安装完成

```
gpssh -f /home/gp/conf/hostlist -e ls -l $GPHOME
```



## 3 创建存储目录

创建存储目录

master，standby上创建：

```Bash
su
mkdir -p /data/master
chown -R gp.gp /data/master/
```

segment上创建目录（也可以在master上用下面命令批量操作segment，建立目录，改权限）

```Bash
gpssh -f /home/gp/conf/seg_hosts -e 'mkdir -p /data/primary'
gpssh -f /home/gp/conf/seg_hosts -e 'mkdir -p /data/mirror'
gpssh -f /home/gp/conf/seg_hosts -e 'chown -R gp.gp /data/primary'
gpssh -f /home/gp/conf/seg_hosts -e 'chown -R gp.gp /data/mirror'
#或者采用以下执行
gpssh -f /home/gp/conf/seg_hosts
=> mkdir -p /data/primary
=> mkdir -p /data/mirror
=> chown -R gp.gp /data/primary
=> chown -R gp.gp /data/mirror
```

## 4 系统设置及配置环境变量(all)

Lingyige 所有主机都需要设定

Standby 设置环境变量，master，standby都设置。

在Master主机，使用NTP守护进程同步所有Segment主机的系统时钟。例如，使用gpssh来完成：

```bash
source /usr/local/greenplum-db/greenplum_path.sh 
gpssh -f /home/gp/conf/hostlist -v -e 'ntpd'
```

添加新行用以加载greenplum_path.sh文件和设置MASTER_DATA_DIRECTORY环境变量。

### 修改.bashrc

```bash
su - gp
cat >>/home/gp/.bashrc<<-EOF
source /usr/local/greenplum-db/greenplum_path.sh 
export MASTER_DATA_DIRECTORY=/data/master/gpseg-1
export PGPORT=5432
export PGDATABASE=gp_sydb
alias ssh='ssh -p36000'
alias scp='scp -P 36000'
EOF
```

### 修改.bash_profile

```bash
su - gp
cat >>/home/gp/.bash_profile<<-EOF
source /usr/local/greenplum-db/greenplum_path.sh
export MASTER_DATA_DIRECTORY=/data/master/gpseg-1
export PGPORT=5432
export PGDATABASE=gp_sydb
alias ssh='ssh -p36000'
alias scp='scp -P 36000'
EOF
```

### 使之生效

```bash
source ~/.bashrc
source ~/.bash_profile
```

### 然后依次复制到各个子节点



## 5 初始化gp数据库(master)

在master上gp用户执行

### ①创建Greenplum数据库初始化配置文件

```bash
su gp
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config   /home/gp/conf/gpinitsystem_config
chmod 775 /home/gp/conf/gpinitsystem_config
```

设置所有必须的参数

vim /home/gp/conf/gpinitsystem_config

1. 以下为文本要修改的属性字段配置      

```bash
#资源目录为在第3步创建的资源目录，配置几次资源目录就是每个子节点有几个实例（推荐4-8个，这里配置了4个，primary与mirror文件夹个数对应）
declare -a DATA_DIRECTORY=(/data/primary /data/primary /data/primary /data/primary)
declare -a MIRROR_DATA_DIRECTORY=(/data/mirror /data/mirror /data/mirror /data/mirror)
MASTER_HOSTNAME=gpmaster                     #主节点名称
MASTER_DIRECTORY=/data/master                #资源目录为第3步创建的资源目录
MASTER_DATA_DIRECTORY=/data/master/gpseg-1   #与第4步配置的系统变量一样
ARRAY_NAME="Greenplum"                         #与第4步配置的系统变量一样
DATABASE_NAME=gp_sydb                        #与第4步配置的系统变量一样
SEG_PREFIX=gpseg 
PORT_BASE=40000
MASTER_PORT=5432
TRUSTED_SHELL=ssh
MACHINE_LIST_FILE=/home/gp/conf/seg_hosts    
MIRROR_PORT_BASE=50000
REPLICATION_PORT_BASE=41000


ENCODING=UNICODE

declare -a DATA_DIRECTORY=(/data/primary)
declare -a MIRROR_DATA_DIRECTORY=(/data/mirror)
MASTER_HOSTNAME=gpmaster                     #主节点名称
MASTER_DIRECTORY=/data/master                #资源目录为第3步创建的资源目录
MASTER_DATA_DIRECTORY=/data/master/gpseg-1   #与第4步配置的系统变量一样
ARRAY_NAME="Greenplum"                         #与第4步配置的系统变量一样
DATABASE_NAME=gp_sydb                        #与第4步配置的系统变量一样
SEG_PREFIX=gpseg 
PORT_BASE=40000
MASTER_PORT=5432
TRUSTED_SHELL=ssh
?CHECK_POINT_SEGMENTS=8
MACHINE_LIST_FILE=/home/gp/conf/seg_hosts    
MIRROR_PORT_BASE=50000
REPLICATION_PORT_BASE=41000
```

### 初始化前检查连通性

检查节点与节点之间文件读取；

```Bash
cd /usr/local/greenplum-db/bin
source /usr/local/greenplum-db/greenplum_path.sh 
gpcheckperf -f /home/gp/conf/hostlist -r N -d /tmp

得到结果
-------------------
--  NETPERF TEST
-------------------
[Warning] retrying with port 23012

====================
==  RESULT
====================
Netperf bisection bandwidth test
gpmaster -> gpseg1 = 193.170000
gpseg2 -> gpseg3 = 194.340000
gpseg1 -> gpmaster = 195.780000
gpseg3 -> gpseg2 = 193.870000

Summary:
sum = 777.16 MB/sec
min = 193.17 MB/sec
max = 195.78 MB/sec
avg = 194.29 MB/sec
median = 194.34 MB/sec

```

### NTP 配置

启用master节点上的ntp，并在Segment节点上配置和启用NTP：

```bash
echo "server gp-master perfer" >>/etc/ntp.conf
gpssh -f /home/gp/conf/hostlist -v -e 'sudo ntpd'
gpssh -f /home/gp/conf/hostlist -v -e 'sudo /etc/init.d/ntpd start && sudo chkconfig --level 35 ntpd on'
```

### ②运行初始化工具初始化数据库

master节点 gp用户执行

```bash
su - gp
source /usr/local/greenplum-db/greenplum_path.sh 
gpinitsystem -c /home/gp/conf/gpinitsystem_config -h /home/gp/conf/seg_hosts -s gpstandby
#当配置了MACHINE_LIST_FILE，可用如下命令
gpinitsystem -c /home/gp/conf/gpinitsystem_config  -S

#gpinitsystem命令参数解释：
-c：指定初始化文件。
-h：指定segment主机文件。
-s：指定standby主机，创建standby节点。
```

若初始化失败，需要删除/data(也就是在第3步配置的目录)下的数据资源目录重新初始化；

```bash
  gpssh -f /home/gp/conf/hostlist
  cd /data/
  rm -rf master/*
  rm -rf mirror/*
  rm -rf primary/*
  exit
```

若初始化成功，那恭喜你已经安装成功了。

```bash
#若成功会有以下结果
gpinitstandby:gpmaster:gp-[INFO]:- Filespace locations
gpinitstandby:gpmaster:gp-[INFO]:------------------------------------------------------
gpinitstandby:gpmaster:gp-[INFO]:-pg_system -> /data/master/gpseg-1
gpinitstandby:gpmaster:gp-[INFO]:-Syncing Greenplum Database extensions to standby
gpinitstandby:gpmaster:gp-[INFO]:-The packages on gpstandby are consistent.
gpinitstandby:gpmaster:gp-[INFO]:-Adding standby master to catalog...
gpinitstandby:gpmaster:gp-[INFO]:-Database catalog updated successfully.
gpinitstandby:gpmaster:gp-[INFO]:-Updating pg_hba.conf file...
gpinitstandby:gpmaster:gp-[INFO]:-pg_hba.conf files updated successfully.
gpinitstandby:gpmaster:gp-[INFO]:-Updating filespace flat files...
gpinitstandby:gpmaster:gp-[INFO]:-Filespace flat file updated successfully.
gpinitstandby:gpmaster:gp-[INFO]:-Starting standby master
gpinitstandby:gpmaster:gp-[INFO]:-Checking if standby master is running on host: gpstandby  in directory: /data/master/gpseg-1
gpinitstandby:gpmaster:gp-[INFO]:-Cleaning up pg_hba.conf backup files...
gpinitstandby:gpmaster:gp-[INFO]:-Backup files of pg_hba.conf cleaned up successfully.
gpinitstandby:gpmaster:gp-[INFO]:-Successfully created standby master on gpstandby
gpinitsystem:gpmaster:gp-[INFO]:-Successfully completed standby master initialization
gpinitsystem:gpmaster:gp-[INFO]:-Scanning utility log file for any warning messages
gpinitsystem:gpmaster:gp-[WARN]:-*******************************************************
gpinitsystem:gpmaster:gp-[WARN]:-Scan of log file indicates that some warnings or errors
gpinitsystem:gpmaster:gp-[WARN]:-were generated during the array creation
gpinitsystem:gpmaster:gp-[INFO]:-Please review contents of log file
gpinitsystem:gpmaster:gp-[INFO]:-/home/gp/gpAdminLogs/gpinitsystem_20190321.log
gpinitsystem:gpmaster:gp-[INFO]:-To determine level of criticality
gpinitsystem:gpmaster:gp-[INFO]:-These messages could be from a previous run of the utility
gpinitsystem:gpmaster:gp-[INFO]:-that was called today!
gpinitsystem:gpmaster:gp-[WARN]:-*******************************************************
gpinitsystem:gpmaster:gp-[INFO]:-Greenplum Database instance successfully created
gpinitsystem:gpmaster:gp-[INFO]:-------------------------------------------------------
gpinitsystem:gpmaster:gp-[INFO]:-To complete the environment configuration, please
gpinitsystem:gpmaster:gp-[INFO]:-update gp .bashrc file with the following
gpinitsystem:gpmaster:gp-[INFO]:-1. Ensure that the greenplum_path.sh file is sourced
gpinitsystem:gpmaster:gp-[INFO]:-2. Add "export MASTER_DATA_DIRECTORY=/data/master/gpseg-1"
gpinitsystem:gpmaster:gp-[INFO]:-   to access the Greenplum scripts for this instance:
gpinitsystem:gpmaster:gp-[INFO]:-   or, use -d /data/master/gpseg-1 option for the Greenplum scripts
gpinitsystem:gpmaster:gp-[INFO]:-   Example gpstate -d /data/master/gpseg-1
gpinitsystem:gpmaster:gp-[INFO]:-Script log file = /home/gp/gpAdminLogs/gpinitsystem_20190321.log
gpinitsystem:gpmaster:gp-[INFO]:-To remove instance, run gpdeletesystem utility
gpinitsystem:gpmaster:gp-[INFO]:-Standby Master gpstandby has been configured
gpinitsystem:gpmaster:gp-[INFO]:-To activate the Standby Master Segment in the event of Master
gpinitsystem:gpmaster:gp-[INFO]:-failure review options for gpactivatestandby
gpinitsystem:gpmaster:gp-[INFO]:-------------------------------------------------------
gpinitsystem:gpmaster:gp-[INFO]:-The Master /data/master/gpseg-1/pg_hba.conf post gpinitsystem
gpinitsystem:gpmaster:gp-[INFO]:-has been configured to allow all hosts within this new
gpinitsystem:gpmaster:gp-[INFO]:-array to intercommunicate. Any hosts external to this
gpinitsystem:gpmaster:gp-[INFO]:-new array must be explicitly added to this file
gpinitsystem:gpmaster:gp-[INFO]:-Refer to the Greenplum Admin support guide which is
gpinitsystem:gpmaster:gp-[INFO]:-located in the /usr/local/greenplum-db/./docs directory
```



### ③查看效果

```bash
ps -ef | grep postgres
```

### ④将standby加入集群

在master上执行

```bash
source /usr/local/greenplum-db/greenplum_path.sh 
#若-s指定了standby，则不需要执行以下命令
gpinitstandby -s gpstandby
```



# greenplum集群操作

## 启动和停止

数据库测试是否能正常启动和关闭，命令如下

```Bash
gpstart
gpstop
#或者
gpstart -a
gpstop -M fast
gpstop -M fast -a
```

## 5.2 登录数据库

```bash
#进入某个数据库
psql -d postgres  
```









安装rpm的

https://blog.csdn.net/king13127/article/details/83989704

有下载截图的是：无standby

https://www.cnblogs.com/renlipeng/p/5685432.html

这个是主备，系统配置参考的是：

http://www.cnblogs.com/gomysql/p/5812591.html

