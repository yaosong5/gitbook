
1、linux下安装python3
 准备编译环境(环境如果不对的话，可能遇到各种问题，比如wget无法下载https链接的文件)

```shell
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel -y
```

​	
2、下载 Python3.6代码包

```shell
wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tar.xz
```

安装解压工具

```shell
yum install xz -y
```

进行解压



报错，解决报错问题