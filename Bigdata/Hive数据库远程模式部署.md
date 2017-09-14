## Hive数据库远程模式部署

由于Hive运行在HDFS上，所以部署Hive之前需要先部署Hadoop,Hadoop部署的三种方式可参考[《Hadoop的三种模式部署-上篇》](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/Hadoop%E7%9A%84%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2-%E4%B8%8A%E7%AF%87.md)和[《Hadoop的三种模式部署-下篇》](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/Hadoop%E7%9A%84%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2-%E4%B8%8B%E7%AF%87.md)，本文将不在重复Hadoop的部署方式，本次Hive的部署基于Hadoop完全分布式部署环境的前提下。

Hive共有三种部署模式，分别为：内置模式，本地模式，远程模式。此处仅介绍生产环境常用的“远程模式”部署。

远程模式需要部署数据库，此处选择MariaDB,MariaDB的部署请参考[Debian8上源码安装MySQL5-6-xx](https://github.com/wing324/helloworld_zh/blob/master/MySQL/Debian8%E4%B8%8A%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85MySQL5-6-xx.md)。

#### 一、基础环境

- 主机信息

  hadoopmaster	192.168.1.1

  hadoopslave1	192.168.1.2

  hadoopslave2	192.168.1.3

  MySQL和Hive server段将部署在hadoopmaster上。

- 软件信息

  Linux: Debian8.2

  MariaDB: 10.1.22

  Java: 1.8.0_144

  Hadoop: 2.8.1

  Hive 2.3.0

- 目录信息

  Hive安装目录： /usr/local/hive

- HDFS目录信息

  由于Hive表创建之前需要在HDFS上存在相应的目录，所以目录规划如下：

  ```shell
  hdfs dfs -mkdir -p /user/hive/warehouse	#Hive的数据目录
  hdfs dfs -mkdir -p /user/hive/tmp	#Hive的临时目录
  hdfs dfs -mkdir -p /user/hive/log	#Hive的日志目录
  hdfs dfs -chmod g+w /user/hive/warehouse
  hdfs dfs -chmod a+w /usr/hive/tmp
  hdfs dfs -chmod g+w /usr/hive/log
  ```

- 三台机器上分别添加环境变量

  ```shell
  vim ~/.bashrc
  # 添加如下环境变量
  export HIVE_HOME=/usr/local/hive
  export PATH=$PATH:$HIVE_HOME/bin
  ```

#### 二、Hive远程模式部署

- 在MySQL中创建hive数据库以及hive用户

  ```sql
  mysql> create database hive;
  mysql> grant select,insert,update,delete,create,drop,index on hive.* to 'hive'@'%' identified by 'hive';
  ```

- 解压hive二进制安装包到/usr/local目录下

  ```shell
  tar apache-hive-2.3.0-bin.tar.gz -C /usr/local
  cd /usr/local
  mv apache-hive-2.3.0 hive
  chown -R hadoop:hadoop /usr/local/hive
  ```

- 重命名hive几个配置文件

  ```shell
  cd /usr/local/hive
  cp conf/hive-default.xml.template conf/hive-default.xml
  cp conf/hive-env.sh.template conf/hive-env.sh
  cp conf/hive-log4j2.properties.template conf/hive-log4j2.properties
  ```

- 在conf目录下新增hive-site.xml文件

  ```shell
  vim conf/hive-site.xml
  添加如下配置：
  <configuration>
      <property>
          <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://192.168.1.1:3306/hive?createDatabaseIfNotExist=true</value>
          <description>JDBC connect string for a JDBC metastore</description>
      </property>
      <property>
          <name>javax.jdo.option.ConnectionDriverName</name>
          <value>com.mysql.jdbc.Driver</value>
          <description>Driver class name for a JDBC metastore</description>
      </property>

      <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>hive</value>
          <description>username to use against metastore database</description>
      </property>
      <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>hive</value>
          <description>password to use against metastore database</description>
      </property>
      <property>
          <name>hive.exec.scratchdir</name>
          <value>/user/hive/tmp</value>
          <description>HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: ${hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
      </property>
      <property>
          <name>hive.metastore.warehouse.dir</name>
          <value>/user/hive/warehouse</value>
          <description>location of default database for the warehouse</description>
      </property>
      <property>
          <name>hive.querylog.location</name>
          <value>/user/hive/log</value>
          <description>Location of Hive run time structured log file</description>
      </property>
  </configuration>
  ```

- 从网上下载MySQL驱动(mysql-connector-java-5.1.44-bin.jar)放置在/usr/local/hive/lib文件夹下。

  驱动下载地址：https://dev.mysql.com/downloads/connector/j/

- 从 Hive 2.1 版本开始, 我们需要先运行 schematool 命令来执行初始化操作。

  ```shell
  schematool -dbType mysql -initSchema
  # 初始化成功之后，可以从MySQL数据库的hive库里面看到初始化的数据表
  ```

- Hive启动metastore服务

  ```shell
  hive --service metastore &
  # 此时通过jps命令，可以发现"RunJar"进程
  # 该进程的关闭，可以通过jps获取"RunJar"进程号，然后kill %jobid即可
  ```

- 访问Hive

  ```shell
  linux:/usr/local/hive $ hive
  SLF4J: Class path contains multiple SLF4J bindings.
  SLF4J: Found binding in [jar:file:/usr/local/hive/lib/log4j-slf4j-impl-2.6.2.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: Found binding in [jar:file:/usr/local/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
  SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
  SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

  Logging initialized using configuration in file:/usr/local/hive/conf/hive-log4j2.properties Async: true
  Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
  hive> show databases;
  OK
  default
  test1
  Time taken: 4.267 seconds, Fetched: 2 row(s)
  hive> use test1;
  OK
  Time taken: 0.024 seconds
  hive> show tables;
  OK
  t
  Time taken: 0.026 seconds, Fetched: 1 row(s)
  hive> show create table t;
  OK
  CREATE TABLE `t`(
    `id` int,
    `name` string)
  ROW FORMAT SERDE
    'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
  STORED AS INPUTFORMAT
    'org.apache.hadoop.mapred.TextInputFormat'
  OUTPUTFORMAT
    'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
  LOCATION
    'hdfs://172.16.0.220:9000/user/hive/warehouse/test1.db/t'
  TBLPROPERTIES (
    'transient_lastDdlTime'='1505356773')
  Time taken: 0.193 seconds, Fetched: 13 row(s)
  hive>
  ```

  ​