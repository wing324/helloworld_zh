## sysbench参数详解

* [sysbench参数详解](#sysbench参数详解)
     * [一、sysbench功能测试参数](#一sysbench功能测试参数)
     * [二、通用参数](#二通用参数)
     * [三、日志参数](#三日志参数)
     * [四、通用数据库参数](#四通用数据库参数)
     * [五、MySQL相关参数](#五mysql相关参数)
     * [六、fileio相关参数](#六fileio相关参数)
     * [七、cpu相关参数](#七cpu相关参数)
     * [八、memory相关参数](#八memory相关参数)
     * [九、threads相关参数](#九threads相关参数)
     * [十、mutex相关参数](#十mutex相关参数)
     * [十一、oltp相关参数](#十一oltp相关参数)



sysbench版本：1.0

#### 一、sysbench功能测试参数

**fileio**

> 磁盘IO测试。

**cpu**

> CPU性能测试。

**memory**

> 内存分配及传输速度测试。

**threads**

> 线程性能测试。

**mutex**

> 互斥性能测试。

**提示：**

还存在oltp测试，为sysbench默认测试。



#### 二、通用参数

**--num-threads=N**

> 使用的线程数量，默认值为1。

**--max-requests=N**

>  总请求数，与`--max-time`选择一个设置即可，默认值为10000。

**--max-time=N**

> 总执行时间，与`--max-requests`选择一个设置即可，单位为s，默认值为0。

**--forced-shutdown=STRING**

> 超过`--max-time`后强制中断，默认为off。

**--thread-stack-size=SIZE**

> 每个线程的stack大小，默认为64K。

**--tx-rate=N**

> sysbench尝试像数据库发送的事务数tps。

**--report-interval=N**

> 表示N秒输出一次测试进度报告，0表示关闭测试进度报告输出，仅输出最终的报告结果，默认值为0。

**--report-checkpoints=[LIST,...]**

> dump full statistics and reset all counters at specified points in time. The argument is a list of comma-separated values representing the amount of time in seconds elapsed from start of test when report checkpoint(s) must be performed. Report checkpoints are off by default. []

**--test=STRING**

> 测试类型，可选项：fileio/cpu/memory/threads/mutex/oltp脚本路径。

**--debug=[on|off]**

>  debug模式输出，默认值为off。

**--validate=[on|off]**

> 在可能的情况下执行验证检查，默认为off。

**--help=[on|off]**

>输出help信息，默认为off。

**--version=[on|off]**

>  输出版本信息，默认为off。

**--rand-type=STRING**

> 表示随机类型的模式，共有4种模式：uniform(固定)，gaussian(高斯)，special(特定)，pareto(帕雷特)，默认值为：special。

**--rand-spec-iter**

> number of iterations used for numbers generation [12]

**--rand-spec-pct=N**

> 对于'special'随机模式中指定值的比例，默认值为75。

**--rand-seed=N**

> seed for random number generator, ignored when 0 [0]

**--rand-pareto-h=N**

> parameter h for pareto distibution [0.2]

**--config-file**

> sysbench配置文件路径。



#### 三、日志参数

**--verbosity=N**

> 初测试报告信息之外的信息输出级别，5为debug信息，0位仅仅输出严重信息，默认值为3。

**--percentile=N**

> 查询相应时间采样的百分比，默认值为95%。



#### 四、通用数据库参数

**--db-driver=STRING**

> 特殊的数据库驱动。

**--db-ps-mode=STRING**

>SQL是否需要预编译，模式有：auto/disable，默认为disable。

**--db-debug=[on|off]**

> 输出数据库层面的debug信息，默认为off。



#### 五、MySQL相关参数

**--mysql-host=[LIST,...]**

> MySQL服务器IP/hostname，默认：localhost。

**--mysql-port=[LIST,...]**

> MySQL端口号，默认：3306。

**--mysql-socket=[LIST,...]**

> MySQL的socket文件。

**--mysql-user=STRING**

> MySQL的用户名，默认：sbtest

**--mysql-password=STRING**

> MySQL用户密码。

**--mysql-db=STRING**

> MySQL数据库。

**--mysql-table-engine=STRING**

> 用户测试表的表结构引擎，可选项：myisam/innodb/bdb/heap/ndbcluster/federated，默认值：innodb。

**--mysql-engine-trx=STRING**

> 存储引擎是否使用事务，可选项：yes,no,auto，默认值：auto。

**--mysql-ssl=[on|off]**

> 使用SSL连接，默认值：off。

**--mysql-ssl-cipher=STRING**

> 为SSL连接指定密码。

**--mysql-compression=[on|off]**

> 使用压缩，默认值：off。

**--myisam-max-rows=N**

> MyISAM表的最大记录数，默认值：1000000。

**--mysql-debug=[on|off]**

> 输出MySQL的debug信息，默认值：off。

**--mysql-ignore-errors=[LIST,...]**

> MySQL忽略的错误代码，可选项：1213/1020/1205

**--mysql-dry-run=[on|off]**

> 假装MySQL所有客户端API都被调用，但实际并不执行它们，默认值：off



#### 六、fileio相关参数

`sysbench --test=fileio help`

**--file-num=N**

> 创建文件的数量，默认值：128。

**--file-block-size=N**

> 每次IO操作的block大小，默认值：16K。

**--file-total-size=SIZE**

> 所有文件大小总和，默认值：2G。

**--file-test-mode=STRING**

>  测试模式：seqwr(顺序写), seqrewr(顺序读写), seqrd(顺序读), rndrd(随机读), rndwr(随机写), rndrw(随机读写)。

**--file-io-mode**

> 文件操作模式：sync(同步),async(异步),mmap(快速map映射)，默认值：sync。

**--file-async-backlog**

> number of asynchronous operatons to queue per thread [128]。

**--file-extra-flags=STRING**

> 使用额外的标志符来打开文件{sync,dsync,direct}。

**--file-fsync-freq=N**

> 在完成N次请求之后，执行fsync()，0表示不使用fsync，默认值：100。

**--file-fsync-all=[on|off]**

> 每次写操作后执行fsync()，默认值：off。

**--file-fsync-end=[on|off]**

> 测试结束后执行fsync()，默认值：on。

**--file-fsync-mode=STRING**

> 使用fsync或fdatasync方法进行同步，默认值：fsync。

**--file-merged-requests=N**

> 尽可能的合并N个IO请求数，0表示不合并，默认值：0。

**--file-rw-ratio=N**

> 测试时候的读写比例，默认值：1.5(即3:2)。



#### 七、cpu相关参数

`sysbench --test=cpu help`

**--cpu-max-prime=N**

> 最大质数生成器的上限，默认值：10000。



#### 八、memory相关参数

`sysbench --test=memory help`

**--memory-block-size=SIZE**

> 测试时内存块大小，默认值：1K。

**--memory-total-size=SIZE**

> 传输数据可使用的最大内存大小，默认值：100G。

**--memory-scope=STRING**

> 内存访问范围：global/local，默认值：global。

**--memory-hugetlb=[on|off**

> 从HugeTLB池分配内存，默认值：off。

**--memory-oper=STRING**

> 内存操作类型：read/ write/none，默认值：write。

**--memory-access-mode=STRING**

> 内存访问方式：seq(顺序)/rnd(随机)，默认值：seq。



#### 九、threads相关参数

`sysbench --test=threads help`

**--thread-yields=N**

> 每个请求产生多少线程，默认值：1000。

**--thread-locks=N**

> 每个线程的锁的数量，默认值：8。



#### 十、mutex相关参数

`sysbench --test=mutex help`

**--mutex-num=N**

> 数组互斥的总大小，默认值：4096。

**--mutex-locks=N**

> 每个线程互斥锁的数量，默认值：50000。

**--mutex-loops=N**

> 内部互斥锁的空循环数量，默认值：10000



#### 十一、oltp相关参数

待研究。