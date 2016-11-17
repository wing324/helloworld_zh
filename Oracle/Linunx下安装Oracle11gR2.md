## Linunx下安装Oracle11gR2


本文档参考Oracle官方文档以及老相Oracle视频。  
建议Oracle安装过程在Xmanager的图形界面以及终端下完成，因为后期oracle安装过程中的弹出框会使用到图形界面。  


一、安装Xmanager
---------------
参考文档：  
http://wenku.baidu.com/link?url=nDEe4RcMgd6-EVTgoyS9_eH13EdwIrskFzkjvQxprY6MPrgnTfeQ7RfhiPJZ4o6AOHz6Z3q7K4T03X1w9MgV3atm-cfFE7xoQEqYHsg-m2q  
##### 1.安装gdm包  
```shell
[root@wing ~]# yum install -y gdm
```
##### 2.打开/etc/inittab ,到最后一行，查看是否为 id：5：initdefault  
```shell
[root@wing ~]# vi /etc/inittab 
```
##### 3.编辑/etc/gdm/customer.conf文件
`vi /etc/gdm/customer.conf`
```shell
我的/etc/gdm/customer.conf文件内容如下
# GDM configuration storage

[daemon]

[security]
AllowRemoteRoot=true
[xdmcp]
Port=177
Enable=1
[greeter]

[chooser]

[debug]
```
##### 4.在防火墙中开启177端口
编辑/etc/sysconfig/iptables文件:  
```shell
vi /etc/sysconfig/iptables
```
在-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT后面添加两行如下内容:  
```shell
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6000:6010 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 177 -j ACCEPT
```
##### 5.重新启动linux
```shell
reboot
```
linux重启后，进入Xmanager里面的Xbrowser里面，即可看到你希望看到的那台linux。


二、安装Oracle之前的准备
----------------------
##### 1.以root用户登录进入Linux中
**cmd:** `su - root`  

##### 2.检查硬件要求
###### 内存
最小内存1G  
推荐内存2G  
**cmd:** `grep MemTotal /proc/meminfo`  
###### swap
是物理内存的2倍  
**cmd:** `grep SwapTotal /proc/meminfo`  
###### /tmp
生产环境中需要至少10G  
**cmd:** `df -k /tmp/`  
###### 操作系统位数
必须64位  
**cmd:** `uname -m`  
###### 操作系统空间
软件至少需要4.7G  
数据文件磁盘至少需要1.7G  
**cmd：** `df -h disk_name`  

##### 3.检查软件要求
###### 操作系统内核
```
# On Oracle Linux 4 and Red Hat Enterprise Linux 4
2.6.9 or later
# On Oracle Linux 5 Update 2 with Red Hat Compatible Kernel
2.6.18 or later
# On Oracle Linux 5 Update 5 with Red Hat Compatible Kernel
2.6.18 or later
# On Oracle Linux 5 Update 5 with Unbreakable Enterprise Kernel
2.6.32-100.0.19 or later
# On Oracle Linux 6
2.6.32-100.28.5.el6.x86_64 or later
# On Oracle Linux 6 with Red Hat Compatible Kernel
# Note: Only the distributions and versions listed in the earlier list are
# supported. Do not install the software on other versions of Linux.
# 8
2.6.32-71.el6.x86_64 or later
# On Oracle Linux 7
3.8.13-33.el7uek.x86_64 or later
# On Oracle Linux 7 with Red Hat Compatible Kernel
3.10.0-54.0.1.el7.x86_64 or later
# On Red Hat Enterprise Linux 5 Update 2
2.6.18 or later
# On Red Hat Enterprise Linux 5 Update 5
2.6.18 or later
# On Red Hat Enterprise Linux 6
2.6.32-71.el6.x86_64 or later
# On Red Hat Enterprise Linux 7
3.10.0-54.0.1.el7.x86_64 or later
# On Asianux Server 3
2.6.18 or later
# On Asianux Server 4
2.6.32-71.el6.x86_64 or later
# On SUSE Linux Enterprise Server 10
2.6.16.21 or later
# On SUSE Linux Enterprise Server 11
2.6.27.19 or later
```
**cmd:**  `uname -r`  
###### 安装包需求
```
# The following or later version of packages for Oracle Linux 4 and Red Hat
# Enterprise Linux 4 must be installed:
binutils-2.15.92.0.2
compat-libstdc++-33-3.2.3
compat-libstdc++-33-3.2.3 (32 bit)
elfutils-libelf-0.97
elfutils-libelf-devel-0.97
expat-1.95.7
gcc-3.4.6
gcc-c++-3.4.6
glibc-2.3.4-2.41
glibc-2.3.4-2.41 (32 bit)
glibc-common-2.3.4
glibc-devel-2.3.4
glibc-headers-2.3.4
libaio-0.3.105
libaio-0.3.105 (32 bit)
libaio-devel-0.3.105
libaio-devel-0.3.105 (32 bit)
libgcc-3.4.6
libgcc-3.4.6 (32-bit)
libstdc++-3.4.6
libstdc++-3.4.6 (32 bit)
libstdc++-devel 3.4.6
make-3.80
numactl-0.6.4.x86_64
pdksh-5.2.14
sysstat-5.0.5

# The following or later version of packages for Oracle Linux 5, Red Hat Enterprise
# Linux 5, and Asianux Server 3 must be installed:
binutils-2.17.50.0.6
compat-libstdc++-33-3.2.3
compat-libstdc++-33-3.2.3 (32 bit)
elfutils-libelf-0.125
elfutils-libelf-devel-0.125
gcc-4.1.2
gcc-c++-4.1.2
glibc-2.5-24
glibc-2.5-24 (32 bit)
glibc-common-2.5
glibc-devel-2.5
glibc-devel-2.5 (32 bit)
glibc-headers-2.5
ksh-20060214
libaio-0.3.106
libaio-0.3.106 (32 bit)
libaio-devel-0.3.106
libaio-devel-0.3.106 (32 bit)
libgcc-4.1.2
libgcc-4.1.2 (32 bit)
libstdc++-4.1.2
libstdc++-4.1.2 (32 bit)
libstdc++-devel 4.1.2
make-3.81
sysstat-7.0.2
# The following or later version of packages for Oracle Linux 6, Red Hat Enterprise
# Linux 6, and Asianux Server 4 must be installed:
binutils-2.20.51.0.2-5.11.el6 (x86_64)
compat-libcap1-1.10-1 (x86_64)
compat-libstdc++-33-3.2.3-69.el6 (x86_64)
compat-libstdc++-33-3.2.3-69.el6.i686
gcc-4.4.4-13.el6 (x86_64)
gcc-c++-4.4.4-13.el6 (x86_64)
glibc-2.12-1.7.el6 (i686)
glibc-2.12-1.7.el6 (x86_64)
glibc-devel-2.12-1.7.el6 (x86_64)
glibc-devel-2.12-1.7.el6.i686
ksh
libgcc-4.4.4-13.el6 (i686)
libgcc-4.4.4-13.el6 (x86_64)
libstdc++-4.4.4-13.el6 (x86_64)
libstdc++-4.4.4-13.el6.i686
libstdc++-devel-4.4.4-13.el6 (x86_64)
libstdc++-devel-4.4.4-13.el6.i686
libaio-0.3.107-10.el6 (x86_64)
libaio-0.3.107-10.el6.i686
libaio-devel-0.3.107-10.el6 (x86_64)
libaio-devel-0.3.107-10.el6.i686
make-3.81-19.el6
sysstat-9.0.4-11.el6 (x86_64)
# The following or later version of packages for Oracle Linux 7, and Red Hat
# Enterprise Linux 7 must be installed:
binutils-2.23.52.0.1-12.el7.x86_64
compat-libcap1-1.10-3.el7.x86_64
gcc-4.8.2-3.el7.x86_64
gcc-c++-4.8.2-3.el7.x86_64
glibc-2.17-36.el7.i686
glibc-2.17-36.el7.x86_64
glibc-devel-2.17-36.el7.i686
glibc-devel-2.17-36.el7.x86_64
ksh
libaio-0.3.109-9.el7.i686
libaio-0.3.109-9.el7.x86_64
libaio-devel-0.3.109-9.el7.i686
libaio-devel-0.3.109-9.el7.x86_64
libgcc-4.8.2-3.el7.i686
libgcc-4.8.2-3.el7.x86_64
libstdc++-4.8.2-3.el7.i686
libstdc++-4.8.2-3.el7.x86_64
libstdc++-devel-4.8.2-3.el7.i686
libstdc++-devel-4.8.2-3.el7.x86_64
libXi-1.7.2-1.el7.i686
libXi-1.7.2-1.el7.x86_64
libXtst-1.2.2-1.el7.i686
libXtst-1.2.2-1.el7.x86_64
make-3.82-19.el7.x86_64
sysstat-10.1.5-1.el7.x86_64
# The following or later version of packages for SUSE Linux Enterprise Server 10
# must be installed:
binutils-2.16.91.0.5
compat-libstdc++-5.0.7
gcc-4.1.0
gcc-c++-4.1.2
glibc-2.4-31.63
glibc-devel-2.4-31.63
glibc-devel-32bit-2.4-31.63
ksh-93r-12.9
libaio-0.3.104
libaio-32bit-0.3.104
libaio-devel-0.3.104
libaio-devel-32bit-0.3.104
libelf-0.8.5
libgcc-4.1.2
libstdc++-4.1.2
libstdc++-devel-4.1.2
make-3.80
numactl-0.9.6.x86_64
sysstat-8.0.4
# The following or later version of packages for SUSE Linux Enterprise Server 11
# must be installed:
binutils-2.19
gcc-4.3
gcc-32bit-4.3
gcc-c++-4.3
glibc-2.9
glibc-32bit-2.9
glibc-devel-2.9
glibc-devel-32bit-2.9
ksh-93t
libaio-0.3.104
libaio-32bit-0.3.104
libaio-devel-0.3.104
libaio-devel-32bit-0.3.104
libstdc++33-3.3.3
libstdc++33-32bit-3.3.3
libstdc++43-4.3.3_20081022
libstdc++43-32bit-4.3.3_20081022
libstdc++43-devel-4.3.3_20081022
libstdc++43-devel-32bit-4.3.3_20081022
libgcc43-4.3.3_20081022
libstdc++-devel-4.3
make-3.81
sysstat-8.1.5
```
**cmd:** `rpm -q package_name`  
**安装所有包cmd:** `yum install -y binutils compat-libcap1-1.10-1 compat-libstdc++-33-3.2.3-69.el6 compat-libstdc++-33-3.2.3-69.el6.i686 gcc gcc-c++ glibc glibc-devel ksh libgcc-4.4.4-13.el6 libgcc libstdc++ libstdc++-devel libaio-0.3.107-10.el6 libaio-0.3.107-10.el6.i686 libaio-devel-0.3.107-10.el6 libaio-devel-0.3.107-10.el6.i686 make sysstat elfutils-libelf-devel`  

##### 4.配置网络
编辑/etc/sysconfig/network-scripts/ifcfg-eth0文件  
**cmd:** `vi /etc/sysconfig/network-scripts/ifcfg-eth0`  
```shell
# 我的配置文件
DEVICE=eth0
HWADDR=00:0C:29:F9:44:18
TYPE=Ethernet
UUID=e89b278a-4312-4581-8d80-b3587da19008
ONBOOT=yes
BOOTPROTO=static
BROADCAST=192.168.98.255
IPADDR=192.168.98.128
NETMASK=255.255.255.0
NETWORK=192.168.98.0

# 修改完配置后，记得重启网络哦
service network restart
```

##### 5.配置主机名
一旦Oracle安装之后，主机名不可变动哦。  
**cmd:** `vi /etc/hosts`  
```shell
# 我的配置文件
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.98.128 wing
```

##### 6.创建操作系统的用户和组
	1) 确认oinstall用户组是否存在：`more /etc/oraInst.loc`，如果oinstall用户组存在，则存在该文件，如果oinstall用户组不存在，则不存在该文件。  
	​```
	[root@wing ~]# more /etc/oraInst.loc 
	inventory_loc=/u01/app/oraInventory
	inst_group=oinstall
	​```
	2) 确认dba用户组存在：`grep dba /etc/group`  
	3) 确认oracle用户存在，并且属于正确的用户组：`id oracle`  
	​```
	# 正确输出如下：
	[root@wing ~]# id oracle
	uid=500(oracle) gid=500(oinstall) groups=500(oinstall),501(dba)
	​```
**创建cmd:**  
```shell
# 添加oinstall用户组
groupadd oinstall
# 添加dba用户组
groupadd dba
# 添加oracle用户，并归属于正确的用户组
useradd -g oinstall -G dba oracle
# 如果oracle用户已存在，修改其用户组
usermod -g oinstall -G dba oracle
# 为oracle用户加密
passwd oracle
```

##### 7.调整Linux参数
###### 调整系统参数
**查看参数值cmd:** `sysctl -a | grep 参数名称`  
**建议参数调整cmd:** `vi /etc/sysctl.conf` 
```
# 添加如下参数
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 980271104
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576

# 使参数生效
sysctl -p
```
###### 调整limits参数
**cmd:** `vi /etc/security/limits.conf`  
```shell
# 添加如下参数
oracle          soft    nproc   2047
oracle          hard    nproc   16384
oracle          soft    nofile  1024
oracle          hard    nofile  65536
```
###### 调整login参数
**cmd:** `vi /etc/pam.d/login`  
```shell
# 添加如下参数
session    required     pam_limits.so
```
###### 调整/etc/profile参数
**cmd:** `vi /etc/profile`  
```shell
# 添加如下参数
if [ $USER = "oracle" ]; then
        if [ $SHELL = "/bin/ksh" ]; then
                ulimit -p 16384
                ulimit -n 65536
        else
                ulimit -u 16384 -n 65536
        fi
fi
```

##### 8.为Oracle软件建立目录
此处假设Oracle软件目录为 /u01/app/oracle  
**cmd：**  
```shell
# 建立目录
mkdir -p /u01/app/oracle
# 修改目录用户和组
chown -R oracle:oinstall /u01/app/
# 修改目录权限
chmod -R 755 /u01/app/
```

##### 9.配置Oracle用户环境变量
**cmd:**  
```shell
# 切换至oracle用户下
su - oracle
# 编辑环境变量文件
vi .bash_profile
# 添加如下环境变量
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1
export ORACLE_SID=wingorcl
export NLS_LANG=american_america.AL32UTF8
export PATH=$PATH:$ORACLE_HOME/bin:.
# 此刻需要重新切换至oracle用户才能生效
```

##### 10.解压oracle11gR2软件至相应目录中
**cmd：**  
```shell
unzip linux.x64_11gR2_database_1of2.zip -d /oraapp/
unzip linux.x64_11gR2_database_2of2.zip -d /oraapp/
```
此时进入*/oraapp*目录下，可以看到database目录，即为我们所需要的Oracle11gR2安装文件  

##### 11.安装前需要在界面操作的terminal中做如下操作
```
[root@wing database]# w
 11:32:46 up  2:29,  2 users,  load average: 0.05, 0.11, 0.06
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1     :0               09:05    2:29m  3.38s  3.38s /usr/bin/Xorg :0 -nr -verbose -audit 4 -auth /var/run/gdm/auth-for-gdm-tBSNYD/database -nolisten tcp vt1
root     pts/1    192.168.98.1     10:27    0.00s  0.26s  0.01s w

[root@wing database]# su - oracle
[oracle@wing database]# export DISPLAY=192.168.98.1:0.0
# 执行xhost +，显示如下命令即可
[oracle@wing ~]# xhost +
access control disabled, clients can connect from any host
```


三、安装Oracle
-------------
##### 1.执行Oracle安装脚本
```shell
[root@wing database]# su - oracle
[oracle@wing ~]$ cd /oraapp/database/
# 使用oracle用户执行安装脚本
[oracle@wing database]$ ./runInstaller
```

##### 2.使用Xmanager连接主机接收图形界面（图片省略）

##### 3.按照图形界面提示执，最后需要以root用户手动执行两个脚本
```shell
sh /u01/app/oraInventory/orainstRoot.sh
sh /u01/app/oracle/product/11.2.0/db_1/root.sh
```


四、在Oracle下建库
-----------------
##### 1.使用netca配置监听
```
[oracle@wing ~]$ xhost +
access control disabled, clients can connect from any host
[oracle@wing ~]$ dbca
[oracle@wing ~]$ netca
# 此处出现图形界面进行配置，配置成功后，输出结果如下
Oracle Net Services Configuration:
Configuring Listener:WING_LISTENER
Listener configuration complete.
Oracle Net Listener Startup:
    Running Listener Control: 
      /u01/app/oracle/product/11.2.0/db_1/bin/lsnrctl start WING_LISTENER
    Listener Control complete.
    Listener started successfully.
Oracle Net Services configuration successful. The exit code is 0
```

##### 2.使用dbca建立数据库
```shell
[root@wing ~]# su - oracle
[oracle@wing ~]$ dbca
# 此时出现图形界面，根据界面提示完成oracle数据库建立

```


五、Oracle常用软件的启动与关闭
---------------------------
##### Oracle启动
listener-->Oracle-->EM
```
##### listener
# listener启动
[oracle@wing ~]$ lsnrctl start

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 11-FEB-2016 19:25:02

Copyright (c) 1991, 2013, Oracle.  All rights reserved.

Starting /u01/app/oracle/product/11.2.0/db_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 11.2.0.4.0 - Production
System parameter file is /u01/app/oracle/product/11.2.0/db_1/network/admin/listener.ora
Log messages written to /u01/app/oracle/diag/tnslsnr/wing/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=wing)(PORT=1521)))

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date                11-FEB-2016 19:25:03
Uptime                    0 days 0 hr. 0 min. 20 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/11.2.0/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/wing/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=wing)(PORT=1521)))
The listener supports no services
The command completed successfully
# listener状态查看
[oracle@wing ~]$ lsnrctl status

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 11-FEB-2016 19:26:44

Copyright (c) 1991, 2013, Oracle.  All rights reserved.

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.4.0 - Production
Start Date                11-FEB-2016 19:25:03
Uptime                    0 days 0 hr. 1 min. 41 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/11.2.0/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/wing/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=wing)(PORT=1521)))
The listener supports no services
The command completed successfully
# listener端口监听查看
[oracle@wing ~]$ netstat -npl | grep 1521
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 :::1521                     :::*                        LISTEN      17985/tnslsnr

##### Oracle
# Oracle启动
[oracle@wing ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Thu Feb 11 19:28:21 2016

Copyright (c) 1982, 2013, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup
ORACLE instance started.

Total System Global Area  780824576 bytes
Fixed Size                  2257312 bytes
Variable Size             511708768 bytes
Database Buffers          264241152 bytes
Redo Buffers                2617344 bytes
Database mounted.
Database opened.
# Oracle状态查看
[root@wing ~]# ps -ef | grep oracle
root     13275 13254  0 18:46 pts/1    00:00:00 su - oracle
oracle   13276 13275  0 18:46 pts/1    00:00:00 -bash
oracle   17985     1  0 19:25 ?        00:00:00 /u01/app/oracle/product/11.2.0/db_1/bin/tnslsnr LISTENER -inherit
oracle   18099     1  0 19:29 ?        00:00:00 ora_pmon_wingdb
oracle   18101     1  0 19:29 ?        00:00:00 ora_psp0_wingdb
oracle   18103     1  7 19:29 ?        00:00:02 ora_vktm_wingdb
oracle   18107     1  0 19:29 ?        00:00:00 ora_gen0_wingdb
oracle   18109     1  0 19:29 ?        00:00:00 ora_diag_wingdb
oracle   18111     1  0 19:29 ?        00:00:00 ora_dbrm_wingdb
oracle   18113     1  0 19:29 ?        00:00:00 ora_dia0_wingdb
oracle   18115     1  0 19:29 ?        00:00:00 ora_mman_wingdb
oracle   18117     1  0 19:29 ?        00:00:00 ora_dbw0_wingdb
oracle   18119     1  0 19:29 ?        00:00:00 ora_lgwr_wingdb
oracle   18121     1  0 19:29 ?        00:00:00 ora_ckpt_wingdb
oracle   18123     1  0 19:29 ?        00:00:00 ora_smon_wingdb
oracle   18125     1  0 19:29 ?        00:00:00 ora_reco_wingdb
oracle   18127     1  0 19:29 ?        00:00:00 ora_mmon_wingdb
oracle   18129     1  0 19:29 ?        00:00:00 ora_mmnl_wingdb
oracle   18131     1  0 19:29 ?        00:00:00 ora_d000_wingdb
oracle   18133     1  0 19:29 ?        00:00:00 ora_s000_wingdb
oracle   18154     1  0 19:30 ?        00:00:00 ora_qmnc_wingdb
oracle   18170     1  1 19:30 ?        00:00:00 ora_cjq0_wingdb
oracle   18172     1  2 19:30 ?        00:00:00 ora_j000_wingdb
oracle   18174     1  0 19:30 ?        00:00:00 ora_j001_wingdb
oracle   18175 13276  0 19:30 pts/1    00:00:00 sqlplus   as sysdba
oracle   18176 18175  0 19:30 ?        00:00:00 oraclewingdb (DESCRIPTION=(LOCAL=YES)(ADDRESS=(PROTOCOL=beq)))
oracle   18178     1  0 19:30 ?        00:00:00 ora_q000_wingdb
oracle   18180     1  0 19:30 ?        00:00:00 ora_q001_wingdb
root     18183 13222  0 19:30 pts/0    00:00:00 grep oracle

##### EM
# EM启动
[oracle@wing ~]$ emctl start dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.2.0.4.0 
Copyright (c) 1996, 2013 Oracle Corporation.  All rights reserved.
https://wing:1158/em/console/aboutApplication
Starting Oracle Enterprise Manager 11g Database Control ........... started. 
------------------------------------------------------------------
Logs are generated in directory /u01/app/oracle/product/11.2.0/db_1/wing_wingdb/sysman/log 
# EM状态
[oracle@wing ~]$ emctl status dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.2.0.4.0 
Copyright (c) 1996, 2013 Oracle Corporation.  All rights reserved.
https://wing:1158/em/console/aboutApplication
Oracle Enterprise Manager 11g is running. 
------------------------------------------------------------------
Logs are generated in directory /u01/app/oracle/product/11.2.0/db_1/wing_wingdb/sysman/log
# EM端口查看
[oracle@wing ~]$ netstat -npl | grep 1158
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 :::1158                     :::*                        LISTEN      20715/java
```

##### Oracle关闭
EM-->listener-->Oracle
```
##### EM
# EM端口监听查看
[oracle@wing ~]$ netstat -npl | grep 1158
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 :::1158                     :::*                        LISTEN      15338/java
# EM关闭操作
[oracle@wing ~]$ emctl stop dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.2.0.4.0 
Copyright (c) 1996, 2013 Oracle Corporation.  All rights reserved.
https://wing:1158/em/console/aboutApplication
Stopping Oracle Enterprise Manager 11g Database Control ... 
 ...  Stopped. 
# EM状态查看
[oracle@wing ~]$ emctl status dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.2.0.4.0 
Copyright (c) 1996, 2013 Oracle Corporation.  All rights reserved.
https://wing:1158/em/console/aboutApplication
Oracle Enterprise Manager 11g is not running.

##### listener
# listener端口监听查看
[oracle@wing ~]$ netstat -npl | grep 1521
# listener关闭操作
[oracle@wing ~]$ lsnrctl stop

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 11-FEB-2016 19:13:55

Copyright (c) 1991, 2013, Oracle.  All rights reserved.

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
The command completed successfully
# listener状态查看
[oracle@wing ~]$ lsnrctl status

LSNRCTL for Linux: Version 11.2.0.4.0 - Production on 11-FEB-2016 19:14:03

Copyright (c) 1991, 2013, Oracle.  All rights reserved.

Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 111: Connection refused

##### Oracle
# Oracle关闭操作
[oracle@wing ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Thu Feb 11 19:14:37 2016

Copyright (c) 1982, 2013, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> 
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> 
# 检查Oracle是否关闭，重新连接，检查是否有数据库连接
SQL> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
[oracle@wing ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Thu Feb 11 19:16:37 2016

Copyright (c) 1982, 2013, Oracle.  All rights reserved.

Connected to an idle instance.

SQL>

```
