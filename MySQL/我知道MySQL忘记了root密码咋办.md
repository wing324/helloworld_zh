## 我知道MySQL忘记了root密码咋办

环境越多，密码越多，没有合理的密码管理工具，很容易很容易忘记密码哒。。那么忘记数据库root密码了，你说咋办啊。。测试环境密码忘记事情比较小，你要是生产环境数据库密码都忘记了，说出去，别人家还敢让你去管理他们的数据库嘛。。别着急，我给你的秘密武器~~~



#### 方法一
此方法是很常用的方法，是需要重启数据库的。就是使用skip-grant-tables来跳过授权表。  
```sql
# 首先，我假装忘记了数据库密码
[root@localhost ~]# mysql -uroot -S /data/mysql/mysqldata3306/sock/mysql.sock -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)

# 此时去my.cnf文件中添加skip-grant-tables参数，用来跳过授权表
[root@localhost ~]# vim /etc/my.cnf
skip-grant-tables

# 重启MySQL
[root@localhost ~]# kill -9 11366
[root@localhost ~]# mysqld_multi start 3306

# 无验证进入MySQL，此时可以看到成功进入
[root@localhost ~]# mysql -uroot -S /data/mysql/mysqldata3306/sock/mysql.sock
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.26-log MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

root@localhost : (none) 06:31:45>

# 使用UPDATE修改密码
root@localhost : (none) 06:33:56> update mysql.user set password=password('123456') where user='root';
Query OK, 4 rows affected (2.73 sec)
Rows matched: 4  Changed: 4  Warnings: 0

root@localhost : (none) 06:34:16> flush privileges;
Query OK, 0 rows affected (0.16 sec)

# 再次重启数据库啦，注释掉my.cnf文件中的skip-grant-tables
[root@localhost ~]# vim /etc/my.cnf
#skip-grant-tables
[root@localhost ~]# mysqladmin shutdown -uroot -S /data/mysql/mysqldata3306/sock/mysql.sock -p
Enter password:
[root@localhost ~]# mysqld_multi start 3306

# 使用新密码进入数据库，可以看到成功进入
[root@localhost ~]# mysql -uroot -S /data/mysql/mysqldata3306/sock/mysql.sock -p123456
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26-log MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

root@localhost : (none) 06:38:41> 
```


#### 方法二
方法一呢需要重启数据库，可是咱是生产环境啊，重启数据库得损失多个亿啊，坚决不能重启数据库，那么此时，只能使用黑客的方法啦。。。
具体思路是，可以将user表的frm/MYD/MYI文件拷贝到另外一个数据库中，然后对密码进行UPDATE，然后在将文件move回来，使用KILL -HUB `pidof mysqld`重新加载配置，然后就可以使用新密码啦~（今天想睡觉了，下次再来补上具体过程）
```sql
# 首先将数据库（此处假设3306实例）中user相关的数据文件移到另一个库(此处假设3307实例)的test目录下
mv /data/mysqldata3306/mydata/mysql/user.* /data/mysqldata3307/mysqldata/test

# 进入3307数据库下
root@localhost : (none) 06:33:56> update test.user set password=password('123456') where user='root';
Query OK, 4 rows affected (2.73 sec)
Rows matched: 4  Changed: 4  Warnings: 0

# 此时将3307的user相关文件再move回原来的数据库目录下
mv /data/mysqldata3307/mysqldata/test /data/mysqldata3306/mydata/mysql/user.*

# 使用KILL -HUB加载mysqld的权限，看到KILL不要怕，KILL -HUB是重新加载配置的意思啦
KILL -HUB `pidof mysqld`

# 此时你就可以使用新的密码登录到数据库里面去了，整个过程完全不需要启动数据库~
[root@localhost ~]# mysql -uroot -S /data/mysql/mysqldata3306/sock/mysql.sock -p123456
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26-log MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

root@localhost : (none) 07:38:41> 
```