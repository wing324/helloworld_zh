## 常用软件部署（Debian版）

##### 一、jdk安装

```shell
cd ~
# 解压jdk压缩包
tar -zxvf ~/jdk-7u45-linux-x64.tar.gz -C /usr/local/
cd /usr/local
mv jdk1.7.0_45/ jdk
# 配置jdk环境变量
vim ~/.bashrc
# 添加环境变量
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
# 使环境变量立即生效
source ~/.bashrc

# 验证jdk安装成功方式
java -version
```

##### 二、tomcat安装

```shell
cd ~
# 解压tomcat压缩包
tar -zxvf apache-tomcat-7.0.68.tar.gz -C /usr/local/
cd /usr/local
mv apache-tomcat-7.0.68/ tomcat
# 启动tomcat
./tomcat/bin/start.sh
# 启动成功输出信息
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.

# 验证tomcat启动成功方式
netstat -npl | grep 'java'
# 输出信息
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      13916/java      
tcp        0      0 127.0.0.1:8005          0.0.0.0:*               LISTEN      13916/java      
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      13916/java
```

##### 三、zookeeper安装

```shell
tar -zxvf zookeeper-3.4.10.tar.gz -C /usr/local/
cd /usr/local/
mv zookeeper-3.4.10 zookeeper
cd zookeeper
cp conf/zoo_sample.cfg conf/zoo.cfg
vim conf/zoo.cfg
# 修改dataDir=/data/zk
# 增加节点：server.1=ip:2888:3888
# 增加节点：server.2=ip:2888:3888
# 创建zk的数据目录
mkdir /data/zk
cd /data/zk
echo $id > myid
# 如server1，此处需要做的神操作是：echi '1' > myid

# 启动zookeeper
bin/zkServer.sh start
# 启动成功输出信息
# ZooKeeper JMX enabled by default
# Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
# Starting zookeeper ... STARTED

# 查看zookeeper当前的状态
bin/zkServer.sh status
# 输出信息
# ZooKeeper JMX enabled by default
# Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
# Mode: follower

```



