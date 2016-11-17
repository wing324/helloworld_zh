## MySQL for Python

作为爱上python的MySQL DBA来说，可以使用python操作MySQL是在是一件太过瘾的事情，但是一段时间不用MySQLdb的库，就会忘记怎么使用python操作MySQL了，所以记忆力太差，我就只能把我的MySQLdb操作记录记载下来，便于以后的快速使用和学习啦  


```mysql
# MySQL中的数据
root@localhost : menu 06:16:42> select * from fish;
+----+---------------+-------+
| ID | NAME          | PRICE |
+----+---------------+-------+
|  1 | catfish       |  8.50 |
|  2 | catfish       |  8.50 |
|  3 | tuna          |  8.00 |
|  4 | catfish       |  5.00 |
|  5 | bass          |  6.75 |
|  6 | haddock       |  6.50 |
|  7 | salmon        |  9.50 |
|  8 | trout         |  6.00 |
|  9 | tuna          |  7.50 |
| 10 | yellowfintuna | 12.00 |
| 11 | yellowfintuna | 13.00 |
| 12 | tuna          |  7.50 |
+----+---------------+-------+
12 rows in set (0.26 sec)
```

```python
# python中的操作
[root@localhost python]# python
Python 2.7.5 (default, Jun 17 2014, 18:11:42) 
[GCC 4.8.2 20140120 (Red Hat 4.8.2-16)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import tab
>>> import MySQLdb

# python与MySQL建立连接
>>> mydb = MySQLdb.connect(user='root',host='127.0.0.1',port=3306,db='menu')
>>> cur=mydb.cursor()

# 在python中执行sql语句
>>> command = cur.execute('select * from fish')

# 获取sql语句执行的结果
>>> results = cur.fetchall()
>>> print results
((1L, 'catfish', Decimal('8.50')), (2L, 'catfish', Decimal('8.50')), (3L, 'tuna', Decimal('8.00')), (4L, 'catfish', Decimal('5.00')), (5L, 'bass', Decimal('6.75')), (6L, 'haddock', Decimal('6.50')), (7L, 'salmon', Decimal('9.50')), (8L, 'trout', Decimal('6.00')), (9L, 'tuna', Decimal('7.50')), (10L, 'yellowfintuna', Decimal('12.00')), (11L, 'yellowfintuna', Decimal('13.00')), (12L, 'tuna', Decimal('7.50')))

# 对sql语句的结果进行友好式的处理
>>> for record in results:                                  
...     print record[0],'-->',record[1],'@',record[2],'each'
... 
1 --> catfish @ 8.50 each
2 --> catfish @ 8.50 each
3 --> tuna @ 8.00 each
4 --> catfish @ 5.00 each
5 --> bass @ 6.75 each
6 --> haddock @ 6.50 each
7 --> salmon @ 9.50 each
8 --> trout @ 6.00 each
9 --> tuna @ 7.50 each
10 --> yellowfintuna @ 12.00 each
11 --> yellowfintuna @ 13.00 each
12 --> tuna @ 7.50 each

# 在sql语句中添加变量
>>> values=7.50  
>>> command = cur.execute('select * from fish where price = %f'%(values))
>>> results = cur.fetchall()
>>> print results
((9L, 'tuna', Decimal('7.50')), (12L, 'tuna', Decimal('7.50')))
>>> for record in results:                               
...     print record[0],'-->',record[1],'(',record[2],')'
... 
9 --> tuna ( 7.50 )
12 --> tuna ( 7.50 )

# 交互式执行sql语句
>>> operation = raw_input('operation:')
operation:=
>>> values = float(raw_input('values:'))
values:7.50
>>> command = cur.execute('select * from fish where price %s %f'%(operation,values))
>>> results = cur.fetchall()
>>> print results
((9L, 'tuna', Decimal('7.50')), (12L, 'tuna', Decimal('7.50')))
>>> for record in results:
...     print record[0],'-->',record[1],'(',record[2],')'
... 
9 --> tuna ( 7.50 )
12 --> tuna ( 7.50 )

# 关闭游标
>>> cur.close()

# 提交sql操作
>>> mydb.commit()

# 关闭MySQL连接
>>> mydb.close()
```

```sql
# 在python建立连接后，在MySQL执行show processlist可以看到的连接变化，多增加了id=6的连接
root@localhost : (none) 06:28:09> show processlist;
+----+-----------------+-----------------+------+---------+------+------------------------+------------------+
| Id | User            | Host            | db   | Command | Time | State                  | Info             |
+----+-----------------+-----------------+------+---------+------+------------------------+------------------+
|  1 | event_scheduler | localhost       | NULL | Daemon  |  762 | Waiting on empty queue | NULL             |
|  3 | root            | localhost       | menu | Sleep   |  721 |                        | NULL             |
|  5 | root            | localhost       | NULL | Query   |    0 | init                   | show processlist |
|  6 | root            | 127.0.0.1:33085 | menu | Sleep   |    1 |                        | NULL             |
+----+-----------------+-----------------+------+---------+------+------------------------+------------------+
4 rows in set (0.00 sec)

# 在python中执行mydb.close()后，在MySQL执行show processlist可以看到的连接变化，id=6的连接消失了
root@localhost : (none) 06:28:47> show processlist;
+----+-----------------+-----------+------+---------+------+------------------------+------------------+
| Id | User            | Host      | db   | Command | Time | State                  | Info             |
+----+-----------------+-----------+------+---------+------+------------------------+------------------+
|  1 | event_scheduler | localhost | NULL | Daemon  | 2544 | Waiting on empty queue | NULL             |
|  3 | root            | localhost | menu | Sleep   | 2503 |                        | NULL             |
|  5 | root            | localhost | NULL | Query   |    0 | init                   | show processlist |
+----+-----------------+-----------+------+---------+------+------------------------+------------------+
3 rows in set (0.00 sec)

# 在python中获取数据库的所有表
>>> cur.close()
>>> mydb = MySQLdb.connect(user='root',host='127.0.0.1',port=3306,db='menu')
>>> cur=mydb.cursor()
>>> command=cur.execute('show tables')
>>> results = cur.fetchall()
>>> print results
(('fish',), ('meat',))
>>> table_list=[]
>>> for record in results:
...     table_list.append(record[0])
... 	#此处注意使用record[0],因为这里的每个reocrd原本都是一个元组
>>> print table_list
['fish', 'meat']
## 或者 当前操作是从1开始计数，因为做了特殊处理，如果不做特殊处理，字典都是从0开始计数的
>>> item_dict={}
>>> for item in xrange(1,len(table_list)+1):
...      item_dict[item]=table_list[item-1]
... 
>>> print item_dict
{1: 'fish', 2: 'meat'}

# 在python中交互式友好的让用户选择需要操作的表
>>> for key in item_dict:                     
...     print '%s ==> %s'%(key,item_dict[key])
... 
1 ==> fish
2 ==> meat
>>> choice = input('please enter your choice of table to be query:')
please enter your choice of table to be query:1

# 验证表确实存在与数据库中
>>> choice = input('please enter your choice of table to be query:')
please enter your choice of table to be query:1
>>> try:
...     table_choice=item_dict[choice]
... except:
...     "error"
... 
>>> choice = input('please enter your choice of table to be query:')
please enter your choice of table to be query:5
>>> try: 
...     table_choice=item_dict[choice]
... except:
...     "error"
... 
'error'

# 显示表中列的信息
>>> sql1 = 'desc %s'%(item_dict[chioce])
>>> command = cur.execute(sql1)
>>> results = cur.fetchall()
>>> print results
(('ID', 'int(11)', 'NO', 'PRI', None, 'auto_increment'), ('NAME', 'varchar(30)', 'NO', '', '', ''), ('PRICE', 'decimal(5,2)', 'NO', '', '0.00', ''))
>>> column_list=[]
>>> for record in results:
...      column_list.append(record[0])
... 
>>> print column_list
['ID', 'NAME', 'PRICE']
```


**注意**  
1、 当数据库中的数据更新后，原来的连接时感知不到数据库的变化，数据更新后的连接才能接受到数据更新后的数据。  
2、 此处的record中的每个元素实际上都是一个元组，所以取值的时候需要用record[0],而不是用record  
```
>>> for record in results:
...     table_list.append(record[0])

>>> for record in results:
...     print type(record)
... 
<type 'tuple'>
<type 'tuple'>
```