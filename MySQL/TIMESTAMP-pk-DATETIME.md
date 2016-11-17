## TIMESTAMP pk DATETIME

本文的比较是基于MySQL5.6的版本，日常使用中比较建议使用TIMESTAMP类型，为什么呢？请允许我娓娓道来~  

**1. TIMESTAMP受时区影响，DATETIME不受时区影响**  
```sql
root@localhost : menu 06:39:51> set session time_zone='+8:00';
Query OK, 0 rows affected (0.00 sec)

root@localhost : menu 06:40:32> show variables like '%zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | +08:00 |
+------------------+--------+
2 rows in set (0.00 sec)

root@localhost : menu 06:40:34> select * from t;
Empty set (0.00 sec)

root@localhost : menu 06:40:43> insert into t values();
Query OK, 1 row affected (0.00 sec)

root@localhost : menu 06:40:54> select * from t;
+---------------------+---------------------+
| id1                 | id2                 |
+---------------------+---------------------+
| 2016-01-12 06:40:54 | 2016-01-12 06:40:54 |
+---------------------+---------------------+
1 row in set (0.00 sec)

root@localhost : menu 06:40:55> set session time_zone='+9:00'; 
Query OK, 0 rows affected (0.01 sec)

root@localhost : menu 06:41:07> show variables like '%zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | +09:00 |
+------------------+--------+

root@localhost : menu 06:41:04> select * from t;
+---------------------+---------------------+
| id1                 | id2                 |
+---------------------+---------------------+
| 2016-01-12 07:40:54 | 2016-01-12 06:40:54 |
+---------------------+---------------------+
1 row in set (0.00 sec)
```

**2. TIMESTAMP与DATETIME取值范围不同**  
DATETIME的范围为‘1000-01-01 00：00：00.000000’--‘9999-12-31 23：59：59.999999’（不受时区影响），在UTC时区下，TIMESTAMP的范围为‘1970-01-01 00：00：01.000000’--‘2038-01-19 03：14：07.999999’  

**3. TIMESTAMP和DATETIME存储所需的空间不同**
在没有微秒的情况下，MySQL5.6.4之后，DATETIME为5个字节，TIMESTAMP为4个字节。  


*注意：*  
从MySQL5.6开始，允许同一个表中存在两种DEFAULT CURRENT_TIMESTAMP的TIMESTAMP类型。