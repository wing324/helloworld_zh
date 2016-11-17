## Nginx的配置文件

前面写了一个Nginx的quickstart,现在来江江Nginx的配置文件nginx.conf文件，一个程序没了配置文件是多么坑爹的事情，还好Nginx是个友好的程序。




配置文件结构
-----------
- 配置文件中的指令包括简单指令和指令块  
  简单指令：  
```
# 名称 参数；
worker_processes  1;
```
指令块：  
```
# 有{}的一系列简单指令

        location / {
            root   /usr/local/nginx/html/host1;
            index  index.html index.htm;
        }

```
- events和http在main指令块内，server在http指令块内，location在server指令块内  
- `#`是注释符号  