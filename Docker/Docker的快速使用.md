## Docker的快速使用

介绍完Docker的安装和基本使用之后，那就进入quick start吧。  
Docker的安装和基本使用：  
http://wing324.github.io/2015/10/06/Docker%E7%9A%84%E5%AE%89%E8%A3%85%E5%8F%8A%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C/  

推荐课程：http://www.jikexueyuan.com/path/docker/  


Docker的配置文件
-------------------------
docker的配置文件位于/etc/default/docker  
使用后续介绍  


Docker的基本操作
-------------------------
1. 启动容器：docker run IMAGE  
   启动交互式容器：docker run -i -t IMAGE /bin/bash  
2. 查看容器：docker ps -a -l  

```
无参数时,返回的是当前运行时的容器
-a 代表列出所有的容器
-l 代表列出最新创建的容器
```
3. 返回容器相关配置信息  

```
docker inspect CONTAINER ID/NAMES
```
4. 自定义容器名  

```
docker run --name=自定义容器名 -i -t  IMAGE 
```
EXAMPLE  

```
[root@localhost ~]# docker run --name=mydocker -i -t hello-world
Usage of loopback devices is strongly discouraged for production use. Either use `--storage-opt dm.thinpooldev` or use `--storage-opt dm.no_warn_on_loop_devices=true` to suppress this warning.

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/

[root@localhost ~]# docker -a
flag provided but not defined: -a
See 'docker --help'.
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS                      PORTS               NAMES
4b1a0607bc88        hello-world             "/hello"            12 seconds ago      Exited (0) 10 seconds ago                       mydocker             # ,名字已换成自定义的mydocker  
1e98f9067f6b        docker.io/hello-world   "/hello"            10 minutes ago      Exited (0) 10 minutes ago                       jovial_pasteur         
1d2c288dcde8        hello-world             "/bin/bash"         28 minutes ago                                                      sharp_hoover           
855231f306a4        hello-world             "/hello"            28 minutes ago      Exited (0) 28 minutes ago                       desperate_kilby        
0ab13a92ed4b        hello-world             "/hello"            32 minutes ago      Exited (0) 32 minutes ago                       clever_sammet          
e1fa10d7a55f        hello-world             "/bin/bash"         35 minutes ago                                                      drunk_albattani        
dad5054db548        hello-world             "/hello"            42 minutes ago      Exited (0) 42 minutes ago                       compassionate_turing   
16ae30c8b4d0        hello-world             "/hello"            45 minutes ago      Exited (0) 45 minutes ago                       cranky_goldstine       
f1ba689673d3        hello-world             "/hello"            21 hours ago        Exited (0) 21 hours ago                         silly_almeida          
8f200d44ba00        hello-world             "/hello"            21 hours ago        Exited (0) 21 hours ago                         loving_mayer 
```

5. 重新启动停止的容器  

```
docker start [-i] 容器名
```

6. 删除停止的容器  

```
docker rm 容器名
# 不能用于删除运行中的容器
```


Docker运行守护式容器
--------------------------------
1. 以守护形式运行容器  

```
docker run -i -t 	IMAGE
Ctrl+P Ctrl+Q #将当前容器置于后台运行
```
2. 附加到运行中的容器 #调出后台运行的容器至前台运行  

```
docker attach 容器名
```
3. 启动守护式容器  

```
docker run -d IMAGE 循环脚本之类的等,让docker一直运行

-d 将docker后台运行,若没有循环脚本之类等,在命令执行结束后即停止,若想让docker一直运行,需在命令后使用-c添加脚本
```
4. 查看容器日志  

```
docker logs [-f] [-t] [--tail] 容器名
-f 表示动态加载日志,相当于LINUX命令里面的tail -f的参数
-t 表示显示时间timestamp
--tail N 表示只显示最后N条记录
```
EXAMPLE  

```
[root@localhost ~]# docker logs mydocker

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

```
# docker logs -t示例
[root@localhost ~]# docker logs  -t mydocker
2015-09-04T13:06:04.469114324Z 
2015-09-04T13:06:04.469184154Z Hello from Docker.
2015-09-04T13:06:04.469191726Z This message shows that your installation appears to be working correctly.
2015-09-04T13:06:04.469196921Z 
2015-09-04T13:06:04.469201788Z To generate this message, Docker took the following steps:
2015-09-04T13:06:04.469206498Z  1. The Docker client contacted the Docker daemon.
2015-09-04T13:06:04.469211437Z  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
2015-09-04T13:06:04.469216436Z  3. The Docker daemon created a new container from that image which runs the
2015-09-04T13:06:04.469221194Z     executable that produces the output you are currently reading.
2015-09-04T13:06:04.469225733Z  4. The Docker daemon streamed that output to the Docker client, which sent it
2015-09-04T13:06:04.469230539Z     to your terminal.
2015-09-04T13:06:04.469235191Z 
2015-09-04T13:06:04.469239605Z To try something more ambitious, you can run an Ubuntu container with:
2015-09-04T13:06:04.469244274Z  $ docker run -it ubuntu bash
2015-09-04T13:06:04.469248764Z 
2015-09-04T13:06:04.469253159Z Share images, automate workflows, and more with a free Docker Hub account:
2015-09-04T13:06:04.469257784Z  https://hub.docker.com
2015-09-04T13:06:04.469262189Z 
2015-09-04T13:06:04.469267096Z For more examples and ideas, visit:
2015-09-04T13:06:04.469271622Z  https://docs.docker.com/userguide/
2015-09-04T13:06:04.469276135Z 
[root@localhost ~]# docker logs  -t -f mydocker
2015-09-04T13:06:04.469114324Z 
2015-09-04T13:06:04.469184154Z Hello from Docker.
2015-09-04T13:06:04.469191726Z This message shows that your installation appears to be working correctly.
2015-09-04T13:06:04.469196921Z 
2015-09-04T13:06:04.469201788Z To generate this message, Docker took the following steps:
2015-09-04T13:06:04.469206498Z  1. The Docker client contacted the Docker daemon.
2015-09-04T13:06:04.469211437Z  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
2015-09-04T13:06:04.469216436Z  3. The Docker daemon created a new container from that image which runs the
2015-09-04T13:06:04.469221194Z     executable that produces the output you are currently reading.
2015-09-04T13:06:04.469225733Z  4. The Docker daemon streamed that output to the Docker client, which sent it
2015-09-04T13:06:04.469230539Z     to your terminal.
2015-09-04T13:06:04.469235191Z 
2015-09-04T13:06:04.469239605Z To try something more ambitious, you can run an Ubuntu container with:
2015-09-04T13:06:04.469244274Z  $ docker run -it ubuntu bash
2015-09-04T13:06:04.469248764Z 
2015-09-04T13:06:04.469253159Z Share images, automate workflows, and more with a free Docker Hub account:
2015-09-04T13:06:04.469257784Z  https://hub.docker.com
2015-09-04T13:06:04.469262189Z 
2015-09-04T13:06:04.469267096Z For more examples and ideas, visit:
2015-09-04T13:06:04.469271622Z  https://docs.docker.com/userguide/
2015-09-04T13:06:04.469276135Z 
```

```
# docker logs --tail演示
[root@localhost ~]# docker logs  -t -f --tail 10  mydocker
2015-09-04T13:06:04.469235191Z 
2015-09-04T13:06:04.469239605Z To try something more ambitious, you can run an Ubuntu container with:
2015-09-04T13:06:04.469244274Z  $ docker run -it ubuntu bash
2015-09-04T13:06:04.469248764Z 
2015-09-04T13:06:04.469253159Z Share images, automate workflows, and more with a free Docker Hub account:
2015-09-04T13:06:04.469257784Z  https://hub.docker.com
2015-09-04T13:06:04.469262189Z 
2015-09-04T13:06:04.469267096Z For more examples and ideas, visit:
2015-09-04T13:06:04.469271622Z  https://docs.docker.com/userguide/
2015-09-04T13:06:04.469276135Z
```
5. 查看容器进程  

```
docker top 容器名
```
6. 在运行中的容器内启动新的进程  

```
docker exec [-d] [-i] [-t] 容器名
```
7. 停止守护式容器

```
docker stop 容器名
docker kill 容器名
```


Docker镜像与仓库
-------------------------
1. 查看镜像  

```
docker images 
无参数时,显示所有当前安装的镜像
-a 显示所有的镜像,包括中间层镜像（仓库为<none>,TAG为<none>）
-f 使用过滤条件
--no-trunc 不截断镜像的ID
-q,--quiet 只显示镜像的ID
```

2. 删除镜像  

```
docker rmi 镜像名
# 注意 'docker rm 容器名'' 的区别
-f,--false 强制删除镜像
--no-prune 不删除被打标签的父镜像
```

3. 查看镜像的详细信息  

```
docker inspect 容器名/镜像名 
-f,--format=''
```
4. 查找镜像  
   Docker Hub官网查找镜像  
   或者  
   docker search TERM  
   --automated 只显示完成的镜像  
   --no-trunc 不截断IMAGE ID  
   -s,--stars   
5. 获取镜像  

```
docker pull NAME:TAG
-a,--all-tags 下载仓库中所有的镜像文件

# 
当网络缓慢时,可以使用--registry-mirror选项在配置文件中添加国内的镜像地址
修改/etc/default/docker
添加DOCKER_OPTS='--registry-mirror=添加URL'
```
6. 推送镜像

```
docker push 用户名/镜像名
```
EXAMPLE

```
doxker push wing324/mysql
```
7. 构建镜像  

```
# 通过容器构建
docker commit  容器名 [REPOSITORY[:TAG]]
-a,--author 作者
-m,--message 提交消息
-p,--pause 提交阶段暂停容器

# 通过Dockerfile构建镜像
docker build
-t,--tag 为镜像去一个名字
```


Dockerfile介绍
-------------------
1. Dockerfile关键字  
##### FROM
```
FROM
FROM <image>
FROM <image>:<tag>
```
**特别说明**  
FROM后的镜像特性：  
已经存在的镜像  
基础镜像  
必须是第一条非注释指令  

##### MAINTAINER
指定镜像的作者信息  

```
MAINTAINER <name>
```

##### RUN
指定当前镜像中的运行命令  

```
# shell模式
RUN <command>
/bin/sh -c command
eg.
RUN echo hello 
```
```
# exec模式
RUN ['executable','参数1','参数2']
RUN ['/bin/bash','-c','echo hello']
```

##### EXPOSE
指定该镜像容器使用的端口  

```
EXPOSE <port>
```

##### CMD
在容器中指定运行的命令  
RUN是在镜像构建中执行的命令  
CMD是在容器运行中执行的命令  
CMD是指定容器运行指令的默认行为,当RUN中指定后,CMD指令将会被RUN指令覆盖  

```
# shell模式
CMD command 参数1 参数2
```

```
# exec模式
CMD ['executable','参数1','参数2']
```

```
# 与ENTRYPOINT搭配使用
CMD ['参数1','参数2']
# 为ENRYPOINT提供默认参数
```

##### ENTRYPOINT
与CMD功能相同,不同的是ENTRYPOINT指令不会被RUN指令覆盖  
可以使用docker run --entrypoint强制使用RUN指令覆盖ENTRYPOINT指令  

```
# shell模式
ENTRYPOINT command 参数1 参数2
```

```
# exec模式
ENTRYPOINT ['executable','参数1','参数2']
```

##### ADD & COPY
将其后的使用文件复制到Dockerfile镜像文件中  
ADD vs COPY  
ADD包含tar解压功能,如果只是单纯的复制文件,Docker推荐使用COPY  

##### VOLUME ['/data']
提供卷的功能,后续讲解  

##### WORKDIR
在容器内部设置执行目录,即CMD,ENTRYPOINT工作目录  

##### ENV 
设置环境变量  

```
ENV <key><value>
ENV <key>=<value>
```

##### USER
指定镜像是什么用户运行  

```
USER daemon
```
EXAMPLE

```
USER mysql
# 基于该镜像的指令使用mysql用户运行
```

##### ONBUILD
镜像触发器