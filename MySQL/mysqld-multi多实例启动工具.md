## mysqld_multi多实例启动工具

我们往往喜欢在一台服务器上安装多个实例，一般使用不同的端口，如3306，3307等，那么怎么才能启动这些实例呢？怎么才能一起启动呢？又怎么才能一个一个启动呢？MySQL很人性化的提供了一款自带的工具：mysqld_multi，可以毫无压力地满足我们对多实例启动的方式。  



mysqld_multi准备知识
-------------------
- mysqld_multi启动会查找my.cnf文件中的[mysqldN]组，N为mysqld_multi后携带的整数值。
- mysqld_multi的固定选项可在配置文件my.cnf中进行配置，在[mysqld_multi]组下配置（如果没有该组，可自行建立）。
- mysqld_multi使用方式如下：

```shell
mysqld_multi [options] {start|stop|reload|report} [GNR[,GNR] ...]
```


mysqld_multi选项
----------------
start 开启MySQL实例
stop 关闭MySQL实例
reload 重新加载MySQL实例
report 返回MySQL当前状态

```shell
# report返回当前MySQL状态
[root@mysql log]# mysqld_multi report 3307
Reporting MySQL servers
MySQL server from group: mysqld3307 is not running

# start开启MySQL实例
[root@mysql log]# mysqld_multi start 3307
[root@mysql log]# mysqld_multi report 3307
Reporting MySQL servers
MySQL server from group: mysqld3307 is running

# reload重新加载MySQL实例
[root@mysql log]# mysqld_multi reload 3307
[root@mysql log]# mysqld_multi report 3307
Reporting MySQL servers
MySQL server from group: mysqld3307 is running

# stop关闭MySQL实例，注意此处是需要一个具有shutdown权限的用户，且密码并被是加密的，也不可以交互式输入密码，Linux又具有history功能，所以为了数据库的安全，还是不要用mysqld_multi stop的方式关闭数据库了吧
[root@mysql log]# mysqld_multi stop 3307 --user=root --password=******
[root@mysql log]# mysqld_multi report 3307
Reporting MySQL servers
MySQL server from group: mysqld3307 is not running
```

- --example 输出一个mysqld_multi配置文件中的配置示例
```shell
 mysqld_multi --example
# This is an example of a my.cnf file for mysqld_multi.
# Usually this file is located in home dir ~/.my.cnf or /etc/my.cnf
#
# SOME IMPORTANT NOTES FOLLOW:
#
# 1.COMMON USER
#
#   Make sure that the MySQL user, who is stopping the mysqld services, has
#   the same password to all MySQL servers being accessed by mysqld_multi.
#   This user needs to have the 'Shutdown_priv' -privilege, but for security
#   reasons should have no other privileges. It is advised that you create a
#   common 'multi_admin' user for all MySQL servers being controlled by
#   mysqld_multi. Here is an example how to do it:
#
#   GRANT SHUTDOWN ON *.* TO multi_admin@localhost IDENTIFIED BY 'password'
#
#   You will need to apply the above to all MySQL servers that are being
#   controlled by mysqld_multi. 'multi_admin' will shutdown the servers
#   using 'mysqladmin' -binary, when 'mysqld_multi stop' is being called.
#
# 2.PID-FILE
#
#   If you are using mysqld_safe to start mysqld, make sure that every
#   MySQL server has a separate pid-file. In order to use mysqld_safe
#   via mysqld_multi, you need to use two options:
#
#   mysqld=/path/to/mysqld_safe
#   ledir=/path/to/mysqld-binary/
#
#   ledir (library executable directory), is an option that only mysqld_safe
#   accepts, so you will get an error if you try to pass it to mysqld directly.
#   For this reason you might want to use the above options within [mysqld#]
#   group directly.
#
# 3.DATA DIRECTORY
#
#   It is NOT advised to run many MySQL servers within the same data directory.
#   You can do so, but please make sure to understand and deal with the
#   underlying caveats. In short they are:
#   - Speed penalty
#   - Risk of table/data corruption
#   - Data synchronising problems between the running servers
#   - Heavily media (disk) bound
#   - Relies on the system (external) file locking
#   - Is not applicable with all table types. (Such as InnoDB)
#     Trying so will end up with undesirable results.
#
# 4.TCP/IP Port
#
#   Every server requires one and it must be unique.
#
# 5.[mysqld#] Groups
#
#   In the example below the first and the fifth mysqld group was
#   intentionally left out. You may have 'gaps' in the config file. This
#   gives you more flexibility.
#
# 6.MySQL Server User
#
#   You can pass the user=... option inside [mysqld#] groups. This
#   can be very handy in some cases, but then you need to run mysqld_multi
#   as UNIX root.
#
# 7.A Start-up Manage Script for mysqld_multi
#
#   In the recent MySQL distributions you can find a file called
#   mysqld_multi.server.sh. It is a wrapper for mysqld_multi. This can
#   be used to start and stop multiple servers during boot and shutdown.
#
#   You can place the file in /etc/init.d/mysqld_multi.server.sh and
#   make the needed symbolic links to it from various run levels
#   (as per Linux/Unix standard). You may even replace the
#   /etc/init.d/mysql.server script with it.
#
#   Before using, you must create a my.cnf file either in /usr/my.cnf
#   or /root/.my.cnf and add the [mysqld_multi] and [mysqld#] groups.
#
#   The script can be found from support-files/mysqld_multi.server.sh
#   in MySQL distribution. (Verify the script before using)
#

[mysqld_multi]
mysqld     = /usr/bin/mysqld_safe
mysqladmin = /usr/bin/mysqladmin
user       = multi_admin
password   = my_password

[mysqld2]
socket     = /tmp/mysql.sock2
port       = 3307
pid-file   = /var/lib/mysql2/hostname.pid2
datadir    = /var/lib/mysql2
language   = /usr/share/mysql/mysql/english
user       = unix_user1

[mysqld3]
mysqld     = /path/to/mysqld_safe
ledir      = /path/to/mysqld-binary/
mysqladmin = /path/to/mysqladmin
socket     = /tmp/mysql.sock3
port       = 3308
pid-file   = /var/lib/mysql3/hostname.pid3
datadir    = /var/lib/mysql3
language   = /usr/share/mysql/mysql/swedish
user       = unix_user2

[mysqld4]
socket     = /tmp/mysql.sock4
port       = 3309
pid-file   = /var/lib/mysql4/hostname.pid4
datadir    = /var/lib/mysql4
language   = /usr/share/mysql/mysql/estonia
user       = unix_user3
 
[mysqld6]
socket     = /tmp/mysql.sock6
port       = 3311
pid-file   = /var/lib/mysql6/hostname.pid6
datadir    = /var/lib/mysql6
language   = /usr/share/mysql/mysql/japanese
user       = unix_user4
```

- --log=file_name 指定一个日志输出文件，如果文件存在则在文件末尾处添加日志信息
- --mysqladmin=pro_name 用于指定一个程序来实现mysqladmin的功能
- --mysqld=pro_name 用于指定一个程序来实现mysqld的功能，如mysqld_safe
- --no-log 将日志信息输出到屏幕上，而不是输入日志文件中
- --password= mysqladmin用户的密码
- --silent 简要信息
- --user= mysqladmin用户，默认为mysql
- --tcp-ip 连接到每个服务器的tcp/ip端口，有时候sock文件丢失，但仍然可以通过tcp/ip端口连接服务器