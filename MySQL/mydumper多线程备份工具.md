## mydumper多线程备份工具

最近在迁移zabbix数据库，500G+的数据，使用mysqldump备份花了3H+，于是想到mydumper多线程备份，备份时间缩短至一半，于是对mydumper多线程备份工具顿生好感。



#### 一、mydumper安装

```shell
apt-get install libpcre3 libglib2.0 zlib1g libglib2.0-dev libmysqlclient15-dev zlib1g-dev libpcre3-dev libssl-dev
cd /usr/local
wget https://launchpadlibrarian.net/225370879/mydumper-0.9.1.tar.gz
tar -zxvf mydumper-0.9.1.tar.gz
cd mydumper-master/
cmake .
make
make install
```



#### 二、mydumper使用

```shell
# 数据库备份：
mydumper -u username -h host -P port -p password -B database -o back_dir  -t thread

# 数据库恢复：
myloader -u username -h host -P port -p passwd -B database  -d back_dir -t thread
```



#### 三、mydumper常用参数

> -B,—database
>
> 备份的数据库。
>
> -T,—table-list
>
> 备份的表，使用',分割。
>
> -o,—outputdir
>
> 备份文件的目录，这是一个目录。
>
> -m,—no-schemas
>
> 不备份表结构。
>
> -d,—no-data
>
> 不备份表数据。
>
> -G,—triggers
>
> 备份触发器。
>
> -E,—events
>
> 备份事件。
>
> -R,—routines
>
> 备份存储过程和函数。
>
> -W,—no-views
>
> 不备份视图。
>
> -L,—logfile
>
> mydumper日志文件，默认是stdout。
>
> -t,—threads
>
> 线程数，默认为4。



#### 四、myloader常用参数

> -d,—directory
>
> 恢复的备份目录
>
> -o,—overwrites-tables
>
> 覆盖写，当表存在时则删除。
>
> -B,—database
>
> 恢复的数据库名称。
>
> -e,—enale-binlog
>
> 允许将备份数据记录binlog中。
>
> -t,—threads
>
> 线程数，默认为4



#### 五、注意事项

1. 主上写入很大的时候，可将innodb_flush_log_at_trx_commit和sync_binlog为0。

