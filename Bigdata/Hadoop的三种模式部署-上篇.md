## Hadoop的三种模式部署(上)

Linux: Debian8.2

Java: 1.8.0_144

Hadoop: Hadoop 3.0.0-alpha4

Hadoop安装包下载地址：http://hadoop.apache.org/releases.html

Java部署请参考[《常用软件部署—Debian版》](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6%E9%83%A8%E7%BD%B2--Debian%E7%89%88.md)



**该篇将分别讲解Hadoop的本地模式和伪分布模式的部署。**

#### 一、基本配置（三种部署模式都需要的基本步骤）

1. 安装依赖包

   ```shell
   apt-get install ssh pdsh
   ```

2. 解压Hadoop安装包至指定的目录下

   ```shell
   tar -zxvf hadoop-3.0.0-alpha4.tar.gz -C /usr/local/
   cd /usr/local
   mv hadoop-3.0.0-alpha4/ hadoop
   cd hadoop/
   ```

3. 配置环境变量

   ```shell
   编辑etc/hadoop/hadoop-env.sh文件
   vim etc/hadoop/hadoop-env.sh
   配置以下选项
   # set to the root of your Java installation
     export JAVA_HOME=/usr/local/jdk
   ```

4. 尝试hadoop命令，验证环境变量是否正确

   ```shell
   bin/hadoop
   #此时如果显示一系列的hadoop命令说明，则环境变量正确。否则，请检查之前的步骤是否正确。
   ```

#### 二、本地模式(Local/Standalone Mode)

1. Hadoop默认配置下，是个local mode，即可以立即使用local operation。

2. 操作示例

   ```shell
   mkdir input
   cp etc/hadoop/*.xml input
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha4.jar grep input output 'dfs[a-z.]+'
   cat output/*
   ```

#### 三、伪分布模式(Pesudo-Distributed Mode)

1. 配置

   - 配置/etc/hosts文件

     ```shell
     添加：
     192.18.16.220 hadoop-master
     ```

   - 配置pdsh的默认使用ssh

     ```shell
     vim ~/.bashrc
     添加：
     export PDSH_RCMD_TYPE=ssh

     source ~/.bashrc
     # 使其在当前会话中立即生效
     ```

   - 配置本机ssh root@localhost免密登录

     具体配置方式见[《SSH免密码登录操作步骤 》](https://github.com/wing324/helloworld_zh/blob/master/Linux/SSH%E5%85%8D%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95%E6%93%8D%E4%BD%9C%E6%AD%A5%E9%AA%A4.md)

   - 配置etc/hadoop/hadoop-env.sh文件

     ```shell
     添加如下变量：(目前我使用的是root用户，所以配置root,如果使用别的用户，请将root用户替换成你使用的用户)
     export HDFS_NAMENODE_USER=root
     export HDFS_DATANODE_USER=root
     export HDFS_SECONDARYNAMENODE_USER=root
     ```

   - 配置etc/hadoop/core-site.xml文件

     ```shell
     <configuration>
         <property>
             <name>fs.defaultFS</name>
             <value>hdfs://192.18.16.220:9000</value>
         </property>
     </configuration>
     ```

   - 配置etc/hadoop/hdfs-site.xml文件

     ```shell
     <configuration>
         <property>
             <name>dfs.replication</name>
             <value>1</value>
         </property>
     </configuration>
     ```

2. 操作示例

   - 初始化文件系统

     ```shell
     bin/hdfs namenode -format
     ```

   - 启动NameNode和DataNode节点

     ```shell
     sbin/start-dfs.sh
     ```

     此时可以登录NameNode的web端：http://192.18.16.220:9870

   - 创建hdfs的目录

     ```shell
      bin/hdfs dfs -mkdir /user
      bin/hdfs dfs -mkdir /user/<username>
     ```

   - 复制input文件的内容到分布式文件系统上

     ```shell
     bin/hdfs dfs -mkdir input
     bin/hdfs dfs -put etc/hadoop/*.xml input
     ```

   - 执行一个示例操作

     ```shell
     bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.0-alpha4.jar grep input output 'dfs[a-z.]+'
     ```

   - 查看输出文件

     ```shell
     bin/hdfs dfs -get output output
     cat output/*
     或者
     bin/hdfs dfs -cat output/*
     ```

   - 停止NameNode和DataNode节点

     ```shell
     sbin/stop-dfs.sh
     ```

