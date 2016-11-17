## MySQL常用的备份工具


最近在面试过程中，总是遇到面试官问：平时工作中经常是如何备份的，使用哪些备份工具。于是还是默默的寻找资料总结一下吧。  



常用备份工具
-----------
1. mysql复制
2. 逻辑备份(mysqldump，mydumper)
3. 物理备份(copy,xtrabackup)


备份工具差异对比
--------------
1.mysql复制相对于其他的备份来说，得到的备份数据比较实时。  
2.逻辑备份：分表比较容易。  
mysqldump备份数据时是将所有sql语句整合在同一个文件中；  
mydumper备份数据时是将SQL语句按照表拆分成单个的sql文件，每个sql文件对应一个完整的表。  
3.物理备份：拷贝即可用，速度快。  
copy:直接拷贝文件到数据目录下，可能引起表损坏或者数据不一致。  
xtrabackup对于innodb表是不需要锁表的，对于myisam表仍然需要锁表。  


拓展
----
###### xtrabackup的原理
xtrabackup利用innodb的crash recovery的原理，备份innodb表时不需要锁表，xtrabackup一页一页复制data pages，并且xtrabackup还有另外一个线程监视redo log，一旦redo log发生变化，xtrabackup就会将log pages复制走（因为redo log是循环的，所以为了防止日志被覆盖，redo log一旦发生变化，就会立刻被xtrabackup复制走）【由于数据是复制出来的，所以备份出来的数据是不一致的，待恢复时，结合redo log利用crush recovery的原理恢复一致性的数据】。xtrabackup对于myisam表是需要锁表的。

###### mysqldump的原理
参考链接：  
http://tencentdba.com/blog/mysqldump-backup-principle/  
首先呢，对于mysqldump来说是需要锁表的，除了--single-transaction单独使用的时候是不需要锁表的。  
以下分析均以innodb存储引擎为例子。  
- mysqldump不添加任何参数
  在mysqldump准备备份某个表的数据之前，mysqldump会对该表添加read表级锁，直到该表数据备份完毕，才会释放锁。
```
# 备份sql
mysqldump -uroot -h127.0.0.1 -P3306 wing t1 > dump1.sql

# general log
160307  9:22:15     3 Connect   root@127.0.0.1 on 
                    3 Query     /*!40100 SET @@SQL_MODE='' */
                    3 Query     /*!40103 SET TIME_ZONE='+00:00' */
                    3 Query     SHOW VARIABLES LIKE 'gtid\_mode'
                    3 Query     SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='wing' AND TABLE_NAME IN ('t1'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE ORDER BY LOGFILE_GROUP_NAME
                    3 Query     SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='wing' AND TABLE_NAME IN ('t1')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
                    3 Query     SHOW VARIABLES LIKE 'ndbinfo\_version'
                    3 Init DB   wing
                    3 Query     SHOW TABLES LIKE 't1'
                    3 Query     LOCK TABLES `t1` READ /*!32311 LOCAL */
					# 表备份开始时，锁表
                    3 Query     show table status like 't1'
                    3 Query     SET SQL_QUOTE_SHOW_CREATE=1
                    3 Query     SET SESSION character_set_results = 'binary'
                    3 Query     show create table `t1`
                    3 Query     SET SESSION character_set_results = 'utf8'
                    3 Query     show fields from `t1`
                    3 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t1`
                    3 Query     SET SESSION character_set_results = 'binary'
                    3 Query     use `wing`
                    3 Query     select @@collation_database
                    3 Query     SHOW TRIGGERS LIKE 't1'
                    3 Query     SET SESSION character_set_results = 'utf8'
                    3 Query     UNLOCK TABLES
					# 表备份结束后u，释放表
                    3 Quit    
```
- 添加--single-trasaction参数的mysqldump
  在备份数据之前，mysqldump会做两件事情，首先将数据库的隔离级别设置为RR模式，然后发出start trasaction语句，开启事务（在RR级别下，此时利用innodb的mvcc得到一份数据快照，保证了数据的一致性），最后开始备份数据，期间无需锁表。  
```
# 备份sql
mysqldump -uroot -h127.0.0.1 -P3306 --single-transaction wing t1 > dump1.sql

# general log
160307  9:30:56     5 Connect   root@127.0.0.1 on 
                    5 Query     /*!40100 SET @@SQL_MODE='' */
                    5 Query     /*!40103 SET TIME_ZONE='+00:00' */
                    5 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
					# 设置RR隔离级别
                    5 Query     START TRANSACTION /*!40100 WITH CONSISTENT SNAPSHOT */
					# 开启一致性读事务
                    5 Query     SHOW VARIABLES LIKE 'gtid\_mode'
                    5 Query     UNLOCK TABLES
                    5 Query     SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='wing' AND TABLE_NAME IN ('t1'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE ORDER BY LOGFILE_GROUP_NAME
                    5 Query     SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='wing' AND TABLE_NAME IN ('t1')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
                    5 Query     SHOW VARIABLES LIKE 'ndbinfo\_version'
                    5 Init DB   wing
                    5 Query     SHOW TABLES LIKE 't1'
                    5 Query     SAVEPOINT sp
                    5 Query     show table status like 't1'
                    5 Query     SET SQL_QUOTE_SHOW_CREATE=1
                    5 Query     SET SESSION character_set_results = 'binary'
                    5 Query     show create table `t1`
                    5 Query     SET SESSION character_set_results = 'utf8'
                    5 Query     show fields from `t1`
                    5 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t1`
                    5 Query     SET SESSION character_set_results = 'binary'
                    5 Query     use `wing`
                    5 Query     select @@collation_database
                    5 Query     SHOW TRIGGERS LIKE 't1'
                    5 Query     SET SESSION character_set_results = 'utf8'
                    5 Query     ROLLBACK TO SAVEPOINT sp
                    5 Query     RELEASE SAVEPOINT sp
                    5 Quit
```
###### 正常使用mysqldump备份必携带--single-transaction,--master-data=2参数  
这两个参数的使用可以保证数据的一致性以及数据的不丢失。但是mysqldump开始的时候会有短暂的锁。  
```
160307 10:30:58     3 Connect   root@127.0.0.1 on 
                    3 Query     /*!40100 SET @@SQL_MODE='' */
                    3 Query     /*!40103 SET TIME_ZONE='+00:00' */
                    3 Query     FLUSH /*!40101 LOCAL */ TABLES
                    3 Query     FLUSH TABLES WITH READ LOCK
					# flush tables
                    3 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
					# 开启RR隔离事务级别
                    3 Query     START TRANSACTION /*!40100 WITH CONSISTENT SNAPSHOT */
					# 开启一致性读事务
                    3 Query     SHOW VARIABLES LIKE 'gtid\_mode'
                    3 Query     SHOW MASTER STATUS
                    3 Query     UNLOCK TABLES
                    3 Query     SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='wing' AND TABLE_NAME IN ('t1'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE ORDER BY LOGFILE_GROUP_NAME
                    3 Query     SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='wing' AND TABLE_NAME IN ('t1')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
                    3 Query     SHOW VARIABLES LIKE 'ndbinfo\_version'
                    3 Init DB   wing
                    3 Query     SHOW TABLES LIKE 't1'
                    3 Query     SAVEPOINT sp
                    3 Query     show table status like 't1'
                    3 Query     SET SQL_QUOTE_SHOW_CREATE=1
                    3 Query     SET SESSION character_set_results = 'binary'
                    3 Query     show create table `t1`
                    3 Query     SET SESSION character_set_results = 'utf8'
                    3 Query     show fields from `t1`
                    3 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t1`
                    3 Query     SET SESSION character_set_results = 'binary'
                    3 Query     use `wing`
                    3 Query     select @@collation_database
                    3 Query     SHOW TRIGGERS LIKE 't1'
                    3 Query     SET SESSION character_set_results = 'utf8'
                    3 Query     ROLLBACK TO SAVEPOINT sp
                    3 Query     RELEASE SAVEPOINT sp
                    3 Quit 
```


Tips
-----
在早上4点备份开始备份，5点备份完毕，那么此时mysqldump --single-transaction --master-data=2到底备份的是几点数据呢？

答案是4点的。对于innodb，使用RR级别以及一致性快照读，获取的是4点的数据快照。对于myisam，使用锁表的方式，等myisam备份完毕后，才会释放锁，得到的仍旧是4点的快照数据。
```sql
# wing数据库下存在的表以及表结构
# 注意每张表不同的存储引擎哦
[wing]root@127.0.0.1 : wing 04:06:52> show tables;
+----------------+
| Tables_in_wing |
+----------------+
| t1             |
| t2             |
| t3             |
| t4             |
+----------------+
4 rows in set (0.00 sec)

[wing]root@127.0.0.1 : wing 04:06:54> show create table t1;
+-------+--------------------------------------------------------------------------------------+
| Table | Create Table                                                                         |
+-------+--------------------------------------------------------------------------------------+
| t1    | CREATE TABLE `t1` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

[wing]root@127.0.0.1 : wing 04:07:03> show create table t2;
+-------+--------------------------------------------------------------------------------------+
| Table | Create Table                                                                         |
+-------+--------------------------------------------------------------------------------------+
| t2    | CREATE TABLE `t2` (
  `id` int(11) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

[wing]root@127.0.0.1 : wing 04:07:06> show create table t3;
+-------+--------------------------------------------------------------------------------------+
| Table | Create Table                                                                         |
+-------+--------------------------------------------------------------------------------------+
| t3    | CREATE TABLE `t3` (
  `id` int(11) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

[wing]root@127.0.0.1 : wing 04:07:11> show create table t4;
+-------+--------------------------------------------------------------------------------------+
| Table | Create Table                                                                         |
+-------+--------------------------------------------------------------------------------------+
| t4    | CREATE TABLE `t4` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------+
1 row in set (0.00 sec)


# general log
/usr/sbin/mysqld, Version: 5.6.28-log (MySQL Community Server (GPL)). started with:
Tcp port: 3307  Unix socket: /data/mysql/mysqldata3307/sock/mysql.sock
Time                 Id Command    Argument
160528 16:02:29    18 Query     show variables like 'general_log'
160528 16:02:53    19 Connect   root@127.0.0.1 on 
                   19 Query     /*!40100 SET @@SQL_MODE='' */
                   19 Query     /*!40103 SET TIME_ZONE='+00:00' */
                   19 Query     FLUSH /*!40101 LOCAL */ TABLES
                   19 Query     FLUSH TABLES WITH READ LOCK
					# flush tables
                   19 Query     SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ
					# 设置RR事务隔离级别
                   19 Query     START TRANSACTION /*!40100 WITH CONSISTENT SNAPSHOT */
					# 开启一致性读事务
                   19 Query     SHOW VARIABLES LIKE 'gtid\_mode'
                   19 Query     SHOW MASTER STATUS
                   19 Query     UNLOCK TABLES
                   19 Query     SELECT LOGFILE_GROUP_NAME, FILE_NAME, TOTAL_EXTENTS, INITIAL_SIZE, ENGINE, EXTRA FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'UNDO LOG' AND FILE_NAME IS NOT NULL AND LOGFILE_GROUP_NAME IN (SELECT DISTINCT LOGFILE_GROUP_NAME FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('wing'))) GROUP BY LOGFILE_GROUP_NAME, FILE_NAME, ENGINE ORDER BY LOGFILE_GROUP_NAME
                   19 Query     SELECT DISTINCT TABLESPACE_NAME, FILE_NAME, LOGFILE_GROUP_NAME, EXTENT_SIZE, INITIAL_SIZE, ENGINE FROM INFORMATION_SCHEMA.FILES WHERE FILE_TYPE = 'DATAFILE' AND TABLESPACE_NAME IN (SELECT DISTINCT TABLESPACE_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA IN ('wing')) ORDER BY TABLESPACE_NAME, LOGFILE_GROUP_NAME
                   19 Query     SHOW VARIABLES LIKE 'ndbinfo\_version'
                   19 Init DB   wing
                   19 Query     SHOW CREATE DATABASE IF NOT EXISTS `wing`
                   19 Query     SAVEPOINT sp
                   19 Query     show tables
                   19 Query     show table status like 't1'
					# 开始备份t1表（innodb）
                   19 Query     SET SQL_QUOTE_SHOW_CREATE=1
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     show create table `t1`
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     show fields from `t1`
                   19 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t1`
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     use `wing`
                   19 Query     select @@collation_database
                   19 Query     SHOW TRIGGERS LIKE 't1'
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     ROLLBACK TO SAVEPOINT sp
                   19 Query     show table status like 't2'
					# 开始备份t2表（myisam）
                   19 Query     SET SQL_QUOTE_SHOW_CREATE=1
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     show create table `t2`
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     show fields from `t2`
                   19 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t2`
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     use `wing`
                   19 Query     select @@collation_database
                   19 Query     SHOW TRIGGERS LIKE 't2'
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     ROLLBACK TO SAVEPOINT sp
                   19 Query     show table status like 't3'
					# 开始备份t3表(myisam)
                   19 Query     SET SQL_QUOTE_SHOW_CREATE=1
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     show create table `t3`
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     show fields from `t3`
                   19 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t3`
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     use `wing`
                   19 Query     select @@collation_database
                   19 Query     SHOW TRIGGERS LIKE 't3'
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     ROLLBACK TO SAVEPOINT sp
                   19 Query     show table status like 't4'
					# 开始备份t4表（innodb）
                   19 Query     SET SQL_QUOTE_SHOW_CREATE=1
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     show create table `t4`
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     show fields from `t4`
                   19 Query     SELECT /*!40001 SQL_NO_CACHE */ * FROM `t4`
                   19 Query     SET SESSION character_set_results = 'binary'
                   19 Query     use `wing`
                   19 Query     select @@collation_database
                   19 Query     SHOW TRIGGERS LIKE 't4'
                   19 Query     SET SESSION character_set_results = 'utf8'
                   19 Query     ROLLBACK TO SAVEPOINT sp
                   19 Query     RELEASE SAVEPOINT sp
                   19 Quit      
```