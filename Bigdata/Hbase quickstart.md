## Hbase quickstart

由于Hbase运行在HDFS上，所以部署Hbase之前需要先部署Hadoop,Hadoop部署的三种方式可参考[《Hadoop的三种模式部署-上篇》](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/Hadoop%E7%9A%84%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2-%E4%B8%8A%E7%AF%87.md)和[《Hadoop的三种模式部署-下篇》](https://github.com/wing324/helloworld_zh/blob/master/Bigdata/Hadoop%E7%9A%84%E4%B8%89%E7%A7%8D%E6%A8%A1%E5%BC%8F%E9%83%A8%E7%BD%B2-%E4%B8%8B%E7%AF%87.md)，本文将不在重复Hadoop的部署方式，本次Hbase的部署基于Hadoop完全分布式部署环境的前提下。  

##### 基础信息：  

OS：Debian8.2  
Java： 1.8.0  
Hadoop：2.8.1  
Hbase：2.0.0

##### 一、部署

Hbase下载地址：http://www.apache.org/dyn/closer.cgi/hbase/  

```shell
# 解压Hbase压缩包到指定目录下（/usr/local）
tar -zxvf hbase-2.0.0-alpha3-bin.tar.gz -C /usr/local/
# 修改hbase安装目录名
mv hbase-2.0.0-alpha3/ hbase
# 修改hbase安装目录的归属
chown -R hadoop:hadoop hbase/
# 创建hbase的数据目录
mkdir /data/hbase
# 修改hbase数据目录的归属
chown -R hadoop:hadoop hbase/
# 进入hbase安装目录添加数据目录相关配置
cd /usr/local/hbase/
vim conf/hbase-site.xml 
### 在配置文件中添加如下配置
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>file:///data/hbase/</value>
    </property>
</configuration>
```

至此，hbase的单机模式安装完毕。

##### 二、Hbase启动与关闭

- 启动  
  `/usr/local/hbase/bin/start-hbase.sh`
- 关闭  
  `/usr/local/hbase/bin/stop-hbase.sh`

##### 三、简单操作

- status  
  查看服务器的运行情况。

  ```shell
  hbase(main):012:0> status
  1 active master, 0 backup masters, 1 servers, 0 dead, 2.0000 average load
  ```

- create  
  创建表。    

  ```shell
  hbase(main):013:0> create 'testtable','colfam1'
  Created table testtable
  ```

- list  
  查看当前的表  

  ```shell
  hbase(main):014:0> list 'testtable'
  TABLE              
  testtable                        
  1 row(s)
  Took 0.0278 seconds

  hbase(main):015:0> list
  TABLE                         
  testtable
  1 row(s)
  ```

- put  
  往表中添加数据  

  ```she
  hbase(main):016:0> put 'testtable','myrow-1','colfam1:q1','value-1'
  hbase(main):022:0* put 'testtable','myrow-2','colfam1:q2','value-2'
  hbase(main):024:0> put 'testtable','myrow-2','colfam1:q3','value-3'
  ```

- scan  
  扫描全表  

  ```shell
  hbase(main):027:0> scan 'testtable'
  ROW                                                    COLUMN+CELL                     
   myrow-1                                               column=colfam1:q1, timestamp=1512888329977, value=value-1                                       
   myrow-2                                               column=colfam1:q2, timestamp=1512888450374, value=value-2
   myrow-2                                               column=colfam1:q3, timestamp=1512888464718, value=value-3 
  ```

- get  

  获取表中的某些数据  

  ```shell
  hbase(main):028:0> get 'testtable','myrow-1'
  COLUMN                                                 CELL                    
   colfam1:q1                                            timestamp=1512888329977, value=value-1
   
  hbase(main):029:0> get 'testtable','myrow-2'
  COLUMN                                                 CELL                      
   colfam1:q2                                            timestamp=1512888450374, value=value-2                         
   colfam1:q3                                            timestamp=1512888464718, value=value-3
  ```

- delete  
  删除表中的某些数据  

  ```shell
  hbase(main):031:0> delete 'testtable','myrow-2','colfam1:q2'
  hbase(main):032:0> scan 'testtable'
  ROW                                                    COLUMN+CELL                 
   myrow-1                                               column=colfam1:q1, timestamp=1512888329977, value=value-1
   myrow-2                                               column=colfam1:q3, timestamp=1512888464718, value=value-3
  ```

- disable/enable  
  禁用/启用表  

  ```shell
  # 禁用表
  disable 'testtable'
  # 启动表
  enable 'testtable'
  ```

- drop  
  删除表  

  ```shell
  # 删除表之前需要先禁用表
  disable 'testtable'
  # 删除表
  drop 'testtable'
  ```

  ​