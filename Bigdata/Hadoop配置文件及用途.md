## Hadoop配置文件及用途

目前先记录所有的用途，配置项待后续阅读Hadoop的官方文档后再来补充。

##### 一、hadoop-env.sh

1. 用途
   运行Hadoop时候的环境变量脚本。
2. 配置项

##### 二、mapred-env.sh

1. 用途
   运行MapReduce(后续简称MR)时候的环境变量脚本。(会覆盖hadoop-env.sh脚本中环境变量)。
2. 配置项

##### 三、yarn-env.sh

1. 用途
   运行YARN时候的环境变量脚本。(会覆盖hadoop-env.sh脚本中环境变量)。
2. 配置项

##### 四、core-site.xml

1. 用途
   Hadoop的内核配置，例如HDFS、MR、YARN的I/O配置等。
2. 配置项

##### 五、hdfs-site.xml

1. 用途
   HDFS守护进程的配置，如NameNode,SecondaryNameNode,DataNode的相关配置。
2. 配置项

##### 六、mapred-site.xml

1. 用途
   MR守护进程的配置，如：JobHistoryServer。
2. 配置项

##### 七、yarn-site.xml

1. 用途
   YARN守护进程的配置，如：ResourceManager、NodeManager、ProxyServer。
2. 配置项

##### 八、slaves

1. 用途
   DataNode和NodeManager的机器列表。
2. 配置项

##### 九、hadoop-metrics2.properties

1. 用途
   Properties for controlling how metrics are published in Hadoop。
2. 配置项

##### 十、log4j.pzroperties

1. 用途
   系统日志文件、NameNode的Audit日志、任务日志的的属性。
2. 配置项

##### 十一、hadoop-policy.xml

1. 用途
   Hadoop在安全模式下运行时，该配置文件用于控制访问列表。
2. 配置项