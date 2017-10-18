## Hive函数大全

参考《Apache Hive Cookbook》。

一、分析型函数

```sql
# 示例表结构
CREATE TABLE `sales`(
  `id` int,
  `fname` string,
  `state` string,
  `zip` int,
  `ip` string,
  `pid` string)
  
# 示例数据
0	Zena	Tennessee	21550	192.168.56.101	PI_09
1	Elaine	Alaska	6429	192.168.56.101	PI_03
2	Sage	Nevada	8899	192.168.56.102	PI_03
3	Cade	Missouri	11233	192.168.56.103	PI_06
4	Abra	New Jersry	21500	192.168.56.101	PI_09
5	Stone	Nebraska	3560	192.168.56.104	PI_08
6	Regina	Tennessee	21550	192.168.56.105	PI_10
7	Donova	New York	95234	192.168.56.106	PI_05
8	Aileen	Illinois	68284	192.168.56.106	PI_02
9	Maraam	Hawaii	95234	192.168.56.107	PI_07
```

1. ROW_NUMBER
   语法：

   - ROW_NUMBER() OVER (ORDER BY col)
     为每个分组记录返回一个排序的数字。
   - ROW_NUMBER() OVER (PARTITION BY col1 ORDER BY col2)
     按照col1分组，在分组内对col2进行排序并返回顺序数字。

   ```sql
   hive> select fname,pid,ip  from sales;
   OK
   Zena	PI_09	192.168.56.101
   Elaine	PI_03	192.168.56.101
   Sage	PI_03	192.168.56.102
   Cade	PI_06	192.168.56.103
   Abra	PI_09	192.168.56.101
   Stone	PI_08	192.168.56.104
   Regina	PI_10	192.168.56.105
   Donova	PI_05	192.168.56.106
   Aileen	PI_02	192.168.56.106
   Maraam	PI_07	192.168.56.107

   hive> select fname,pid,ip,row_number() over (order by ip) from sales;
   Abra	PI_09	192.168.56.101	1
   Elaine	PI_03	192.168.56.101	2
   Zena	PI_09	192.168.56.101	3
   Sage	PI_03	192.168.56.102	4
   Cade	PI_06	192.168.56.103	5
   Stone	PI_08	192.168.56.104	6
   Regina	PI_10	192.168.56.105	7
   Aileen	PI_02	192.168.56.106	8
   Donova	PI_05	192.168.56.106	9
   Maraam	PI_07	192.168.56.107	10

   hive> select fname,pid,ip,row_number() over (partition by pid order by ip) from sales;
   Aileen	PI_02	192.168.56.106	1
   Elaine	PI_03	192.168.56.101	1
   Sage	PI_03	192.168.56.102	2
   Donova	PI_05	192.168.56.106	1
   Cade	PI_06	192.168.56.103	1
   Maraam	PI_07	192.168.56.107	1
   Stone	PI_08	192.168.56.104	1
   Abra	PI_09	192.168.56.101	1
   Zena	PI_09	192.168.56.101	2
   Regina	PI_10	192.168.56.105	1
   ```

2. RANK

   - RANK() OVER (ORDER BY col)
     和ROW_NUMBER()差不多，但是排序时候相同的字段会返回相同的数字。
   - RANK() OVER (PARTITION by col1 ORDER BY col2)
     和ROW_NUMBER()差不多，但是排序时候同等级的字段会返回相同的数字。

   ```sql
   hive> select fname,ip,rank() over (order by ip) from sales;
   Abra	192.168.56.101	1
   Elaine	192.168.56.101	1
   Zena	192.168.56.101	1
   Sage	192.168.56.102	4
   Cade	192.168.56.103	5
   Stone	192.168.56.104	6
   Regina	192.168.56.105	7
   Aileen	192.168.56.106	8
   Donova	192.168.56.106	8
   Maraam	192.168.56.107	10

   hive> select fname,pid,ip,rank() over (partition by pid order by ip) from sales;
   Aileen	PI_02	192.168.56.106	1
   Elaine	PI_03	192.168.56.101	1
   Sage	PI_03	192.168.56.102	2
   Donova	PI_05	192.168.56.106	1
   Cade	PI_06	192.168.56.103	1
   Maraam	PI_07	192.168.56.107	1
   Stone	PI_08	192.168.56.104	1
   Abra	PI_09	192.168.56.101	1
   Zena	PI_09	192.168.56.101	1
   Regina	PI_10	192.168.56.105	1
   ```

3. DENSE_RANK
   和RANK()差不多，但是RANK()的排序数字存在空洞，见一-2-第一个示例。而DENSE_RANK()则不会存在排序数字空洞。

   ```sql
   hive> select fname,ip,dense_rank() over (order by ip) from sales;
   Abra	192.168.56.101	1
   Elaine	192.168.56.101	1
   Zena	192.168.56.101	1
   Sage	192.168.56.102	2
   Cade	192.168.56.103	3
   Stone	192.168.56.104	4
   Regina	192.168.56.105	5
   Aileen	192.168.56.106	6
   Donova	192.168.56.106	6
   Maraam	192.168.56.107	7
   ```

4. PERCENT_RANK

   - PERCENT_RANK() OVER (ORDER BY col)  当前行的rank-1/总行数-1
   - PERCENT_RAANK() OVER (PARTITION BY col1 ORDER BY col2) 按照col1分组后，分组内当前行的rank-1/分组内的总行数-1

   ```sql
   hive> select id,ip,percent_rank() over (order by id)from sales;
   0	192.168.56.101	0.0
   1	192.168.56.101	0.1111111111111111
   2	192.168.56.102	0.2222222222222222
   3	192.168.56.103	0.3333333333333333
   4	192.168.56.101	0.4444444444444444
   5	192.168.56.104	0.5555555555555556
   6	192.168.56.105	0.6666666666666666
   7	192.168.56.106	0.7777777777777778
   8	192.168.56.106	0.8888888888888888
   9	192.168.56.107	1.0

   hive> select id,ip,percent_rank() over (partition by ip order by id)from sales;
   0	192.168.56.101	0.0
   1	192.168.56.101	0.5
   4	192.168.56.101	1.0
   2	192.168.56.102	0.0
   3	192.168.56.103	0.0
   5	192.168.56.104	0.0
   6	192.168.56.105	0.0
   7	192.168.56.106	0.0
   8	192.168.56.106	1.0
   9	192.168.56.107	0.0
   ```

   ​

5. CUME_DIST

   - CUME_DIST() OVER (ORDER BY col) 小于等于col当前值的行数/总行数。
   - CUME_DIST() OVER (PARTITION BY col1 ORDER BY col2) 按照col1分组后，分组内部小于等于col2当前值的行数/总行数。

   ```sql
   hive> select id,ip from sales;
   OK
   0	192.168.56.101
   1	192.168.56.101
   2	192.168.56.102
   3	192.168.56.103
   4	192.168.56.101
   5	192.168.56.104
   6	192.168.56.105
   7	192.168.56.106
   8	192.168.56.106
   9	192.168.56.107

   hive> select id,ip,cume_dist() over (order by id)from sales;
   0	192.168.56.101	0.1
   1	192.168.56.101	0.2
   2	192.168.56.102	0.3
   3	192.168.56.103	0.4
   4	192.168.56.101	0.5
   5	192.168.56.104	0.6
   6	192.168.56.105	0.7
   7	192.168.56.106	0.8
   8	192.168.56.106	0.9
   9	192.168.56.107	1.0

   hive> select id,ip,cume_dist() over (partition by ip order by id)from sales;
   0	192.168.56.101	0.3333333333333333
   1	192.168.56.101	0.6666666666666666
   4	192.168.56.101	1.0
   2	192.168.56.102	1.0
   3	192.168.56.103	1.0
   5	192.168.56.104	1.0
   6	192.168.56.105	1.0
   7	192.168.56.106	0.5
   8	192.168.56.106	1.0
   9	192.168.56.107	1.0
   ```

6. NTILE
   用于将分组数据按照顺序切分成n片，并返回当前切片值。

   ```sql
   hive> select id,ip, ntile(2) over (partition by ip order by id)from sales;
   0	192.168.56.101	1
   1	192.168.56.101	1
   4	192.168.56.101	2
   2	192.168.56.102	1
   3	192.168.56.103	1
   5	192.168.56.104	1
   6	192.168.56.105	1
   7	192.168.56.106	1
   8	192.168.56.106	2
   9	192.168.56.107	1
   ```

   ​