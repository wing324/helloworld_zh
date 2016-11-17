## MySQL之sql_mode

曾经我一直忽视了sql_mode的作用，直到有一天认识了ONLE_FULL_GROUP_BY模式，我才重新认识了sql_mode,才发现sql_mode还是有很多不容小视的设置。  


说明
----
**MySQL版本**  
```
root@localhost : wing 06:18:33> select version();
+------------+
| version()  |
+------------+
| 5.6.26-log |
+------------+
1 row in set (0.00 sec)
```


SQL Modes
---------
##### 常用的sql_mode
- ANSI     
- STRICT_TRANS_TABLES  
  对于事务型表，insert非法值之后，回滚整条语句；对于非事务型表，如果是单值语句或者是第一条记录就是非法的多值插入将回滚整条语句，对于第一条记录不是非法的多值插入，将会给非法值替换为可能值。  
```
# 事务型表结构
root@localhost : wing 06:24:46> show create table innodb_test;
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table       | Create Table                                                                                                                                           |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| innodb_test | CREATE TABLE `innodb_test` (
  `id` int(10) unsigned NOT NULL,
  `name` varchar(8) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 非事务型表结构
root@localhost : wing 06:27:28> show create table myisam_test;  
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table       | Create Table                                                                                                                                           |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| myisam_test | CREATE TABLE `myisam_test` (
  `id` int(10) unsigned NOT NULL,
  `name` varchar(8) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 |
+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : wing 06:42:03> show variables like 'sql_mode';
+---------------+---------------------+
| Variable_name | Value               |
+---------------+---------------------+
| sql_mode      | STRICT_TRANS_TABLES |
+---------------+---------------------+
1 row in set (0.00 sec)

# 单值插入非法语句
root@localhost : wing 06:23:56> insert into innodb_test values(-9,'ABCDEFGHIJKLMNOPQ');
ERROR 1264 (22003): Out of range value for column 'id' at row 1

root@localhost : wing 06:24:22> insert into myisam_test values(-9,'ABCDEFGHIJKLMNOPQ');
ERROR 1264 (22003): Out of range value for column 'id' at row 1

root@localhost : wing 06:24:30> select * from innodb_test;
Empty set (0.00 sec)

root@localhost : wing 06:24:36> select * from myisam_test;
Empty set (0.00 sec)


# 第一条记录非法的多值插入
root@localhost : wing 06:28:03> insert into innodb_test values(-1,'A'),(2,'B'),(3,'C');
ERROR 1264 (22003): Out of range value for column 'id' at row 1

root@localhost : wing 06:35:08> insert into myisam_test values(-1,'A'),(2,'B'),(3,'C');
ERROR 1264 (22003): Out of range value for column 'id' at row 1

root@localhost : wing 06:35:14> select * from innodb_test;
Empty set (0.00 sec)

root@localhost : wing 06:35:22> select * from myisam_test;      
Empty set (0.00 sec)

# 第一条记录不是非法的多值插入
root@localhost : wing 06:35:26> insert into innodb_test values(1,'A'),(2,'B'),(-3,'C'),(4,'D');
ERROR 1264 (22003): Out of range value for column 'id' at row 3
root@localhost : wing 06:36:28> insert into myisam_test values(1,'A'),(2,'B'),(-3,'C'),(4,'D');
Query OK, 4 rows affected, 1 warning (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 1

Warning (Code 1264): Out of range value for column 'id' at row 3
root@localhost : wing 06:36:35> select * from innodb_test;
Empty set (0.00 sec)

root@localhost : wing 06:36:42> select * from myisam_test;      
+----+------+
| id | name |
+----+------+
|  1 | A    |
|  2 | B    |
|  0 | C    |
|  4 | D    |
+----+------+
4 rows in set (0.00 sec)

```
-  TRADITIONAL  

##### sql_mode全员
- ALLOW_INVALID_DATES  
  对于日期只检查月份在1到12之间，日期在1到31之间。该sql_mode只适用于DATE和DATETIME类型，对于TIMESTAMP类型还是需要有效的日期。  
```
# 只开启严格模式（不能插入非法日期）
root@localhost : wing 06:56:03> set sql_mode='STRICT_TRANS_TABLES'; 
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:56:08> truncate table t;
Query OK, 0 rows affected (2.64 sec)

root@localhost : wing 06:56:14> insert into t(d,dt) values('2015-4-31','2015-4-31 00:00:00');
ERROR 1292 (22007): Incorrect date value: '2015-4-31' for column 'd' at row 1
Error (Code 1292): Incorrect date value: '2015-4-31' for column 'd' at row 1
Error (Code 1292): Incorrect datetime value: '2015-4-31 00:00:00' for column 'dt' at row 1
root@localhost : wing 06:56:18> select * from t;
Empty set (0.00 sec)

# 不开启严格模式（非法日期转换）
root@localhost : wing 06:56:23> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:56:31> insert into t(d,dt) values('2015-4-31','2015-4-31 00:00:00');
Query OK, 1 row affected, 2 warnings (0.01 sec)

Warning (Code 1264): Out of range value for column 'd' at row 1
Warning (Code 1264): Out of range value for column 'dt' at row 1
root@localhost : wing 06:56:33> select * from t;
+------------+---------------------+---------------------+
| d          | dt                  | ts                  |
+------------+---------------------+---------------------+
| 0000-00-00 | 0000-00-00 00:00:00 | 2015-11-06 06:56:33 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)

# 只开启ALLOW_INVALID_DATES（允许插入非法日期）
root@localhost : wing 06:56:38> set sql_mode='ALLOW_INVALID_DATES';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:58:09> truncate table t;
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:58:15> insert into t(d,dt) values('2015-4-31','2015-4-31 00:00:00');
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 06:58:17> select * from t;
+------------+---------------------+---------------------+
| d          | dt                  | ts                  |
+------------+---------------------+---------------------+
| 2015-04-31 | 2015-04-31 00:00:00 | 2015-11-06 06:58:17 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)

# 即开启严格模式，又开启ALLOW_INVALID_DATES（允许插入非法日期）
root@localhost : wing 06:58:21> set sql_mode='STRICT_TRANS_TABLES,ALLOW_INVALID_DATES';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:59:10> truncate table t;
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:59:14> insert into t(d,dt) values('2015-4-31','2015-4-31 00:00:00');
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 06:59:16> select * from t;
+------------+---------------------+---------------------+
| d          | dt                  | ts                  |
+------------+---------------------+---------------------+
| 2015-04-31 | 2015-04-31 00:00:00 | 2015-11-06 06:59:16 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)
```
- ANSI_QUOTES  
  未开启ANSI_QUOTES模式时，``和""是两种不同的标识符，""与''作用相同；开启ANSI_QUOTES模式时，``和""是作用相同的标识符，此时""与''作用不同。  
```
# 未启用ANSI_QUOTES模式
root@localhost : wing 12:54:34> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 12:56:54> select * from t where ts='2015-11-06 06:59:16';
+------------+---------------------+---------------------+
| d          | dt                  | ts                  |
+------------+---------------------+---------------------+
| 2015-04-31 | 2015-04-31 00:00:00 | 2015-11-06 06:59:16 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)

root@localhost : wing 12:57:23> select * from t where ts=`2015-11-06 06:59:16`; 
ERROR 1054 (42S22): Unknown column '2015-11-06 06:59:16' in 'where clause'
root@localhost : wing 12:57:34> select * from t where ts="2015-11-06 06:59:16";
+------------+---------------------+---------------------+
| d          | dt                  | ts                  |
+------------+---------------------+---------------------+
| 2015-04-31 | 2015-04-31 00:00:00 | 2015-11-06 06:59:16 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)

# 启用ANSI_QUOTES模式
root@localhost : wing 12:57:41> set sql_mode='ANSI_QUOTES';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 12:57:47> select * from t where ts='2015-11-06 06:59:16';
+------------+---------------------+---------------------+
| d          | dt                  | ts                  |
+------------+---------------------+---------------------+
| 2015-04-31 | 2015-04-31 00:00:00 | 2015-11-06 06:59:16 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)

root@localhost : wing 12:57:57> select * from t where ts=`2015-11-06 06:59:16`;
ERROR 1054 (42S22): Unknown column '2015-11-06 06:59:16' in 'where clause'
root@localhost : wing 12:58:01> select * from t where ts="2015-11-06 06:59:16";
ERROR 1054 (42S22): Unknown column '2015-11-06 06:59:16' in 'where clause'
```
- ERROR_FOR_DIVISION_BY_ZERO  
  声明：该参数将会被弃用，对于这类模式均生成一个warning。  
  未启用ERROR_FOR_DIVISION_BY_ZERO模式，对于使用被除数为0的值，会用NULL代替，但不会报错warning；启用ERROR_FOR_DIVISION_BY_ZERO模式将会在INSERT/UPDATE时，对于使用被除数为0的值，将会报出warning,并用NULL代替；启用ERROR_FOR_DIVISION_BY_ZERO+STRICT_TRANS_TABLES模式，在INSERT/UPDATE时，对于使用被除数为0的值，将会报错error,并不会INSERT或UPDATE任何值。  

```
# 未启用ERROR_FOR_DIVISION_BY_ZERO模式
root@localhost : wing 01:09:37> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 01:09:56> insert into sql_mode_test values(mod(5,0));
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 01:10:22> select * from sql_mode_test;
+------+
| num  |
+------+
| NULL |
+------+
1 row in set (0.00 sec)

# 启用ERROR_FOR_DIVISION_BY_ZERO模式
root@localhost : wing 01:10:32> set sql_mode='ERROR_FOR_DIVISION_BY_ZERO';
Query OK, 0 rows affected, 1 warning (0.00 sec)

Warning (Code 1681): 'ERROR_FOR_DIVISION_BY_ZERO' is deprecated and will be removed in a future release.
root@localhost : wing 01:10:43> insert into sql_mode_test values(mod(5,0));
Query OK, 1 row affected, 1 warning (0.00 sec)

Warning (Code 1365): Division by 0
root@localhost : wing 01:10:49> select * from sql_mode_test;
+------+
| num  |
+------+
| NULL |
| NULL |
+------+
2 rows in set (0.00 sec)

# 启用ERROR_FOR_DIVISION_BY_ZERO+STRICT_TRANS_TABLES模式
root@localhost : wing 01:10:54> set sql_mode='ERROR_FOR_DIVISION_BY_ZERO,STRICT_TRANS_TABLES';
Query OK, 0 rows affected, 1 warning (0.00 sec)

Warning (Code 1681): 'ERROR_FOR_DIVISION_BY_ZERO' is deprecated and will be removed in a future release.
root@localhost : wing 01:11:04> insert into sql_mode_test values(mod(5,0));
ERROR 1365 (22012): Division by 0
root@localhost : wing 01:11:12> select * from sql_mode_test;
+------+
| num  |
+------+
| NULL |
| NULL |
+------+
2 rows in set (0.00 sec)
```
- HIGH_NOT_PRECEDENCE  
  未开启HIGH_NOT_PRECEDENCE模式，NOT为最低优先级，例如对于NOT a BETWEEN b AND c的表达式可表示为：NOT （a BETWEEN b AND c）;开启HIGH_NOT_PRECEDENCE模式，NOT为最高优先级，例如对于NOT a BETWEEN b AND c的表达式可表示为：(NOT a) BETWEEN b AND c;
```
# 未开启HIGH_NOT_PRECEDENCE模式
root@localhost : wing 01:30:40> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 01:30:56> SELECT NOT 1 BETWEEN -5 AND 5;
+------------------------+
| NOT 1 BETWEEN -5 AND 5 |
+------------------------+
|                      0 |
+------------------------+
1 row in set (0.00 sec)

#　开启HIGH_NOT_PRECEDENCE模式
root@localhost : wing 01:30:58> set sql_mode='HIGH_NOT_PRECEDENCE';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 01:31:09> SELECT NOT 1 BETWEEN -5 AND 5;
+------------------------+
| NOT 1 BETWEEN -5 AND 5 |
+------------------------+
|                      1 |
+------------------------+
1 row in set (0.00 sec)
```
- IGNORE_SPACE  
  MySQL默认是不忽略空格，'count'和'count '是不相同的，开启IGNORE_SPACE模式后，'count'和'count '是相同的。  
```
# 未启用IGNORE_SPACE模式
root@localhost : wing 02:03:57> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:04:14> CREATE TABLE count(id int);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'count(id int)' at line 1
root@localhost : wing 02:04:17> CREATE TABLE count (id int);
Query OK, 0 rows affected (0.01 sec)

root@localhost : wing 02:04:21> drop table count;
Query OK, 0 rows affected (0.10 sec)

# 启用IGNORE_SPACE模式
root@localhost : wing 02:04:38> set sql_mode='IGNORE_SPACE';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:04:54> CREATE TABLE count(id int);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'count(id int)' at line 1
root@localhost : wing 02:04:59> CREATE TABLE count (id int);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'count (id int)' at line 1
```
- NO_AUTO_CREATE_USER  
  未开启NO_AUTO_CREATE_USER模式，对于没有使用IDENTIFIED BY/IDENTIFIED WITH的GRANT语句赋予权限时，若用户不存在，则MySQL会自动创建该用户；开启NO_AUTO_CREATE_USER模式，对于没有使用IDENTIFIED BY/IDENTIFIED WITH的GRANT语句赋予权限时，若用户不存在，则会报出Error错误。  
```
# 未启用NO_AUTO_CREATE_USER模式
root@localhost : wing 02:34:14> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:34:18> show grants for 'wing'@'localhost';
ERROR 1141 (42000): There is no such grant defined for user 'wing' on host 'localhost'
root@localhost : wing 02:34:19> grant select on *.* to 'wing_test'@'localhost';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:34:31> show grants for 'wing'@'localhost';                 
ERROR 1141 (42000): There is no such grant defined for user 'wing' on host 'localhost'
root@localhost : wing 02:34:32> show grants for 'wing_test'@'localhost';
+------------------------------------------------+
| Grants for wing_test@localhost                 |
+------------------------------------------------+
| GRANT SELECT ON *.* TO 'wing_test'@'localhost' |
+------------------------------------------------+
1 row in set (0.00 sec)

# 启用NO_AUTO_CREATE_USER模式
root@localhost : wing 02:34:53> set sql_mode='NO_AUTO_CREATE_USER';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:35:18> show grants for 'wing'@'localhost';
ERROR 1141 (42000): There is no such grant defined for user 'wing' on host 'localhost'
root@localhost : wing 02:35:55> grant select on *.* to 'wing'@'localhost';     
ERROR 1133 (42000): Can't find any matching row in the user table
root@localhost : wing 02:36:06> grant select on *.* to 'wing'@'localhost' identified by '123456';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:36:25> show grants for 'wing'@'localhost';            
+--------------------------------------------------------------------------------------------------------------+
| Grants for wing@localhost                                                                                    |
+--------------------------------------------------------------------------------------------------------------+
| GRANT SELECT ON *.* TO 'wing'@'localhost' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
+--------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
- NO_AUTO_VALUE_ON_ZERO  
  该参数对auto_increment列有影响，未启用NO_AUTO_VALUE_ON_ZERO时，当auto_increment列遇到0时，会自动插入当前自增列的最大值代替，启用NO_AUTO_VALUE_ON_ZERO时，当auto_increment列遇到0时，会在auto_increment列插入0。
```
# 未启用NO_AUTO_VALUE_ON_ZERO模式
root@localhost : wing 06:27:42> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:27:55> show create table sql_mode_test;
+---------------+----------------------------------------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                                                     |
+---------------+----------------------------------------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+----------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : wing 06:28:04> select * from sql_mode_test;
Empty set (0.00 sec)

root@localhost : wing 06:28:12> insert into sql_mode_test values(0);
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 06:28:24> select * from sql_mode_test;
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.00 sec)

# 已启用NO_AUTO_VALUE_ON_ZERO模式
root@localhost : wing 06:29:00> set sql_mode='NO_AUTO_VALUE_ON_ZERO';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:29:16> show create table sql_mode_test;
+---------------+----------------------------------------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                                                     |
+---------------+----------------------------------------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+----------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : wing 06:29:32> select * from sql_moed_test;
ERROR 1146 (42S02): Table 'wing.sql_moed_test' doesn't exist
root@localhost : wing 06:29:39> select * from sql_mode_test;  
Empty set (0.00 sec)

root@localhost : wing 06:29:43> insert into sql_mode_test values(0);
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 06:29:55> select * from sql_mode_test;
+----+
| id |
+----+
|  0 |
+----+
1 row in set (0.00 sec)
# 此时再插入0时，会造成主键重复
root@localhost : wing 06:34:32> insert into sql_mode_test values(0);
ERROR 1062 (23000): Duplicate entry '0' for key 'PRIMARY'
```
- NO_BACKSLASH_ESCAPES  
  不适用该参数'\'反斜杠符号表示转义符，启用该参数'\'反斜杠符号仅代表它自己
```
# 未启用NO_BACKSLASH_ESCAPES模式
root@localhost : wing 06:36:42> create table sql_mode_test(vc varchar(16));
Query OK, 0 rows affected (1.84 sec)

root@localhost : wing 06:34:35> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:37:01> insert into sql_mode_test values('i\'m');
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 06:37:36> select * from sql_mode_test;
+------+
| vc   |
+------+
| i'm  |
+------+
1 row in set (0.00 sec)

# 启用NO_BACKSLASH_ESCAPES模式
root@localhost : wing 06:37:42> set sql_mode='NO_BACKSLASH_ESCAPES';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 06:37:59> insert into sql_mode_test values('i\'m');
    '> '
    -> \c
root@localhost : wing 06:38:05> insert into sql_mode_test values("i\'m"); 
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 06:38:15> select * from sql_mode_test;
+------+
| vc   |
+------+
| i'm  |
| i\'m |
+------+
2 rows in set (0.00 sec)
```
-  NO_DIR_IN_CREATE
   创建表时，忽视所有INDEX DIRECTORY和DATA DIRECTORY命令，该选项对slave较有用。

-  NO_ENGINE_SUBSTITUTION  
   对于使用不支持的ENGINE(存储引擎)类型时，未启用NO_ENGINE_SUBSTITUTION模式时，CREATE TABLE或ALTER TABLE会成功，但使用的存储类型为默认类型（当前版本InnoDB）,报出warning,启用NO_ENGINE_SUBSTITUTION模式时，CREATE TABLE或ALTER TABLE时会报出error执行终止。  
```
# 未启用NO_ENGINE_SUBSTITUTION模式
root@localhost : wing 11:08:39> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| BLACKHOLE          | NO      | /dev/null storage engine (anything you write to it disappears) | NULL         | NULL | NULL       |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)

root@localhost : wing 11:12:17> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 11:12:35> create table sql_mode_test(id int) engine=BLACKHOLE ;
Query OK, 0 rows affected, 2 warnings (0.09 sec)

Warning (Code 1286): Unknown storage engine 'BLACKHOLE'
Warning (Code 1266): Using storage engine InnoDB for table 'sql_mode_test'
root@localhost : wing 11:12:36> show create table sql_mode_test;
+---------------+-------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                    |
+---------------+-------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `id` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+-------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

#　启用NO_ENGINE_SUBSTITUTION模式
root@localhost : wing 11:12:45> set sql_mode='NO_ENGINE_SUBSTITUTION';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 11:13:10> drop table sql_mode_test;
Query OK, 0 rows affected (0.10 sec)

root@localhost : wing 11:13:17> create table sql_mode_test(id int) engine=BLACKHOLE ;
ERROR 1286 (42000): Unknown storage engine 'BLACKHOLE'
root@localhost : wing 11:13:25> show create table sql_mode_test;
ERROR 1146 (42S02): Table 'wing.sql_mode_test' doesn't exist
```
- NO_FIELD_OPTIONS、NO_KEY_OPTIONS、NO_TABLE_OPTIONS  
  用于mysqldump的兼容性，实际操作未实现。  
- NO_UNSIGNED_SUBTRACTION  
  未启用NO_UNSIGNED_SUBTRACTION模式，无符号类型的数值减法后还是无符号类型，对于负数将直接报错，启用NO_UNSIGNED_SUBTRACTION模式，无符号类型的数值减法可以是负数，而不会报错。  
```
# 未启用NO_UNSIGNED_SUBTRACTION模式
root@localhost : wing 12:29:46> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 12:30:26> select cast(3 as unsigned) - 4;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(cast(3 as unsigned) - 4)'

# 启用NO_UNSIGNED_SUBTRACTION模式
root@localhost : wing 12:30:29> set sql_mode='NO_UNSIGNED_SUBTRACTION';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 12:30:44> select cast(3 as unsigned) - 4;
+-------------------------+
| cast(3 as unsigned) - 4 |
+-------------------------+
|                      -1 |
+-------------------------+
1 row in set (0.00 sec)
```
- NO_ZERO_DATE  
  声明：该参数将会被弃用，对于这类模式均生成一个warning。  
  对于插入'0000-00-00 00:00:00'这种类型包含0月份或者0日期的日期，未启用NO_ZERO_DATE模式时，可以插入并且无任何warning,启用NO_ZERO_DATE时，可以插入但是会有warning,启用NO_ZERO_DATE+STRICT_TRANS_TABLES模式，无法插入且报出error,只启用STRICT_TRANS_TABLES模式，可以插入并且无任何warning。  
```
# 未启用NO_ZERO_DATE模式
root@localhost : wing 01:19:31> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 01:19:39> insert into sql_mode_test values('0000-00-00 00:00:00');
Query OK, 1 row affected (1.13 sec)

root@localhost : wing 01:20:07> select * from sql_mode_test;
+---------------------+
| ts                  |
+---------------------+
| 0000-00-00 00:00:00 |
+---------------------+
1 row in set (0.00 sec)

# 启用NO_ZERO_DATE模式
root@localhost : wing 01:20:16> set sql_mode='NO_ZERO_DATE';
Query OK, 0 rows affected, 1 warning (0.00 sec)

Warning (Code 1681): 'NO_ZERO_DATE' is deprecated and will be removed in a future release.
root@localhost : wing 01:20:37> insert into sql_mode_test values('0000-00-00 00:00:00');
Query OK, 1 row affected, 1 warning (0.00 sec)

Warning (Code 1264): Out of range value for column 'ts' at row 1
root@localhost : wing 01:20:42> select * from sql_mode_test;
+---------------------+
| ts                  |
+---------------------+
| 0000-00-00 00:00:00 |
| 0000-00-00 00:00:00 |
+---------------------+
2 rows in set (0.00 sec)

# 启用NO_ZERO_DATE和STRICT_TRANS_TABLES模式
root@localhost : wing 01:20:51> set sql_mode='NO_ZERO_DATE,STRICT_TRANS_TABLES';
Query OK, 0 rows affected, 1 warning (0.00 sec)

Warning (Code 1681): 'NO_ZERO_DATE' is deprecated and will be removed in a future release.
root@localhost : wing 01:21:29> 
root@localhost : wing 01:21:30> insert into sql_mode_test values('0000-00-00 00:00:00');
ERROR 1292 (22007): Incorrect datetime value: '0000-00-00 00:00:00' for column 'ts' at row 1
root@localhost : wing 01:21:37> select * from sql_mode_test;
+---------------------+
| ts                  |
+---------------------+
| 0000-00-00 00:00:00 |
| 0000-00-00 00:00:00 |
+---------------------+
2 rows in set (0.00 sec)

# 只启用STRICT_TRANS_TABLES模式
root@localhost : wing 01:21:42> set sql_mode='STRICT_TRANS_TABLES';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 01:32:37> insert into sql_mode_test values('0000-00-00 00:00:00');
Query OK, 1 row affected (0.00 sec)

root@localhost : wing 01:32:42>  select * from sql_mode_test;
+---------------------+
| ts                  |
+---------------------+
| 0000-00-00 00:00:00 |
| 0000-00-00 00:00:00 |
| 0000-00-00 00:00:00 |
+---------------------+
3 rows in set (0.00 sec)
```
- NO_ZERO_IN_DATE  
  声明：该参数将会被弃用，对于这类模式均生成一个warning。  
  与NO_ZERO_DATE模式相差不大。  
- ONLY_FULL_GROUP_BY  
  未启ONLY_FULL_GROUP_BY模式时，group by跟数据表中的任何字段均可，启用ONLY_FULL_GROUP_BY模式时，select的字段必须出现在group by中，group by字段可以不出现在select字段中。  
```
root@localhost : wing 02:42:49> show create table sql_mode_test;
+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                                                                                       |
+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL,
  `score` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 未启用ONLY_FULL_GROUP_BY模式
root@localhost : wing 02:42:57> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:43:03> select id from sql_mode_test group by name;
Empty set (0.00 sec)

# 启用ONLY_FULL_GROUP_BY模式
root@localhost : wing 02:43:18> set sql_mode='ONLY_FULL_GROUP_BY';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:43:36> select id from sql_mode_test group by name;
ERROR 1055 (42000): 'wing.sql_mode_test.id' isn't in GROUP BY
root@localhost : wing 02:43:37> select id from sql_mode_test group by id,name;
Empty set (0.00 sec)

```
- PAD_CHAR_TO_FULL_LENGTH  
  未启用PAD_CHAR_TO_FULL_LENGTH模式，对于CHAR类型会截断尾部空格，启用PAD_CHAR_TO_FULL_LENGTH模式，对于CHAR类型不会截断尾部空格。该模式对VARCHAR类型不起作用。  
```
root@localhost : wing 02:55:07> create table sql_mode_test(name char(10));
Query OK, 0 rows affected (1.39 sec)

root@localhost : wing 02:58:30> insert into sql_mode_test values('wing');
Query OK, 1 row affected (0.00 sec)

# 未启用PAD_CHAR_TO_FULL_LENGTH模式
root@localhost : wing 02:59:18> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)
root@localhost : wing 02:59:50> select name,char_length(name) from sql_mode_test;  
+------+-------------------+
| name | char_length(name) |
+------+-------------------+
| wing |                 4 |
+------+-------------------+
1 row in set (0.00 sec)

启用PAD_CHAR_TO_FULL_LENGTH模式
root@localhost : wing 02:59:55> set sql_mode='PAD_CHAR_TO_FULL_LENGTH';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 03:00:09> select name,char_length(name) from sql_mode_test;
+------------+-------------------+
| name       | char_length(name) |
+------------+-------------------+
| wing       |                10 |
+------------+-------------------+
1 row in set (0.00 sec)
```
- PIPES_AS_CONCAT  
  未启用PIPES_AS_CONCAT模式，将'||'视为'或'的功能，启用PIPES_AS_CONCAT模式，将'||'视为CONCAT()函数。  
```
root@localhost : wing 03:05:46> show create table sql_mode_test\G
*************************** 1. row ***************************
       Table: sql_mode_test
Create Table: CREATE TABLE `sql_mode_test` (
  `name` char(10) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

# 未启用PIPES_AS_CONCAT模式
root@localhost : wing 03:00:11> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 03:03:47> select concat(name,' hello') from sql_mode_test;
+-----------------------+
| concat(name,' hello') |
+-----------------------+
| wing hello            |
+-----------------------+
1 row in set (0.00 sec)

root@localhost : wing 03:04:19> select name||' hello' from sql_mode_test;
+----------------+
| name||' hello' |
+----------------+
|              0 |
+----------------+
1 row in set, 2 warnings (0.00 sec)

Warning (Code 1292): Truncated incorrect DOUBLE value: 'wing                          '
Warning (Code 1292): Truncated incorrect DOUBLE value: ' hello'

# 启用PIPES_AS_CONCAT模式
root@localhost : wing 03:05:12> set sql_mode='PIPES_AS_CONCAT';  
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 03:05:18> select concat(name,' hello') from sql_mode_test;
+-----------------------+
| concat(name,' hello') |
+-----------------------+
| wing hello            |
+-----------------------+
1 row in set (0.00 sec)

root@localhost : wing 03:05:21> select name||' hello' from sql_mode_test;
+----------------+
| name||' hello' |
+----------------+
| wing hello     |
+----------------+
1 row in set (0.00 sec)
```
- REAL_AS_FLOAT  
  未启用REAL_AS_FLOAT模式时，REAL类型相当于DOUBLE类型，启用REAL_AS_FLOAT模式时，REAL类型相当于FLOAT类型。  
```
# 未启用REAL_AS_FLOAT模式
root@localhost : wing 07:02:50> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 07:03:13> create table sql_mode_test(salary real);
Query OK, 0 rows affected (0.01 sec)

root@localhost : wing 07:06:06> show create table sql_mode_test;
+---------------+----------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                       |
+---------------+----------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `salary` double DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+----------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 启用REAL_AS_FLOAT模式
root@localhost : wing 07:06:17> set sql_mode='REAL_AS_FLOAT';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 07:06:34> show create table sql_mode_test;
+---------------+----------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                       |
+---------------+----------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `salary` double DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+----------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : wing 07:06:36> drop table sql_mode_test;
Query OK, 0 rows affected (0.09 sec)

root@localhost : wing 07:06:48> create table sql_mode_test(salary real);
Query OK, 0 rows affected (0.10 sec)

root@localhost : wing 07:06:57> show create table sql_mode_test;
+---------------+---------------------------------------------------------------------------------------------------+
| Table         | Create Table                                                                                      |
+---------------+---------------------------------------------------------------------------------------------------+
| sql_mode_test | CREATE TABLE `sql_mode_test` (
  `salary` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+---------------+---------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
- STRICT_ALL_TABLES  
  对所有的表都启用严格模式。  
- STRICT_TRANS_TABLES  
  对事务型的表启用严格模式。  

##### 组合的sql_mode
- ANSI  
  REAL_AS_FLOAT + PIPES_AS_CONCAT + ANSI_QUOTES + IGNORE_SPACE  
- DB2  
  PIPES_AS_CONCAT + ANSI_QUOTES + IGNORE_SPACE + NO_KEY_OPTIONS + NO_TABLE_OPTIONS + NO_FIELD_OPTIONS  
- MAXDB  
  PIPES_AS_CONCAT + ANSI_QUOTES + IGNORE_SPACE + NO_KEY_OPTIONS + NO_TABLE_OPTIONS + NO_FIELD_OPTIONS + NO_AUTO_CREATE_USER  
- MSSQL  
  PIPES_AS_CONCAT + ANSI_QUOTES + IGNORE_SPACE + NO_KEY_OPTIONS + NO_TABLE_OPTIONS + NO_FIELD_OPTIONS  
- MYSQL323  
  MYSQL323 + HIGH_NOT_PRECEDENCE  
- MYSQL40  
  MYSQL40 + HIGH_NOT_PRECEDENCE  
- ORACLE  
  PIPES_AS_CONCAT + ANSI_QUOTES + IGNORE_SPACE + NO_KEY_OPTIONS + NO_TABLE_OPTIONS +NO_FIELD_OPTIONS, NO_AUTO_CREATE_USER  
- POSTGRESQL  
  PIPES_AS_CONCAT + ANSI_QUOTES + IGNORE_SPACE + NO_KEY_OPTIONS + NO_TABLE_OPTIONS, + NO_FIELD_OPTIONS  
- TRADITIONAL  
  STRICT_TRANS_TABLES + STRICT_ALL_TABLES + NO_ZERO_IN_DATE + NO_ZERO_DATE + ERROR_FOR_DIVISION_BY_ZERO + NO_AUTO_CREATE_USER + NO_ENGINE_SUBSTITUTION  

##### 严格的sql_mode模式
严格处理MySQL的INSERT/UPDATE的无效的值。  
对于查询，无效的值将会返回一个warning,而不是error。  
严格模式包括：STRICT_ALL_TABLES/STRICT_TRANS_TABLES。  