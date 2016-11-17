## MySQL实时监控sql工具之orztop

orztop是一款实时show full processlist的工具，我们可以实时看到数据库有哪些线程，执行哪些语句等。工具使用方便简单。解决了我们需要手动刷新show full processlist的痛苦。  



orztop结果图
-----------
此处我正在对我的mysql使用tpcc-mysql工具进行压测，此时orztop得到的结果图如下：  
![](http://i.imgur.com/x0eWUFM.png)


orztop命令行
-----------
orztop命令行均为较简单的英文，这些英文的阅读能力是作为一个IT人员的必备专业技能。。  
```
[root@wing orztop]# ./orztop --help
==========================================================================================
Info  :
                Created By zhuxu@taobao.com
Usage :
Command line options :

        -help       Print Help Info. 
        -h,--host   Hostname/Ip to use for mysql connection.
        -u,--user   User        to use for mysql connection.
        -p,--pwd    Password    to use for mysql connection.
        -P,--port   Port        to use for mysql connection(default 3306). 

        -S,--socket Socket      to use for mysql connection.

        -t          Time(second) Interval.  

==========================================================================================

```