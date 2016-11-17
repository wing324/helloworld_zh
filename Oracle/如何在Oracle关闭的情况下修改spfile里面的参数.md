## 如何在Oracle关闭的情况下修改spfile里面的参数

在Oracle中pfile参数是可以手动更改的，但是spfile是二进制文件所以不可以手动更改，那么万一碰到了我遇到的情况（见下），修改参数错误，导致Oracle启动不了，一定要修改spfile该怎么办呢？  




问题
----
我使用的Oracle11g，当我敲下如下一段命令后，就让我傻眼了。。  
```sql
alter system set sga_max_size=960M scope=spfile;
shutdown immediate
startup
```
此时的startup报错了，错误为：  
```sql
SQL> startup
ORA-00844: Parameter not taking MEMORY_TARGET into account
ORA-00851: SGA_MAX_SIZE 985661440 cannot be set to more than MEMORY_TARGET 784334848.
```


原因分析
-------
原来在Oracle11g中增加了memory_target参数，sga_max_size必须必memory_target参数小。那么问题来了，此时我已经关闭Oracle了，spfile文件是二进制文件，又不能手动修改，那么我该怎么办呢。。好捉急好捉急。。。  


思路
----
通过pfile启动Oracle-->在Oracle中通过create pfile='' from spfile=''取出spfile的内容（pfile是可以手动修改的）-->修改新建的pfile-->以新的pfile启动Oracle-->在Oracle中通过create spfile='' from pfile=''获得修改后的spfile


实战
----
```sql
[oracle@wing ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Mon Feb 15 14:04:46 2016

Copyright (c) 1982, 2013, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> create pfile='/home/oracle/pfile.new' from spfile='/u01/app/oracle/product/11.2.0/db_1/dbs/spfilewingdb.ora';
File created.

SQL> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

通过vi修改pfile.new文件中相应的参数（本文档中是memory_target参数），修改后保存  

[oracle@wing ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Mon Feb 15 14:04:46 2016

Copyright (c) 1982, 2013, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup pfile='/home/oracle/pfile.new'
ORACLE instance started.

Total System Global Area  810090496 bytes
Fixed Size                  2257520 bytes
Variable Size             415239568 bytes
Database Buffers          390070272 bytes
Redo Buffers                2523136 bytes
Database mounted.
Database opened.

SQL> create spfile='/u01/app/oracle/product/11.2.0/db_1/dbsspfilewingdb.ora' from pfile='/home/oracle/pfile.new';

File created.

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> exit
Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.4.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options
[oracle@wing ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.4.0 Production on Mon Feb 15 14:08:40 2016

Copyright (c) 1982, 2013, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> 
SQL> startup
ORACLE instance started.

Total System Global Area  810090496 bytes
Fixed Size                  2257520 bytes
Variable Size             415239568 bytes
Database Buffers          390070272 bytes
Redo Buffers                2523136 bytes
Database mounted.
Database opened.

SQL> show parameter memory    

NAME                                 TYPE
------------------------------------ --------------------------------
VALUE
------------------------------
hi_shared_memory_address             integer
0
memory_max_target                    big integer
800M
memory_target                        big integer
800M
shared_memory_address                integer
0
SQL> show parameter sga

NAME                                 TYPE
------------------------------------ --------------------------------
VALUE
------------------------------
lock_sga                             boolean
FALSE
pre_page_sga                         boolean
FALSE
sga_max_size                         big integer
776M
sga_target                           big integer
740M

# 至此Oracle使用新的spfile启动成功，参数也得到相应的修改
```