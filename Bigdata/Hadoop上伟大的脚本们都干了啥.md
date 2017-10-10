## Hadoop上伟大的脚本们都干了啥

感觉Hadoop上bin/sbin目录下脚本们都挺伟大了，靠着他们维护了一个那么庞大的Hadoop集群。最近正好在学习《Hadoop权威指南》,于是跟着Timwhite学习了一把Hadoop上伟大的脚本们。

##### 一、sbin/start-dfs.sh

1. 作用
   启动HDFS守护进程。
2. sbin/start-dfs.sh脚本启动过程
   - 通过`hdfs getconf -namenodes `获得NameNode的机器列表，并在这些机器上启动NameNode;
   - 在所有etc/hadoop/slaves的机器列表上启动DataNode;
   - 通过`hdfs getconf -secondarynamenodes `获得SecondaryNameNode的机器列表，并在这些机器上启动SecondaryNameNode(如果集群配置了该项的前提下);
   - 通过`hdfs getconf -confKey dfs.namenode.shared.edits.dir`获取JournalNode的机器列表，并在这些机器上启动JournalNode(如果集群配置了该项的前提下)；
   - 通过`hdfs getconf -confKey dfs.ha.automatic-failover.enabled`获取zkfc的机器列表，并在这些机器上启动zkfc(如果集群配置了该项的前提下)。

##### 二、sbin/start-yarn.sh

1. 作用
   启动YARN守护进程。
2. sbin/start-yarn.sh脚本启动过程
   - 在本机上启动ResourceManager;
   - 在所有etc/hadoop/slaves的机器列表上启动NodeManager。

##### 三、sbin/stop-dfs.sh

1. 作用
   关闭HDFS守护进程。
2. sbin/stop-dfs.sh脚本关闭过程
   - 通过`hdfs getconf -namenodes `获得NameNode的机器列表，并在这些机器上关闭NameNode;
   - 在所有etc/hadoop/slaves的机器列表上关闭DataNode;
   - 通过`hdfs getconf -secondarynamenodes `获得SecondaryNameNode的机器列表，并在这些机器上关闭daryNameNode(如果集群配置了该项的前提下);
   - 通过`hdfs getconf -confKey dfs.namenode.shared.edits.dir`获取JournalNode的机器列表，并在这些机器上关闭JournalNode(如果集群配置了该项的前提下)；
   - 通过`hdfs getconf -confKey dfs.ha.automatic-failover.enabled`获取zkfc的机器列表，并在这些机器上关闭zkfc(如果集群配置了该项的前提下)。

#### 四、sbin/stop-yarn.sh

1. 作用
   关闭YARN守护进程。
2. sbin/stop-yarn.sh脚本关闭过程
   - 本机上关闭ResourceManager;
   - 在所有etc/hadoop/slaves的机器列表上关闭NodeManager。

##### 五、后续待补充

