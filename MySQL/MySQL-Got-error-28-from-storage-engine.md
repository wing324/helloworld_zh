## MySQL Got error 28 from storage engine

今天进入数据库后，show tables居然报了一个error:Got error 28 from storage engine，第一次遇到，我还以为我的数据库出现了大问题了，但心里不着急，因为这只是我个人使用的测试库，但还是想赶紧去看看这个error是什么鬼。  


经过google，在stackoverflow中别人说是磁盘不够的原因，于是我敲下了如下命令：  
```shell
[root@host-192-168-200-153 ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root  6.7G  6.6G     0 100% /
tmpfs                         939M     0  939M   0% /dev/shm
/dev/vda1                     485M   32M  428M   7% /boot
```
纳尼，果然是磁盘是真的不够，但是不知道是不是这个原因导致的呢。于是我删除了一些无用的东西之后，再敲下同样的命令：  
```shell
[root@host-192-168-200-153 ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root  6.7G  6.2G  118M  99% /
tmpfs                         939M     0  939M   0% /dev/shm
/dev/vda1                     485M   32M  428M   7% /boot
```
现在磁盘够了，我赶紧去数据库再去执行show tables，然后然后，重要的时刻出现了，居然不再报错了。。原来真的是磁盘不够哇。。。  
这个问题就这么解决了。。