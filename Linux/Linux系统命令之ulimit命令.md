## Linux系统命令之ulimit命令

ulimit命令作为一款Linux内置的命令，可以限制资源的使用，利用鸟哥的例子来说明ulimit的作用吧：
想像一个状況：我的 Linux 主机里面同时登陆了十个人，这十个人不知怎么搞的， 同时开启了 100 个文件，每个文件的大小约10MBytes ，请问一下， 我的 Linux 主机的内存要有多大才够？ 10*100*10 = 10000 MBytes = 10GBytes ... 老天爷，这样，系統不挂掉才有鬼呢！为了要预防这个情况的发生，所以我们得『限制使用者的某些系统资源』才行，包括可以开启的文件数量， 可以使用的 CPU 时间，可以使用的内存总量等等。如何设置呢？用 ulimit 吧！(参考链接：http://linux.vbird.org/linux_basic/0320bash.php#variable_ulimit)



参数详解
-------
>-S 软限制，资源使用超过该值会出现警告
>-H 硬限制，资源使用不能超过该值
>-a 列出所有的资源使用限制情况
>-b socket文件的buffer大小
>-c 当某些程序发生错误时，系统会将内存中的信息写到文件中，该文件即为core file，该参数就是限制每个core file的最大容量
>-d 一个程序的数据段（data segement）最大限制
>-e 调度优先级的最大限制
>-f 使用shell建立文件的最大限制
>-i 等待信号的最大数量
>-l 程序可锁住内存的最大值
>-m 可使用内存的最大限制，单位为KB
>-n 打开文件的最大数量
>-p 管道缓冲区的大小
>-q POSIX消息队列的最大字节数
>-r 实时调度优先级的最大限制
>-s stack的最大限制
>-t 可使用的最大CPU时间（单位为秒）
>-u 每个用户可使用的最大程序数量
>-v 可使用虚拟内存的最大限制
>-x 锁定文件的最大数量（file locks并不确定为锁定文件，该参数实际含义不详）


实战演练
-------
```shell
# 查看当前用户的所有资源使用限制情况
[test@localhost ~]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7230
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

# 修改当前用户的file size的限制为512blocks,注意file size的大小有所变动，并且注意该方法的设置在当前用户登出系统时，设置失效，即此时的file size依旧会变为原来的unlimited
[test@localhost ~]$ ulimit -f 512
[test@localhost ~]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) 512
pending signals                 (-i) 7230
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

#测试file size是否真的限制文件大小
[test@localhost ~]$ dd if=/dev/zero of=test bs=4K count=1204
文件大小超出限制
[test@localhost ~]$ dd if=/dev/zero of=test bs=4K count=513
文件大小超出限制
[test@localhost ~]$ dd if=/dev/zero of=test bs=4K count=512
文件大小超出限制
[test@localhost ~]$ dd if=/dev/zero of=test bs=4K count=511
文件大小超出限制
[test@localhost ~]$ dd if=/dev/zero of=test bs=4K count=256
文件大小超出限制
[test@localhost ~]$ dd if=/dev/zero of=test bs=4K count=128
记录了128+0 的读入
记录了128+0 的写出
524288字节(524 kB)已复制，0.000737911 秒，711 MB/秒
```
