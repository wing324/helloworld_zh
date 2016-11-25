## Oracle启动与关闭

Oralce的启动与关闭。  

Oracle启动
----------
oracle启动分为三个步骤：  
###### startup nomount  
Oracle找到参数文件，根据参数文件设置将内存空间划分出来并打开相应的进程。  
```sql
SQL> startup nomount
ORACLE instance started.

Total System Global Area   814264320  bytes
Fixed Size                   2257560  bytes
Variable Size              415239528  bytes
Database Buffers           394264576  bytes
Redo Buffers                 2502656  bytes

```
###### alter database mount
根据参数文件设置的控制文件位置，打开控制文件。  
```sql
SQL> alter database mount;
Database altered.
```
###### alter database open
开启Oracle数据库。  
```sql
SQL> alter database open;
Database altered.
```
**注意**  
`startup`直接全部执行"nomount","mount","open"步骤。  


Oracle关闭
---------
###### abort
	模拟突然掉电
	内存被清空、内存中的数据没有写入数据文件
	事务被立即中断
	没有提交、没有回滚
###### immediate（常用）
	强制中断当前正在运行的所有事务，回滚这些事务
	回滚完毕，强制中断所有的连接
	讲实例中的所有数据写入数据文件
###### transactional（常用）
	等待正在运行的事务，一直到他们提交或者回滚
	所有事务主动结束以后（提交或者回滚），强行中断连接
	将实例里面的数据写入数据文件
	清空缓存
	如果有事务一直没有提交或者回滚，实例无法关闭
###### normal(默认)
	等待事务的主动提交或者回滚
	等待用户主动断开连接
	如果有一个用户没有断开连接，那么数据库无法关闭


修改参数的scope含义
------------------
###### scope=both
参数立即生效并且永久生效  
###### scope=spfile
参数需要重启后生效
###### scope=memory
参数立即生效，但重启后将不再生效
