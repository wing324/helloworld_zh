## mysqladmin管理MySQL服务器的客户端

mysqladhmin是一款作为管理MySQL的客户端工具，你可以用它来检查服务器的配置和当前状态等等。  



语法
---
```shell
# 语法
mysqladmin [options] command [command-arg]
# 示例
[root@mysql ~]# mysqladmin --user=root --socket=/data/mysqldata3306/sock/mysql.sock status -p
Enter password: 
Uptime: 86415  Threads: 3  Questions: 8545  Slow queries: 2  Opens: 79  Flush tables: 1  Open tables: 68  Queries per second avg: 0.098
```


Command
-------
- create *db_name*  
  创建新的数据库  
- debug  
  使服务器将debug信息写到error log里  
- drop *db_name*  
  删除该数据库及所有表  
- extended-status  
  输出服务器的状态变量，相当于show global status;  
- flush-hosts  
  刷新host cache中的所有信息  
- flus-status  
  清除所有的状态变量  
- flush-tables  
  刷新所有的表  
- flush-threads  
  刷新所有的线程缓存  
- kill id,id,...  
  杀死服务器线程，id为thread id  
- old-password new-password  
  修改旧密码  
- password new-password  
  设置新密码  
- ping  
  检查服务器是否可用  
```
[root@mysql ~]# mysqladmin --login-path=root3306 ping
mysqld is alive
```
- processlist  
  相当于show processlist  
```
[root@mysql ~]# mysqladmin --login-path=root3306 processlist
+----+-----------------+-----------+----+---------+-------+------------------------+------------------+
| Id | User            | Host      | db | Command | Time  | State                  | Info             |
+----+-----------------+-----------+----+---------+-------+------------------------+------------------+
| 1  | event_scheduler | localhost |    | Daemon  | 88456 | Waiting on empty queue |                  |
| 39 | root            | localhost |    | Query   | 0     | init                   | show processlist |
+----+-----------------+-----------+----+---------+-------+------------------------+------------------+
```
- reload  
  加载权限表（grant table）  
- refresh  
  刷新所有表并且先关闭当前binlog,重新开启新的binlog  
- shutdown  
  关闭MySQL服务器  
- start-slave  
  开启slave,相当于start slave  
- stop-slave  
  关闭slave,相当于stop slave  
- status  
  输出当前服务器的状态 
```
[root@mysql ~]# mysqladmin --login-path=root3306 status
Uptime: 89174  Threads: 2  Questions: 8594  Slow queries: 2  Opens: 79  Flush tables: 3  Open tables: 0  Queries per second avg: 0.096
# 对上述的输出名词解释
Uptime			当前服务器运行了多长时间
Threads			活动线程（active threads）的数量
Questions		服务器运行以来执行的总查询数，不论是正确的SQL还是错误的SQL
Slow queries	慢查询数量
Opens			服务器开启的表数量
Flush tables	服务器执行flush-*/refresh/reload的数量
Open tables		服务器当前打开的表的数量
```
- variables  
  输出当前服务器的系统变量，相当于show global variables;  


Option
------
以下选项可在配置文件中的[mysqladmin]组下配置。（以下仅列出一些个人觉得有用的选项，具体可通过mysqladmin --help查看）  
- --compress 压缩所有的信息在服务器与客户端之间传输
- --force 即使SQL语句出错，仍然继续执行
- --login-path 一种连接MySQL的方式，详情请参考：  
  http://wing324.github.io/2015/09/28/mysql-config-editor%E5%B7%A5%E5%85%B7/
- --print-default 输出默认选项
```
[root@mysql ~]# mysqladmin --login-path=root3306 --print-defaults
mysqladmin would have been started with the following arguments:
--user=root --password=root --socket=/data/mysqldata3306/sock/mysql.sock
```
- --sleep=N 每N分钟重复执行相同命令
```
[root@mysql ~]# mysqladmin --login-path=root3306 status --sleep=5
Uptime: 101052  Threads: 3  Questions: 8633  Slow queries: 2  Opens: 79  Flush tables: 3  Open tables: 0  Queries per second avg: 0.085
Uptime: 101057  Threads: 3  Questions: 8634  Slow queries: 2  Opens: 79  Flush tables: 3  Open tables: 0  Queries per second avg: 0.085
Uptime: 101062  Threads: 3  Questions: 8635  Slow queries: 2  Opens: 79  Flush tables: 3  Open tables: 0  Queries per second avg: 0.085
```
- --relative 与--sleep一起使用，显示当前状态值和之前状态值的差，仅能用于extended-status

