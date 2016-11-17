## Linux修改时区

之前一直修改时区都没有效果，今天总算让我可以成功的修改了，抓紧记录下。。。



修改时区
-------
```
# 以中国为例
[root@ip-172-31-1-8 ~]# date
Wed Jan 13 04:28:50 UTC 2016

[root@ip-172-31-1-8 ~]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
cp: overwrite ‘/etc/localtime’? y

[root@ip-172-31-1-8 ~]# date
Wed Jan 13 12:31:48 CST 2016
```

