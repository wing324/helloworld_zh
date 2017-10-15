## Sqoop quickstart

##### 一、基础软件

Linux Debian8.2  
Sqoop 1.4.6  
Hadoop 2.8.1  
MySQL mariadb10.1.22  
jdbc驱动 5.1.44  
Sqoop的安装需要建立在HDFS之上，HDFS是Hadoop的一种文件系统，Hadoop的安装请参考[Hadoop的三种模式部署-上篇](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/Hadoop%E7%9A%84%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2-%E4%B8%8A%E7%AF%87.md)和[Hadoop的三种模式部署-下篇](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/Hadoop%E7%9A%84%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2-%E4%B8%8B%E7%AF%87.md), 本文将不再赘述。

##### 二、Sqoop的作用

Sqoop是一款用于同步关系型数据库到HDFS上的开源工具。官方介绍请移步[这里](http://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_introduction)。

##### 三、Sqoop的安装

1. 下载地址  
   http://www.apache.org/dyn/closer.lua/sqoop/  

2. 安装步骤  

   ```shell
   # 解压到/usr/local下
   tar sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz -C /usr/local
   mv sqoop-1.4.6.bin__hadoop-2.0.4 sqoop

   # 在/etc/profile添加Sqoop环境变量
   export SQOOP_HOME=/usr/local/sqoop
   export PATH=$PATH:$SQOOP_HOME/bin

   # 使环境变量立即生效
   source /etc/profile
   ```

##### 四、Sqoop的使用

1. 查看帮助信息  

   ```shell
   sqoop help
   ```

2. 查看子命令的帮助信息  

   ```shell
   sqoop import --help
   ```

3. 从MySQL中表数据导出到HDFS上

   ```shell
   # 从192.168.0.1:3306的hadoopguide数据库上导出widgets表的全部数据到HDFS上
   sqoop import --connect jdbc:mysql://192.168.0.1:3306/hadoopguide --table widgets -m 1 --username wing -P
   # 此处的-P为交互式输入MySQL密码

   # 查看HDFS是否从MySQL中导出该表
   Linux> hdfs dfs -cat widgets/part-m-00000
   1,sprocket,0.25,2010-02-10,1,Connects two gizmons
   2,gizmo,4.00,2009-11-30,4,null
   3,gadget,99.99,1983-08-13,13,Our flagship product
   # 由此可见该命令已经将MySQL中表数据成功导出到HDFS上
   ```

   **TIPS**  

   - 默认情况下,$SQOOP_HOME/lib下没有JDBC驱动，需要自行下载并移动到该目录下，否则会爆出找不到JDBC的错误。
   - `y-m 1`这个参数表示指定一个map任务，默认情况下-m=4，此处指定-m 1的目的是为了查看HDFS文件的时候所有的数据都在一个文件内。
   - Sqoop默认导出文件的分隔符为逗号(',')分隔。

4. 将导出数据的过程生成源代码  
   默认使用3中的导出语句会在当前的目录下生成一个Widgets.java源代码文件，那么如何手动生成源代码文件呢？  

   ```shell
   sqoop codegen --connect jdbc:mysql://192.168.0.1:3306/hadoopguide --table widgets --username wing -P --class-name Widget
   # 此时会在当前目录下生成一个Widget.java的源代码文件。该操作并不会真正的从MySQL中导出数据，但仍然会做一些检查性的工作。
   ```

5. 使用Sqoop将MySQL数据表导入到Hive中  

   ```shell
   # 将widgets表的数据导入到HDFS上
   sqoop create-hive-table  --connect jdbc:mysql:/192.168.0.1:3306/hadoopguide --table widgets --username wing -P --fields-terminated-by ','

   # 此时可以在/user/Hadoop目录下看到多出来的widgets目录

   # 在Hive中创建widgets表结构
   CREATE TABLE `widgets` (
     `id` int,
     `widget_name` string,
     `price` decimal,
     `design_date` date,
     `version` int,
     `design_comment` string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
   ```

   ​

   ​