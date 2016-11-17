## keepalived的安装

由于日后在工作中会需要用到keepalived做高可用，故记录安装过程，为日后工作方便。。。


1. 创建keepalived目录  
```
mkdir -p /usr/keepalived  
```
2. 解压keepalvied安装包到keepalived目录下  
```
tar -zxvf keepalived-1.2.19.tar.gz -C /usr/keepalived/  
```
3. 安装依赖包  
```
yum install -y gcc gcc-c++
yum install -y openssl openssl-devel -y  
```
4. 编译安装keepalived  
```
./configure --prefix=/usr/keepalived  
make
make install
```
5. 设置keepalived自启动  
```
cp /usr/keepalived/sbin/keepalived /usr/sbin/
cp /usr/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
cp /usr/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
```
6. 设置keepalived的配置文件  
```
mkdir -p /etc/keepalived
cd /etc/keepalived/
vi keepalived.conf
```
