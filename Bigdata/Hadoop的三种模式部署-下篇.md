## Hadoop的三种模式部署（下）

Linux: Debian8.2

Java: 1.8.0_144

Hadoop: Hadoop 2.8.1

Hadoop安装包下载地址：<http://hadoop.apache.org/releases.html>

Java部署请参考[《常用软件部署—Debian版》](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6%E9%83%A8%E7%BD%B2--Debian%E7%89%88.md)



**该篇将演示Hadoop的完全分布式部署，并且以hadoop用户启动。**

参考文档链接：https://chu888chu888.gitbooks.io/hadoopstudy/content/Content/4/chapter0404.html



#### 一、基础环境准备

- 主机信息

  hadoopmaster	192.168.1.1

  hadoopslave1	192.168.1.2

  hadoopslave2	192.168.1.3

- 软件信息

  - Debian8.2
  - JAVA1.8.0_144
  - Hadoop 2.8.1

- 目录信息

  Java安装目录: `/usr/local/jdk`

  Hadoop安装目录: `/usr/local/hadoop`

  Hadoop数据目录: `/data/hadoop`

- 安装依赖包，三台主机分别执行如下命令：

  ```shell
  apt-get install ssh pdsh
  ```

- 三台机器配置hadoop:hadoop组合用户，三台主机分别执行如下命令：

  ```shell
  addgroup hadoop
  adduser -ingroup hadoop hadoop
  ```

- 三台主机分别配置/etc/hosts文件

  ```shell
  vim /etc/hosts
  添加如下主机名
  192.168.1.1	hadoopmaster
  192.168.1.2 hadoopslave1
  192.168.1.3 hadoopslave2
  ```

- 三台主机使用hadoop用户登录，并添加如下环境变量：

  ```shell
  vim /home/hadoop/.bashrc
  添加如下变量：
  export JAVA_HOME=/usr/local/jdk
  export JRE_HOME=/usr/local/jdk/jre
  export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
  export HADOOP_HOME=/usr/local/hadoop
  export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$HADOOP_HOME/bin
  export PDSH_RCMD_TYPE=ssh
  if [[ $- == *i* ]]
  then
      PS1="\[$(tput bold)\]\[$(tput setaf 3)\]\u\[$(tput setaf 7)\]@\[$(tput setaf 5)\]\h:\[$(tput setaf 2)\]\w\[$(tput setaf 4)\] \\$\[$(tput sgr0)\] "
  fi

  # 立即生效环境变量
  source /home/hadoop/.bashrc
  ```

- 三台主机相互之前SSH免密码登录（主机和主机本身也需要SSH免密码登录），详细信息请参考[《SSH免密码登录操作步骤》](https://github.com/wing324/helloworld_zh/blob/master/Linux/SSH%E5%85%8D%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95%E6%93%8D%E4%BD%9C%E6%AD%A5%E9%AA%A4.md)

  ​

#### 二、Hadoop完全分布式配置

**以下所有操作没有特殊说明的前提下，使用的是hadoop用户进行操作。**

- hadoopmaster机器上解压文件包(以root用户操作)

  ```shell
  tar -zxvf hadoop-2.8.1.tar.gz -C /usr/local
  mv hadoop-2.8.1 hadoop
  chown -R hadoop:hadoop /usr/local/hadoop
  ```

- hadoopmaster机器上新增/data/hadoop/tmp数据目录(以root用户操作)

  ```shell
  mkdir /data/hadoop/tmp
  chown -R hadoop:hadoop /data/hadoop/tmp
  ```

- hadoopmaster机器上配置hadoop-env.sh文件

  ```shell
  cd /usr/local/hadoop
  vim etc/hadoop/hadoop-env.sh
  更新如下变量：
  export JAVA_HOME=/usr/local/jdk
  ```

- hadoopmaster机器上配置集群环境，正常启动最小配置文件： slaves、core-site.xml、hdfs-site.xml、mapred-site.xml、yarn-site.xml

  - 配置etc/hadoop/slaves文件

    ```shell
    vim etc/hadoop/slaves
    添加如下文件：
    192.168.1.2
    192.168.1.3
    ```

  - 配置core-site.xml文件

    ```shell
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://192.168.1.1:9000</value>
        </property>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>file:/data/hadoop/tmp</value>
            <description>Abase for other temporary directories.</description>
        </property>
    </configuration>
    ```

  - 配置hfs-site.xml文件

    ```shell
    <configuration>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>192.168.1.1:50090</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>file:/data/hadoop/tmp/dfs/name</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/data/hadoop/tmp/dfs/data</value>
        </property>
    </configuration>
    ```

  - 配置mapred-site.xml文件

    ```shell
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.address</name>
            <value>172.168.1.1:10020</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>192.168.1.1:19888</value>
        </property>
    </configuration>
    ```

  - 配置yarn-site.xml文件

    ```shell
    <configuration>

    <!-- Site specific YARN configuration properties -->
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>192.168.1.1</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
    ```

  hadoopslave1和hadoopslave2可以将hadoopmaster的/usr/local/hadoop完全copy一份至本机的/usr/local/hadoop目录下。其中/usr/local/hadoop/etc/hadoop/slaves文件在两台slave机器上是无用的，但是该文件存在并无其他影响。

  ​

  #### 三、Hadoop完全分布式启动

  - hadoopmaster上执行初始化，执行一次即可

    ```shell
    hdfs namenode -format
    ```

  - hadoopmaster上执行

    ```shell
    sbin/start-dfs.sh
    sbin/start-yarn.sh
    sbin/mr-jobhistory-daemon.sh start historyserver
    ```



#### 四、Hadoop完全分布式部署完成验证

- 三台主机上分别执行jps查看java进程

  ```shell
  hadoopmaster:/usr/local/hadoop $ jps
  19360 JobHistoryServer
  19012 ResourceManager
  18806 SecondaryNameNode
  18620 NameNode
  20190 Jps

  hadoopslave1:/usr/local/hadoop $ jps
  17393 NodeManager
  17282 DataNode
  17907 Jps

  hadoopslave2:/usr/local/hadoop $ jps
  21720 NodeManager
  21609 DataNode
  21837 Jps
  ```

- hadoopmaster执行`hdfs dfsadmin -report`查看各个数据节点是否正常

  ```shell
  hadoopmaster:/usr/local/hadoop $ hdfs dfsadmin -report
  Configured Capacity: 42398629888 (39.49 GB)
  Present Capacity: 11841167360 (11.03 GB)
  DFS Remaining: 11841110016 (11.03 GB)
  DFS Used: 57344 (56 KB)
  DFS Used%: 0.00%
  Under replicated blocks: 0
  Blocks with corrupt replicas: 0
  Missing blocks: 0
  Missing blocks (with replication factor 1): 0
  Pending deletion blocks: 0

  -------------------------------------------------
  Live datanodes (2):

  Name: 192.168.1.2:50010 (hadoopslave1)
  Hostname: hadoopslave1
  Decommission Status : Normal
  Configured Capacity: 21199314944 (19.74 GB)
  DFS Used: 28672 (28 KB)
  Non DFS Used: 14452621312 (13.46 GB)
  DFS Remaining: 5654732800 (5.27 GB)
  DFS Used%: 0.00%
  DFS Remaining%: 26.67%
  Configured Cache Capacity: 0 (0 B)
  Cache Used: 0 (0 B)
  Cache Remaining: 0 (0 B)
  Cache Used%: 100.00%
  Cache Remaining%: 0.00%
  Xceivers: 1
  Last contact: Wed Sep 13 12:00:35 CST 2017


  Name: 192.168.1.2:50010 (hadoopslave2)
  Hostname: hadoopslave2
  Decommission Status : Normal
  Configured Capacity: 21199314944 (19.74 GB)
  DFS Used: 28672 (28 KB)
  Non DFS Used: 13920976896 (12.96 GB)
  DFS Remaining: 6186377216 (5.76 GB)
  DFS Used%: 0.00%
  DFS Remaining%: 29.18%
  Configured Cache Capacity: 0 (0 B)
  Cache Used: 0 (0 B)
  Cache Remaining: 0 (0 B)
  Cache Used%: 100.00%
  Cache Remaining%: 0.00%
  Xceivers: 1
  Last contact: Wed Sep 13 12:00:35 CST 2017
  ```

- 打开浏览器，验证是否能打开如下四个网址：

  MapReduce JobHistory Server		http://192.168.1.1:19888

  ResourceManager				http://192.168.1.1:8088/

  NameNode						http://192.168.1.1:50070

  DataNode						http://192.168.1.2:50075 http://192.168.1.3:50075



#### 五、Hadoop集群关闭

```shell
stop-yarn.sh
stop-dfs.sh
mr-jobhistory-daemon.sh stop historyserver
```

