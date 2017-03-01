## SELECT … FOR UPDATE如何影响INNODB的锁级别

如果`SELECT ... FOR UPDATE `生效，需要在noautocommit的环境下，即`BEGIN;COMMIT/ROLLBACK;`或者`SET AUTOCOMMIT=0`的前提下。本文使用`BEGIN;COMMIT/ROLLBACK;`创造noautocommit的环境研究`SELECT ... FOR UPDATE `对于INNODB的锁级别影响。

**约定**

- 表结构如下

  ```sql
  CREATE TABLE `t` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(16) NOT NULL,
    `num1` int(11) NOT NULL,
    `num2` int(11) NOT NULL,
    `num3` int(11) NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_name` (`name`),
    KEY `ix_num1_num2` (`num1`,`num2`)
  ) ENGINE=InnoDB AUTO_INCREMENT=67 DEFAULT CHARSET=utf8
  ```

- 表数据如下

  ```sql
  select * from t;
  +----+-------+------+------+------+
  | id | name  | num1 | num2 | num3 |
  +----+-------+------+------+------+
  |  1 | AAAAA |    0 |    2 |    2 |
  |  2 | BBBBB |    1 |    2 |    2 |
  |  3 | CCCCC |    0 |    0 |    0 |
  |  4 | DD    |    4 |    1 |    1 |
  |  5 | EE    |    0 |    5 |    5 |
  | 66 | FFFFF |    0 |    5 |    5 |
  +----+-------+------+------+------+
  6 rows in set (0.00 sec)
  ```




一、WHERE条件使用主键

```sql
# session 1
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t where id=1;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | t     | const | PRIMARY       | PRIMARY | 4       | const |    1 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)

mysql> select * from t where id=1 for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  1 | AAAAA |    0 |    2 |    2 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

# session 2
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t where id=2 for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  2 | BBBBB |    1 |    2 |    2 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where name="DD" for update;
+----+------+------+------+------+
| id | name | num1 | num2 | num3 |
+----+------+------+------+------+
|  4 | DD   |    4 |    1 |    1 |
+----+------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where id=1 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where name="AAAAA" for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where num1=0 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where num3=2 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```



二、WHERE条件使用唯一索引

```sql
# session 1
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t where name='AAAAA' ;
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
| id | select_type | table | type  | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | t     | const | ux_name       | ux_name | 50      | const |    1 | NULL  |
+----+-------------+-------+-------+---------------+---------+---------+-------+------+-------+
1 row in set (0.00 sec)

# session 2
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t where name ="DD" for update;
+----+------+------+------+------+
| id | name | num1 | num2 | num3 |
+----+------+------+------+------+
|  4 | DD   |    4 |    1 |    1 |
+----+------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where id=2 for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  2 | BBBBB |    1 |    2 |    2 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where name='AAAAA' for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where id=1 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where num1=0 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where num3=2 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

三、WHERE条件使用普通索引

```sql
# session 1
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t where num1=0 and num2=5;
+----+-------------+-------+------+---------------+--------------+---------+-------------+------+-------+
| id | select_type | table | type | possible_keys | key          | key_len | ref         | rows | Extra |
+----+-------------+-------+------+---------------+--------------+---------+-------------+------+-------+
|  1 | SIMPLE      | t     | ref  | ix_num1_num2  | ix_num1_num2 | 8       | const,const |    2 | NULL  |
+----+-------------+-------+------+---------------+--------------+---------+-------------+------+-------+
1 row in set (0.00 sec)

mysql> select * from t where num1=0 and num2=5 for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  5 | EE    |    0 |    5 |    5 |
| 66 | FFFFF |    0 |    5 |    5 |
+----+-------+------+------+------+
2 rows in set (0.00 sec)

# session 2
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t where id=1 for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  1 | AAAAA |    0 |    2 |    2 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where name="AAAAA" for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  1 | AAAAA |    0 |    2 |    2 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where num1=0 and num2=5 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where num2=5 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where num3=5 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```



四、WHERE条件使用联合索引的部分索引

```sql
# session 1
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t where num1=1 ;
+----+-------------+-------+------+---------------+--------------+---------+-------+------+-------+
| id | select_type | table | type | possible_keys | key          | key_len | ref   | rows | Extra |
+----+-------------+-------+------+---------------+--------------+---------+-------+------+-------+
|  1 | SIMPLE      | t     | ref  | ix_num1_num2  | ix_num1_num2 | 4       | const |    1 | NULL  |
+----+-------------+-------+------+---------------+--------------+---------+-------+------+-------+
1 row in set (0.00 sec)

mysql> select * from t where num1=1 for update;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  2 | BBBBB |    1 |    2 |    2 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

# session 2
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t where id =3;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  3 | CCCCC |    0 |    0 |    0 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where id =3 for update
    -> ;
+----+-------+------+------+------+
| id | name  | num1 | num2 | num3 |
+----+-------+------+------+------+
|  3 | CCCCC |    0 |    0 |    0 |
+----+-------+------+------+------+
1 row in set (0.00 sec)

mysql> select * from t where num1=4 for update;
+----+------+------+------+------+
| id | name | num1 | num2 | num3 |
+----+------+------+------+------+
|  4 | DD   |    4 |    1 |    1 |
+----+------+------+------+------+
1 row in set (0.00 sec)

mysql> explain select * from t where num1=4 for update;
+----+-------------+-------+------+---------------+--------------+---------+-------+------+-------+
| id | select_type | table | type | possible_keys | key          | key_len | ref   | rows | Extra |
+----+-------------+-------+------+---------------+--------------+---------+-------+------+-------+
|  1 | SIMPLE      | t     | ref  | ix_num1_num2  | ix_num1_num2 | 4       | const |    1 | NULL  |
+----+-------------+-------+------+---------------+--------------+---------+-------+------+-------+
1 row in set (0.00 sec)

mysql> select * from t where num1=0 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

mysql> explain select * from t where num1=0 for update;
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | t     | ALL  | ix_num1_num2  | NULL | NULL    | NULL |    6 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)
```



五、WHERE条件不使用索引

```sql
# session 1
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> explain select * from t where num3=1 ;
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | t     | ALL  | NULL          | NULL | NULL    | NULL |    6 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

mysql> select * from t where num3=1 for update;
+----+------+------+------+------+
| id | name | num1 | num2 | num3 |
+----+------+------+------+------+
|  4 | DD   |    4 |    1 |    1 |
+----+------+------+------+------+
1 row in set (0.00 sec)

# session 2
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from t where id=1 for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
mysql> select * from t where name='BBBBB' for update;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

