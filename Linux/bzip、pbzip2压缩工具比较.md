## bzip、pbzip2压缩工具比较

Linux版本：Debian8.5

pbzip2安装：`apt-get install pbzip2`

#### bzip(单线程压缩工具)

```shell
# 压缩单个文件测试
# 单个文件大小
root@wing:/data # du -h 2016.sql
3.4G	2016.sql
# tar bzip2 压缩命令
time tar -jcf 2016.sql.bz2 2016.sql
# 单个文件压缩时间
real	10m7.996s
user	10m4.632s
sys	0m13.276s
# 压缩后文件大小
root@wing:/data # du -sh 2016.sql.bz2
220M	2016.sql.bz2

# 压缩目录测试
# 目录文件大小
root@wing:/data # du -sh 20161122/
6.9G	20161122/
# tar bzip 只能使用一个核进行压缩
time tar -jcvf 20161122_bzip.bz2 20161122/*
# 目录压缩时间
real	24m30.013s
user	22m51.936s
sys	0m23.872s
# 压缩后文件大小
root@wing:/data # du -h 20161122.bz2
356M	20161122.bz2
```

#### pbzip2(多线程压缩工具)

```shell
# 压缩单个文件测试
# 单个文件大小
root@wing:/data # du -h 2016.sql
3.4G	2016.sql
# pbzip2压缩命令
time pbzip2 -p3 -k 2016.sql 
# 单个文件压缩时间
real	3m22.909s
user	9m55.092s
sys	0m16.284s
# 压缩后文件大小
root@wing:/data # du -sh 2016.pbzip.bz2
221M	2016.pbzip.bz2

# 压缩目录测试
# 目录文件大小
root@wing:/data # du -sh 20161122/
6.9G	20161122/
# tar bzip pbzip 使用3个核进行压缩
time tar -c 20161122 | pbzip2 -p3 -c > 20161122.tar.bz2
# 目录压缩时间
real	7m31.688s
user	22m5.736s
sys	0m42.520s
# 压缩后文件大小
root@wing:/data # du -h 20161122.tar.bz2
358M	20161122.tar.bz2

```

#### 总结：

|               |    bzip    | pbzip（3个线程） |
| :-----------: | :--------: | :---------: |
|     原文件大小     |    3.4G    |    3.4G     |
| 文件压缩时间( real) | 10m7.996s  |  3m22.909s  |
|    文件压缩大小     |    220M    |    221M     |
|     原目录大小     |    6.9G    |    6.9G     |
| 目录压缩时间(real)  | 24m30.013s |  7m31.688s  |
|    目录压缩大小     |    356M    |    358M     |

**注意：**压缩时间使用real计算，而不使用user+sys计算的原因是，多线程下user的时间是每个线程时间之和，与我们可以感知到的时间偏差较大，所以选择real，该服务器上都是初始化的job，所以real更接近用户感知的时间。

从上面表格可以得出，**pbzip2开启3个线程压缩的前提下，无论是压缩单个文件还是压缩目录，时间上比单线程bzip2压缩快了接近3倍，而压缩比也基本相同。**