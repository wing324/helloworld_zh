## 无效的grant语句导致主从复制断开


执行无效的grant语句，会导致主从复制断开以及主库二进制日志重新创建，从库报错：The incident LOST_EVENTS occured on the master. Message: error writing to the binary log，以及二进制日志会有如下记录：# Incident: LOST_EVENTS
RELOAD DATABASE; # Shall generate syntax error #。  



场景重现
-------
测试版本：MySQL 5.6.11  
首先主从复制正常：  
![](/img/invaild grant 1.png)  
主库：  
执行grant file on test.* to test@127.0.0.1语句  
![](/img/invaild grant 2.png)  
从库：  
 查看从库状态show slave status\G  
![](/img/invaild grant 3.png)  

查看二进制日志（注意：此时查看的二进制日志是show master status的上一个日志，因为在无效的grant语句执行后，主库会创建新的二进制日志），会有如下记录  
![](/img/invaild grant 4.png)  


错误原因分析
----------
因为file权限不能只赋予某个库，需要赋予所有库，即grant file on *.* to user@host才是有效的grant语句。  


解决方法
-------
1. 使用sql_slave_skip_counter跳过事件，但此方法只适用于基于二进制日志原理的复制，不适用于基于GTID原理的复制。  
2. 使用slave_skip_errors跳过错误。  
3. 在从库上做change master操作，重新切换master_log_file和master_log_pos。（由于无效的grant语句执行后会创建新的二进制日志，所以可以指定主库show master status的master_log_file和master_log_pos）  


规避方案
-------
避免无效的grant语句的执行。例如，file,process,super权限都要赋予所有数据库上，而不是赋予某个库或者某几个库。  


Bug地址
-------
MySQL Bug#68892：  
http://bugs.mysql.com/bug.php?id=68892  