## MySQL5.6之XA事务

MySQL5.6新特性之XA事务，XA事务也即是分布式事务。



XA事务SQL语句
-------------

```sql
XA {START|BEGIN} xid [JOIN|RESUME]

XA END xid [SUSPEND [FOR MIGRATE]]

XA PREPARE xid

XA COMMIT xid [ONE PHASE]

XA ROLLBACK xid

XA RECOVER
```

**特别说明**  
1. 对于XA START语句,JOIN/RESUME语句不支持;对于XA END语句,SUSPEND [FOR MIGRATE]语句并不支持;  
2. 每一个XA语句都是以XA开头的;  
3. xid的值是有客户提供的或者有MySQL服务器生成;  
4. xid的组成如下  

```sql
xid: gtrid [, bqual [, formatID ]]
```
*gtrid*是全局事务ID,64字节的字符串;  
*bqual*是分支(branch)的修饰符,64字节的字符串;  
*formatID*是用于标识gtrid和bqual的格式,无符号的整数数值;  
xid是由transaction manager管理;  
每一个XA事务都必须拥有其唯一的xid值;  


XA事务state
-----------
1. XA事务的处理过程如下  
   1) 使用XA START开启一个xa事务,并将其置为active状态;  
   2) 对于active状态的xa事务,处理完事务内的SQL语句后,使用XA END语句,XA END语句将事务置为IDLE状态;  
   3) 对于IDLE状态的xa事务,可以使用XA PREPARE或XA XOMMIT ... ONE PHASE语句;  
   XA PREPARE将xa事务置于prepared状态;  
   XA RECOVER将prepare状态的xa事务全部列出来;  
   XA COMMIT ... ONE PHASE准备并提交事务,此时的xid值将不会出现在XA RECOVER语句中;  
   4) 对于prepared状态的XA事务,可以使用XA COMMIT或者XA ROLLBACK来提交或回滚XA事务。  
   **实战演练**  

```sql
#开启新的xa事务,将xa事务置于active状态
root@localhost : wing 02:34:24> xa start 'xatest';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:34:42> insert into xa_test values (1);
Query OK, 1 row affected (0.00 sec)

# 结束xa事务,将xa事务置于idle状态,此时事务并未提交
root@localhost : wing 02:34:48> xa end 'xatest';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:35:06> xa recover 'xatest';
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''xatest'' at line 1

#由于xa事务没有置于prepare状态,故xa recover没有显示结果
root@localhost : wing 02:35:26> xa recover;
Empty set (0.00 sec)

#将xa事务置于prepared状态
root@localhost : wing 02:35:55> xa prepare 'xatest';
Query OK, 0 rows affected (0.00 sec)

#xa事务进入prepared状态之后,可以在xa recover中查看到prepare的xa事务
root@localhost : wing 02:35:59> xa recover;
+----------+--------------+--------------+--------+
| formatID | gtrid_length | bqual_length | data   |
+----------+--------------+--------------+--------+
|        1 |            6 |            0 | xatest |
+----------+--------------+--------------+--------+
1 row in set (0.00 sec)

#提交xa事务,此时可开启另一个会话,提交xa事务,事务只有在进入prepared状态以后才可以提交
root@localhost : wing 02:36:05> xa commit 'xatest';
Query OK, 0 rows affected (0.00 sec)
```
2. 同一个会话中,不允许并行开启两个XA事务,除非前一个xa事务必须提交,常规的START TRANSACTION也一样  

```sql
root@localhost : wing 02:48:58> xa start 'xa1';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:51:51> xa start 'xa2';
ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in the  ACTIVE state

root@localhost : wing 02:53:49> xa end 'xa1';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:54:01> xa start 'xa2';
ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in the  IDLE state

root@localhost : wing 02:54:05> xa prepare 'xa1';
Query OK, 0 rows affected (0.00 sec)

root@localhost : wing 02:54:11> xa start 'xa2';
ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in the  PREPARED state

root@localhost : wing 02:54:15> xa commit 'xa1';
Query OK, 0 rows affected (0.00 sec)

# 直到'xa1'提交后,方可创建新的xa事务'xa2'
root@localhost : wing 02:54:24> xa start 'xa2';
Query OK, 0 rows affected (0.00 sec)
```
3. 当事务处于active时,不能发出任何导致隐式提交的语句。(此处隐式提交的语句与常规事务的隐式提交语句一致)  


XA事务的限制
------------
1. XA事务仅适用于Innodb存储引擎;  
2. XA START语句不支持JOIN/RESUME语句;  
3. XA END语句不支持SUSPEND [FOR MIGRATE]语句;  
4. 当XA事务在prepared阶段时,此时MySQL Server宕机，该事物可以在服务器重启后继续执行，然而客户端重连后，即使事务已经提交，但是它并不会记录到binlog中，这将导致主从复制不一致。  
