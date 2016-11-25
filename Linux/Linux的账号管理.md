## Linux的账号管理

记忆力不好，对于Linux最基本的账号管理都记不住，只能默默的写下来，此处列出常用的让自己记住。。。


查看用户相关信息
--------------
- /etc/passwd查看用户，/etc/shadow查看密码，/etc/group查看组
- id命令查看自己或他人相关的UID/GID信息
```shell
# id命令看起来有点像这样
[test@localhost ~]$ id test1
uid=1001(test1) gid=1001(test1) 组=1001(test1)
[test@localhost ~]$ id test
uid=1000(test) gid=1000(test) 组=1000(test)
[test@localhost ~]$ id root
uid=0(root) gid=0(root) 组=0(root)
# 可以直接通过id查看当前用户的UID/GID
[test@localhost ~]$ id
uid=1000(test) gid=1000(test) 组=1000(test) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```


新增用户
-------
```shell
# 只添加用户
useradd test1

# -g表示初始化的组，-G表示还可以加入的组
[root@localhost ~]# useradd test2 -g test -G test2

# useradd添加的账户是没有密码的，所以我们需要用passwd命令，修改账户密码
[root@localhost ~]# passwd test2		
更改用户 test2 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```


修改用户
-------
```shell
# 为账户test2修改注释，接着可在/etc/passwd文件中查看相应的修改
[root@localhost ~]# usermod -c 'just test' test2
该语句执行前：test2:x:1002:1000::/home/test2:/bin/bash
该语句执行后：test2:x:1002:1000:just test:/home/test2:/bin/bash

# 修改账户test2的初始化组，此处test1组的GID为1001
[root@localhost ~]# usermod -g test1 test2
该语句执行前：test2:x:1002:1000:just test:/home/test2:/bin/bash
该语句执行后：test2:x:1002:1001:just test:/home/test2:/bin/bash

# 修改账户test2的次要组
[root@localhost ~]# usermod -G test test2
```


删除用户
-------
```shell
# 仅删除test2用户
[root@localhost ~]# userdel test2

# 删除test2用户及其home
[root@localhost ~]# userdel -r test2
```
使用userdel需要谨慎，因为也许可能你存在文件的user是test2用户创建的，如果删除test2用户，用户名就会变成一串数字。  


新增用户组
---------
```shell
# 只添加用户组
[root@localhost ~]# groupadd group1

# 创建用户组时，指定GID
[root@localhost ~]# groupadd -g 324 group2
```


修改用户组
---------
```shell
# 更改用户组的GID 
[root@localhost ~]# groupmod -g 125 group2
```


删除用户组
---------
```shell
# 删除用户组
[root@localhost ~]# groupdel group2

# 为什么这个用户组删除不了呢
[root@localhost ~]# groupdel test1
groupdel：不能移除用户“test1”的主组
# 该用户组删除不了的原因为，它为某个账号的初始化组即initial group,所以此时删除不了，此时可将该账号的GID修改为其他或者删除该账号
[root@localhost ~]# groupdel test1
groupdel：不能移除用户“test1”的主组
[root@localhost ~]# usermod -g 1000 test1
[root@localhost ~]# groupdel test1
```


组管理员
-------
```shell
# 建立群组
[root@localhost ~]# groupadd testgroup
# 为该群组添加密码
[root@localhost ~]# gpasswd testgroup
正在修改 testgroup 组的密码
新密码：
请重新输入新密码：
# 将test设为testgroup的组管理员
[root@localhost ~]# gpasswd -A test testgroup

#以test登陆，将自己和test2加入到testgroup群组中
[test@localhost ~]$ gpasswd -a test testgroup
正在将用户“test”加入到“testgroup”组中
[test@localhost ~]$ gpasswd -a test2 testgroup
正在将用户“test2”加入到“testgroup”组中
```