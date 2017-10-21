## Hive实用函数大全

为什么叫实用函数，因为有些不实用的函数没有写进文档中，哈哈。

参考文档：

《Apache Hive Cookbook》

[Hive分析窗口函数(一) SUM,AVG,MIN,MAX](http://lxw1234.com/archives/2015/04/176.htm)

```sql
# 示例表结构sales
CREATE TABLE `sales`(
  `id` int,
  `fname` string,
  `state` string,
  `zip` int,
  `ip` string,
  `pid` string)
  
# 示例数据sales
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

# 示例表结构po
CREATE TABLE `po`(
  `cookieid` string,
  `createtime` string,
  `pv` int)
  
# 示例数据po
cookie1	2015-04-10	1
cookie1	2015-04-11	5
cookie1	2015-04-12	7
cookie1	2015-04-13	3
cookie1	2015-04-14	2
cookie1	2015-04-15	4
cookie1	2015-04-16	5
```

##### 一、分析型函数

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

##### 二、窗口型函数

1. LEAD
   - LEAD() OVER (PARTITION BY col1 ORDER BY col2) 按照col1分组后，返回结果集中的下一个col2
2. LAG
   - OVER (PARTITION BY col1 ORDER BY col2) 按照col1分组后，返回结果集中的上一个col2
3. FIRST_VALUE
   - OVER (PARTITION BY col1 ORDER BY col2) 按照col1分组后，返回结果集中的第一个col2
4. LAST_VALUE
   - OVER (PARTITION BY col1 ORDER BY col2) 按照col1分组后，返回结果集中的最后一个col2
5. OVER
   - 聚集OVER
   - COUNT
   - MIN
   - MAX
   - AVG
   - OVER WITH PARTITION BY
   - OVER WITH PARTITION BY and ORDER BY

```sql
# 首先需要了解函数OVER(PARTITION BY col1 ORDER BY col2)
SELECT cookieid,
createtime,
pv,
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime) AS pv1, -- 默认为从起点到当前行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS pv2, --从起点到当前行，结果同pv1 
SUM(pv) OVER(PARTITION BY cookieid) AS pv3,								--分组内所有行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) AS pv4,   --当前行+往前3行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN 3 PRECEDING AND 1 FOLLOWING) AS pv5,    --当前行+往前3行+往后1行
SUM(pv) OVER(PARTITION BY cookieid ORDER BY createtime ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS pv6   ---当前行+往后所有行  
FROM po;
cookie1	2015-04-16	5	27	27	27	14	14	5
cookie1	2015-04-15	4	22	22	27	16	21	9
cookie1	2015-04-14	2	18	18	27	17	21	11
cookie1	2015-04-13	3	16	16	27	16	18	14
cookie1	2015-04-12	7	13	13	27	13	16	21
cookie1	2015-04-11	5	6	6	27	6	13	26
cookie1	2015-04-10	1	1	1	27	1	6	27
# 其他聚集函数方式相同。

# LEAD
hive> select fname,ip,pid,lead(pid) over (partition by ip order by ip) from sales;
Abra	192.168.56.101	PI_09	PI_03 -- ip的分组中下一个pid的值为PI_03
Elaine	192.168.56.101	PI_03	PI_09
Zena	192.168.56.101	PI_09	NULL
Sage	192.168.56.102	PI_03	NULL
Cade	192.168.56.103	PI_06	NULL
Stone	192.168.56.104	PI_08	NULL
Regina	192.168.56.105	PI_10	NULL
Aileen	192.168.56.106	PI_02	PI_05
Donova	192.168.56.106	PI_05	NULL
Maraam	192.168.56.107	PI_07	NULL

# LAG
hive> select fname,ip,pid,lag(pid) over (partition by ip order by ip) from sales;
Abra	192.168.56.101	PI_09	NULL  
Elaine	192.168.56.101	PI_03	PI_09  -- ip的分组中上一个pid的值为PI_09
Zena	192.168.56.101	PI_09	PI_03
Sage	192.168.56.102	PI_03	NULL
Cade	192.168.56.103	PI_06	NULL
Stone	192.168.56.104	PI_08	NULL
Regina	192.168.56.105	PI_10	NULL
Aileen	192.168.56.106	PI_02	NULL
Donova	192.168.56.106	PI_05	PI_02
Maraam	192.168.56.107	PI_07	NULL

# FIRST_VALUE
hive> select fname,ip,pid,first_value(pid) over (partition by ip order by ip) from sales;
Abra	192.168.56.101	PI_09	PI_09  -- ip的分组中第一个pid的值为PI_09
Elaine	192.168.56.101	PI_03	PI_09
Zena	192.168.56.101	PI_09	PI_09
Sage	192.168.56.102	PI_03	PI_03
Cade	192.168.56.103	PI_06	PI_06
Stone	192.168.56.104	PI_08	PI_08
Regina	192.168.56.105	PI_10	PI_10
Aileen	192.168.56.106	PI_02	PI_02  -- ip的分组中第一个pid的值为PI_02
Donova	192.168.56.106	PI_05	PI_02
Maraam	192.168.56.107	PI_07	PI_07

# LAST_VALUE
hive> select fname,ip,pid,last_value(pid) over (order by ip) from sales;
Abra	192.168.56.101	PI_09	PI_09 -- ip的分组中最后一个pid的值为PI_09
Elaine	192.168.56.101	PI_03	PI_09
Zena	192.168.56.101	PI_09	PI_09
Sage	192.168.56.102	PI_03	PI_03
Cade	192.168.56.103	PI_06	PI_06
Stone	192.168.56.104	PI_08	PI_08
Regina	192.168.56.105	PI_10	PI_10
Aileen	192.168.56.106	PI_02	PI_05
Donova	192.168.56.106	PI_05	PI_05 -- ip的分组中最后一个pid的值为PI_05
Maraam	192.168.56.107	PI_07	PI_07
```

##### 二、数值型函数

1. abs(x)
   返回x的绝对值。

   ```sql
   hive> select abs(-1);
   1

   hive> select abs(1);
   1

   hive> select abs(0);
   0
   ```

2. bin(x)
   返回x的二进制格式。

   ```sql
   hive> select bin(123);
   1111011

   hive> select bin(2);
   10

   hive> select bin(3);
   11
   ```

3. ceil(x),ceiling(x)
   返回x向上取整整数。

   ```sql
   hive> select ceil(10.01);
   11

   hive> select ceil(0.01);
   1

   hive> select ceil(-10.01);
   -10
   ```

4. conv(x,y,z)
   进制转换，将x从y进制转换为z进制。

   ```sql
   # 将10从十进制转换为二进制
   hive> select conv(10,10,2);
   1010
   ```

5. floor(x)
   返回x向下取整整数。

   ```sql
   hive> select floor(10.01);
   10

   hive> select floor(11.01);
   11

   hive> select floor(11.77);
   11
   ```

6. greatest(x,y,z...)
   返回x,y,z....中数值最大的值。

   ```sql
   hive> select greatest(1,8,2,3,-10,-12);
   8
   ```

7. least(x,y,z,...)
   返回x,y,z...中数值最小的值。

   ```sql
   hive> select least(1,8,2,3,-10,-12);
   -12
   ```

8. negative(x)
   返回x的负值。

   ```sql
   hive> select negative(1);
   -1
   hive> select negative(-1);
   1
   ```

9. round(x)
   x取整。

   ```sql
   hive> select round(10.12);
   10

   hive> select round(10.92);
   11

   hive> select round(10);
   10

   hive> select round(10.5);
   11
   ```

10. sign(x)
    当x是正数时返回1，当x是负数时返回-1，当x是0时返回0。

##### 三、类型转换型函数

1. binary(x)
   将x以二进制的方式存储。

2. cast(x as T)
   将x转换为T类型。

   ```sql
   hive> select cast(10 as STRING);
   10

   hive> select cast(10 as INT);
   10

   hive> select cast('A' as INT);
   NULL
   ```

##### 四、日期函数

1. add_month(x,y)
   在日期x的基础上加上y月。

   ```sql
   hive> select add_months('2017-10-09',1);
   2017-11-09

   hive> select add_months('2017-10-09',4);
   2018-02-09
   ```

2. current_date()
   获取当前日期。

   ```sql
   hive> select current_date();
   OK
   2017-10-20
   ```

3. current_timestamp()
   获取当前时间。

   ```sql
   hive> select current_timestamp();
   2017-10-20 18:59:52.19
   ```

4. date_add(x,y)
   在日期x的基础上加上y天。

   ```sql
   hive> select date_add('2017-10-09',4);
   2017-10-13

   hive> select date_add('2017-12-30',4);
   2018-01-03
   ```

5. date_format(x,y)
   将日期x的格式化为y形式的时间。
   y的格式[请点击参考](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html)。

6. date_sub(x,y)
   在日期x的基础上减去y天。

7. datediff(x,y)
   返回日期x,y之间的时间差(天数)。

   ```sql
   hive> select datediff('2017-10-09','2017-10-01');
   8
   ```

8. unix_timestamp()
   获取当前时区的UNIX时间戳

9. from_unixtime(x)
   返回时间戳x的直观日期。

   ```sql
   hive> select from_unixtime(unix_timestamp());
   2017-10-20 19:13:11
   ```

10. last_day(x)
    返回日期x的月份的最后一天的日期。

    ```sql
    hive> select last_day('2017-02-02');
    2017-02-28
    ```

11. months_between(x,y)
    返回x,y之间的月份差。

    ```sql
    hive> select months_between('2017-10-01','2017-02-02');
    OK
    7.96774194
    ```

##### 四、字符型函数

1. concat(x,y,….)
   连接x,y....字符串合并为一个字符串。

   ```sql
   hive> select concat('ABC','abc','zzz');
   ABCabczzz
   ```

2. concat_ws(z,x,y,….)
   实用分隔符z连接x,y....字符串。

   ```sql
   hive> select concat_ws('/','ABC','abc','zzz');
   ABC/abc/zzz
   ```

3. find_in_set('x','y,z,...')
   检查x是否存在与y,z...中，存在则返回位置值，否则返回0。

   ```sql
   hive> select find_in_set('abc','a,b,c,ab,abc,bc');
   5

   hive> select find_in_set('abc','a,b,c,ab,bc');
   0
   ```

4. in_file(x,y)
   检查字符串x是否为文件y的一行。

5. initcap(x)
   将字符串x的首字母大写，然后将其他字母小写。

   ```sql
   hive> select initcap('hello');
   Hello

   hive> select initcap('hellO');
   Hello

   hive> select initcap('hELLO');
   Hello

   hive> select initcap('HeLLO');
   Hello

   hive> select initcap('HELLO');
   Hello
   ```

6. instr(x,y)
   返回y在x中的第一个位置值。

   ```sql
   hive> select instr('abc','c');
   3

   hive> select instr('abc','d');
   0
   ```

7. length(x)
   返回x的字符个数

   ```sql
   hive> select length('abc');
   3
   hive> select length('中国');
   2
   ```

8. lower(x),lcase(x)
   返回x的小写字符串。

   ```sql
   hive> select lower('ABC');
   abc
   ```

9. locate(x,y,z)
   返回x在y的位置z之后第一次出现的位置。

10. lpad(x,y,z)
    将x左侧用字符串z填充，xz组合的总长度为z

    ```sql
    hive> select lpad('a',2,'b');
    ba

    hive> select lpad('a',3,'b');
    bba

    hive> select lpad('aaaa',3,'b');
    aaa
    ```

11. ltrim(x)
    去除x左侧的空格。

    ```sql
    hive> select ltrim('  AA');
    AA
    ```

12. repeat(x,y)
    将x重复y次。

    ```sql
    hive> select repeat('a',5);
    aaaaa
    ```

13. reverse(x)
    将x逆向输出。

    ```sql
    hive> select reverse('SUV');
    VUS
    ```

14. rpad(x,y,z)
    将x左侧用字符串z填充，xz组合的总长度为z。

15. rtrim(x)
    去除x右侧的空格。

16. split(x,y)
    用y分割字符串x,y为正则表达式。

17. substr(x,y),substring(x,y)
    返回x从位置y直到结尾的字符串。

18. substr(x,y,z),substring(x,y,z)
    返回x从位置y开始的字符串，长度为z的子串。

19. trim(x)
    去掉x两端的空格。

##### 五、条件函数

1. case when

   ```sql
   # 方式1
   case when a=b then b1 when a=c then c1 else a end as col

   # 方式2
   case a when b then b1 when c then c1 else a end as col
   ```

2. coalsce(x,y,z)
   当x is null时返回y，当x is not null 返回z。

3. if(x,y,z)
   当条件x成立时返回y，当条件x不成立时返回z。

4. isnotnull(x)
   当x is not null时返回true，当x i null 返回false。

5. isnull(x)
   当x is null时返回true，当x is not null 返回false。

6. nvl(x,y)
   当x is null时返回y，当x is not null 返回x。

   ```sql
   hive> select nvl(1,100);
   1

   hive> select nvl(null,100);
   100
   ```

##### 六、UDAF函数

1. avg()
   返回平均值
2. count()
   返回总数
3. max()
   返回最大值
4. min()
   返回最小值
5. sum()
   返回总和
6. variance()
   返回方差