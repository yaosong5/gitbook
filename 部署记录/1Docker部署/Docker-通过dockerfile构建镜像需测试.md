```dockerfile
# 设置基本的镜像，后续命令都以这个镜像为基础

FROM kinogmt/centos-ssh:6.7

# 作者信息

MAINTAINER yaosong, http://yaosong5@qq.com

# RUN命令会在上面指定的镜像里执行任何命令

RUN yum install passwd openssl openssh-server -y \

​```
    && echo '123456' | passwd --stdin root \
    && touch /etc/ssh/ssh_host_rsa_key \
    && touch /etc/ssh/ssh_host_ecdsa_key \
    &&  chmod 600 /etc/ssh/ssh_host_rsa_key  \
    &&  chmod 600 /etc/ssh/ssh_host_dsa_key  \
    &&  chown root /etc/ssh/ssh_host_rsa_key  \
    &&  chown root /etc/ssh/ssh_host_dsa_key  \
    && mkdir -p /root/.ssh  \
    && mkdir /var/run/sshd  \
    && touch /root/.ssh/config  \
    && echo "StrictHostKeyChecking no" > /root/.ssh/config  \
    && sed -i "a UserKnownHostsFile /dev/null" /root/.ssh/config \
    && ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key -y \
    && ssh-keygen -t dsa -f /etc/ssh/ssh_host_rsa_key -y \
    && sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config  \
    && yum  install vim -y \
    && yum install -y openssh-server  \
    && sed -i "s/#Port 22/Port 22/g" /etc/ssh/sshd_config    \
    && sed -i "s/#ListenAddress0.0.0.0/ListenAddress 0.0.0.0/g" /etc/ssh/sshd_config    \
    && sed -i "s/#ListenAddress::/ListenAddress ::/g" /etc/ssh/sshd_config    \
    && sed -i "s/#PermitRootLoginyes/PermitRootLogin yes/g" /etc/ssh/sshd_config    \
    && sed -i "s/#PasswordAuthenticationyes/PasswordAuthentication yes/g" /etc/ssh/sshd_config    \
    && sed -i "s/#PermitEmptyPasswordsno/PermitEmptyPasswords yes/g" /etc/ssh/sshd_config   \
    && mkdir -p /root/.ssh && chown root.root /root && chmod 700 /root/.ssh
​```

EXPOSE 22

# 启动sshd服务并且暴露22端口

CMD ["/usr/sbin/sshd", "-D"] 

# 暴露ssh端口22

EXPOSE 22

# 设定运行镜像时的默认命令：输出ip，并以daemon方式启动sshd

# CMD ip addr ls eth0 | awk '{print $2}' | egrep -o '([0-9]+\.){3}[0-9]+';/usr/sbin/sshd -D

CMD ["/usr/sbin/sshd", "-D"]

```



# 构造镜像

docker build -f Dockerfile -t hadoop:v1 .

也可以指定Dockerfile如下：

docker build -t centos:tt - < Dockerfile