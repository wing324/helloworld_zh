## Percona-Xtrabackup安装

Percona-Xtrabackup为innodb和xtraDB的无加锁的热备份工具。



**声明**  
操作系统：CentOS6.5  
MySQL版本：MySQL5.6.26  
Percona-Xtrbackup版本：Percona-Xtrabackup2.2.12-1.el6  


Percona-Xtrbackup的优点
----------------------------------
1. 更快更可靠的备份;  
2. 备份过程中不会打断事务的执行;  
3. 节省磁盘空间和网络带宽;  
4. 自动备份验证;  
5. 由于更快的恢复时间,所以提高了数据库正常的运行时间。  


Percona-Xtrbackup安装
-------------------------------
此处选择YUM安装  
yum install http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm  
yum list | grep percona-xtrabackup.x86_64 即可查找到对应的Xtrabackup安装包   
yum install -y percona-xtrabackup.x86_64 安装percona-xtrabackup工具  


Percona-Xtrabackup相关工具
---------------------------------------  
1. innobackupex  
   备份整个MySQL数据库实例（包括myisam表,innodb表,XtraDB表）  
2. xtrabackup  
   只复制InnoDB和XtraDB的数据  
3. xbcrypt  
   用于加密解密备份文件  
4. xbstream  
   形成或提取xbstram流文件  

innobackupex和xtrabackup的详细使用,请参考该目录下的同名文件，xbcrypt和xbstream不做介绍。


