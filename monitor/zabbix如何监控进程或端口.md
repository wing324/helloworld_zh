## zabbix如何监控进程或端口

前几天生产环境有个job没有起来，结果测试人员一直以为是个bug,后来开发人员觉得这块没有bug逻辑啊，怎么回事啊。。终于，经过很长时间的排查，发现有个job快一天没有启动了。。。原因是:前一天晚上新版本上线，然后忘记启动这个job了。。。。这个尴尬的事情，终于让人明白对于进程或者端口的监控是多么的重要~~~本来想写个python脚本来监控的，后来上网一看，哇塞，zabbix好牛逼哦，用zabbix简单的配置下就能搞定了，何必重复造轮子，我写的估计性能也赶不上zabbix,于是我果断选择了用zabbix来监控进程或端口。。。  
<!-- more -->


首先，zabbix的安装部署我就不讲了，将来我会整理一份zabbix的安装部署出来好了。  
其次，无论监控进程还是端口，都需要在对应的机器上部署一个zabbix_agented。  
最后，我的前提是zabbix已经正常监听了进程/端口对应的host。  
接下来，就分别说说如何用zabbix监控进程以及监控端口。  

本文只讲到如何触发trigger,至于trigger触发后如何报警，请参考：  
[http://http://wing324.github.io/2016/05/28/zabbix%E4%BD%BF%E7%94%A8python%E8%84%9A%E6%9C%AC%E5%8F%91%E9%80%81%E6%8A%A5%E8%AD%A6%E9%82%AE%E4%BB%B6/](http://http://wing324.github.io/2016/05/28/zabbix%E4%BD%BF%E7%94%A8python%E8%84%9A%E6%9C%AC%E5%8F%91%E9%80%81%E6%8A%A5%E8%AD%A6%E9%82%AE%E4%BB%B6/ "zabbix使用python脚本发送报警邮件")

**zabbix版本:3.0**  


zabbix监控进程
-------------
##### 使用zabbix的item中的key来做进程监控。  

> 监控进程的key,<cmdline>是模糊匹配
> proc.num[<name>,<user>,<state>,<cmdline>]
>
> key对应的释义
> The number of processes. Returns integer
> 统计指定process的总数
>
> 示例中的key值配置
> proc.num[,mysql,all,/usr/sbin/mysqld --basedir=/usr --datadir=/data/mysql/mysqldata3307/mydata]

zabbix中item的配置如下：  
![](http://i.imgur.com/gR16yUe.png)


##### 报警设置当然用zabbix的trigger
相应的trigger对应的Expression如下:
{test_agentd:proc.num[,mysql,all,/usr/sbin/mysqld --basedir=/usr --datadir=/data/mysql/mysqldata3307/mydata].last(,0)}=0

zabbix中trigger的配置如下：
![](http://i.imgur.com/4q13j5I.png)


zabbix监控端口
-------------
##### 使用zabbix的item中的key来做端口

> 监控进程的key
> net.tcp.listen[port]	
>
> key对应的释义
> Checks if this TCP port is in LISTEN state. Returns 0 - it is not in LISTEN state; 1 - it is in LISTEN state
> 检测tcp端口的状态，监听中返回1，未监听返回0
>
> 示例中的key值配置
> net.tcp.listen[3307]

zabbix中item的配置如下：  
![](http://i.imgur.com/tWJXqly.png)

##### 报警设置当然也用zabbix的trigger
相应的trigger对应的Expression如下:  
{test_agentd:net.tcp.listen[3307].last(0)}=0  

zabbix中trigger的配置如下：  
![](http://i.imgur.com/jjtZZeR.png)

Tips：
加上zabbix邮件报警，你就可以停掉对应的进程或者端口来检测是否达到预期期望，即触发了trigger，而后发送报警邮件。  