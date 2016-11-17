## Linux限流工具之pv

pv是一款Liunx下的限流工具，可以使用该工具查看任务进度，传输速率，使用时间以及预估完成时间，还能限制传输速率  



pv示例
------
```
# 我的MySQL数据库中有张表t，数据量如下
wing@3306>select count(*) from t;
+----------+
| count(*) |
+----------+
| 25165824 |
+----------+
1 row in set (10.94 sec)

[root@wing wing]# du -h t.*
12K     t.frm
741M    t.ibd

# 此时我不使用pv工具完成该表的备份,在无任何输出的界面上等待一段时间
[root@wing ~]# mysqldump -uroot -P3306 -h127.0.0.1 wing t > dump.sql
[root@wing ~]# 

# 此时我使用pv工具完成该表的备份,我们可以看到文件已经传输的大小，使用时间，传输速率，已经进度条（感觉进度条不是特别准确的样子）
[root@wing ~]# mysqldump -uroot -P3306 -h127.0.0.1 wing t | pv > dump.sql
 429MiB 0:00:32 [13.1MiB/s] [                                                                                                      <=>                                                      ]

```


pv参数详解
---------
Usage: pv [OPTION] [FILE]...  
Concatenate FILE(s), or standard input, to standard output,with monitoring.  

###### -p, --progress  
show progress bar  
显示进度条（目测不是很准的样子==）（默认）  
###### -t, --timer  
show elapsed time  
显示任务已经进行的时长（默认）  
###### -e, --eta  
 show estimated time of arrival (completion)  
显示剩余多长时间完成（默认，但好像并不能显示）  
###### -r, --rate  
show data transfer rate counter  
显示当前传输速率（默认）  
###### -a, --average-rate  
show data transfer average rate counter  
显示平均传输速率  
###### -b, --bytes  
show number of bytes transferred  
###### -F, --format FORMAT  
set output format to FORMAT  
###### -n, --numeric  
output percentages, not visual information  
显示进度百分比  
###### -q, --quiet  
do not output any transfer information at all  
不输出任何信息
###### -W, --wait  
 display nothing until first byte transferred  
###### -s, --size SIZE  
set estimated data size to SIZE bytes  
###### -l, --line-mode  
count lines instead of bytes  
###### -i, --interval SEC  
update every SEC seconds  
###### -w, --width WIDTH  
assume terminal is WIDTH characters wide  
###### -H, --height HEIGHT  
assume terminal is HEIGHT rows high  
###### -N, --name NAME  
prefix visual information with NAME  
###### -f, --force  
output even if standard error is not a terminal  
###### -c, --cursor  
use cursor positioning escape sequences  
###### L, --rate-limit RATE  
limit transfer to RATE bytes per second  
限制每秒的传输速率，RATE可为n,nK,nM,nG
###### -B, --buffer-size BYTES  
use a buffer size of BYTES  
###### -E, --skip-errors  
skip read errors in input  
###### -S, --stop-at-size  
stop after --size bytes have been transferred  
###### -R, --remote PID  
update settings of process PID  
###### -P, --pidfile FILE  
save process ID in FILE  
###### -h, --help  
show this help and exit  
###### -V, --version  
show version information and exit  

Please report any bugs to Andrew Wood <andrew.wood@ivarch.com>.