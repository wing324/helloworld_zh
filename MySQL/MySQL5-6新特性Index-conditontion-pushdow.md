## MySQL5.6新特性Index conditontion pushdow

index condition pushdown是MySQL5.6的新特性，主要是对MySQL索引使用的优化。  


Index condition push简称ICP，索引条件下推，将索引条件从server层下推到storage engine层。  
![](/img/ICP1.png)  
此时出现ICP现象，但ICP到底是个什么现象呢。  
1、ICP的开启和关闭方法：  
set optimizer_switch=’index_condition_pushdown=on|off’  
2、ICP使用限制：  
ICP适用于MyISAM和InnoDB表，对于InnoDB表只能用于二级索引。  
MySQL5.6版本的分区表还没有ICP功能，官网文档介绍将于5.7版本中解决。  
3、ICP的目的在于减少完整读取一条记录的次数，从而减少IO的操作。  
4、
当优化器没有使用ICP的时候，数据的访问和提取：  
当storage engine层读取记录的时候，首先使用索引定位并读取整条记录，server层再根据where条件决定该条记录是否使用。  
![](/img/ICP2.png)
当优化器使用ICP的时候，数据的访问和提取：  
当storage engine层读取记录的时候，首先测试where条件中索引列（由server层下推到storage engine层的索引条件）决定该条记录是否可用，如果可用，则索引定位的记录完整读取出来，让server层根据where条件决定该条记录是否使用。  
![](/img/ICP3.png)  

5、实例演示：  
首先有如下表结构：  

```sql
CREATE TABLE `icp_test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `code` int(11) DEFAULT NULL,
  `name` varchar(16) DEFAULT NULL,
  `address` varchar(16) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_c_n_a` (`code`,`name`,`address`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

分析如下操作：  

```sql
select * from icp_test where code=18 and name like '%A%' ;
select * from icp_test where code=18 and name like '%A%' ;
```
做如下分析：  
优化器在没有ICP的条件下：  
select * from icp_test where code=18 and name like '%A%' ;  
![](/img/ICP4.png)  
优化器在有ICP的条件下：  
![](/img/ICP5.png)  

分析：  
在没有ICP的条件下，虽然有idx_c_n_a(code,name.address)索引，但由于where条件中name like ‘%A%’不使用索引，所以storage engine层只能使用code=18过滤得到4条记录返回到server层，此时server层根据name like ‘%A%’得到两条符合条件的记录返回客户端。  

在有ICP的条件下，有idx_c_n_a(code,name,address)索引，where条件中name like ‘%A%’不使用索引，所以storage engine层只能使用code=18过滤得到4条记录，但此时不立即返回server层，由于MySQL5.6版本的索引条件下推（ICP）的特性，stora engine层还会根据索引中的name列name like ‘%A%’过滤得到2条记录返回客户端，此时没有非索引列的where条件，最终返回客户端。  