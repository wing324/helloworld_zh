## 四下让你明白MySQL函数之last_insert_id()

遇到一个小伙伴说，为什么last_insert_id()得到的结果与预期的不一样呢，于是，我认真的去研究的一下这个参数，研究结果请点击全文。  



#### 首先，举个栗子
```sql
wing@3306>show create table tt;
+-------+-----------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                          |
+-------+-----------------------------------------------------------------------------------------------------------------------+
| tt    | CREATE TABLE `tt` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+-------+-----------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 没有指定值的时候，last_insert_id()符合预期希望
wing@3306>insert into tt values();
Query OK, 1 row affected (0.00 sec)

wing@3306>select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                1 |
+------------------+
1 row in set (0.00 sec)

wing@3306>insert into tt values();
Query OK, 1 row affected (0.00 sec)

wing@3306>select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                2 |
+------------------+
1 row in set (0.00 sec)

# what?不是应该是5么，为什么是第一个插入的值3？last_insert_id开始有一点不符合预期了。。
wing@3306>insert into tt values(),(),();
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

wing@3306>select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                3 |
+------------------+
1 row in set (0.00 sec)

wing@3306>insert into tt values(),(),();
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

wing@3306>select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                6 |
+------------------+
1 row in set (0.00 sec)

# 纳尼？按照预期不是10么？为什么还是之前的6？last_insert_id()我不懂你啊。。
wing@3306>insert into tt values(10);
Query OK, 1 row affected (0.01 sec)

wing@3306>select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
|                6 |
+------------------+
1 row in set (0.00 sec)
```


#### 其次，研究一下
查阅MySQL官方文档，真的太重要了。。。  
官方出处：  
http://dev.mysql.com/doc/refman/5.6/en/information-functions.html#function_last-insert-id  
1. 官方文档原话：  
   With no argument, LAST_INSERT_ID() returns a 64-bit value representing the first automatically generated value successfully inserted for an AUTO_INCREMENT column as a result of the most recently executed INSERT statement.  
   翻译：  
   没有参数的last_insert_id()返回的是**最近一次**针对**autoincrement列**执行的**INSERT语句**的**第一个自动生成**的值。  
2. 官方文档原话：  
   If you insert multiple rows using a single INSERT statement, LAST_INSERT_ID() returns the value generated for the first inserted row only. The reason for this is to make it possible to reproduce easily the same INSERT statement against some other server.  
   翻译：  
   如果你在单条INSERT语句中插入多个值，那么last_insert_id()返回的是该INSERT语句**第一个自动生成**的值。  


然后，剖析一下
------------
请认真阅读上述翻译中的黑色字体，牢记last_insert_id()的约束。  
1. 为什么插入指定的值，last_insert_id()就失效了呢？  
   官方文档明明说了，是**自动生成**的值啊，不是你指定的值啊，是由autoincremnt计数器自己生成的才能被last_insert_id()追踪到哇。。  
2. 为什么多值插入的时候，显示的是第一条插入值啊，last不是最后一个值的意思么啊啊啊。。  
   官方文档明明说了，是**最近一次**的**INSERT语句****自动生成的第一个值**哇哇哇。。  


#### 最后，总结一下
记住last_insert_id()的约束。。  
**最近一次INSERT语句在autpincrement列上自动生成的第一个值**。  
总结的这句话比翻译的那句话感觉顺口多了==  