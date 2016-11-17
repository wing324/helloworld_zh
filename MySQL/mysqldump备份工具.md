## mysqldump备份工具

作为一名DBA，备份是件很重要的工作，如果备份没有做好，那就大事不好啦。。mysqldump作为MySQL自带一款备份工具，还是大有用途的。
经常与mysqldump工具做比较的另一款Percona Xtrabackup，详情请见：
http://wing324.github.io/tags/Percona-Xtrabackup/  



简介
----
mysqldump是一款逻辑备份工具，针对原库中的定义和数据产生一系列可执行的SQL语句。


权限
----
1. select:备份表需要的权限
2. show view:备份视图需要的权限
3. trigger:备份触发器需要的权限
4. lock tables:如果没有使用--single-transaction时需要的权限
5. 导入表时，需要有相应的权限执行SQL语句，例如CREATE权限等
6. 其他权限视选项而定  
   **注意**  
   mysqldump中可能会含有ALTER DATABASE语句,这种情况是发生在使用--routines选项备份存储过程时（此处存储过程一定要存在才会发生这种现象），在存储过程写下后，又更改了数据库的字符集，此时才会发现mysqldump中会包含ALTER DATABASE语句,此处在导入表时，需要有ALTER 权限。  
```
# 原来数据库字符集为utf8
root@localhost : wing 10:44:01> show create database wing;
+-----------+--------------------------------------------------------------------+
| Database  | Create Database                                                    |
+-----------+--------------------------------------------------------------------+
| wing      | CREATE DATABASE `wing` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+-----------+--------------------------------------------------------------------+

#内部包含存储过程，此时执行如下命令
[root@mysql tmpdir]# mysqldump -uroot --socket=/data/mysqldata3306/sock/mysql.sock -p --routines --single-transaction wing   > dumpfile.sql
[root@mysql tmpdir]# less dumpfile.sql
此时会发现如下这段信息，此处不存在ALTER DATABASES语句：
--
-- Dumping routines for database 'wing'
--
/*!50003 DROP PROCEDURE IF EXISTS `sp_in` */;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8 */ ;
/*!50003 SET character_set_results = utf8 */ ;
/*!50003 SET collation_connection  = utf8_general_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `sp_in`(in id int)
BEGIN
SELECT id AS id_init;
IF (id is not null) THEN
set id = id + 1;
END IF;
SELECT id AS id_out;
END ;;
DELIMITER ; 

# 此时更改数据库字符集
root@localhost : wing 11:03:20> alter database wing character set = latin1;
[root@mysql tmpdir]# mysqldump -uroot --socket=/data/mysqldata3306/sock/mysql.sock -p --routines --single-transaction wing   > dumpfile1.sql
[root@mysql tmpdir]# less dumpfile1.sql
此时会发现如下这段信息，此处存在ALTER DATABASES语句：
--
-- Dumping routines for database 'wing'
--
/*!50003 DROP PROCEDURE IF EXISTS `sp_in` */;
ALTER DATABASE `wing` CHARACTER SET utf8 COLLATE utf8_general_ci ;
/*!50003 SET @saved_cs_client      = @@character_set_client */ ;
/*!50003 SET @saved_cs_results     = @@character_set_results */ ;
/*!50003 SET @saved_col_connection = @@collation_connection */ ;
/*!50003 SET character_set_client  = utf8 */ ;
/*!50003 SET character_set_results = utf8 */ ;
/*!50003 SET collation_connection  = utf8_general_ci */ ;
/*!50003 SET @saved_sql_mode       = @@sql_mode */ ;
/*!50003 SET sql_mode              = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION' */ ;
DELIMITER ;;
CREATE DEFINER=`root`@`localhost` PROCEDURE `sp_in`(in id int)
BEGIN
SELECT id AS id_init;
IF (id is not null) THEN
set id = id + 1;
END IF;
SELECT id AS id_out;
END ;;
DELIMITER ;
```


语法
----
```
mysqldump [options] db_name [tbl_name ...]
mysqldump [options] --databases db_name ...
mysqldump [options] --all-databases
```


参数详解
-------
此处仅列出个人认为还是比较重要的参数，毕竟参数选项太多了，更多的参数可通过mysqldump --help查看。  
#### 连接参数
- --compress,-C  
  压缩所有的信息在服务器和客户端之间传输  
- --host.-h  
  连接MySQL服务器的主机名  
- --login-path  
  连接MySQL服务器的一种方式，详情请见：  
  http://wing324.github.io/2015/09/28/mysql-config-editor%E5%B7%A5%E5%85%B7/
- --password,-p  
  连接MySQL服务器的密码  
- --port，-P  
  连接MySQL服务器的端口  
- --socket,-S  
  连接MySQL服务器的socket文件  
- --user,-u  
  连接MySQL服务器的用户名  
- --max_allowed_packet  
  服务器/客户端之间传输包的最大数量  
- --net_buffer_length  
  用于建立连接时的连接缓冲和结果缓冲  

#### 设置文件参数
- --print-defaults  
  输出mysqldump默认选项  
```
# 只是一个例子啦
[root@mysql log]# mysqldump --login-path=root3306 --print-defaults
mysqldump would have been started with the following arguments:
--quick --max_allowed_packet=2G --default-character-set=utf8 --user=root --password=root --socket=/data/mysqldata3306/sock/mysql.sock
```
- --defaults-file  
  指定配置文件  

#### DDL参数
- --add-drop-database  
  在每个CREATE DATABASE前面添加DROP DATABASE语句  
- --add-drop-table  
  在每个CREATE TABLE前面添加DROP TABLE语句  
- --add-drop-trigger  
  在每个CREATE TRIGGER前面添加DROP TRIGGER语句  
- --all-tablespaces  
  导出所有的表空间，不会有表空间的相关信息出现在mysqldump的输出文件中，该参数只适用于MySQL集群表  
- --no-create-db,-n  
  使用--databases或者--all-databases不使用CREATE DATABASE语句  
```
# 不添加-n参数可发现存在CREATE DATABASE语句
[root@mysql tmpdir]#  mysqldump --login-path=root3306 --single-transaction --databases test > t1.sql
[root@mysql tmpdir]# less t1.sql
-- ------------------------------------------------------
-- Server version       5.6.26-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Current Database: `test`
--

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `test` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `test`;

--
-- Table structure for table `t1`
--

# 添加-n则不存在CREATE DATABASE语句
[root@mysql tmpdir]#  mysqldump --login-path=root3306 --single-transaction -n --databases  test > t2.sql
[root@mysql tmpdir]# less t2.sql
-- ------------------------------------------------------
-- Server version       5.6.26-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Current Database: `test`
--

USE `test`;

--
-- Table structure for table `t1`
--
```
- --no-create-info,-t  
  在备份表时，不写CREATE TABLE语句，即该参数不导出表结构，只导出数据  
- --no-tablespances,-y  
  mysqldump输出文件中不包含CREATE LOGFILE GROUP/CREATE TABLESPACE语句  
- --replace  
  使用REPLACE语句代替INSERT语句  
```
# 未使用--replace
mysqldump --login-path=root3306 --single-transaction --databases test > t2.sql
[root@mysql tmpdir]# less t2.sql 
-- 部分输出如下
LOCK TABLES `t1` WRITE;
/*!40000 ALTER TABLE `t1` DISABLE KEYS */;
INSERT INTO `t1` VALUES (1,'FMmer58',15),(2,'INLGVwn',19),(3,'AVJTHCw',31),(4,'WsA2Oae',39),(5,'gTnLLnW',99),(6,'95D4ooj',12),(7,'ZnyNFxz',59),(8,'8ajAUfF',24),(9,'hsAx4YD',69),(10,'huG3rP1',18);
/*!40000 ALTER TABLE `t1` ENABLE KEYS */;
UNLOCK TABLES;

#使用--replace
[root@mysql tmpdir]#  mysqldump --login-path=root3306 --single-transaction --replace --databases test > t3.sql
[root@mysql tmpdir]# less t3.sql
-- 部分输出如下
LOCK TABLES `t1` WRITE;
/*!40000 ALTER TABLE `t1` DISABLE KEYS */;
REPLACE INTO `t1` VALUES (1,'FMmer58',15),(2,'INLGVwn',19),(3,'AVJTHCw',31),(4,'WsA2Oae',39),(5,'gTnLLnW',99),(6,'95D4ooj',12),(7,'ZnyNFxz',59),(8,'8ajAUfF',24),(9,'hsAx4YD',69),(10,'huG3rP1',18);
/*!40000 ALTER TABLE `t1` ENABLE KEYS */;
UNLOCK TABLES;

# 观察发现REPLACE语句取代了INSERT语句
```
- --allow-keywords  
  允许列名为关键字  
- --comment,-i  
  附加注释信息  
- --force,-f
  即使在备份表时出现SQL错误，依旧继续备份。即使一个view的基表被删除，那么该view将会无效，但如果使用--force，会出现报错信息，但mysqldump以及继续工作。  
- --log-error  
  错误信息和警告信息都将追加到该文件中  
- --skip-comments  
  不备份注释信息  

#### 复制选项

- --apply-slave-statements  
  使用--dump-slave选项，在CHANGE MASTER TO之前添加STOP SLAVE,在备份文件末尾添加START SLAVE  
- --delete-master-logs  
  该选项会在备份时，像服务器发送PURGE BINARY LOGS语句删除二进制日志，该选项会自动开启--master-data选项。  
- --dump-slave=0/1/2
  与参数--master-slave相似，用来备份一个slave机器的数据文件可以直接用于建立另一个slave作为备份文件的master的从机。使用该参数的备份文件中存在master机器的CHANGE MASTER TO的MASTER_LOG_FILE和MASTER_LOG_POS参数，用于为该master新建一个slave机器使用。当--dump-slave和--master-data一起使用时，--master-data参数会被忽略。  
  值为0：  
  并没有CHANGE MASTER TO的信息；  
```
# --dump-slave=0
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `company`
--
```
值为1：  
存在CHANGE MASTER TO的信息，并且为可执行的；  
```
--dump-slave=1
--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--

CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=70606941;

--
-- Table structure for table `company`
--
```
值为2：
存在CHANGE MASTER TO的信息，以注释方式出现。  
```
# --dump-slave=2
--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--

-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=70607833;

--
-- Table structure for table `company`
--
```
**注意**  
```
# 使用--dump-slave参数，会输出备份机器的master机器的如下CHANGE MASTER TO信息，注意该信息并非是备份机器的信息，而是备份机器的master机器信息。
--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--

CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=70575052;
```
- --include-master-host-port  
  对于--dump-slave参数产生的CHANGE MASTER TO语句，会增加MASTER_HOST和MASTER_PORT参数。  
  **拓展**  
  如果master机器已经存在一个或多个slave,其实想给master机器增加更多的slave,那么要做的事情就是对其slave从机使用--dump-slave做备份文件。  
```
# 使用命令
mysqldump -uroot --socket=/data/mysql/mysqldata3306/sock/mysql.sock --single-transaction --dump-slave=1 --apply-slave-statements --include-master-host-port -p --all-databases > 备份文件名称
Enter password: 

# 和复制相关信息
--
-- stop slave statement to make a recovery dump)
--

STOP SLAVE;

--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--

CHANGE MASTER TO MASTER_HOST='192.168.200.158', MASTER_PORT='3306', MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=70729145;

--
-- Table structure for table `company`
--
...
（备份数据完毕后）
--
-- start slave statement to make a recovery dump)
--

START SLAVE;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2015-10-24 10:41:21
```
- --master-data  
  该参数用于master机器的备份文件用于设置另一个机器为该master机器的slave从机，它会产生备份机器的CHANGE MASTER TO的MASTER_LOG_FILE和MASTER_LOG_POS参数。  
  值为0：  
  备份文件中并不会存在CHANGE MASTER TO的参数信息；  
```
# --master-data=0
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `company`
--
```
值为1：  
CHANGE MASTER TO产生的参数以语句的形式出现在备份文件中，导入文件时起作用；  
```
# --master-data=1
--
-- Position to start replication or point-in-time recovery from
--

CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000007', MASTER_LOG_POS=74942041;

--
-- Table structure for table `company`
--
```
值为2：  
CHANGE MASTER TO产生的参数以注释形式出现在备份文件中，导入文件时并不起作用。  
```
# --master-data=2
--
-- Position to start replication or point-in-time recovery from
--

-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000007', MASTER_LOG_POS=74942749;

--
-- Table structure for table `company`
--
```
**注意**  
1. 该参数的开启需要RELOAD权限，且需要开启二进制日志；  
2. 该参数会自动关闭--lock-tables,在没有使用参数--single-transaction时也会自动开启--lock-all-tables。  

- --set-gtid-purged=OFF/ON/AUTO  
  该参数表示是否在备份文件中添加SET @@global.gtid_purged。  
  值为OFF:  
  不添加SET @@global.gtid_purged至备份文件中；  
  值为ON：  
  添加SET @@global.gtid_purged至备份文件中，但需要开启GTID参数，否则会报错；  
  值为AUTO:  
  如果服务器开启了GTID，则添加SET @@global.gtid_purged至备份文件中，否则不添加。  

#### 格式参数
- --compact  
  开启--skip-add-drop-table,--skip-add-locks,--skip-comments,--skip-disable-keys,--skip-set-charset参数，以达到压缩的目的。  
```
# 使用--compact
[root@host-192-168-200-154 tmpdir]# du -h t0.sql 
4.0K    t0.sql
# 其备份文件如下
[root@host-192-168-200-154 tmpdir]# less t0.sql 
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `company` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `company_address` (
  `id` bigint(20) unsigned NOT NULL,
  `address` varchar(256) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `createtest` (
  `id` varchar(1) DEFAULT NULL
...

# 未使用--compact
[root@host-192-168-200-154 tmpdir]# du -h t1.sql 
8.0K    t1.sql
#其备份文件如下
[root@host-192-168-200-154 tmpdir]# less t1.sql 
-- MySQL dump 10.13  Distrib 5.6.11, for Linux (x86_64)
--
-- Host: localhost    Database: hotdb_01
-- ------------------------------------------------------
-- Server version       5.6.11-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `company`
--

DROP TABLE IF EXISTS `company`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `company` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Table structure for table `company_address`
--

DROP TABLE IF EXISTS `company_address`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `company_address` (
  `id` bigint(20) unsigned NOT NULL,
  `address` varchar(256) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;
...
```

- --compatible=name  
  告诉MySQL,导出的备份文件尽量与name兼容;name可以为ansi, mysql323, mysql40, postgresql, oracle, mssql, db2, maxdb, no_key_options, no_table_options, no_field_options这些字段。  

- --complete-insert  
  INSERT语句包括完整的列名。  

- --create-options  
  CREATE TABLE语句中包括所有的MySQL表选项。  

- --tab  
  对备份的数据库中每个表都会产生两个文件，一份是table_name.sql，记录的是CREATE TABLE创建的表结构，一份是table_name.txt，记录的是该表的数据，并非包含INSERT语句。  
```
# 使用--tab参数
mysqldump --login-path=root3306 --single-transaction --tab=/data/mysqldata3306/tmpdir/  hotdb

# 结果
-rw-r--r-- 1 root  root   1391 Oct 25 21:10 company_address.sql
-rw-rw-rw- 1 mysql mysql   120 Oct 25 21:10 company_address.txt
-rw-r--r-- 1 root  root   1395 Oct 25 21:10 company.sql
-rw-rw-rw- 1 mysql mysql    16 Oct 25 21:10 company.txt
-rw-r--r-- 1 root  root   3755 Oct 25 21:10 createtest.sql
-rw-rw-rw- 1 mysql mysql     0 Oct 25 21:10 createtest.txt
-rw-r--r-- 1 root  root   1973 Oct 25 21:10 customer.sql
-rw-rw-rw- 1 mysql mysql  6251 Oct 25 21:10 customer.txt
-rw-r--r-- 1 root  root   1829 Oct 25 21:10 orderlist.sql
-rw-rw-rw- 1 mysql mysql 28054 Oct 25 21:10 orderlist.txt
-rw-r--r-- 1 root  root   1711 Oct 25 21:10 product.sql
-rw-rw-rw- 1 mysql mysql   329 Oct 25 21:10 product.txt

# 查看.sql和.txt文件
[root@mysql tmpdir]# less company_address.sql 
-- MySQL dump 10.13  Distrib 5.6.26, for Linux (x86_64)
--
-- Host: localhost    Database: hotdb
-- ------------------------------------------------------
-- Server version       5.6.26-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `company_address`
--

DROP TABLE IF EXISTS `company_address`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `company_address` (
  `id` bigint(20) unsigned NOT NULL,
  `address` varchar(256) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2015-10-25 21:17:58

[root@mysql tmpdir]# less company_address.txt 
1       北京省北京市某某区
2       上海省上海市某某区
3       江苏省南京市某某区
4       广东省广州市某某区
```
**特别说明**  
1. 使用该参数需要FILE权限。  
2. 服务器必须拥有写入到目录的权限。  
- --xml,-X  
  备份文件为xml文件。  
```
# 使用--xml参数
mysqldump --login-path=root3306 --single-transaction --xml hotdb> hotdb.sql
[root@mysql tmpdir]# ll
total 168
-rw-r--r-- 1 root root 170151 Oct 25 21:26 hotdb.sql
[root@mysql tmpdir]# less hotdb.sql 
<?xml version="1.0"?>
<mysqldump xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<database name="hotdb">
        <table_structure name="company">
                <field Field="id" Type="bigint(20) unsigned" Null="NO" Key="PRI" Extra="auto_increment" Comment="" />
                <field Field="name" Type="varchar(64)" Null="NO" Key="" Extra="" Comment="" />
                <key Table="company" Non_unique="0" Key_name="PRIMARY" Seq_in_index="1" Column_name="id" Collation="A" Cardinality="4" Null="" Index_type="BTREE" Comment="" Index_comment="" />
                <options Name="company" Engine="InnoDB" Version="10" Row_format="Compact" Rows="4" Avg_row_length="4096" Data_length="16384" Max_data_length="0" Index_length="0" Data_free="0" Auto_increment="5" Create_time="2015-10-22 08:11:26" Collation="utf8_general_ci" Create_options="" Comment="" />
        </table_structure>
        <table_data name="company">
        <row>
                <field name="id">1</field>
                <field name="name">A</field>
        </row>
        <row>
                <field name="id">2</field>
                <field name="name">B</field>
        </row>
...
```

#### 过滤选项
- --all-databases,-A  
  备份所有数据库。  
- --database,-B  
  备份指定的数据库，备份几个数据库。  
- --events,-E  
  备份数据库的event,需要EVENT权限。  
- --ignore-table=db_name.table_name  
  不备份指定的表。  
- --no-data,-d  
  不备份表数据。  
- --routines,-R  
  备份数据库的stored routines(存储过程+函数)，需要对mysql.proc表的SELECT权限。  
- --tables  
  只备份指定的表,会忽略--databases/-B参数。  
```
# 使用--tables参数，其后接的第一个参数是数据库名，从第二个参数开始均为表名
mysqldump --login-path=root3306 --tables hotdb company customer > hotdb.sql
```
- --triggers  
  备份数据库的触发器。  
- --where='where条件'，-w 'where条件'  
  只备份指定条件的数据。  
```
# 使用--where参数
mysqldump --login-path=root3306 -w 'id=1' --tables hotdb customer > hotdb.sql

[root@mysql tmpdir]# less hotdb.sql 
...
--
-- Dumping data for table `customer`
--
-- WHERE:  id=1

LOCK TABLES `customer` WRITE;
/*!40000 ALTER TABLE `customer` DISABLE KEYS */;
INSERT INTO `customer` VALUES (1,'赵合肥','13912340001',1,'Anhui','合肥','某某街某某号');
/*!40000 ALTER TABLE `customer` ENABLE KEYS */;
UNLOCK TABLES;
...

mysqldump --login-path=root3306 --single-transaction  -w "city=' 上海'" --tables hotdb customer > hotdb.sql

[root@mysql tmpdir]# mysqldump --login-path=root3306 --single-transaction  -w "city=' 上海'" --tables hotdb customer > hotdb.sql
...
--
-- Dumping data for table `customer`
--
-- WHERE:  city='上海'

LOCK TABLES `customer` WRITE;
/*!40000 ALTER TABLE `customer` DISABLE KEYS */;
INSERT INTO `customer` VALUES (75,'罗上海','13912340075',25,'Shanghai','上海','某某街
某某号'),(76,'毕上海','13912340076',25,'Shanghai','上海','某某街某某号'),(77,'郝上海','13912340077',25,'Shanghai','上海','某某街某某号'),(78,'邬上海','13912340078',25,'Shanghai','上海','某某街某某号'),(79,'安上海','13912340079',25,'Shanghai','上海','某某街某某号'),(80,'常上海','13912340080',25,'Shanghai','上海','某某街某某号'),(81,'乐上海','13912340081',25,'Shanghai','上海','某某街某某号'),(82,'于上海','13912340082',25,'Shanghai','上海','某某街某某号'),(83,'时上海','13912340083',25,'Shanghai','上海','某某街某某号'),(84,'傅上海','13912340084',25,'Shanghai','上海','某某街某某号'),(85,'皮上海','13912340085',25,'Shanghai','上海','某某街某某号'),(86,'卞上海','13912340086',25,'Shanghai','上海','某某街某某号'),(87,'齐上海','13912340087',25,'Shanghai','上海','某某街某某号'),(88,'康上海','13912340088',25,'Shanghai','上海','某某街某某号'),(89,'伍上海','13912340089',25,'Shanghai','上海','某某街某某号'),(90,'余上海','13912340090',25,'Shanghai','上海','某某街某某号');
/*!40000 ALTER TABLE `customer` ENABLE KEYS */;
UNLOCK TABLES;
...
```

#### 性能选项
- --extended-insert,-e  
  产生批量插入的SQL语句，可以使用--skip-extended-insert产生单次插入语句。  
- --insert-ignore  
  使用INSERT IGNORE INTO代替INSERT INTO语句。  
- --opt  
  该参数选项是默认开启的，是 --add-drop-table，--add-locks，--create-options，--disable-keys，--extended-insert，--lock-tables，--quick，--set-charset的集合。  
- --quick,-q  
  该参数适用于备份大表，没有该参数，则mysqldump在导出结果前将整个结果集装载到内存中，如果使用该参数，则mysqldump将直接导出结果集至文件中，不经过内存。  
- --skip-opt  
  跳过--opt参数。  

#### 事务选项
- --add-locks  
  每张表备份前后添加LOCK TABLES和UNLOCK TABLES语句。  
- --flush-logs,-F  
  备份前FLUSH数据库日志，这个参数要求RELOAD权限。  
  **注意**  
  如果--flush-logs与--all-databases一起使用，则日志将会在每备份一个库时FLUSH一次；  
  如果--flush-logs与--lock-all-tables/--master-data/--single-transaction一起使用，日志将仅仅被刷新一次。  
- --flush-privileges  
  备份mysql(指的是系统库)库完毕后，添加FLUSH PRIVILEGES语句。  
- --lock-all-tables,-x  
  在整个备份的过程中，将有一个全局的读锁锁住所有的数据库，该参数将会自动关闭--single-transaction和--lock-tables参数。  
- --lcok-tables,-l  
  备份每个库之前对该库的每张表添加读锁，对于事务型存储引擎而言，--single-trnsaction比--lock-tables更适合，因为--single-transaction根本不需要锁。  
- --no-autocommit  
  为每张表的INSERT语句添加SET autocommit = 0和COMMIT语句。  
```
--
-- Dumping data for table `t1`
--

LOCK TABLES `t1` WRITE;
/*!40000 ALTER TABLE `t1` DISABLE KEYS */;
set autocommit=0;
INSERT INTO `t1` VALUES (1,'fzqX83OfCJ3eLs6a',9),(2,'TeijyzUju26j1AmY',34),(3,'By4SYpW2MYzGtaIq',59),(4,'8DepDafmEtpARRgp',42),(5,'OUVJWkV5Z2J1mW1M',97),(6,'gzQyudMSB4v86uKZ',84),(7,'J1IdGyYhvkQQy0mO',90),(8,'WBSR3OW4QTwKz8HT',2),(9,'YG6pAUHdDOdkCcrt',19),(10,'3lEboOZkvhgW8C80',12);
/*!40000 ALTER TABLE `t1` ENABLE KEYS */;
UNLOCK TABLES;
commit;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
```
- --order-by-primary  
  备份每张表的数据按主键或者第一个唯一索引排序。  
- --single-transaction  
  该参数将事务隔离级别设置为REPEATABLE READ,并且在开始备份前发送START TRANSACTION语句至MySQL数据库中，这样可以保证备份的数据具有一致性。但是该参数仅适用于Innodb表，此时备份数据时，不阻塞任何英语程序。  
  **注意**  
1. 当使用--single-transaction参数备份时，为了保证一个有效的备份文件，禁止执行以下语句：  
   ALTER TABLE;  
   CREATE TABLE;  
   DROP TABLES;  
   RENAME TABLE;  
   TRUNCATE TABLE;  
2. --single-transaction和--lock-tables两个参数是互斥的，因为前者不阻塞应用，后者会阻塞应用。  
3. 当备份InnoDB类型的大表时，--single-transaction和--quick一起使用更佳。  


推荐mysqldump使用语句
--------------------
以下均只适用于事务型存储引擎，如InnoDB  
#### 常规备份
``` 
mysqldump --login-path=root3306 --single-transaction --master-data=2 --quick --no-autocommit --extended-insert  --complete-insert --events --routines --comments test  > test.sql
```

#### 压缩备份
```
mysqldump --login-path=root3306 --single-transaction --master-data=2 --quick --no-autocommit  --events --routines --compact test  > test.sql
```

#### 表结构与数据分开
```
mysqldump --login-path=root3306 --single-transaction  --quick --comments --tab=/data/mysql/mysqldata3306/tmpdir/ test
```

#### 备份slave数据库
```
mysqldump --login-path=root3306 --single-transaction --dump-slave=2 --include-master-host-port --quick --no-autocommit --compact --events --routines test  > test.sql
```


mysqldump限制
-------------
1. mysqldump默认不会备份INFORMATION_SCHEMA和performance_schema数据库，备份这些数据库时，需要在命令行指定这两个数据库。  
2. mysqldump不会备份MySQL集群的ndbnfo数据库。  
3. 在mysql5.6.6之前，mysqldump不会备份general_log和slow_log表。