## Nginx安装模块

记一次Nginx出现的错误，登录浏览器，出现如下错误：  
SSL received a record that exceeded the maximum permissible length.  


出现问题如上，所以开始google造成这个问题的原因，然后google告诉我需要安装http_ssl_module模块，可是作为Nginx小白的我连模块都不会安装，于是继续google，找到了如何安装模块，就是需要重新编译。。  

```
# http_ssl_module模块需要以来于OpenSSL,所以找到OpenSSL的压缩包，然后解压到OpenSSL目录下即可。
./configure --with-http_ssl_module --with-openssl=/usr/local/openssl/openssl-1.0.2d
make
make install

# 通过-V参数可看到安装了相应的模块
[root@wing nginx]# ./sbin/nginx -V
nginx version: nginx/1.8.0
built by gcc 4.8.3 20140911 (Red Hat 4.8.3-9) (GCC) 
built with OpenSSL 1.0.2d 9 Jul 2015
TLS SNI support enabled
configure arguments: --with-http_ssl_module --with-openssl=/usr/local/openssl/openssl-1.0.2d
```
于是问题解决了。。。。  
