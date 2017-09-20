## Hive常见的数据导入方式

参考文档：[HIVE的四种数据导入方式](https://yq.aliyun.com/articles/100910?spm=5176.10017275.843861.53.9aaSS4)

1. 从本地文件导入到Hive表中；
2. 从HDFS上导入到Hive表中；
3. 从别的表中查询出相应的数据并导入到表中；
4. Hive多表插入；
5. 创建表的时候通过从别的表中查询出相应的记录并插入到表中，即表和数据是在一条SQL语句中完成的。

##### 一、从本地文件导入到Hive表中

```shell
# 建表语句
hive> create table t1 (id int,name string,age int) row format delimited fields terminated by ',' lines terminated by '\n' STORED AS TEXTFILE;

# /tmp/sample.txt文件内容
linux>  cat /tmp/sample.txt
1,Technical manager,10
2,Proof reader,20
3,Wing,30

# 导入本地文件/tmp/sample.txt至t1表中
hive> load data local inpath '/tmp/sample.txt' into table t1;
No rows affected (0.325 seconds)
hive> select * from t1;
+--------+--------------------+---------+
| t1.id  |      t1.name       | t1.age  |
+--------+--------------------+---------+
| 1      | Technical manager  | 10      |
| 2      | Proof reader       | 20      |
| 3      | Wing               | 30      |
+--------+--------------------+---------+
3 rows selected (0.119 seconds)

# 注意
# 此时只是将本地文件的数据插入到hive表中，本地文件sample.txt仍然在本地目录/tmp上，并不是被移动到HDFS上。
# 此时可以再进行一次相同的load，会发现数据依然可以被加载到t1去
hive> load data local inpath '/tmp/sample.txt' into table t1;
hive> select * from t1;
+--------+--------------------+---------+
| t1.id  |      t1.name       | t1.age  |
+--------+--------------------+---------+
| 1      | Technical manager  | 10      |
| 2      | Proof reader       | 20      |
| 3      | Wing               | 30      |
| 1      | Technical manager  | 10      |
| 2      | Proof reader       | 20      |
| 3      | Wing               | 30      |
+--------+--------------------+---------+
6 rows selected (0.119 seconds)
```

##### 二、从HDFS上导入到Hive表中

```shell
# 查看Linux上的本地文件
linux>  cat /tmp/sample.txt
1,Technical manager,10
2,Proof reader,20
3,Wing,30

# 将Linux上的本地文件上传到HDFS上
linux>  hdfs dfs -put /tmp/sample.txt /tmp

# 建表语句
hive> create table t2 (id int,name string,age int) row format delimited fields terminated by ',' lines terminated by '\n' STORED AS TEXTFILE;

# 导入HDFS文件至t2表中
hive> load data  inpath '/tmp/sample.txt' into table t2;
hive> select * from t2;
+--------+--------------------+---------+
| t2.id  |      t2.name       | t2.age  |
+--------+--------------------+---------+
| 1      | Technical manager  | 10      |
| 2      | Proof reader       | 20      |
| 3      | Wing               | 30      |
+--------+--------------------+---------+
3 rows selected (0.117 seconds)

# 注意
# 此时查看HDFS目录可以发现，原本在HDFS的/tmp目录上的sample.txt文件没有了，反而在e/warehouse/test1.db/t2目录下面多出来了一个sample.txt文件，因为此时的操作是将HDFS上的sample.txt文件移动到了表目录下面。
# 此时可以再进行一次相同的load，会发现报错"件不存在"
hive> load data  inpath '/tmp/sample.txt' into table t2;
Error: Error while compiling statement: FAILED: SemanticException Line 1:18 Invalid path ''/tmp/sample.txt'': No files matching path hdfs://172.16.0.220:9000/tmp/sample.txt (state=42000,code=40000)
```

##### 三、从别的表中查询出相应的数据并导入到表中

```shell
# 建表语句
hive> create table t3(id int,name string) partitioned by (age int) row format DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n' STORED AS TEXTFILE;

# 指定分区插入数据
hive> insert into table t3 partition (age=20) select id,name from t2;
hive> select * from t3;
OK
1	Technical manager	20
2	Proof reader	20
3	Wing	20
Time taken: 0.137 seconds, Fetched: 3 row(s)
# 注意
# 在t2表里面，其实3条记录的age并不全是20，为什么在t3里面3条记录的age全是20呢？hive修改了我的数据吗？？
# 解答：其实，下载t3的数据文件之后，可以发现，在t3的数据文件中并没有age这一栏，只是这个文件记录的所有age都为20而已。

# 动态插入分区
hive> set hive.exec.dynamic.partition.mode=nonstrict;
hive> insert overwrite table t3 partition (age) select id,name,age from t2;
hive> select * from t3;
OK
1	Technical manager	10
2	Proof reader	20
3	Wing	30
Time taken: 0.168 seconds, Fetched: 3 row(s)
# 注意
# 此时可以看到t3和t2长的一模一样了。进入t3的HDFS目录下可以看到3个文件，分别为age=10,age=20,age=30三个文件。这是hive在导入数据的时候已经将数据分区好了。hive是不是很棒棒～
```

##### 四、Hive多表插入

HIve支持扫描一次源表，通过一条SQL将数据插入到多个表中。

```shell
# 创建t4,t5表
hive> create table t4 (id int,name string,age int) row format delimited fields terminated by ',' lines terminated by '\n' STORED AS TEXTFILE;
OK
Time taken: 0.073 seconds
hive> create table t5 (id int,name string,age int) row format delimited fields terminated by ',' lines terminated by '\n' STORED AS TEXTFILE;
OK
Time taken: 0.043 seconds

# 通过一条SQL，同时向t4,t5表插入数据。
hive> from t2
    > insert into table t4 select id,name,age
    > insert into table t5 select id,name,age where age=30;
hive> select * from t4;
OK
1	Technical manager	10
2	Proof reader	20
3	Wing	30
Time taken: 0.068 seconds, Fetched: 3 row(s)
hive> select * from t5;
OK
3	Wing	30
Time taken: 0.128 seconds, Fetched: 1 row(s)
```

##### 五、创建表的时候通过从别的表中查询出相应的记录并插入到表中

即建表和导入数据在一条SQL中完成。

```shell
# 建表+导入数据+不带t3分区字段age
hive> create table t6 as select id,name from t3;
hive> show create table t6;
OK
CREATE TABLE `t6`(
  `id` int,
  `name` string)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://172.16.0.220:9000/user/hive/warehouse/test1.db/t6'
TBLPROPERTIES (
  'transient_lastDdlTime'='1505904374')
Time taken: 0.027 seconds, Fetched: 13 row(s)
hive> select * from t6;
OK
1	Technical manager
2	Proof reader
3	Wing
Time taken: 0.109 seconds, Fetched: 3 row(s)

# 建表+导入数据+带t3分区字段age
hive> create table t7 as select id,name,age from t3;
hive> show create table t7;
OK
CREATE TABLE `t7`(
  `id` int,
  `name` string,
  `age` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://172.16.0.220:9000/user/hive/warehouse/test1.db/t7'
TBLPROPERTIES (
  'transient_lastDdlTime'='1505904594')
Time taken: 0.026 seconds, Fetched: 14 row(s)
hive> select * from t7;
OK
1	Technical manager	10
2	Proof reader	20
3	Wing	30
Time taken: 0.065 seconds, Fetched: 3 row(s)
```



