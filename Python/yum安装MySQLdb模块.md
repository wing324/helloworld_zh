## yum安装MySQLdb模块

在我使用的python2.7版本中，MySQLdb模块还不是python的内置模块，但是MySQLdb模块又是Python与MySQL连接的桥梁，对于作为MySQL DBA又很喜欢Python语言的我来说，MySQLdb真的是必需品呢。



1. MySQLdb依赖于mysql-devel包，所以首先我们需要先安装mysql-devel包  
   可以去官网下载mysqldevel的rpm包，然后安装在服务器上。  

2. 直接用yum安装MySQLdb  
   yum install -y MySQLdb-python  

3. 检验MySQLdb模块是否安装成功  
```
[root@ip-172-31-1-8 ~]# python
Python 2.7.10 (default, Dec  8 2015, 18:25:23) 
[GCC 4.8.3 20140911 (Red Hat 4.8.3-9)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import MySQLdb
>>> 

# import之后不出现任何提示或者错误，即使MySQLdb模块包安装成功

```