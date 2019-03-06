[TOC]

# 记录docker启动记录



1. 启动虚拟机

   (vagrant suspend 暂停)若是处于暂停需要启动

   vagrant resume  启动

   ​

2. 启动docker-machine

   列出 ：docker-machine ls

   启动：docker-machine start default

   启动后检查是否配置了宿主与容器共享ip段，参考日志[]()

   ​

   > 由于我是采用共享目录，所以需要挂载目录 [参考:Docker-machine挂载本地文件夹](https://www.jianshu.com/p/2ecef54c1e33)
   >
   > (1).在docker-machine 虚拟机上设置共享文件夹
   > ![](https://ws4.sinaimg.cn/large/006tKfTcgy1g0ryyzp0vrj30zi0bm0t9.jpg)
   > (2).再在docker-machine虚拟机中执行，是将共享的文件夹
   >
   > ```shell
   > sudo mount -t vboxsf vagrant  /Users/yaosong
   > ```
   >
   > 那么在virtualbox中设置共享文件夹的share名称对应mac的目录
   > 以后相当于操作宿主机/Users/yaosong/Yao/share/就是操作的mac系统/Users/yaosong/Yao/share/

3. 设置默认环境

   ```shell
   eval $(docker-machine env default) # Setup the environment
   ```

   ​

4. 启动docker-容器

   列出：docker ps

   - 如果没有容器：创建容器

     ```Shell
     docker run -itd  --net=br  --name spark --hostname spark yaosong5/spark:2.1.0 &> /dev/null
     ```

   - 如果有容器

     启动：docker start 容器名

     > 容器名是docker ps 命令展示出的已存在的容器名

5. 使用容器


# Docker虚拟机关闭流程

1. 暂停docker-machine

   vagrent suspend

2. ​

   ​