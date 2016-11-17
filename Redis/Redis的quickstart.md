## Redis的quickstart

NoSQL现在真是如火如荼，听的最多的局势Redis了，而我并不了解这个东西，但是有很多很多人都在用，所以我决定学习学习，看看这个东东好在哪里。。。




Redis安装
--------
参考链接：  
http://redis.io/download  
操作系统：CentOS 7  
Redis版本：3.0.5  

1. 在参考链接中获得Redis安装包，或者通过如下方式获得：  
   wget http://download.redis.io/releases/redis-3.0.5.tar.gz  
2. 解压软件至指定目录  
   mkdir -p /usr/local/redis  
   tar -zxvf redis-3.0.5.tar.gz -C /usr/local/redis/  
3. 编译安装包  
   cd /usr/local/redis/redis-3.0.5  
   make  
4. 启动Redis服务  
   /usr/local/redis/redis-3.0.5/src/redis-server &  
5. 验证Redis启动  
   /usr/local/redis/redis-3.0.5/src/redis-cli  
   127.0.0.1:6379> keys *  
   (empty list or set)  
6. 推出Redis  
   ctrl+D  



Redis启动
---------
- 直接启动  
  redis-server &  

- 指定配置文件启动  
  redis-server /usr/local/redis/etc/redis.conf  
```shell
# 修改配置文件redis.conf设置redis为后台启动
# 将redis.conf文件中daemonize no修改为：
daemonize yes

# 启动Redis
[root@localhost etc]# redis-server /usr/local/redis/etc/redis.conf 
[root@localhost etc]# ps -ef | grep redis
root       2907      1  0 01:45 ?        00:00:00 redis-server *:6379
root       2911   2827  0 01:45 pts/0    00:00:00 grep --color=auto redis
```

- 使用Redis自启动脚本
  Redis的启动脚本位于redis-3.0.5/utils/redis_init_script下  
  **启动Redis自启动脚本步骤**  
  1.新建目录 /etc/redis用来存放Redis的配置文件  
  2.复制redis.conf到/etc/redis目录下并命名为6379.conf  
  3.修改6379.conf配置文件  
  4.复制redis_init_script脚本文件到/etc/init.d目录中，并命名为redis  
  5.执行随系统自动启动命令  
  chmod a+x /etc/init.d/redis  
  service redis start  



Redis停止
---------
1. 使用Ctrl+C  
2. 在客户端执行SHUTDOWN  
3. kill -9 *PID*  



Redis配置文件
------------
Redis有两种配置文件，redis.conf文件用于redis server的配置文件，sentinel.conf用于redis sentinel配置文件，用于监控。更详细的内容请看未来更新的文章~
