删除重新初始化



gpssh -f /home/gp/conf/hostlist

gpinitsystem -c /home/gp/conf/gpinitsystem_config -h /home/gp/conf/seg_hosts -s gpstandby



```bash
gpssh -f /home/gp/conf/hostlist
cd /data/
rm -rf master/*
rm -rf mirror/*
rm -rf primary/*
exit
```



# 系统参数

一样的

vim /etc/sysctl.conf

```
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
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
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 1025 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
```



sysctl -p

一样的


```
vim /etc/security/limits.conf
# End of file
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```





```
gpssh -f /home/gp/conf/hostlist
```





```
ARRAY_NAME="Greenplum"
SEG_PREFIX=gpseg
PORT_BASE=40000
declare -a DATA_DIRECTORY=(/data/primary)
MASTER_HOSTNAME=gpmaster
MASTER_DIRECTORY=/data/master
MASTER_PORT=5432
TRUSTED_SHELL=/usr/bin/ssh
MIRROR_PORT_BASE=50000
REPLICATION_PORT_BASE=41000
MIRROR_REPLICATION_PORT_BASE=44000
declare -a MIRROR_DATA_DIRECTORY=(/data/mirror)
MACHINE_LIST_FILE=/home/gp/conf/seg_hosts






ARRAY_NAME="Greenplum"
SEG_PREFIX=gpseg
PORT_BASE=40000
declare -a DATA_DIRECTORY=(/data/primary /data/primary /data/primary)
MASTER_HOSTNAME=gpmaster
MASTER_DIRECTORY=/data/master
MASTER_PORT=5432
TRUSTED_SHELL=ssh
MIRROR_PORT_BASE=50000
REPLICATION_PORT_BASE=41000
MIRROR_REPLICATION_PORT_BASE=44000
declare -a MIRROR_DATA_DIRECTORY=(/data/mirror /data/mirror /data/mirror)
MACHINE_LIST_FILE=/home/gp/conf/seg_hosts
DATABASE_NAME=gp_sydb
MASTER_DATA_DIRECTORY=/data/master/gpseg-1

设置了会报错？
CHECK_POINT_SEGMENTS=8
```







