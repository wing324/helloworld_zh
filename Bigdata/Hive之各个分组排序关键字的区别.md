## Hive之各个分组排序关键字的区别

#####  一、Group by  

按照Distribute  by 后的字段分组。

##### 二、Distribute by 

按照Distribute by后的字段分组，相似与Group by。将map中输出相似的数据分配到同一个reduce中。

##### 三、Order by

按照Order by后的字段排序（全局排序），在Hive的严格模式下，Order by需要与Limit一起使用。  
Hive使用Order by的时候会设置reduce=1，在一个reduce里面进行排序，当然是个全局排序啦，但该方法效率低下。

##### 四、 Sort by

按照Sort by后的字段排序（部分排序，只对每个reduce里面进行排序[mapred.reduce.tasks>1的前提下]）,它在数据进入reducer前就完成了排序。

##### 五、Cluster by  

对同一个字段的相似的数据分发到同一个reduce task中，并保证该reduce task中数据有序输出。当distributeb by 和sort by的字段相同时：cluster by = distribute by + sort by。但是排序只能是ASC排序，不能指定排序规则，即不能使用DESC进行排序

##### 六、如何使用sort by实现全局排序

先sort by 在order by :`(distribute by x sort by y) order by y`，例如：

```sql
hive> select a.* from (select * from test cluster by id ) a order by a.id ;
MapReduce Jobs Launched: 
Job 0: Map: 1  Reduce: 3   Cumulative CPU: 4.5 sec   HDFS Read: 305 HDFS Write: 448 SUCCESS
Job 1: Map: 1  Reduce: 1   Cumulative CPU: 1.96 sec   HDFS Read: 1232 HDFS Write: 32 SUCCESS
Total MapReduce CPU Time Spent: 6 seconds 460 msec
OK
1	a
1	a
2	b
2	b
3	c
3	c
4	d
4	d
Time taken: 118.261 seconds, Fetched: 8 row(s)
```



