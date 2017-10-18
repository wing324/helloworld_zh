## Hive函数大全

参考《Apache Hive Cookbook》。

一、分析型函数

- RANK
- DENSE_RANK
- ROW_NUMBER
- PERCENT_RANK
- CUME_DIST
- NTILE

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

1. RANK
2. DENSE_RANK
3. PERCENT_RANK
4. CUME_DIST
5. NTILE