[TOC]

GreenPlum使用记录，测试报告

# 创建用户及分配权限

连接参数：

-d，指定要连接的数据库，基本每次登录GreenPlum数据库都需要使用这个参数。
-l，列出可用的所有数据库，如果忘记了要登录数据库的名字，可以使用这个参数查看。
-h，指定要连接的数据库服务器的IP地址，默认是本机(localhost)。
-p，指定数据库的端口号，默认是5432.
-U，连接数据库的用户名，默认是安装gp的用户。

## 创建密码

由于greenplum集群搭建成功，默认是免密的
使用\password命令，为gp用户设置一个密码(这个用户为配置greenplum集群时的用户)

```bash
\password gp
#后面为默认用户的用户名
#回车后输入密码
#或者也可以使用下面命令更改：
ALTER USER gp WITH PASSWORD 'kngp';
```

登录本地数据库可以不指定-h参数，如果端口使用默认的5432，也不需要指定-p参数，默认使用gpadmin管理员用户登录数据库，如果使用gpadmin用户登录，也可以不指定-U参数。

如果是设置密码后没有起到输入命令也未提示密码：
那么请注意查看配置文件：pg_hba.conf的配置项每行最后一列都是trust，若是，即为免密

![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1f99c0qyxj30ui04mabj.jpg)





 psql -h 10.105.201.140 -p 5432 -U gp

10.105.201.140



报错： 

```
gpstop failed. (Reason='fe_sendauth: no password supplied
') exiting...
```

参考[fe_sendauth: no password supplied](https://stackoverflow.com/questions/17996957/fe-sendauth-no-password-supplied)

用命令：pg_ctl reload

```
[gp@gpmaster gpseg-1]$ pg_ctl reload -D /data/master/gpseg-1
server signaled
```

pg_ctl reload -D /data/master/gpseg-1 



service postgresql reload server signaled







## 创建数据库用户dw

并设置密码

```sql
CREATE USER dw WITH PASSWORD 'kndw';
```

创建数据库

```sql
CREATE DATABASE dw_test OWNER dw;
```

## 创建测试用户

```sql
CREATE USER test WITH PASSWORD 'test'；
CREATE DATABASE testgp OWNER test;
```



## 



# 开启远程访问

## 修改配置文件

就是在配置greenplum集群时，master存储数据的目录中： /data/master/gpseg-1/
1. 在所有ip位置监听
   vim  /data/master/gpseg-1/postgresql.conf
   添加/修改：在所有IP地址上监听，从而允许远程连接到数据库服务器
   ```bash
   listening_address: '*'
   ```
2. 文件：pg_hba.conf
   vim /data/master/gpseg-1/pg_hba.conf
   添加/修改：允许任意用户从任意机器上以密码方式访问数据库，把下行添加为第一条规则：
   ```bash
   host    all             all             0.0.0.0/0               md5
   ```

修改pg_hba.conf文件不需要重启数据库，但是需要使用gpstop –u参数重新加载后才能使之生效

```bash
sudo systemctl restart postgresql
```

参考[GreenPlum公司数据库登录和基本的帮助手册信息](http://www.dbdream.com.cn/2016/01/greenplum%E6%95%B0%E6%8D%AE%E5%BA%93%E7%99%BB%E5%BD%95%E5%92%8C%E5%9F%BA%E6%9C%AC%E7%9A%84%E5%B8%AE%E5%8A%A9%E6%89%8B%E5%86%8C%E4%BF%A1%E6%81%AF/)