## Oracle的TIMESTAMP的几种类型

Oralce中TIMESTAMP的几种类型。  

TIMESTAMP的几种类型比较
---------------------
###### TIMESTAMP
时间戳类型，与date的区别在于，date不能精确到毫秒，而timestamp可以精确到毫秒，毫秒的位数为0-9位，默认为6位。  
```sql
SQL> select tp from timestamp_test;
TP
--------------------------------------------------------------------------------
01-3月 -16 09.22.33.000000 上午
```
###### TIMESTAMP WITH TIME ZONE
TIMESTAMP WITH TIME ZONE 与 TIMESTAMP的区别在于，前者输出显示携带存入该时间值的数据库时区，后者输出不携带时区。  
```sql
SQL> select tp_tz from timestamp_test;
TP_TZ
--------------------------------------------------------------------------------
01-3月 -16 09.22.33.000000 上午 +08:00
```
###### TIMESTAMP WITH LOCAL TIME ZONE与TIMESTAMP的区别在于，前者的输出受时区影响，会跟着时区的变化而变化，而后者存入数据库后将不受时区影响。即前者以数据库本地时区保存数据，输出时将转换成客户端时区输出。  
```sql
SQL> select tp_l_tz from timestamp_test;
TP_L_TZ
--------------------------------------------------------------------------------
01-3月 -16 09.22.33.000000 上午
```


实战演练
-------
```sql
# 创建timestamp_test测试表
SQL> create table timestamp_test(dt date,tp timestamp(6),tp_tz timestamp(6) with time zone,tp_l_tz timestamp(6) with local time zone);
Table created

# 在测试表中添加数据
SQL> insert into timestamp_test values(sysdate,sysdate,sysdate,sysdate);
1 row inserted

SQL> commit;
Commit complete

# 查看数据库的时区和当前会话的时区
SQL> select dbtimezone,sessiontimezone from dual;
DBTIMEZONE SESSIONTIMEZONE
---------- ---------------------------------------------------------------------------
+00:00     +08:00

# 查看当前时间
SQL> select sysdate from dual;
SYSDATE
-----------
2016/3/1 9:

# 查看测试表的数据
SQL> select * from timestamp_test;
DT          TP                                                                               TP_TZ                                                                            TP_L_TZ
----------- -------------------------------------------------------------------------------- -------------------------------------------------------------------------------- --------------------------------------------------------------------------------
2016/3/1 9: 01-3月 -16 09.22.33.000000 上午                                                  01-3月 -16 09.22.33.000000 上午 +08:00                                           01-3月 -16 09.22.33.000000 上午

# 修改当前会话的时区
SQL> alter session set time_zone='+10:00';
Session altered

# 查看当前会话时区修改后的测试表的数据
SQL> select dbtimezone,sessiontimezone from dual;
DBTIMEZONE SESSIONTIMEZONE
---------- ---------------------------------------------------------------------------
+00:00     +10:00

SQL> select * from timestamp_test;
DT          TP                                                                               TP_TZ                                                                            TP_L_TZ
----------- -------------------------------------------------------------------------------- -------------------------------------------------------------------------------- --------------------------------------------------------------------------------
2016/3/1 9: 01-3月 -16 09.22.33.000000 上午                                                  01-3月 -16 09.22.33.000000 上午 +08:00                                           01-3月 -16 11.22.33.000000 上午


```
