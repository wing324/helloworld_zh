## Nginx的quickstart

众所周知，IT这个行业什么都要懂一点才能成长起来，嗯，所以我就学学Nginx了，主要的目标还是通过Nginx实现方向代理。最直观的需求就是，我可以通过电脑（Windows）的浏览器，敲入虚拟机(Linux)地址，从而访问该虚拟机内容。下面仅介绍如何在Linux上安装Nginx,并学会Nginx的简单使用。  



Nginx的安装
-----------
1. 通过Nginx官网下载想要的版本（以nginx1.8.0为例）  
   http://nginx.org/  
2. 解压Nginx安装包  
   tar -zxvf nginx-1.8.0.tar.gz  
3. 进入解压后的Nginx目录  
   cd nginx-1.8.0  
4. 安装编译相关的工具  
   yum -y install gcc gcc-c++ autoconf automake  
5. 安装需要的依赖包  
- yum -y install pcre pcre-devel  
  如不安装该依赖包，会报如下错误：  
  ./configure: error: the HTTP rewrite module requires the PCRE library.
  You can either disable the module by using --without-http_rewrite_module
  option, or install the PCRE library into the system, or build the PCRE library
  statically from the source with nginx by using --with-pcre=<path> option.  
- yum install -y zlib zlib-devel  
  如不安装该依赖包，会报如下错误：  
  ./configure: error: the HTTP rewrite module requires the zlib library.
  You can either disable the module by using --without-http_rewrite_module
  option, or install the PCRE library into the system, or build the PCRE library
  statically from the source with nginx by using --with-pcre=<path> option.  
5. 进行编译安装  
   ./configure  
   make  
   make install  
   至此Nginx安装完成，进入/usr/local可查看是否存在nginx目录来确定Nginx是否安装成功。  


Nginx的启动
-----------
- 启动命令如下  
```shell
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
# Nginx执行命令 -c Nginx配置文件

#可通过如下命令查看Nginx是否启动
[root@mysql local]# ps -ef |grep nginx
root     31672     1  0 21:03 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   31673 31672  0 21:03 ?        00:00:00 nginx: worker process                                          
root     31675 26425  0 21:04 pts/2    00:00:00 grep nginx
#如上为Nginx的启动成功的显示结果
```
- 控制
  一旦Nginx启动后，可以使用-s参数来控制Nginx的可执行文件。  
```
nginx -s signal

# signal可为以下值
stop:快速关闭
quit:缓慢关闭
reload:重新加载配置文件
reopen:重新打开日志文件
```


Nginx的停止
-----------
1. 缓慢停止  
   kill -QUIT Nginx主进程号  
2. 快速停止  
   kill -TERM Nginx主进程号  
   或者  
   kill -INT Nginx主进程号
3. 强制停止  
   pkill -9 nginx  
```
#缓慢停止的实例
[root@mysql local]# ps -ef | grep nginx
root     31679     1  0 21:10 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   31680 31679  0 21:10 ?        00:00:00 nginx: worker process                                          
root     31694 26425  0 21:12 pts/2    00:00:00 grep nginx

[root@mysql local]# kill -QUIT 31679

[root@mysql local]# ps -ef | grep nginx
root     31696 26425  0 21:12 pts/2    00:00:00 grep nginx
```


Nginx验证配置文件正确性
---------------------
- 方法一  
```shell
[root@mysql local]# cd /usr/local/nginx/sbin/

# 使用-t参数，可看到ok,即配置文件是正确的
[root@mysql sbin]# ./nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

```
- 方法二  
```shell
# 注意此处-t参数需要在-c参数前面
[root@mysql sbin]# /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```


Nginx的重启
----------
- 方法一  
```shell
[root@mysql sbin]# cd /usr/local/nginx/sbin/
[root@mysql sbin]# ./nginx -s reload
```

- 方法二  
```shell
[root@mysql sbin]# ps -ef | grep nginx
root     31711     1  0 21:20 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   31716 31711  0 21:21 ?        00:00:00 nginx: worker process                                          
root     31718 26425  0 21:22 pts/2    00:00:00 grep nginx

# 通过发送信号量的方式，kill -HUP Nginx主进程号
[root@mysql sbin]# kill -HUP 31711
```

说明
----
若想通过浏览器访问Nginx,需要考虑将端口添加到防火墙或者关闭防火墙。  
##### 关闭防火墙
- centos7之前  
  service iptables stop  
- centos7之后  
  service firewalld stop
  或者
  systemctl stop firewalld.service
  禁止firewalld开机自启
  systemctl disable firewalld.service  

