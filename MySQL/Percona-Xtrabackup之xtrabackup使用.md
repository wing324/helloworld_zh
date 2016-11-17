## Percona-Xtrabackup之xtrabackup使用

xtrabackup为Percona Xtrabackup工具之一，文章从xtrabackup工具的工作原理与使用对其进行了介绍。  

Percona Xtrabackup的安装详见：  
http://wing324.github.io/2015/10/04/Percona-Xtrabackup%E5%AE%89%E8%A3%85/



配置xtrabackup
--------------------
xtrabackup可以在my.cnf文件[xtrabackup]下配置，但注意[xtrabackup]的相关配置会取代[mysqld]相关配置。  

```

[xtrabackup]

target_dir = /data/backup
```


xtrabackup完全备份演示
----------------------------------
##### 创建完全备份
**xtrabackup备份时的两个主要任务**  
1. 在后台运行log-copying线程，监控innodb重做日志文件，当日志文件发生改变，就将更改的部分复制到xtrabackup_logfile文件中，当备份完成后，该线程也将会停止。  
2. 将innodb数据文件复制到备份目录下，并不是简单的复制，而是从数据目录中一页一页的复制。  

```shell
xtrabackup --user=percona --host=127.0.0.1 --password=percona --port=3306 --defaults-group='mysqld3306' --backup  --target-dir=/data/backup/x_1
# 备份成功后的输出信息为：
xtrabackup: Transaction log of lsn (1575937353) to (1584357142) was copied.
```

##### 恢复准备阶段

```shell
# 第一次恢复准备
xtrabackup --prepare --target-dir=/data/backup/x_1/
# 目的
使数据文件保持一致，但没有创建新的InnoDB日志文件，如果在此时恢复数据并启动MySQL,此时MySQL需要创建新的日志文件，比较耗时。
# 准备成功后的输出信息为：
InnoDB: Shutdown completed; log sequence number 1584357152

# 第二次恢复准备
xtrabackup --prepare --target-dir=/data/backup/x_1/
# 目的
此时xtrbackup囧创建日志文件，在启动MySQL时，将更减少时间。
# 成功后输出信息有点像下面这样
[root@oracle01 backup]# xtrabackup --prepare --target-dir=/data/backup/x_1/
xtrabackup version 2.2.12 based on MySQL server 5.6.24 Linux (x86_64) (revision id: 8726828)
xtrabackup: cd to /data/backup/x_1/
xtrabackup: This target seems to be already prepared.
xtrabackup: notice: xtrabackup_logfile was already used to '--prepare'.
.
.
.
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
InnoDB: FTS optimize thread exiting.
InnoDB: Starting shutdown...
InnoDB: Shutdown completed; log sequence number 1584357408 
```

##### 恢复备份

```shell
rsync -avrP ibdata1 /data/mysqldata3307/innodb_ts/
rsync -avrP ib_logfile* /data/mysqldata3307/innodb_log/
rsync -avrP mysql/ wing/ xtrabackup_binlog_pos_innodb xtrabackup_checkpoints xtrabackup_logfile /data/mysqldata3307/mydata/
# xtrabackup未提供任何用于恢复的参数
# 修改权限
chown -R mysql:mysql /data/mysqldata3307/
```


Xtrabackup增量备份演示
----------------------------------
##### 创建增量备份

```shell
# 由于增量备份时建立在完全备份的基础上，所以首先创建完全备份
xtrabackup --user=percona --port=3306 --password=percona --host=192.168.200.143 --defaults-group='mysqld3306' --backup --target-dir=/data/backup/full
# 创建增量备份
xtrabackup --user=percona --port=3306 --password=percona --host=192.168.200.143 --defaults-group='mysqld3306' --backup --target-dir=/data/backup/inc1 --incremental-basedir=/data/backup/full/
# --target-dir 增量备份存储目录
# --incemental-base-dir 基于该目录做增量备份

```

##### 增量备份恢复前准备阶段

```shell
# 对完全备份进行准备
xtrabackup --prepare --apply-log-only --target-dir=/data/backup/full/
# 对增量备份进行准备
xtrabackup --prepare  --target-dir=/data/backup/full/ --incremental-dir=/data/backup/inc1/
# --target-dir指的是全备份目录
# --incremental-dir指的是增量备份目录
```

##### 恢复增量备份
恢复增量备份与完全备份相似，此处省略。  


Xtrabackup部分演示
---------------------------
##### 创建部分备份
备份单个表或者单个数据库。  

```shell
# 使用-tables创建部分备份
xtrabackup --user=percona --host=127.0.0.1 --port=3306 --password=percona --defaults-group='mysqld3306' --backup --target-dir=/data/backup/t_full --tables='^wing[.]t'
# 使用--tables-file，--databases，--databases-file创建部分备份,具体参考innobackupex部分备份
```
##### 恢复部分备份准备阶段

```shell
xtrabackup --prepare --export --target-dir=/data/backup/t_full/
```

##### 恢复部分备份
该部分与innobackupex部分备份恢复相似，请参考之。切记要首先创建恢复表的表结构。  
