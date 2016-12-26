## sysbench之基于MySQL的OLTP测试

[请点击此处查看sysbench部署](https://github.com/wing324/helloworld_zh/blob/master/Linux/sysbench/sysbench%E9%83%A8%E7%BD%B2.md)

[请点击此处查看sysben相关参数](https://github.com/wing324/helloworld_zh/blob/master/Linux/sysbench/sysbench%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3.md)

参考文档[来自yejr博客文章1](http://imysql.com/2014/10/17/sysbench-full-user-manual.shtml)，[文章2](http://imysql.com/node/312)

MySQL版本：MySQL5.6.19

sysbench版本：1.0

#### 一、测试前准备数据

```
sysbench --mysql-user=db_user --mysql-host=db_host --mysql-port=db_port --mysql-password=db_password --test=/usr/local/sysbench/sysbench/tests/db/oltp.lua --oltp_tables_count=10 --oltp-table-size=100000 prepare
```

注：/usr/local/sysbench为sysbench安装目录。

**--test**

> oltp脚本。

**--oltp_tables_count**

> oltp表的数量。

**--oltp-table-size**

> 每张OLTP表的记录数。

#### 二、oltp测试

```
sysbench --mysql-user=db_user --mysql-host=db_host --mysql-port=db_port --mysql-password=db_password --test=/usr/local/sysbench/sysbench/tests/db/oltp.lua --oltp_tables_count=10 --oltp-table-size=100000 --num-threads=8 --oltp-read-only=off --oltp-test-mode=complex -report-interval=10 --rand-type=uniform --max-time=3600 --max-requests=0 --percentile=99 run
```

注：/usr/local/sysbench为sysbench安装目录。

**--oltp-read-only**

> 仅仅执行SELECT语句。

[其他参数请点击此处。](https://github.com/wing324/helloworld_zh/blob/master/Linux/sysbench/sysbench%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3.md)

#### 三、oltp测试结果解读

```
sysbench 1.0:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 8
Report intermediate results every 10 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

# 第10S测试结果 | 线程数 | TPS | 每秒读 | 每秒写 | 99%以上的响应时长 | 错误数 | 重试数  
[  10s] threads: 8, tps: 91.50, reads: 1291.09, writes: 368.50, response time: 1023.91ms (99%), errors: 0.00, reconnects:  0.00
[  20s]
[  30s]
	.
	.
	.
[3600s]
OLTP test statistics:
    queries performed:
        read:                            2540048	--读总数
        write:                           725728		--写总数
        other:                           362864		--除SELECT/INSERT/UPDATE/DELETE之外操作
        total:                           3628640	--总操作数
    transactions:                        181432 (50.39 per sec.)	--总事务数(每秒事务数)
    read/write requests:                 3265776 (907.10 per sec.)	--读写总数(每秒读写数)
    other operations:                    362864 (100.79 per sec.)	--其他操作总数(每秒其他操作数)
    ignored errors:                      0      (0.00 per sec.)		--总错误数(每秒发生错误数)
    reconnects:                          0      (0.00 per sec.)		--总重试数(每秒发生重试数)

General statistics:
    total time:                          3600.2202s		--sysbench总执行时间
    total number of events:              181432			--sysbench总发生的事务数
    total time taken by event execution: 28800.2820s	--所有事务的总操作时间
    response time:
         min:                                  2.69ms	--最小响应时间
         avg:                                158.74ms	--平均响应时间
         max:                               5750.77ms	--最大响应时间
         approx.  99 percentile:            1404.98ms	--99%的平均耗时

Threads fairness:
    events (avg/stddev):           22679.0000/199.81
    execution time (avg/stddev):   3600.0352/0.07
```

