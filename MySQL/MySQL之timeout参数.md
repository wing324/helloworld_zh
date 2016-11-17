## MySQL之timeout参数

总是记不住MySQL里面和timeout相关参数的含义，所以决定对其进行整理。  


首先查看下与MySQL相关的timeout参数：  
```sql
mysql> show variables like '%timeout%';
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| connect_timeout             | 10       |
| delayed_insert_timeout      | 300      |
| innodb_flush_log_at_timeout | 1        |
| innodb_lock_wait_timeout    | 120      |
| innodb_rollback_on_timeout  | ON       |
| interactive_timeout         | 172800   |
| lock_wait_timeout           | 31536000 |
| net_read_timeout            | 30       |
| net_write_timeout           | 60       |
| rpl_stop_slave_timeout      | 31536000 |
| slave_net_timeout           | 3600     |
| wait_timeout                | 172800   |
+-----------------------------+----------+
12 rows in set (0.00 sec)
```


connect_timeout
---------------
![](http://i.imgur.com/GgJb9Fv.png)  
释义：client与MySQL建立连接时，MySQL允许client建立连接的秒数，如果client建立与MySQL的连接超过connect_timeout，MySQL将拒绝client此次的连接（例如client登录MySQL时，client超过connect_timeout依旧没有登录到MySQL,MySQL将拒接client的此次登录）。MySQL会报出错误：Lost connection to MySQL server at 'XXX', system error: errno.  


delayed_insert_timeout
----------------------
![](http://i.imgur.com/aXc59Cw.png)  
INSERT DELAYED的超时时间。  


innodb_flush_log_at_timeout
---------------------------
![](http://i.imgur.com/PUlkJpj.png)  
每隔N秒刷新一次MySQL日志。  


innodb_lock_wait_timeout
------------------------
innodb事务中行锁等待超时时间（只适用于innodb行锁），一旦innodb行锁超过该值，将会回滚事务，对于事务回滚方式由innodb_rollback_on_timeout决定。  


innodb_rollback_on_timeout
--------------------------
![](http://i.imgur.com/Bm0BeFC.png)  
当innodb行锁等待超时后，如果innodb_rollback_on_timeout=0,则只回滚超时事务的最后一个语句(MySQL的默认处理方式)，如果innodb_rollback_on_timeout=1，则回滚整个超时事务（推荐）。  


interactive_timeout
-------------------
![](http://i.imgur.com/PC7AjXS.png)  
对于交互式的连接，当该连接超过interactive_timeout时间没有活动时，MySQL将关闭该交互式连接。  


lock_wait_timeout
-----------------
表锁等待超时的时间(官网上说的是metadata lock)，如LOCK TABLES,FLUSH TABLES WITH READ LOCK，HANDLER语句均会使用到metadata lock。  


net_read_timeout
----------------
服务器等待客户端发送数据包的超时时间。 （服务器读）  


net_write_timeout
-----------------
客户端等待服务器发送数据包的超时时间。（服务器写）  


rpl_stop_slave_timeout
----------------------
![](http://i.imgur.com/ADMglQk.png)  
为了方式stop slave与其他的slave SQL线程发生死锁，所以通过rpl_stop_slave_timeout控制STOP SLAVE的等待时间。  


slave_net_timeout
-----------------
![](http://i.imgur.com/Sa6EUho.png)  
当slave超过slave_net_timeout时间没有读到来自master的数据，就认为master宕机了，于是slave断开连接，尝试重连。  


wait_timeout
------------
![](http://i.imgur.com/FsXJLAX.png)  
对于非交互式的连接，一个持续sleep状态的线程超时时间。即通过show processlist看到的sleep状态的线程，当它持续sleep时间超过interactive_timeout之后，将会倍MySQL主动KILL掉。

