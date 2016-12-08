## pbzip2安装及使用

Linux版本：Debian8.5

#### 简介

平时大文件的压缩喜欢使用bzip2，虽然bzip2的压缩率很高，但是压缩时长实在无法忍受，于是，通过强大的Google找到了pbzip2这款工具，pbzip2作为多线程版本的bzip2工具，压缩比和bzip2基本相当，但是压缩时间比bzip2减少了线程数倍数，毕竟bzip2是单线程工作，pbzip2是多线程工作。详情请见：[bzip2与pbzip2压缩工具比较](https://github.com/wing324/helloworld/blob/master/Linux/bzip%E3%80%81pbzip2%E5%8E%8B%E7%BC%A9%E5%B7%A5%E5%85%B7%E6%AF%94%E8%BE%83.md)

#### pbzip2安装

```apt-get install pbzip2```

#### pbzip2参数详解

> Usage: pbzip2 \[-1 .. -9][-b#cdfhklm#p#qrS#tVz] \<filename> \<filename2> \<filenameN>
>
> -1…-9
>
> 设置BWT(一种压缩技术算法)的block大小为100k...900k(默认为900k)
>
> -b#
>
> block大小，单位是100k（默认9=900k）
>
> -c,--stdout
>
> 输出到stdout
>
> -d,--decompress
>
> 解压文件
>
> -f,--force
>
> 覆盖已经存在的输出文件
>
> -h,--help
>
> 输出帮助信息
>
> -k,--keep
>
> 保留被压缩的文件(默认删除被压缩文件)，这里是个大坑，所以使用pbzip2压缩时，切记一定要携带-k参数
>
> -l,--loadavg
>
> 由load average(平均负载)决定使用CPU的最大数量
>
> -m#
>
> 最大内存使用量，单位：1MB(默认 100=100MB)
>
> -p#
>
> 指定CPU数，即线程数(默认自动检测，检测失败后为2)
>
> -q,--quiet
>
> 静默模式
>
> -r,--read
>
> 读取整个文件进入内存，并在各个CPU分开处理
>
> -S#
> 子线程的stack(堆栈)大小，单位：1KB
>
> -t,--test
> 完整的测试压缩文件
>
> -v,--verbose
> 详细信息模式
>
> -V,--version
>
> 输出pbzip2的版本信息
>
> -z,--compress
>
> 压缩文件(默认值)
>
> --ignore-trailing-garbage=#
> 是否忽略文件末尾对齐数据块(1忽略，0禁止)

#### pbzip2常用示例

- 压缩单个文件（指定3个线程）

```shell
pbzip2 test.sql -z -p3 -k > test.sql.bz2
```

- 压缩目录（指定3个线程）

```shell
tar -c test_dir/* | pbzip2 -c -p3 -k > test_dir.tar.bz2
```

- 解压文件（指定3个线程）

```shell
pbzip2 -d -p3 -k test.sql.bz2
```

- 解压目录（指定3个线程）

```shell
pbzip2 -d -p3 -k test_dir.tar.bz2
tar -xf test_dir.tar
# 或者
pbzip2 -d -p3 -k test_dir.tar.bz2 && tar -xf test_dir.tar
```

## pbzip2限制

由于pbzip2只能压缩文件，不能对目录进行压缩，所以如果想使用pbzip2压缩目录，则需要借助tar工具。