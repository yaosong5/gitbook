# 新建用户

新建greenplum用户

 可以直接利用adduser创建新用户（adduser +用户名）这样在/home目录下会自动创建同名文件夹 也可用useradd  -m 用户名

```bash
#创建用户gb
useradd -m gp 
#给已创建的用户testuser设置密码
passwd gp 
greenplum
```

> 如果删除用户，只需使用一个简单的命令“userdel 用户名”即可。不过最好将它留在系统上的文件也删除掉，你可以使用“userdel -r 用户名”来实现这一目的。
>
> ```Bash
> userdel -r 用户名
> ```



# ssh免密登录

## 本机自身实现无密码登录：

```Bash
su gp
```

生成公钥、私钥对

    ssh-keygen
一路enter

![](https://ws4.sinaimg.cn/large/006tKfTcgy1g17xk706byj316k0m4n9l.jpg)

进入到生成密钥文件夹中，默认在用户的家目录下面，一个隐藏的.ssh文件夹中。

```shell
   cd ~/.ssh/
```
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g17xl4ei4dj30vs06ctb2.jpg)



查看是否有“authorized_keys”文件，如果有，直接将公钥追加到“authorized_keys”文件中，如果没有，创建“authorized_keys”文件，并修改权限为“600”

```Bash
touch authorized_keys
chmod 600 authorized_keys 
```
追加公钥到“authorized_keys”文件中
```Bash
cat id_rsa.pub >> authorized_keys 
```
## 集群机器相互免密登录

主机间实现无密码登录：

主机分别生成公钥、私钥对（上一步已经生成）
```bash
ssh-keygen
```
进入到生成密钥文件夹中，默认在用户的家目录下面，一个隐藏的.ssh文件夹中。
```bash
cd ~/.ssh/
```
node1上(小心别把node2的id_rsa.pub覆盖掉)

```bash
scp -P 36000 id_rsa.pub gp@10.105.201.140:~/.ssh/id_rsa.pub1
```



使用scp命令，将A主机公钥发送给B主机，将B主机公钥发送给A主机。

```bash
scp id_rsa.pub  gp@10.105.201.140:~/
scp id_rsa.pub  gp@10.105.19.24:~/
scp id_rsa.pub  gp@10.105.3.145:~/
scp id_rsa.pub  gp@10.105.20.38:~/
scp id_rsa.pub  gp@10.105.192.27:~/
```
分别查看A，B主机是否有“authorized_keys”文件，如果有，直接将需无密码登录的主机公钥追加到“authorized_keys”文件中，如果没有，创建“authorized_keys”文件，并修改权限为“600”



> 也可以直接将各个机器中id_rsa.pub 的所有内容拼接起来，直接追加到各个机器的authorized_keys文件中



