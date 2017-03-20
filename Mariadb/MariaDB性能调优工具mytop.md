## MySQL/MariaDB性能调优工具mytop

mytop为MariaDB自带工具，但MySQL并没有携带该工具，需要自行安装。安装方式请自行google==

#### 一、使用方式

```shell
./mytop --prompt -u xxx -h xxx -P xxx -d xxx
```



#### 二、结果解释

- 常用结果展示

  ```
  MariaDB on localhost (10.1.18-MariaDB)                       up 77+08:53:14 [20:08:00]
  Queries: 17.6G   qps: 2833 Slow:  106.7k         Se/In/Up/De(%):    48/07/11/00
  Sorts:   380 qps now: 3196 Slow qps: 0.2  Threads:  199 (   3/ 102) 52/05/12/00
  Handler: (R/W/U/D)  9372/ 5371/  709/    0        Tmp: R/W/U:   297/  278/    0
  ISAM Key Efficiency: 99.7%  Bps in/out: 586.6k/ 2.1M   Now in/out: 521.9k/ 1.8M
  Replication IO:No SQL:No
        Id      User         Host/IP         DB      Time    Cmd Query or State
         --      ----         -------         --      ----    --- ----------
          2      root       localhost      mysql         0  Query show full processlist                          
         16      root       localhost                    0  Sleep
         17      root       localhost     testdb         0  Query SELECT * FROM dept_emp
  ```

  第一行：MariaDB版本；数据库运行时间；

  第二行：Queries为数据库启动之后处理的总queries；qps:数据库启动之后平均的qps；Slow：数据库启动之后的慢查询总数；Se/In/Up/De(%): SELECT/INSERT/UPDATE/DELETE所占的比例；

  第三行：Sorts：没看明白；qps now:自mytop上次刷新后的平均qps；Slow qps: 自mytop上次刷新后的平均慢查询qps；Threads: 连接数据库线程总数(活跃的线程数/SLEEP状态的线程数)；52/05/12/00：自mytop上次刷新后SELECT/INSERT/UPDATE/DELTE的比例；

  第四行：个人目前没有看明白，也没有找到相关的描述；

  第五行:	ISAM Key Efficiency:myisam的key buffer的命中率，Bps：数据库启动之后平均的网络流量，Now：自mytop上次刷新后的平均网络流量。

  ​

#### 三、常用快捷键

?	显示帮助信息。

c	命令的总结视图(基于Com_*的统计)。

C	关闭／开启颜色模式。

d	仅仅显示指定的数据库。

e	将指定的thread_id对应的query在数据库上的explain结果展示出来。

E	展示当前复制的error信息。

f	显示指定query的完整信息。

h	显示指定的host的连接信息。

H	只显示mytop的头信息。

I	show innodb status的信息。（大写i）

k	kill指定的thread id。

p	显示暂停。

l	高亮慢查询。（小写L）

m	只展示qps的信息。

M	显示状态信息。

o	反序排序。

s	显示信息的refresh间隔。

u	显示指定用户的连接信息。

V	show variables的相关信息。（大写v）

