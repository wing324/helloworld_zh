## LOCK TABLES & TRIGGER那些事

LOCK TABLES和TRIGGER之间会产生哪些奇妙的锁呢？




测试用例
-------------
1. 创建测试相关的表与触发器  

```sql
root@localhost : sp_test_db 02:55:50> create table t1(id int);
Query OK, 0 rows affected (0.04 sec)

root@localhost : sp_test_db 02:56:50> create table t2(id int);
Query OK, 0 rows affected (0.02 sec)

root@localhost : sp_test_db 02:56:53> create table t3(id int);
Query OK, 0 rows affected (0.04 sec)

root@localhost : sp_test_db 02:56:56> create table t4(id int);

DELIMITER //
CREATE TRIGGER t1_after_ins AFTER INSERT ON t1 FOR EACH ROW
BEGIN
UPDATE t4 SET id=id+1
WHERE id=NEW.id AND EXISTS (SELECT id FROM t3);
INSERT INTO t2 VALUES(100);
END//
DELIMITER ;
```

2. 执行LOCK TABLES 语句  

```sql
LOCK TABLES t1 WRITE, t2 READ;
```

3. 测试相关表的锁情况，为了测试方便，将参数innodb_lock_wait_timeout设置为2  

```sql
root@localhost : sp_test_db 03:18:00> set session innodb_lock_wait_timeout=2;
Query OK, 0 rows affected (0.02 sec)

root@localhost : sp_test_db 03:18:29> select * from t1;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

root@localhost : sp_test_db 03:18:38> select * from t2;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

root@localhost : sp_test_db 03:18:47> select * from t3;
Empty set (0.00 sec)

root@localhost : sp_test_db 03:19:19> insert into t3 values(1);
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction

root@localhost : sp_test_db 03:19:36> select * from t4;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```


分析过程
-------------
1. t1表持有写锁，因为LOCK TABLES显示指定;  
2. t2表持有写锁，虽然LOCK TABLES显示指定读锁，但是触发器中需要做INSERT操作;  
3. t3表持有读锁，因为触发器中只需要做SELECT操作;  
4. t4表持有写锁，因为触发器中需要UPDATE操作。  



结论
------
如果显示指定了LOCK TABLES,那么与它相关的触发器中的所有的表都会被隐式的指定LOCK TABLE。  
1. 使用LOCK TABLES指定的表被相应的锁锁住,如果在触发器中没有该表相关的语句需要更高范围的锁;  
2. 触发器里的表持有的锁取决于其SQL语句,如果只需要读,则持有读锁,否则为写锁;  
3. 如果一个表被LOCK TABLES显示指定读锁,但在触发器中需要写锁,那么该表将持有写锁。  