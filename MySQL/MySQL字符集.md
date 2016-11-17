## MySQL字符集

啊啊啊啊啊啊啊，乱码真的是个太恼人的东西了，而且一乱码就心慌......好吧，好好学习下字符集吧。。



字符集简介
----------
1. 字符集不仅影响到数据的存储,也影响到客户端和MySQL服务器之间的通信,如果你需要客户端和服务器之间通信不同于默认的字符集,则需要指定字符集通过以下的方式：
   set names charset_name;  
   **特别说明**  
   set names修改字符集,修改的参数如下：  
   character_set_client  
   character_set_connection  
   character_set_results  
   collation_connection  
2. 通过SHOW CHARACTER SET查看MySQL支持的字符集(SHOW CHARACTER SET LIKE 'utf8');
   通过SHOW COLLATION查看MySQL支持的排序规则(SHOW COLLATION LIKE 'utf8%')。
3. 排序规则的特点

- 不同的字符集有不同的排序规则
- 每一个字符集都有一个默认的排序规则
- 排序规则中,_ci(case insensitive)大小写不敏感,_cs(case sensitive)大小写敏感,_bin(binary)二进制
```sql
# _ci

root@localhost : character_set_test 01:36:24> show create table t1;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                              |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------+
| t1    | CREATE TABLE `t1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) CHARACTER SET gbk DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:37:04> select * from t1;
+------+------+
| id   | name |
+------+------+
|    1 | ABd  |
+------+------+
1 row in set (0.01 sec)

root@localhost : character_set_test 01:37:10> select * from t1 where name like 'A%';
+------+------+
| id   | name |
+------+------+
|    1 | ABd  |
+------+------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:37:21> select * from t1 where name like 'a%';
+------+------+
| id   | name |
+------+------+
|    1 | ABd  |
+------+------+
1 row in set (0.01 sec)
```
```sql
# _cs

root@localhost : character_set_test 01:37:24> show create table t2;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                              |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------+
| t2    | CREATE TABLE `t2` (
  `name` varchar(16) CHARACTER SET latin1 COLLATE latin1_general_cs DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

root@localhost : character_set_test 01:37:56> select * from t2;
+------+
| name |
+------+
| ABd  |
+------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:38:06> select * from t2 where name like 'A%';
+------+
| name |
+------+
| ABd  |
+------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:38:14> select * from t2 where name like 'a%';
Empty set (0.00 sec)
```
```sql
# _bin
root@localhost : character_set_test 01:40:09> show create table t3;
+-------+------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                       |
+-------+------------------------------------------------------------------------------------------------------------------------------------+
| t3    | CREATE TABLE `t3` (
  `name` varchar(16) CHARACTER SET latin1 COLLATE latin1_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:40:17> select * from t3 ;
+------+
| name |
+------+
| ABd  |
+------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:40:37> select * from t3 where name like 'A%';
+------+
| name |
+------+
| ABd  |
+------+
1 row in set (0.00 sec)

root@localhost : character_set_test 01:40:47> select * from t3 where name like 'a%';
Empty set (0.00 sec)
```
- 一个字符集会有多个排序规则,具体如何排序,请参照 http://www.collation-charts.org/


字符集&排序规则
---------------
1. 字符集和排序规则可以设置4个级别：server,database,table,column
2. CHARACTRE SET=CHARSET

#### Server character set & collation
MySQL服务器的字符集和排序规则
character_set_server
collation_server
可在线修改参数

##### Database character set & collation
1. 数据库的字符集和排序规则
   character_set_database
   collation_database
   可在线修改
   用于默认的数据库字符集和排序规则
2. 所有的数据库选项被保存在数据库目录下的db.opt文件里
```sql
[root@wing character_set_test]# cat db.opt 
default-character-set=utf8
default-collation=utf8_general_ci
```
3. 如果没有指定表的字符集和排序规则,那么表的字符集和排序规则默认使用character_set_database和collation_database。
4. 对于LOAD DATA语句,如果没有指定character set,默认使用character_set_database。
5. 对于存储过程和函数来说,如果没有指定character set和collation,默认使用character_set_database和collation_database。

##### Table character set & collation
1. 在CREATE TABLE 和 ALTER TABLE中指定character set & collation。
2. 表的字符集和排序规则被使用在列上,当列没有指定的字符集和排序规则。

##### Column character set & collation
1. 在CREATE TABLE和ALTER TABLE中指定character set & collation。
2. 如果列没有指定的字符集和排序规则,那么将使用表的字符集和排序规则。
3. 如果列指定了字符集,那么排序规则将使用字符集默认的排序规则,而不是表的字符集。

##### 字符串的character set & collation
```sql
[_charset_name]'string' [COLLATE collation_name]
```
EXAMPLE
```sql
root@localhost : character_set_test 01:59:42> select '中文';
+--------+
| 中文   |
+--------+
| 中文   |
+--------+
1 row in set (0.01 sec)

root@localhost : character_set_test 02:01:29> select _gbk'中文';
+---------+
| 涓?枃   |
+---------+
| 涓?枃   |
+---------+
1 row in set (0.00 sec)

root@localhost : character_set_test 02:01:34> select _gbk'中文' collate gbk_bin;
+------------------------------+
| _gbk'中文' collate gbk_bin   |
+------------------------------+
| 涓?枃                        |
+------------------------------+
1 row in set (0.00 sec)
```
**特别说明**
1. 相关参数
   character_set_connection
   collation_connection
2. _*charset_name*表达式旨在告诉解析器：该字符串使用指定的charset

##### Connection character set & collation
1. 相关参数
   character_set_client：客户端发来的字符集,如client端发来的语句使用该字符集到server端
   character_set_connection collation_connection：server端接收该语句后使用的字符集和排序规则,server端将语句从character_set_client转换到character_set_connection(除了使用_charset_name指定的字符串)
   character_set_database collation_database：指定数据库的字符集和排序规则
   character_set_filesystem
   character_set_results：服务器返回结果给客户端的字符集
   character_set_server collation_server：server端默认的内部操作字符集
   character_set_system
2. 设置字符集语句
   1)set names '*charset_name*' collate '*collate_name*'
   指明了客户端发来语句时使用的字符集以及客户端收到语句时使用的字符集
   set names修改字符集,修改的参数如下:
   character_set_client = *character_name*
   character_set_connection = *character_name*
   character_set_results = *character_name*
   collation_connection = *collation_name*
   2)set character name '*charset_name*'
   修改的参数如下:
   character_set_client = *character_name*
   character_set_results = *character_name*
   collation_connection = collation_database
   由于collation_connection的改变会引起character_set_connection的改变
   3)使用mysql客户端时,可以直接使用charset '*charset_name*',与set names一样
```sql
root@localhost : (none) 04:13:54> charset latin1
Charset changed
root@localhost : (none) 04:14:02> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

##### 应用程序的character set & collection
1. 应用程序使用的MySQL默认charset&collation(latin1,latin1_swedish_ci),如果需要不同的charset&collection,可使用如下几种方式
   1) 为每个数据库指定一个charset&collation
   2) 服务器启动时指定charset&collation
   3) 在配置文件中指定charset&collation

##### 错误信息的charset & collation
描述服务器如何使用字符集构建错误日志以及返回客户端
1. MySQL使用utf8构建错误信息,以character_set_results返回到客户端
2. 错误信息的模板为utf8
   错误信息模板中参数被特定值取代时：
   标识值如表名/列名在内部使用utf8;
   字符串值(非二进制形式)则将它们从charset转换成utf8;
   二进制字符串在 (0x20,0x7E)范围内使用十六进制,其他的使用十进制
3. 返回错误信息时,MySQL服务器会将错误信息从utf8转换到character_set_results设置的字符集中。

##### 字符集排序规则(collation)
_ci：大小写不敏感
_cs：大小写敏感
_bin：字符基于二进制编码,大小写敏感

##### MySQL元数据的字符集
1. MySQL默认元数据的字符集为UTF-8。
2. 参数character_set_system设置的便是MySQL元数据字符集。
3. 字符集转换
   1) 元数据与数据之间字符集转换
```sql
SELECT * FROM t1 WHERE USER() = latin1_column;
# latin1_column自动转换成utf8字符集
```
```sql
INSERT INTO t1 (latin1_column) SELECT USER();
# user()自动转换成latin1字符集
```
2) 列的字符集转换
- 如果列为二进制的数据类型(BINARY,VARBINARY,BLOB),所有的值必须使用同一个字符集,如果使用多个字符集,MySQL没法知道哪些值使用了那些字符集导致MySQL不能正确转换数据。
- 如果列为非二进制的数据类型(CHAR,VARCHAR,TEXT),列值将会被列的字符集编码,如果列值在不同的字符集里,可先将列值转换成二进制,然后将二进制列转换成需要的列。

4. 如果某列由于字符集原因导致乱码,可将该列转换为二进制模式(binary/varbinary/blob)
```sql
# 一个乱码小实验
root@localhost : ym 02:14:53> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

root@localhost : ym 02:15:01> show create table t;
+-------+------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                           |
+-------+------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : ym 02:15:16> select * from t;
+------+--------+
| id   | name   |
+------+--------+
|    1 | 中国   |
+------+--------+
1 row in set (0.00 sec)

root@localhost : ym 02:15:21> set names gbk;
Query OK, 0 rows affected (0.00 sec)

root@localhost : ym 02:15:26> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | gbk                        |
| character_set_connection | gbk                        |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | gbk                        |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

root@localhost : ym 02:15:30> show create table t;
+-------+------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                           |
+-------+------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : ym 02:15:36> select * from t;
+------+------+
| id   | name |
+------+------+
|    1 |     |
+------+------+
1 row in set (0.00 sec)

root@localhost : ym 02:15:41> alter table t modify name varbinary(16);
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

root@localhost : ym 02:16:00> select * from t;
+------+--------+
| id   | name   |
+------+--------+
|    1 | 中国   |
+------+--------+
1 row in set (0.00 sec)

root@localhost : ym 02:16:04> show create table t;
+-------+--------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                             |
+-------+--------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `name` varbinary(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+--------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
5. 对于binary类型转换为char类型,末尾会有0x00结尾符,体现在结果集中为异常的空格,可使用TRIM()函数去除
   EXAMPLE
```sql
root@localhost : ym 02:21:40> create table t(id int,name binary(50));
Query OK, 0 rows affected (0.01 sec)

root@localhost : ym 02:22:04> insert into t values(1,'中国');
Query OK, 1 row affected (0.00 sec)

root@localhost : ym 02:22:15> select * from t;
+------+----------------------------------------------------+
| id   | name                                               |
+------+----------------------------------------------------+
|    1 | 中国                                               |
+------+----------------------------------------------------+
1 row in set (0.00 sec)

root@localhost : ym 02:22:19> alter table t modify name char(50);
Query OK, 1 row affected (0.02 sec)
Records: 1  Duplicates: 0  Warnings: 0

root@localhost : ym 02:22:45> select * from t;
+------+----------------------------------------------------+
| id   | name                                               |
+------+----------------------------------------------------+
|    1 | 中国                                               |
+------+----------------------------------------------------+
1 row in set (0.00 sec)
# 中国后面有异常的空格
# 正常非转换过的结果集如下
# root@localhost : ym 02:24:02> create table t1 (id int,name char(50));
# Query OK, 0 rows affected (0.01 sec)

# root@localhost : ym 02:22:46> update t set name =trim(trailing 0x00 from name);
# Query OK, 1 row affected (0.00 sec)
# Rows matched: 1  Changed: 1  Warnings: 0

# root@localhost : ym 02:24:16> insert into t1 values(1,'中国');
# Query OK, 1 row affected (0.00 sec)

# root@localhost : ym 02:24:35> select * from t1;
# +------+--------+
# | id   | name   |
# +------+--------+
# |    1 | 中国   |
# +------+--------+
# 1 row in set (0.00 sec)

# root@localhost : ym 02:23:25> select * from t;
# +------+--------+
# | id   | name   |
# +------+--------+
# |    1 | 中国   |
# +------+--------+
# 1 row in set (0.00 sec)

# 使用trim()函数解决该情况
root@localhost : ym 02:22:46> update t set name =trim(trailing 0x00 from name);
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

root@localhost : ym 02:23:25> select * from t;
+------+--------+
| id   | name   |
+------+--------+
|    1 | 中国   |
+------+--------+
1 row in set (0.00 sec)
```

6. 两个字符集的切换,保险起见,还是应该先将字符集切换为二进制类型(binary/varbinary/blob),而后转换成自己想要的字符集。
7. 如果需要将表中所有列的字符集转换成另外的字符集,可以使用语句
```sql
ALTER TABLE table_name CONVERT TO CHARACTER SET charset_name;
```


总结
----
**一个完整的用户请求的字符集转换过程**
![](/img/MySQL_Character_Set.png)

**特别说明**
1. 相关变量
   character_set_client：client端发出请求使用的字符集
   character_set_connection：Server端接受到Client端的请求后,使用的字符集
   character_set_database：数据库默认字符集,如果没有默认数据库,那就使用character_set_server指定的字符集
   character_set_filesystem：文件系统的字符集，把操作系统上文件名转化成此字符集,即把character_set_client转换character_set_filesystem,该参数默认字符集为binary则不需做任何转换
   character_set_results：返回结果集的字符集
   character_set_server：数据库服务器的默认字符集
   character_set_system：MySQL服务器的元数据字符集,默认为utf8

