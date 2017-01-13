## Linux监视进程之ps命令

操作系统：Debian8.5

**仅仅以下常用的三种组合命令的解释，更加齐全的命令请自行`ps --help all` 查看。**

#### 一、ps -ef

输出展示：

```shell
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0  2016 ?        00:00:28 /lib/systemd/systemd --system --deserialize 15
root         2     0  0  2016 ?        00:00:00 [kthreadd]
root         3     2  0  2016 ?        00:00:25 [ksoftirqd/0]
root         5     2  0  2016 ?        00:00:00 [kworker/0:0H]
root         7     2  0  2016 ?        00:04:57 [rcu_sched]
root         8     2  0  2016 ?        00:00:00 [rcu_bh]
......
```

输出字段说明：

>UID:	进程所属的用户。
>
>PID:		进程的ID。
>
>PPID:	父进程的PID。
>
>C:		CPU的使用/调度信息。
>
>STIME:	启动进程的时间。
>
>TTY:		控制终端。
>
>TIME:	消耗CPU的时间。
>
>CMD:	进程执行的命令行。

#### 二、ps aux

输出展示：

```shell
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0 176040  3272 ?        Ss    2016   0:28 /lib/systemd/systemd --system --deserialize 15
root         2  0.0  0.0      0     0 ?        S     2016   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S     2016   0:25 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<    2016   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S     2016   4:57 [rcu_sched]
root         8  0.0  0.0      0     0 ?        S     2016   0:00 [rcu_bh]
......
```

输出字段说明：

> USER:	进程的所属用户。
>
> PID:		进程的ID。
>
> %CPU:	该进程正在使用的CPU百分比。
>
> %MEM:	该进程正在使用的内存百分比。
>
> VSZ:	进程的虚拟大小。
>
> RSS:	内存中页的数量。
>
> TTY:		控制终端。
>
> STAT:	当前进程状态。
>
> ​		R=可运行；D=不可中断的休眠状态(如正在等待磁盘)；S=休眠状态(Sleep)；T=被跟踪或者被停止(Stop)；Z=僵尸进程(Zombie)；
>
> ​		附加标志：
>
> ​		W=进程被交换出去(Progress is swapping out)；<=进程优先级高于普通优先级；N=进程优先级低于普通优先级；L=有些页面被锁在内存中；s=进程是会话的先导进程(Process is a session leader)
>
> START:	进程开始时间。
>
> TIME:	进程已经消耗掉的CPU时间。
>
> COMMAND:	进程执行的命令行。

### 三、ps lax

ps lax运行速度比ps aux快，因为它不必将每个UID转换成用户名。

输出展示：

```shell
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0     1     0  20   0 176040  3272 -      Ss   ?          0:28 /lib/systemd/systemd --system --deserialize 15
1     0     2     0  20   0      0     0 -      S    ?          0:00 [kthreadd]
1     0     3     2  20   0      0     0 -      S    ?          0:25 [ksoftirqd/0]
1     0     5     2   0 -20      0     0 -      S<   ?          0:00 [kworker/0:0H]
1     0     7     2  20   0      0     0 -      S    ?          4:58 [rcu_sched]
1     0     8     2  20   0      0     0 -      S    ?          0:00 [rcu_bh]
```

输出字段说明：

> F:		进程标志。
>
> UID:	进程所属用户的ID。
>
> PID:		进程ID。
>
> PPID:	父进程ID。
>
> PRI:		进程的优先级，值越小代表优先级越高。
>
> NI:		进程的谦和度，也可以理解为进程的优先级，值越小代表优先级越高。
>
> VSZ:	进程的虚拟大小。
>
> RSS:	内存中页的数量。
>
> WCHAN:	进程正在等待的对象地址。
>
> STAT:	当前进程状态。
>
> ​		R=可运行；D=不可中断的休眠状态(如正在等待磁盘)；S=休眠状态(Sleep)；T=被跟踪或者被停止(Stop)；Z=僵尸进程(Zombie)；
>
> ​		附加标志：
>
> ​		W=进程被交换出去(Progress is swapping out)；<=进程优先级高于普通优先级；N=进程优先级低于普通优先级；L=有些页面被锁在内存中；s=进程是会话的先导进程(Process is a session leader)
>
> TTY:		控制终端。
>
> TIME:	进程已经消耗掉的CPU时间。
>
> COMMAND:	进程执行的命令行。



本文参考《UNIX/Linux系统管理技术手册》。