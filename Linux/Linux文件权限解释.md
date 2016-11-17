## Linux文件权限解释

Linux下如何修改权限及权限的含义，比起windows，Linux修改权限还真是方便，比windows方便多了，不过是我太笨了，不会用windows命令。。



查看文件：ls -al  

```
-rw-r--r--.  1 root root        0 9月  18 06:59 test.txt
# 前9个字符则代表权限，user/group/others(3/3/3)
```


修改权限命令
-----------------

- chgrp 修改文件所属组  
  注：group存在于/etc/group文件中  

```shell
# 修改文件所属组
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 root root        0 9月  18 06:59 test.txt
[root@localhost ~]# chgrp test test.txt 
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 root test        0 9月  18 06:59 test.txt
```

- chown 修改文件拥有者  
  注：用户存在于/etc/passwd文件中  

```shell
# 修改文件用户
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 root test        0 9月  18 06:59 test.txt
[root@localhost ~]# chown test test.txt 
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 test test        0 9月  18 06:59 test.txt

# 修改文件用户和用户组
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 test test        0 9月  18 06:59 test.txt
[root@localhost ~]# chown root.root test.txt 
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 root root        0 9月  18 06:59 test.txt
```

- chmod 修改文件权限  

```shell
# 修改文件权限
[root@localhost ~]# ls -l | grep test.txt 
-rw-r--r--. 1 root root        0 9月  18 06:59 test.txt
[root@localhost ~]# chmod 777 test.txt 
[root@localhost ~]# ls -l | grep test.txt 
-rwxrwxrwx. 1 root root        0 9月  18 06:59 test.txt
```


r/w/x权限
---------
##### 含义
r(4):可读  
w(2):可写  
x(1):可执行  
-：无权限  

##### 对于文件
r4：可读，可读取该文件内容  
w2：可写，可编辑/增加/修改该文件内容，但不包含删除文件  
x1：可执行，该文件具有被系统执行的权限  

##### 对于目录
r4:读取目录下的文件  
w2:增删改查该目录及该目录下的文件  
x1:拥有cd到该目录下的权限  






