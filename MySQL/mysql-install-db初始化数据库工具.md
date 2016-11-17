## mysql_install_db初始化数据库工具

编译安装MySQL的时候，我们可以编译指定MySQL各种数据目录，比如datadir，binlog等等，那么在RPM安装MySQL时怎么办呢？别担心，我们有mysql_install_db神器，哈哈哈哈哈~  



mysql_install_db对MySQL做什么
----------------------------
1. 初始化MySQL的数据目录，该数据目录在--defaults-file指定的配置文件中配置，或者不指定时默认为/etc/my.cnf（需要说明的是，这些数据目录需要执行自行创建好，并修改目录的user&group为适当权限的用户，如mysql:mysql）；  
2. 创建MySQL的系统表，即mysql和information_schema库和表（此处可能包含performance_schema,还没有研究过performance_schema,研究之后再来确定）；  
3. 初始化系统表空间和需要管理Innodb表的数据结构；  


mysql_install_db使用
--------------------
- 语法
```
mysql_install_db [options]
```
**使用说明**
1. 由于mysql_install_db在运行期间mysqld需要进入到数据目录中，所以要使用与运行mysqld相同账号运行，所以可能需要指定--user。也许还需要指定--basedir和--datadir,当不使用默认的安装目录和数据目录。
```
mysql_install_db --user=mysql --datadir=/data/mysqldata3308/mydata/
```
2. 注意mysql_install_db读取my.cnf配置文件时，只读取[mysqld]组，所以请在运行该工具之前，将[mysqlN]修改为[mysqld],运行完mysql_install_db之后，再将[mysqld]修改回[mysqldN]。  


参数选项
-------
- --basedir MySQL  
  安装目录
- --builddir 
- --cross_bootstrap  
  内核参数，暂不做解释
- --datadir MySQL  
  数据目录
- --defaults-extra-file  
  额外的配置文件，在读取配置文件之后读取该额外的配置文件
- --defaults-file  
  指定配置文件
- --force  
  运行mysql_install_db即使在DNS不工作的情况下
- --keep-my-cnf  
  运行mysql_install_db时，不创建配置文件模板my.cnf
- --ldata=path  
  MySQL数据库目录，相当于--datadir
- --no-defaults  
  不读取配置文件中的任意选项
- --random-passwords  
  为MySQL的所有root账号创建一个随机密码并设置过期时间
- --rpm  
  内核参数，暂不做解释
- --skip-name-resolve  
  创建权限表时，使用IP地址取代主机名，用于DNS不工作的情况下
- --srcdir  
  MySQL源码目录
- --user  
  运行mysqld的用户名
- --windows  
  内核参数，暂不做解释  
  除以上参数外，还支持mysqld参数。  

mysql_install_db由于主机解析不了导致的Warning的解决方法：
http://wing324.github.io/2015/10/08/mysql-install-db%E7%9A%84Warning%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/