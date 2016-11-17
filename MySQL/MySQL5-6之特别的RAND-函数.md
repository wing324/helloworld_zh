## MySQL5.6之特别的RAND()函数

RAND()函数特别之处在于它在主从复制下是安全的函数，即使是基于binlog_format=STATEMENT模式下也不会导致主从复制数据不一致。 



约定
-------
1.在没有特殊说明的情况下，默认binlog_format=STATEMENT,由于binlog_format=ROW/MIXED模式下，RAND()函数以行格式记录，所以此处不做讨论。  
2.测试表结构如下：  

```sql
CREATE TABLE `t` (
  `id` decimal(5,3) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```


测试过程
--------------
1.RAND()函数不在同一个事务中  
①　测试原理：在autocommit=ON的情况下，多次执行INSERT INTO t VALUES(RAND());语句  
![](/img/rand函数1.png)  
②　binlog_format=STATEMENT模式下的。二进制日志记录如下：  
![](/img/rand函数2.png)  
③　主从复制的数据  
MASTER:  
![](/img/rand函数3.png)  
SLAVE:  
![](/img/rand函数4.png)  
2.RAND()函数在同一个事务中  
①　测试原理：在START TRANSACTION;和COMMIT;之间执行多次INSERT INTO t VALUES(RAND());语句  
![](/img/rand函数5.png)  
②　binlog_format=STATEMENT模式下的。二进制日志记录如下：  
![](/img/rand函数6.png)  
③　主从复制的数据  
MASTER:  
![](/img/rand函数7.png)  
SLAVE:  
![](/img/rand函数8.png)  


结论
------
由测试过程可得,不论是同一个事务中还是非同一个事务中,包含RAND()函数的SQL语句之前都会记录两个会话级的参数RAND_SEED1和RAND_SEED2，由这两个参数根据RAND()产生随机数的算法便可得到一个确定的数值，所以即使在binlog_format=STATEMENT模式下，主从复制之间使用RAND()函数可以确保数据一致。  