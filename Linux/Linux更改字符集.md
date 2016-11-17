## Linux更改字符集

为了便于自己记忆，将其方法整理出来。  
参考链接：http://linux.vbird.org/linux_basic/0160startlinux.php#cmd_cmd_lang  


临时更改字符集
-------------
```shell
# 修改字符集为en_US.utf8
[root@localhost ~]# LANG=en_US.utf8
# 更改某个显示值得字符集
[root@localhost ~]# export LC_ALL=en_US.utf8
# 更改所有显示值得字符集
[root@localhost ~]# locale
# 显示当前LINUX的字符集
LANG=en_US.utf8
LC_CTYPE="en_US.utf8"
LC_NUMERIC="en_US.utf8"
LC_TIME="en_US.utf8"
LC_COLLATE="en_US.utf8"
LC_MONETARY="en_US.utf8"
LC_MESSAGES="en_US.utf8"
LC_PAPER="en_US.utf8"
LC_NAME="en_US.utf8"
LC_ADDRESS="en_US.utf8"
LC_TELEPHONE="en_US.utf8"
LC_MEASUREMENT="en_US.utf8"
LC_IDENTIFICATION="en_US.utf8"
LC_ALL=en_US.utf8
[root@localhost ~]# date
# 使用date测试当前字符集
Thu Sep 17 06:59:28 CST 2015


# 修改字符集为zh_CN.UTF-8
[root@localhost ~]# LANG=zh_CN.UTF-8
[root@localhost ~]# LC_ALL=zh_CN.UTF-8
[root@localhost ~]# locale
LANG=zh_CN.UTF-8
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=zh_CN.UTF-8
[root@localhost ~]# date
2015年 09月 17日 星期四 06:59:40 CST
```


永久更改字符集方式
----------------
```shell
# 编辑/etc/sysconfig/i18n文件，如果没有该文件可自行创建
[root@localhost /]# vi /etc/sysconfig/i18n
修改为中文添加：
LANG="zh_CN.UTF-8"
SYSFONT="latarcyrheb-sun16"
修改为英文添加：
LANG="en_US.UTF-8"
SYSFONT="latarcyrheb-sun16"

# 修改完成后，保存下文件后,source命令加载后，即可永久更改字符集
[root@localhost /] source /etc/sysconfig/i18n

```