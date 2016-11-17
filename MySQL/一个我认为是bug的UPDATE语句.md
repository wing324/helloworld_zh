## 一个我认为是bug的UPDATE语句

这个我认为的bug，反馈给MySQL官方，但是MySQL官方认为这并不是一个bug，并给出了解释，我认为这个解释是合理的，但是不可避免的是这条语句实在太危险了。



#### 问题描述

示例表结构与表数据：

```sql
# 表结构
mysql> show create table t;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                      |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------+
| t     | CREATE TABLE `t` (
  `id` int(11) DEFAULT NULL,
  `c1` int(11) DEFAULT NULL,
  `c2` varchar(16) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

# 表数据
mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |    1 | A    |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)
```

正常的UPDATE语句应该长这样子的，SET后接的并列字段分隔符为"逗号(,)"：

```sql
mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |    1 | A    |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)

mysql> update t set c1=11,c2='AA' where id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |   11 | AA   |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)
```

不合理的UPDATE应该是什么样子的呢，是将SET后接的并列字段分隔符改为"AND"，**注意这样写的话，MySQL并不会报错，还会执行成功，但是语义完全和"逗号"作为分隔符是两码事**：

```sql
# 先来个实验
mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |    1 | A    |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)

mysql> update t set c1=11 and c2='AA' where id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |    0 | A    |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)
# 实验走到这里，觉得这个update不是我想要的，但是这个c1咋变为0了呢，c2咋没变呢？


# 再来个实验
mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |    1 | A    |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)

mysql> update t set  c2='AA' and c1= 11 where id=1;
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> select * from t;
+------+------+------+
| id   | c1   | c2   |
+------+------+------+
|    1 |    1 | 0    |
|    2 |    2 | B    |
|    3 |    3 | C    |
+------+------+------+
3 rows in set (0.00 sec)
# 咦？这里c2变了，咋变为了0呢？我明明是个字母啊，再怎么改也不能改成一个字符串型数字啊？c2咋依旧没变呢？
```



#### 问题原因

原来在UPDATE …  SET后接分隔符为"AND"的语句，由于AND的优先级较高，所以先处理“AND”，再处理“＝”，于是“＝”后面的值只有逻辑运算的结果true(1) / false(0)，直接上实验好了＝＝

```sql
# 第一个实验解析
update t set c1=11 and c2='AA' where id=1;
# 在这条语句中MySQL将c1=11 and c2='AA'解析成了c1=(11 and c2='AA'),而在这张表中(11 and c2='AA')是一个假语句(false),所以MySQL将c1值解析为c1=0
mysql> select 11 and c2='AA' from t where id=1;
+----------------+
| 11 and c2='AA' |
+----------------+
|              0 |
+----------------+
1 row in set (0.00 sec)
# 所以最终出现了c1变为0，c2没有任何改变的现象


# 第二个实验解析
update t set  c2='AA' and c1= 11 where id=1;
# 在这条语句中MySQL将c2='AA' and c1= 11解析成了c2=('AA' and c1= 11),而在这张表中('AA' and c1= 11)是一个假语句(false),所以MySQL将c2值解析为c2=0,然后隐式转换，将'0'存储到c2列
mysql> select 'AA' and c1= 11 from t where id=1;
+-----------------+
| 'AA' and c1= 11 |
+-----------------+
|               0 |
+-----------------+
1 row in set, 1 warning (0.01 sec)
# 所以最终出现了c2变为'0'，c1依旧没变
```



#### 问题总结

这个问题告诉我们，SQL语句一定要写规范了，执行SQL语句前一定要清楚这条SQL的运行逻辑，对于UPDATE／DELETE手动执行之前，一定要按照如下顺序来＝＝

```sql
begin;
update / delete ...;
select 校验数据;
commit; --数据校验成功
rollback; --数据校验失败
```





我投递的bug地址：https://bugs.mysql.com/bug.php?id=82647