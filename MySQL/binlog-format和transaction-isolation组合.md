## binlog_format和transaction_isolation组合

你知道吗？MySQL中并不是所有的binlog_format和所有的transaction_isolation都可以在一起愉快的合作哦。  


测试环境：MySQL5.6.11
### 组合产生的效果
![](/img/binlog_format&transaction_isolation.png)

YES 代表支持binlog_format+transaction_isolation组合  
NO 代表不支持binlog_format+transaction_isolation组合  


### WHY NO
（以测试的INSERT、UPDATE、DELETE语句为例）  
1. STATEMENT + READ UNCOMMITTED  
   INSERT、UPDATE、DELETE语句无法写入二进制日志中，进而导致主库上无法执行该类型语句。  
2. STATEMENT + READ COMMITED  
   INSERT、UPDATE、DELETE语句无法写入二进制日志中，进而导致主库上无法执行该类型语句。  

MySQL5.6官方文档有如下解释：  
If you are using InnoDB tables and the transaction isolation level is READ COMMITTED or READ UNCOMMITTED, only row-based logging can be used. It is possible to change the logging format to STATEMENT, but doing so at runtime leads very rapidly to errors because InnoDB can no longer perform inserts.  

解释原因：  
在READ COMMITTED、READ UNCOMMITED事务隔离级别下，基于binlog_format=’STATEMENT’模式下，SQL语句记录的顺序会导致主从数据不一致。  


### 测试
以READ COMMITED为例，假设STATEMENT模式下能写入二进制日志中。  
此处需要提醒一个很重要的概念：RC级别是会话提交后，其他的会话就可以读到提交过后的数据（即有幻读）；RR级别由于MVCC多版本控制，会话提交后，其他会话未提交的状态下，仍旧读到其他会话中第一次读取的版本（即无幻读）
![](/img/binlog_format&transaction_isolation_test.png)

分析：  
如上，在主库上执行一次如上操作，二进制日志记录SQL语句的方式为commit之后写入二进制日志中。  
故记录二进制日志的顺序为SESSION 2 , SESSION 1  
此时如果有从库，那么从库执行的结果为：

```sql  
start transaction;  
delete from t1 where c1 = 2;  
select * from t1;  
+------+------+  
| c1   | c2   |  
+------+------+  
|    1 |    1 |  
|    1 |    1 |  
+------+------+  
commit;  

start transaction;  
update t2 set c2 = 3 where c1 in (select c1 from t1);  
select * from t2;  
+------+------+  
| c1   | c2   |  
+------+------+  
|    1 |    3 |  
|    2 |    2 |  
|    1 |    3 |  
|    2 |    2 |  
+------+------+  
update t2 set c2 = 4 where c1 in (select c1 from t1);  
select * from t2;  
+------+------+  
| c1   | c2   |  
+------+------+  
|    1 |    4 |  
|    2 |    2 |  
|    1 |    4 |  
|    2 |    2 |  
+------+------+  
commit;  
```

此时对比主库与从库的t2表：  
![](/img/binlog_format&transaction_isolation_test01.png)

由此可见数据不一致现象，read uncommitted+statement同理可得。  


### 解决方法
1. 使用binlog_foemat=row/mixed  
2. 使用repeatable read 或 serializable事务隔离级别  
3. 关闭将语句记录到二进制日志的功能，set session sql_log_bin=0，但是注意使用该参数之后，执行的语句将不被记录到二进制日志中。主从会造成数据不一致。  

注：  
REPEATABLE READ + MIXED  SERIALIZABLE + MIXED  
INSERT、UPDATE、DELETE是以STATEMENT写入二进制日志中  

READ UNCOMMITTED + MIXED   READ COMMITTED + MIXED  
INSERT、UPDATE、DELETE是以ROW写入二进制日志中  