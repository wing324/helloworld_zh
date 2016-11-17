## oak-online-alter-table工具

oak-online-alter-table作为一款实现非阻塞的在线ALTER TABLE操作的工具，与online DDL的区别在于其最后不需要应用online log，故最后一部分不需要进行锁表。  



使用限制
--------
1. 表至少含有一个单列的唯一索引;  
2. 修改的表和源表之间共享单列的唯一索引;  
3. 表上没有‘AFTER’触发器(因为该工具会在执行过程中创建AFTER触发器);  
4. 表没有外键;  
5. 表名不超过57个字符。  

程序运行时,源表上只能有INSERT/UPDATE/DELETE/REPLACE操作,不允许源表上有TRUNCATE/ALTER/REPAIR/OPTIMIZER等操作,且运行的表必须有表级别锁。  


基本功能
--------
1. 非阻塞的ALTER TABLE操作  
   存在限制如下：  
   ADD COLUMN：新列必须有一个默认值;  
   DROP COLUMN：源表和修改的表必须有共享的单列唯一索引;  
   MODIFY COLUMN：无;  
   ADD KEY：无;
   DROP KEY：源表和修改的表必须有共享的单列唯一索引;  
   CHANGE ENGINE：无。

2. 空的ALTER操作,将会重建表  
   重组数据并释放无用的磁盘空间。  

3. 创建一个源表的镜像表,可以在数据库中看到。  
   限制：  
   ALTER TABLE不操作在源表上;  
   源表上不能有TRUNCATE操作;  
   源表上没有load data infile操作;  
   源表上没有optimize操作;  


原理
----
通过在oak-online-alter-table工具运行期间创建源表的镜像表,而后与源表进行缓慢的同步,当同步完成后,该镜像表就取代了源表。  
镜像表取代源表的过程中,存在一个将源表数据复制到镜像表的过程,该过程中需要将源表的数据放进chunk中,将源表数据读入到chunck中,对于MyISAM表需要表级读锁,对于InnoDB表需要对读取到chunk的数据上加上行级读锁,所以此处需要锁。chunk的大小可以有参数-c设置;chunk较小时读锁持有时间较小且允许更多的并发,但数据复制过程可能需要更久;chunk较大时,读锁持有时间较长且允许并发更少,但数据复制过程时间短。  


工具选项
--------
-h,--help				显示help页面  
-u,--user				MySQL连接用户  
-H,--host				MySQL连接主机(默认localhost)  
--ask-pass				MySQL连接密码  
-P,--port				MySQL连接端口,默认为3306端口  
-S,--socket				MySQL连接socket文件  
--defaults-file			指定MySQL配置文件  
-d,--database			指定需要做online-alter-table对应的数据库  
-t,--table 				指定需要做online-alter-table对应的表  
-g,--ghost				显示创建一个镜像表,与源表保持同步  
-a,--alter 				alter-table的语句,该工具自身会执行alter table *table_name*,故alter-table语句只需alter table *table_name*后的语句  
-l,--lock-chunks 		每个chunk被复制之前使用lock tables语句  
-N,--skip-binlog 		不记录到binlog中  
-r,--max-lock-retries	当出现死锁或者超过lock_wait_timeout时,重试的次数,默认为10,0代表无穷大  
--skip-delete-pass		不执行DELETE语句  
--sleep 				两个chunk复制数据之间的间隔时间,单位为：微秒,默认为0  
--sleep-ratio 			睡眠时间与执行时间的比例,睡眠时间将与每部分的执行时间成比例,而不是一成不变的--sleep时间,默认为0  
--cleanup 				删除自定义的触发器,如果程序意外终止,但没有该选项,那么没有删除的自定义触发器并不会得到删除。  
-v,--verbose 			输出更详细的信息  
-q,--quiet 				输出简要信息  


实战演练
--------
测试添加普通索引以及删除索引的过程。

```sql
# 源表表结构
root@localhost : wing 11:19:23> show create table oak;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                           |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| oak   | CREATE TABLE `oak` (
  `id` int(11) NOT NULL DEFAULT '0',
  `name` varchar(16) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# oak-online-alter-table添加普通索引执行语句
[root@oracle01 ~]# oak-online-alter-table  --user=root --socket=/data/mysqldata3306/sock/mysql.sock  --database=wing --table=oak --alter='add key idx_test(name)';

# oak-online-alter-table添加普通索引输出信息
-- Connecting to MySQL
-- Table wing.oak is of engine innodb
-- Checking for UNIQUE columns on wing.oak, by which to chunk
-- Possible UNIQUE KEY column names in wing.oak:
-- - id
-- Table wing.__oak_oak has been created
-- Table wing.__oak_oak has been altered
-- Checking for UNIQUE columns on wing.__oak_oak, by which to chunk
-- Possible UNIQUE KEY column names in wing.__oak_oak:
-- - id
-- Checking for UNIQUE columns on wing.oak, by which to chunk
-- - Found following possible unique keys:
-- - id (int)
-- Chosen unique key is 'id'
-- Shared columns: id, name
-- Created AD trigger
-- Created AU trigger
-- Created AI trigger
-- Attempting to lock tables

-- Tables locked WRITE
-- id (min, max) values: ([1L], [3L])
-- Tables unlocked
-- - Reminder: altering wing.oak: add key idx_test(name)...
-- Copying range (1), (3), progress: 0%
-- Copying range 100% complete. Number of rows: 3
-- - Reminder: altering wing.oak: add key idx_test(name)...
-- Deleting range (1), (3), progress: 0%
-- Deleting range 100% complete. Number of rows: 0
-- Table wing.oak has been renamed to wing.__arc_oak,
-- and table wing.__oak_oak has been renamed to wing.oak
-- Table wing.__arc_oak was found and dropped
-- ALTER TABLE completed

# oak-online-alter-table添加普通索引成功
root@localhost : wing 11:26:22> show create table oak;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                      |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| oak   | CREATE TABLE `oak` (
  `id` int(11) NOT NULL DEFAULT '0',
  `name` varchar(16) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_test` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# oak-online-alter-table删除普通索引执行语句
[root@oracle01 ~]# oak-online-alter-table  --user=root --socket=/data/mysqldata3306/sock/mysql.sock  --database=wing --table=oak --alter='drop key idx_test'

# oak-online-alter-table删除普通索引输出信息
-- Connecting to MySQL
-- Table wing.oak is of engine innodb
-- Checking for UNIQUE columns on wing.oak, by which to chunk
-- Possible UNIQUE KEY column names in wing.oak:
-- - id
-- Table wing.__oak_oak has been created
-- Table wing.__oak_oak has been altered
-- Checking for UNIQUE columns on wing.__oak_oak, by which to chunk
-- Possible UNIQUE KEY column names in wing.__oak_oak:
-- - id
-- Checking for UNIQUE columns on wing.oak, by which to chunk
-- - Found following possible unique keys:
-- - id (int)
-- Chosen unique key is 'id'
-- Shared columns: id, name
-- Created AD trigger
-- Created AU trigger
-- Created AI trigger
-- Attempting to lock tables

-- Tables locked WRITE
-- id (min, max) values: ([1L], [3L])
-- Tables unlocked
-- - Reminder: altering wing.oak: drop key idx_test...
-- Copying range (1), (3), progress: 0%
-- Copying range 100% complete. Number of rows: 3
-- - Reminder: altering wing.oak: drop key idx_test...
-- Deleting range (1), (3), progress: 0%
-- Deleting range 100% complete. Number of rows: 0
-- Table wing.oak has been renamed to wing.__arc_oak,
-- and table wing.__oak_oak has been renamed to wing.oak
-- Table wing.__arc_oak was found and dropped
-- ALTER TABLE completed

# oak-online-alter-table删除普通索引成功
root@localhost : wing 11:28:48> show create table oak;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                           |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| oak   | CREATE TABLE `oak` (
  `id` int(11) NOT NULL DEFAULT '0',
  `name` varchar(16) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

**特别说明**  
oak-online-alter-table工具与online DDL的区别：  
online DDL在操作执行完毕后,需要一次锁表将online log应用到当前表中,而oak-online-alter-table则在操作执行完毕即可,无需再次锁表。

