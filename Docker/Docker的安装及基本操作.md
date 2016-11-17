## Docker的安装及基本操作

如今的Docker容器越来越活，之前趁着周末对其进行了肤浅的学习，以下是其安装和基本使用。  


Docker的组成
-------------------
Docker Client 客户端  
Docker Daemon 守护进程/服务器  
Docker Image 镜像：存放用于启动容器中的各种条件，相当于容器的源代码  
Docker Container 容器：通过镜像启动，容器的执行来源  
Docker Registry 仓库：保存用户构建的镜像，公有(如docker hub)仓库和私有仓库  


Docker依赖的Linux内核特性
--------------------------------------
##### Namespace 命名空间
1. PID（Process ID）进程隔离  
2. NET（Network）管理网络接口  
3. IPC（InterProcess Communication）管理跨进程通信的访问  
4. MNT（Mount）管理挂载点  
5. UTS（Unix Timesharing System）隔离内核和版本标识  

##### Control Groups(cgroups) 控制组
资源限制  
优先级设定  
资源计量  
资源控制  


Docker容器的能力
-------------------------
1. 文件系统隔离  
   每个容器都有自己的root文件系统  
2. 进程隔离  
   每个容器都运行在自己的进程环境中  
3. 网络隔离  
   荣期间的虚拟网络接口个IP地址都是分开的  
4. 资源的隔离和分组  
   使用cgroups将CPU和内存之类的资源独立分配给每个Docker容器  


Docker在Linux下的安装
---------------------------------
操作系统：CentOS7  
Docker版本：1.7.1

安装Docker命令

```shell
yum install -y docker-io
```


Docker守护进程的配置和操作
-----------------------------------------
service docker start：启动docker守护进程  
service docker stop：关闭docker守护进程  
service docker restart：重启docker守护进程  
