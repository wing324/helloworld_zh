## Liunx系统命令之tree命令

Linux下有一个树状图展示目录的命令，是一款在颜值上优先于ls的命令。  



tree安装
------------

```shell
tar -zxvf tree-1.7.0.tgz
cd tree-1.7.0
make
cp -af tree /usr/bin
```
或者  

```
yum install -y tree
```


tree命令详解
------------------
1. tree参数详解  
> -a   显示所有文件和目录。  
> -A   使用ASNI绘图字符显示树状图而非以ASCII字符组合。  
>  -C   在文件和目录清单加上色彩，便于区分各种类型。  
> -d   显示目录名称而非内容。  

```shell
[root@oracle01 /]# tree -d /data/mysqldata3306
/data/mysqldata3306
├── binlog
├── innodb_log
├── innodb_ts
├── log
├── mydata
│   ├── mysql
│   ├── performance_schema
│   ├── test
│   └── ym
├── relaylog
├── sock
└── tmpdir
```
> -D   列出文件或目录的更改时间。  
> -f   在每个文件或目录之前，显示完整的相对路径名称。  
> -F   在执行文件，目录，Socket，符号连接，管道名称名称，各自加上"*","/","=","@","|"号。  
> -g   列出文件或目录的所属群组名称，没有对应的名称时，则显示群组识别码。  
> -i   不以阶梯状列出文件或目录名称。  
> -I<范本样式>   不显示符合范本样式的文件或目录名称。  
> -l   如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录。  
> -n   不在文件和目录清单加上色彩。  
> -N   直接列出文件和目录名称，包括控制字符。  
> -p   列出权限标示。  

```shell
[root@oracle01 /]# tree -dCp /data/mysqldata3306
/data/mysqldata3306
├── [drwxr-xr-x]  binlog
├── [drwxr-xr-x]  innodb_log
├── [drwxr-xr-x]  innodb_ts
├── [drwxr-xr-x]  log
├── [drwxr-xr-x]  mydata
│   ├── [drwx------]  mysql
│   ├── [drwx------]  performance_schema
│   ├── [drwx------]  test
│   └── [drwx------]  ym
├── [drwxr-xr-x]  relaylog
├── [drwxr-xr-x]  sock
└── [drwxr-xr-x]  tmpdir
```
> -P<范本样式>   只显示符合范本样式的文件或目录名称。  
> -q   用"?"号取代控制字符，列出文件和目录名称。  
> -s   列出文件或目录大小。  

```shell
[root@oracle01 /]# tree -s /data/mysqldata3306/binlog/
/data/mysqldata3306/binlog/
├── [      65302]  mysql-bin.000001
├── [    1046158]  mysql-bin.000002
├── [        556]  mysql-bin.000003
├── [       1125]  mysql-bin.000004
├── [       1510]  mysql-bin.000005
├── [      30019]  mysql-bin.000006
└── [        264]  mysql-bin.index

0 directories, 7 files
```
> -t   用文件和目录的更改时间排序,从最新开始排序。  

```shell
[root@oracle01 ym]# ll
total 220
-rw-rw---- 1 mysql mysql    61 Aug 18 14:01 db.opt
-rw-rw---- 1 mysql mysql  8556 Aug 18 17:04 s.frm
-rw-rw---- 1 mysql mysql 98304 Aug 18 17:04 s.ibd
-rw-rw---- 1 mysql mysql  8556 Aug 18 16:36 t.frm
-rw-rw---- 1 mysql mysql 98304 Aug 18 16:36 t.ibd
[root@oracle01 ym]# tree -t /data/mysqldata3306/mydata/ym/
/data/mysqldata3306/mydata/ym/
├── s.ibd
├── s.frm
├── t.ibd
├── t.frm
└── db.opt
```
> -u   列出文件或目录的拥有者名称，没有对应的名称时，则显示用户识别码。  
> -x   将范围局限在现行的文件系统中，若指定目录下的某些子目录，其存放于另一个文件系统上，则将该子目录予以排除在寻找范围外。  
