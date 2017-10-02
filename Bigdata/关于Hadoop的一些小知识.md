## 关于Hadoop的一些小知识

#### HDFS

1. 在`hdfs fs -ls`结果显示中，文件的有配置的备份数显示，而目录却没有备份呢？
   答：因为目录是元数据(metadata)，被存储在namenode中，而不是datanode中。 