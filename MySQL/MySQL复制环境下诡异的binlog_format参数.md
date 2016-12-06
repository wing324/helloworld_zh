##  MySQL复制环境下诡异的binlog_format参数

#### 一、问题描述：

有一次在MySQL从库上执行binlog_format=row之后，发现binlog的格式咋还是statement，并没有变化为binlog_format，查阅MySQL官方手册发现binlog_format的确是个动态修改的参数(http://dev.mysql.com/doc/refman/5.6/en/server-options.html#option_mysqld_binlog-format)，这是为啥呢？



#### 二、假想测试：

```sql
# 测试表结构
CREATE TABLE t(id INT NOT NULL DEFAULT '0');
```



1. 官方文档错误？毕竟我曾经的确给官方文档提出一个BUG，并且MySQL官网很快给修改了。。

   ```sql
   # 主库上测试
   # 修改主库的binlog_format
   set global binlog_format=statement;
   set session binlog_format=statement;
   insert into t values(6);

   # 查看binlog
   # at 441
   #161206 20:40:13 server id 2263333  end_log_pos 542 CRC32 0x3eeb2569 	Query	thread_id=1215	exec_time=0	error_code=0
   use `fianna`/*!*/;
   SET TIMESTAMP=1481028133/*!*/;
   insert into t values(6)
   /*!*/;
   # at 542

   ```

   结论：官方文档并没有欺骗我，的确是可以动态修改的。。。

2. 莫非从库上binlog_format并不是动态修改型参数？

   ```sql
   # 从库上测试
   # 修改从库binlog_format
   set global binlog_format=statement;
   set session binlog_format=statement;
   insert into t values(11);

   # 查看binlog
   # at 984
   #161206 20:41:31 server id 2273333  end_log_pos 1032 CRC32 0xa3865eac 	Rows_query
   # insert into t values(11)
   # at 1032
   ```

   结论：从库上的binlog_format也是动态修改参数。。

3. 难道从库上从主库复制来的SQL才会导致binlog_format没有修改？

   ```sql
   # 主从同时测试
   # 修改主库binlog_format
   set global binlog_format=row;
   set session binlog_format=row;
   # 修改从库binlog_format
   set global binlog_format=row;
   set session binlog_format=row;

   #  主库执行
   insert into t values(5);

   # 主库binlog
   # at 740
   #161206 20:43:00 server id 2263333  end_log_pos 780 CRC32 0x0ab3fb46 	Write_rows: table id 3523 flags: STMT_END_F

   BINLOG '
   VLJGWB0liSIALwAAALYCAACAABdpbnNlcnQgaW50byB0IHZhbHVlcyg2KeLjoRw=
   VLJGWBMliSIALgAAAOQCAAAAAMMNAAAAAAEABmZpYW5uYQABdAABAwABl8Yg5A==
   VLJGWB4liSIAKAAAAAwDAAAAAMMNAAAAAAEAAgAB//4GAAAARvuzCg==
   '/*!*/;
   ### INSERT INTO `fianna`.`t`
   ### SET
   ###   @1=5 /* INT meta=0 nullable=1 is_null=0 */
   # at 780

   # 从库binlog
   # at 984
   #161206 20:43:01 server id 2273333  end_log_pos 1032 CRC32 0xa3865eac 	Rows_query
   # insert into t values(5)
   # at 1032

   ```

   结论：好像binlog_format对于主从复制的SQL并没有影响。。

4. 如果restart slave，binlog_format是否有影响呢？

   ```sql
   # 主从同时测试
   # 修改主库binlog_format
   set global binlog_format=row;
   set session binlog_format=row;
   # 修改从库binlog_format
   set global binlog_format=row;
   set session binlog_format=row;
   stop slave;
   start slave;

   #  主库执行
   insert into t values(2);

   # 主库binlog
   # at 998
   #161206 20:53:00 server id 2263333  end_log_pos 998 CRC32 0x0ab3fb46 	Write_rows: table id 3523 flags: STMT_END_F

   BINLOG '
   VLJGWB0liSIALwAAALYCAACAABdpbnNlcnQgaW50byB0IHZhbHVlcyg2KeLjoRw=
   VLJGWBMliSIALgAAAOQCAAAAAMMNAAAAAAEABmZpYW5uYQABdAABAwABl8Yg5A==
   VLJGWB4liSIAKAAAAAwDAAAAAMMNAAAAAAEAAgAB//4GAAAARvuzCg==
   '/*!*/;
   ### INSERT INTO `fianna`.`t`
   ### SET
   ###   @1=2 /* INT meta=0 nullable=1 is_null=0 */
   # at 998

   # 从库binlog
   # at 1078
   #161206 20:53:01 server id 2273333  end_log_pos 1118 CRC32 0x81a97c17 	Write_rows: table id 47129 flags: STMT_END_F

   BINLOG '
   F7RGWB01sCIAMAAAAAgEAACAABhpbnNlcnQgaW50byB0IHZhbHVlcygxMSmsXoaj
   F7RGWBM1sCIALgAAADYEAAAAABm4AAAAAAEABmZpYW5uYQABdAABAwAB3jnZ6w==
   F7RGWB41sCIAKAAAAF4EAAAAABm4AAAAAAEAAgAB//4LAAAAF3ypgQ==
   '/*!*/;
   ### INSERT INTO `fianna`.`t`
   ### SET
   ###   @1=2 /* INT meta=0 nullable=1 is_null=0 */
   # at 1118
   ```

   结论：restart slave之后,binlog_format立即动态被修改。



#### 三、问题总结

主从复制中，从库的binlog_format动态修改后，并不能立即于来自于主库的SQL(即复制SQL)立即生效，需要在从库执行`stop slave;``start slave;`之后，方可生效。