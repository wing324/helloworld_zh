## mysql.event中诡异的execute_at&last_executed

你发现了吗？对于不是UTC时区的MySQL，mysql.event与information_schema.events中execute_at、last_executed、starts、ends的时间是不一样的。你以为是Bug吗？NONONO,这才不是一个Bug呢。这是MySQL的精心设计之处。  



问题发现
-------
一次意外的发现创建完Event之后，information_schema.events和mysql.event表中execute_at和last_executed字段均相差8个小时，mysqld实例的时区为：  
![](/img/eventtime1.png)  


测试过程
-------
-  创建测试数据库  
   ![](/img/eventtime2.png)  
-  创建测试数据表  
   ![](/img/eventtime3.png)  
-  创建Event  
   ![](/img/eventtime4.png)  
-  执行SQL语句查询mysqld实例的时区和时间  
   ![](/img/eventtime5.png)  
-  执行SQL语句查询information_schema.events表和mysql.event表中execute_at和last_executed字段值  
   ![](/img/eventtime6.png)  
   ![](/img/eventtime7.png)  


问题原因
-------
为了确保Event的执行不受时区的影响，使得Event可以准确执行，MySQL将mysql.event表的事件调度时间（execute_at和last_executed）转换成UTC时间。  

文档描述：  
Times in the ON SCHEDULE clause are interpreted using the current session time_zone value. This becomes the event time zone; that is, the time zone that is used for event scheduling and is in effect within the event as it executes. These times are converted to UTC and stored along with the event time zone in the mysql.event table. This enables event execution to proceed as defined regardless of any subsequent changes to the server time zone or daylight saving time effects.  

文档链接：  
http://dev.mysql.com/doc/refman/5.6/en/create-event.html  

**注意**  
mysql.event表中除了execute_at和last_execute诡异外，starts和ends也是同样原因引起的诡异。  