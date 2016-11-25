## Redis命令随笔

Redis有着各种各样的命令，并且记录是个很好的习惯......

事务
----
使用'multi'开启一个事务，期间执行各种命令都是在一个线程上操作，保持其原子性；使用'exec'提交事务，执行事务中的所有命令；
```
# session 1
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> get user
(nil)
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set user 1
QUEUED
127.0.0.1:6379> incr user 
QUEUED
127.0.0.1:6379> incr user 
QUEUED
127.0.0.1:6379> incr user 
QUEUED
127.0.0.1:6379> get user
QUEUED

# session 2
127.0.0.1:6379> get user
(nil)

# session 1
127.0.0.1:6379> exec
1) OK
2) (integer) 2
3) (integer) 3
4) (integer) 4
127.0.0.1:6379> get user
"4"

# session 2
127.0.0.1:6379> get user
"4"
```


慢日志
------
##### cofig set slowlog-log-slower-than *times*
设置慢日志的时长  
##### slowlog get
获取所有的慢日志时间  
##### slowlog get *counts*
获取最近的*counts*个数的慢日志  
```
# 设置slowlog的时长为0
127.0.0.1:6379> config set slowlog-log-slower-than 0

# 获取所有的慢日志
127.0.0.1:6379> slowlog get
1) 1) (integer) 1
   2) (integer) 1449934538
   3) (integer) 32
   4) 1) "slowlog"
      2) "get"

# 获取最近出现的两个慢日志
127.0.0.1:6379> slowlog get 2
1) 1) (integer) 9
   2) (integer) 1449934878
   3) (integer) 23
   4) 1) "slowlog"
      2) "get"
2) 1) (integer) 8
   2) (integer) 1449934877
   3) (integer) 3
   4) 1) "flushdb"
```
##### 获取慢日志结果详解
```
127.0.0.1:6379> slowlog get

8) 1) (integer) 1
   2) (integer) 1449934514
   3) (integer) 54
   4) 1) "config"
      2) "get"
      3) "*"

9) 1) (integer) 0
# 自增ID
   2) (integer) 1449934507
# 命令发生的时间戳
   3) (integer) 5
# 命令运行的时长（微秒）
   4) 1) "config"
      2) "set"
      3) "slowlog-log-slower-than"
      4) "0"
# 运行的命令与其参数
```
Redis的慢日志在默认情况下，只保留1024条慢日志，并且slowlog是保存在内存里的，所以一旦reids重启之后，slowlog也会丢失。  


排序
----
sort  
对list、set、sorted set进行排序  
```
# list
127.0.0.1:6379> lrange name 0 -1
1) "5"
2) "2"
3) "3"
127.0.0.1:6379> sort name
1) "2"
2) "3"
3) "5"
127.0.0.1:6379> lrange name 0 -1
1) "5"
2) "2"
3) "3"

# set
127.0.0.1:6379> sadd name wing fianna bibi chou zhou
(integer) 5
127.0.0.1:6379> smembers name
1) "bibi"
2) "fianna"
3) "chou"
4) "wing"
5) "zhou"
127.0.0.1:6379> sort name alpha
1) "bibi"
2) "chou"
3) "fianna"
4) "wing"
5) "zhou"
127.0.0.1:6379> sort name limit 0 3  alpha
1) "bibi"
2) "chou"
3) "fianna"

# sorted name
127.0.0.1:6379> zadd name  0 wing
(integer) 1
127.0.0.1:6379> zadd name  1 fianna
(integer) 1
127.0.0.1:6379> zadd name  2 bibi
(integer) 1
127.0.0.1:6379> zadd name  3 chou
(integer) 1
127.0.0.1:6379> zadd name  4 zhou
(integer) 1
127.0.0.1:6379> zrange name 0 -1
1) "wing"
2) "fianna"
3) "bibi"
4) "chou"
5) "zhou"
127.0.0.1:6379> sort name alpha
1) "bibi"
2) "chou"
3) "fianna"
4) "wing"
5) "zhou"
127.0.0.1:6379> sort name limit 0 3 desc alpha
1) "zhou"
2) "wing"
3) "fianna"
127.0.0.1:6379> sort name limit 0 3 asc alpha
1) "bibi"
2) "chou"
3) "fianna"

```


管理命令
-------
- config get *parameter*  
  获取配置参数信息  
```
# 获取全部参数信息
127.0.0.1:6379> config get *
  1) "dbfilename"
  2) "dump.rdb"
  3) "requirepass"
  4) ""
...
127) "notify-keyspace-events"
128) ""
129) "bind"
130) ""

# 获取模糊参数信息
127.0.0.1:6379> config get *log*
 1) "logfile"
 2) ""
...
13) "loglevel"
14) "notice"

# 获取指定参数信息
127.0.0.1:6379> config get slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "10000"
```


关键字（key）
-----------
- DEL  
  删除命令，返回的是实际删除的key的数量。  
```
127.0.0.1:6379> set name wing
OK
127.0.0.1:6379> get name
"wing"
127.0.0.1:6379> del name 
(integer) 1
127.0.0.1:6379> get name
(nil)

# 注意 del命令返回的整数是实际删除的key的数量
127.0.0.1:6379> set name wing
OK
127.0.0.1:6379> set user wing
OK
127.0.0.1:6379> del name user school
(integer) 2
```
- DUMP  
  用于获取存储在Redis中键数据的序列化版本，返回的是序列化的值。  
```
127.0.0.1:6379> set name wing 
OK
127.0.0.1:6379> dump name
"\x00\x04wing\x06\x00\x0fQD\x89^\xc7u\xc5"
127.0.0.1:6379> set name fianna
OK
127.0.0.1:6379> get name
"fianna"
127.0.0.1:6379> dump name
"\x00\x06fianna\x06\x00\xce\x1aa\xf3K\x0eh\x1b"
```
- EXISTS  
  返回key是否存在,返回值1表示存在，返回值0表示不存在。  
```
127.0.0.1:6379> get name
"fianna"
127.0.0.1:6379> exists name
(integer) 1
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> exists name
(integer) 0
```
- EXPIRE key seconds  
  设置key过期时间。返回值1表示key已经过期，返回值0表示key已经删除或者不能设置为过期。  
```
127.0.0.1:6379> set name wing
OK
127.0.0.1:6379> expire name 10
(integer) 1
127.0.0.1:6379> ttl name
(integer) 7
127.0.0.1:6379> ttl name
(integer) 4
127.0.0.1:6379> ttl name
(integer) 3
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> exists name
(integer) 0
```
- EXPIREAT *unix timestamp*  '
  与EXPIRE相似，EXPIREAT后接的是unix时间戳  
```
# unix时间戳
mysql> select unix_timestamp('2015-12-27 00:00:00');
+---------------------------------------+
| unix_timestamp('2015-12-27 00:00:00') |
+---------------------------------------+
|                            1451174400 |
+---------------------------------------+
1 row in set (0.12 sec)

# 当前时间2015/12/27 20：11：00
127.0.0.1:6379> set name wing
OK
127.0.0.1:6379> expireat name  1451174400
(integer) 1
127.0.0.1:6379> exists name
(integer) 0
127.0.0.1:6379> exists name
(integer) 0
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> ttl name

# unix时间戳
mysql> select unix_timestamp('2015-12-27 20:13:00');    
+---------------------------------------+
| unix_timestamp('2015-12-27 20:13:00') |
+---------------------------------------+
|                            1451247180 |
+---------------------------------------+
1 row in set (0.00 sec)

# 当前时间2015/12/27 20：11：53
127.0.0.1:6379> set name wing
OK
127.0.0.1:6379> expireat name 1451247180
(integer) 1
127.0.0.1:6379> ttl name
(integer) 28866
127.0.0.1:6379> ttl name
(integer) 28863
127.0.0.1:6379> ttl name
(integer) 0
```
- KEYS  

