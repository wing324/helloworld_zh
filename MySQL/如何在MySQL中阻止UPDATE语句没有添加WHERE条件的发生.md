## 如何在MySQL中阻止UPDATE语句没有添加WHERE条件的发生

如果在生产环境中使用UPDATE语句更新表数据，此时如果忘记携带本应该添加的WHERE条件，那么。。Oh,no...后果可能不堪设想。。。之前就遇到一个同事在生产环境UPDATE忘记携带WHERE条件。。于是只能从binlog日志中找到相关数据，然后去恢复。。宝宝当时表示心好累。。。那么有没有什么办法可以阻止这样的事情发生，又不使用任何的审核工具呢。。。办法当然是有的，请听我说~  



sql_safe_updates
----------------
sql_safe_updates这个MySQL自带的参数就可以完美的解决我们的问题，并且该参数是可以在线变更的哦~  
当该参数开启的情况下，你必须要在UPDATE语句后携带WHERE条件，否则就会报出ERROR。。  


举个栗子
-------
```sql
# sql_safe_updates=0,即未开启
root@127.0.0.1 : test 07:58:34> set sql_safe_updates=0;
Query OK, 0 rows affected (0.00 sec)

root@127.0.0.1 : test 07:58:43> show variables like 'sql_safe_updates';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| sql_safe_updates | OFF   |
+------------------+-------+
1 row in set (0.00 sec)

root@127.0.0.1 : test 07:58:55> select * from t;
+-------+
| pd    |
+-------+
| hello |
| mysql |
+-------+
2 rows in set (0.00 sec)

root@127.0.0.1 : test 07:58:59> begin;
Query OK, 0 rows affected (0.00 sec)

root@127.0.0.1 : test 07:59:04> update t set pd='MySQL';
Query OK, 2 rows affected (0.00 sec)
Rows matched: 2  Changed: 2  Warnings: 0

root@127.0.0.1 : test 07:59:12> select * from t;
+-------+
| pd    |
+-------+
| MySQL |
| MySQL |
+-------+
2 rows in set (0.00 sec)

# sql_safe_updates=1,即开启
root@127.0.0.1 : test 08:00:00> set sql_safe_updates=1;
Query OK, 0 rows affected (0.00 sec)

root@127.0.0.1 : test 08:00:11> show variables like 'sql_safe_updates';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| sql_safe_updates | ON    |
+------------------+-------+
1 row in set (0.00 sec)

root@127.0.0.1 : test 08:00:16> select * from t;
+-------+
| pd    |
+-------+
| hello |
| mysql |
+-------+
2 rows in set (0.00 sec)

root@127.0.0.1 : test 08:00:25> begin;
Query OK, 0 rows affected (0.00 sec)

root@127.0.0.1 : test 08:00:27> update t set pd='MySQL';
ERROR 1175 (HY000): You are using safe update mode and you tried to update a table without a WHERE that uses a KEY column

```
如上属的栗子所示，当参数sql_safe_updates开启的时候，UPDATE语句不携带WHERE条件将会爆出一个错误。。所以小心使用UPDATE语句是真的很重要哇。。。