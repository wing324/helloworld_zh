## mysql_config_editor工具

mysql_config_editor是MySQL自带的一款用于安全加密登录的工具，对于多实例的MySQL数据库来说，每次登陆需要指定host,port,password真的有点烦躁呢，所以可以使用mysql_config_editor工具轻松搞定。  


实战演练
-------
首先对该工具进行一个快速上手吧。  

```shell
# 首先根据你的MySQL客户端登陆需求，设置登陆信息，不用害怕，这些信息不会泄露，都会被MySQL加密为别人看不懂的格式
[root@mysql ~]# mysql_config_editor set --login-path=wing3306 --user=wing --host=127.0.0.1 --port=3306 --password
Enter password: 

# 为了相信真的是别人看不懂，还是看一下保存信息吧，信息保存为~/.mylogin.cnf，其保存信息为二进制文件，反正这结果我是看不懂哒
[root@mysql ~]# hexdump ~/.mylogin.cnf 
0000000 0000 0000 1912 0410 0d08 0a0d 0307 1216
0000010 0b13 0700 1800 021e 0010 0000 00d2 4a12
0000020 229e 43c1 0aba ba4e 2899 314a 0010 0000
0000030 f41c 5f8d 0b8b 1efc bf69 c6b2 95de 865c
0000040 0020 0000 3913 f6af 32bf 3a02 f6d0 2007
0000050 9e60 9ff7 ae3b 6f04 e9ad 59c8 38b2 688a
0000060 d143 ffb7 0020 0000 a2b5 c420 3de7 99e3
0000070 2909 085f f4c7 2740 769f 86e9 8208 d7d6
0000080 3ad8 93cc aafd 4389 0010 0000 6bd2 327a
0000090 94dc fbd5 86af d82b a87b 27aa          
000009c

# 此时登陆MySQL,我们可以很轻松的看到MySQL登陆成功的界面啦
[root@mysql ~]# mysql --login-path=wing3306
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 24
Server version: 5.6.26-log MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

wing@127.0.0.1 : (none) 03:22:19> 
```


相关选项
-------
##### mysql_config_editor
- set 对login path进行登陆信息设置

```
[root@mysql ~]# mysql_config_editor set --login-path=wing3306 --user=wing --host=127.0.0.1 --port=3306 --password
Enter password:
 
```

- print 显示指定的login path所有信息

```
# 再次验证没有泄露password哦
[root@mysql ~]# mysql_config_editor print --login-path=wing3306
[wing3306]
user = wing
password = *****
host = 127.0.0.1
port = 3306
```

- remove 从登陆文件中删除所有的login path

```
# 第一次可以登录
[root@mysql ~]# mysql --login-path=wing3306
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 26
Server version: 5.6.26-log MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

wing@127.0.0.1 : (none) 04:04:04> \q
Bye

# 将指定的login path删除，同样的方式将无法登陆
[root@mysql ~]# mysql_config_editor remove --login-path=wing3306
[root@mysql ~]# mysql --login-path=wing3306
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (111)

[root@mysql ~]# mysql_config_editor print --login-path=wing3306
```

- reset 删除登陆日志的所有内容


```
[root@mysql ~]# mysql_config_editor reset
```

##### mysql_config_editor set 
- -h,--host=name 添加host到登陆文件中
- -G，--login-path=name 在登录文件中为login path添加名字（默认为client）
- -p,--password 在登陆文件中添加密码（该密码会被mysql_config_editor自动加密）
- -u，--user 添加用户名到登陆文件中
- -S,--socket=name 添加sock文件路径到登陆文件中
- -P，--port=name 添加登陆端口到登陆文件中

##### mysql_config_editor print
--all 输出所有的login path的登陆信息
--login-path 指定login path，输出指定的login path登陆信息

```
[root@mysql ~]# mysql_config_editor print --all
[wing3306]
user = wing
password = *****
host = 127.0.0.1
port = 3306
[wing3307]
user = wing
password = *****
host = 127.0.0.1
port = 3307

[root@mysql ~]# mysql_config_editor print --login-path=wing3306
[wing3306]
user = wing
password = *****
host = 127.0.0.1
port = 3306
```

##### mysql_config_editor remove
- -h,--host 删除login path中的host信息
- -G，--login-path 指定删除的login path(默认为client)
- -p,--password 删除login path中的password信息
- -u，--user 删除login path中的用户名信息
- -S，--socket 删除login path中的sock文件信息
- -P，--port 删除login path中的port信息

##### mysql_config_editor reset
无选项。  