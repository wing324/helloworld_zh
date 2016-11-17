## mysql_install_db的Warning解决方法

每次在使用mysql_install_db的时候都会有一个恼人的Waring:
WARNING: The host 'mysql' could not be looked up with /usr/bin/resolveip.
在强迫症的眼里，Warning就是个Error啊！

参考链接：
http://yangrong083.blog.163.com/blog/static/113406097201526101243232/

Warning如下：
WARNING: The host 'mysql' could not be looked up with /usr/bin/resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !

Warning的原因是主机名host不能被解析。  

解决方法：
- cat /etc/hosts
  此处可以发现并没有对应的ip：主机名
```
[root@mysql mysqldata3308]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```
- vi /etc/hosts添加对应的ip:主机名
```
[root@mysql mysqldata3308]# vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.200.143 mysql
```
- 使用resoveip解析下主机名
```
[root@mysql mysqldata3308]# /usr/bin/resolveip mysql
IP address of mysql is 192.168.200.143
# 此时看到完美解析
```

此时再次使用mysql_install_db将不再出现该Warning，至少我已经成功解决了~  

mysql_install_db工具的使用：
http://wing324.github.io/2015/10/08/mysql-install-db%E5%88%9D%E5%A7%8B%E5%8C%96%E6%95%B0%E6%8D%AE%E5%BA%93%E5%B7%A5%E5%85%B7/