## CentOS7下源码安装MySQL5.7.6+

Linux版本：Centos7

MySQL版本：MySQL5.7.16

该文档适用于MySQL版本>=5.7.6

1. 卸载CentOS7默认携带的mariadb包

   ```shell
   # 检查mariadb安装包
   [root@wing ~]# rpm -qa | grep -i mysql
   [root@wing ~]# rpm -qa | grep -i mariadb
   mariadb-libs-5.5.50-1.el7_2.x86_64

   # 卸载mariadb安装包
   [root@wing ~]# rpm -e mariadb-libs-5.5.50-1.el7_2.x86_64
   error: Failed dependencies:
   	libmysqlclient.so.18()(64bit) is needed by (installed) postfix-2:2.10.1-6.el7.x86_64
   	libmysqlclient.so.18(libmysqlclient_18)(64bit) is needed by (installed) postfix-2:2.10.1-6.el7.x86_64
   [root@wing ~]# rpm -e mariadb-libs-5.5.50-1.el7_2.x86_64 postfix-2:2.10.1-6.el7.x86_64
   ```

2. 获得MySQL所有版本(5.0.15-latest)地址传送门
   http://downloads.mysql.com/archives/community/

3. 安装编译软件

   ```shell
   yum install -y cmake make gcc gcc-c++
   ```

4. 创建MySQL安装目录

   ```shell
   # 如MySQL安装目录为：/usr/local/mysql
   mkdir -p /usr/local/mysql
   ```

5. 解压MySQL源码包

   ```shell
   tar -zxvf mysql-5.7.16.tar.gz
   ```

6. 创建mysql用户和用户组

   ```shell
   # 创建用户组
   groupadd mysql
   # 创建mysql用户，所属组为mysql
   useradd -s /bin/bash -m -g mysql mysql
   ```

7. 安装MySQL相关依赖包

   ```shell
   yum install -y ncurses-devel openssl-devel bison-devel libaio libaio-devel
   ```

8. boost库安装

   ```shell
   # 该步骤可以省略，在cmake阶段添加参数-DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost即可
   # boost库安装
   wget http://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
   tar -zxvf boost_1_59_0.tar.gz -C /usr/local
   mv /usr/local/boost_1_59_0 /usr/local/boost
   cd /usr/local/boost
   ./bootstrap.sh
   ./b2 stage threading=multi link=shared
   ./b2 install threading=multi link=shared
   ```

9. 创建MySQL相关目录

   | 目录         | 含义                                     | 配置参数                                     |
   | :--------- | -------------------------------------- | ---------------------------------------- |
   | bin_log    | 二进制日志目录                                | log_bin_basename<br />log_bin_index      |
   | mydata     | 数据文件目录                                 | datadir                                  |
   | innodb_log | InnoDB重做日志目录                           | innodb_log_group_home_dir                |
   | innodb_ts  | InnoDB共享表空间目录                          | innodb_data_home_dir                     |
   | log        | 日志文件目录(error log+general log+slow log) | log_error<br />general_log_file<br />slow_query_log_file |
   | relay_log  | InnoDB中继日志目录                           | relay_log_basename<br />relay_log_index  |
   | tmpdir     | 临时文件目录                                 | tmpdir                                   |
   | undo_log   | InnoDB回滚日志目录                           | innodb_undo_directory                    |

   ```shell
      mkdir -p /data/mysql/mysql3306/bin_log
      mkdir -p /data/mysql/mysql3306/db_file
      mkdir -p /data/mysql/mysql3306/innodb_log
      mkdir -p /data/mysql/mysql3306/innodb_ts
      mkdir -p /data/mysql/mysql3306/log
      mkdir -p /data/mysql/mysql3306/relay_log
      mkdir -p /data/mysql/mysql3306/tmpdir
      mkdir -p /data/mysql/mysql3306/undo_log
   ```

10. 修改步骤9创建的目录的所属用户与所属组为mysql:mysql

   ```
   chown -R mysql:mysql /data/mysql/mysql3306
   ```

11. 将MySQL配置文件my.cnf放置到/etc目录下

   > 默认情况下，MySQL会依次按顺序查找如下几个路径来获取MySQL配置问文件：
   >
   > /etc/my.cnf
   >
   > /etc/mysql/my.cnf
   >
   > /etc/my.cnf/my.cnf
   >
   > /usr/local/mysql/my.cnf
   >
   > ~/.my.cnf
   >
   > 使用过程中可通过--defaults-file=xxx来指定配置文件。

```shell
# 修改MySQL配置文件所属用户与所属组
chown -R mysql:mysql my.cnf
```

12. 编译安装MySQL5.7.6+

```shell
   # 切换到mysql-5.7.16源码目录下
   cd /path/mysql-5.7.16
   
   # cmake
   cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/mysql3306/mydata -DSYSCONFDIR=/etc/my.cnf -DWITH_INNOBASE_STORAGE_ENGINE=1  -DWITH_PARTITION_STORAGE_ENGINE=1 -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLE_DEBUG_SYNC=0 -DENABLED_LOCAL_INFILE=1 -DENABLED_PROFILING=1 -DMYSQL_TCP_PORT=3306 -DMYSQL_UNIX_ADDR=/data/mysql/mysql3306/tmpdir/my-3306.sock -DWITH_DEBUG=0 -DWITH_SSL=yes -DWITH_SAFEMALLOC=OFF -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/usr/local/boost
  
   # make 
   # 该命令中可以通过添加-j参数指定多线程工作，如make -j2 && make install -j2 则使用2个CPU核进行make
   # 该步骤执行完毕后，可以到CMAKE_INSTALL_PREFIX参数指定的目录下，即MySQL安装目录下查看到mysql相关目录与文件
   make && make install
   
   # 修改MySQL安装目录的所属用户与用户组为mysql:mysql
   chown -R mysql:mysql /usr/local/mysql   
```

13.  初始化MySQL

    ```shell
       # 进入到MySQL安装目录下
       cd /usr/local/mysql
       # 初始化MySQL，切记--defaults-file=/etc/my.cnf要放在参数的第一位，初始化信息可以在MySQL的errorlog中查看，并且在errorlog会生成一个root的随机密码，该随机密码仅仅为root@localhost用户所有。
       mysqld --defaults-file=/etc/my.cnf --initialize --basedir=/usr/local/mysql --datadir=/data/mysql/mysql3306/mydata --user=mysql
    ```

14.  添加MySQL环境变量

    ```shell
       vim /etc/profile

       # 在~/.bashrc文件下添加如下语句
       export MYSQL_HOME=/usr/local/mysql
       export PATH=${MYSQL_HOME}/bin:$PATH

       # 保存后，使环境变量生效
       source /etc/profile
    ```

15.  启动MySQL

    ```shell
    mysqld_safe --defaults-file=/etc/my.cnf &
    # 此时可以通过ps -ef | grep mysql看到相关进程
    ```

16.  登陆MySQL

    ```shell
    mysql -uroot -S /data/mysql/mysql3306/tmpdir/mysql.sock -p
    # 输入errorlog中生成的随机密码，即可登陆MySQL

    # 登陆mysql需要修改root密码，否则会出现下列情况：
    root@localhost : (none) 11:16:52> show databases;
    ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
    ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
    ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

    # 修改root密码
    set password='MYSQL';
    # 目前版本可以使用直接的字符串代替以前password('xxx')的加密方式，目前版本提示如下：
    root@localhost : (none) 11:16:54> set password=password('MYSQL');
    Query OK, 0 rows affected, 1 warning (0.00 sec)
    Warning (Code 1287): 'SET PASSWORD = PASSWORD('<plaintext_password>')' is deprecated and will be removed in a future release. Please use SET PASSWORD = '<plaintext_password>' instead
    root@localhost : (none) 11:19:27> set password='MYSQL';
    ```

17.  关闭MySQL

    ```shell
    mysqladmin shutdown -uroot -S /data/mysql/mysql3306/tmpdir/mysql.sock -p
    # 使用新密码
    ```

18.  初始化的MySQL5.7.6+与MySQL5.6.xx不同之处

    > - 初始化工具不同
    >
    >   MySQL5.6.xx使用的是mysql_install_db，MySQL5.7.6+官方推荐使用mysqld --initialize。
    >
    > - 初始化数据库不同
    >
    >   MySQL5.6.xx初始化之后存在mysql,information_schema,performance_schema,test四个数据库，MySQL5.7.6+初始化之后存在mysql,information_schema,performance_schema,sys四个数据库。
    >
    > - 初始化用户不同
    >
    >   MySQL5.6.xx初始化之后存在root@localhost,root@'::1',root@'hostname',''@'localhost',''@'hostname'五个用户，MySQL5.7.6+初始化之后存在mysql.sys,root@localhost用户
    >
    > - 初始化root密码
    >
    >   MySQL5.6.xx初始化之后root用户密码为空，MySQL5.7.6+初始化之后会为root@localhost用户生成随机密码。