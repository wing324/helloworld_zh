## 手动制造GTID空洞

在网上看了下同名文章，但是宝宝没有实现。后来情急之下，通过宝宝的小领导一提醒，宝宝就实现了如何手动制造GTID空洞。


- 首先，要创建基于GTID的主从复制哒，该步骤省略。。

- 建立GTID同步，从库上`show slave status\G`可以看到如下结果：

  ```sql
  mysql> show slave status\G
  *************************** 1. row ***************************
                 Slave_IO_State: Waiting for master to send event
                    Master_Host: 192.168.0.101
                    Master_User: repl
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: mysql-bin.000003
            Read_Master_Log_Pos: 1167
                 Relay_Log_File: mysql-relay-log.000007
                  Relay_Log_Pos: 464
          Relay_Master_Log_File: mysql-bin.000003
               Slave_IO_Running: Yes
              Slave_SQL_Running: Yes
                Replicate_Do_DB:
            Replicate_Ignore_DB:
             Replicate_Do_Table:
         Replicate_Ignore_Table:
        Replicate_Wild_Do_Table:
    Replicate_Wild_Ignore_Table:
                     Last_Errno: 0
                     Last_Error:
                   Skip_Counter: 0
            Exec_Master_Log_Pos: 1167
                Relay_Log_Space: 5364
                Until_Condition: None
                 Until_Log_File:
                  Until_Log_Pos: 0
             Master_SSL_Allowed: No
             Master_SSL_CA_File:
             Master_SSL_CA_Path:
                Master_SSL_Cert:
              Master_SSL_Cipher:
                 Master_SSL_Key:
          Seconds_Behind_Master: 0
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 0
                  Last_IO_Error:
                 Last_SQL_Errno: 0
                 Last_SQL_Error:
    Replicate_Ignore_Server_Ids:
               Master_Server_Id: 330601
                    Master_UUID: 80544330-385c-11e6-8262-005056900269
               Master_Info_File: /data/mysql/3333/db_file/master.info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
             Master_Retry_Count: 86400
                    Master_Bind:
        Last_IO_Error_Timestamp:
       Last_SQL_Error_Timestamp:
                 Master_SSL_Crl:
             Master_SSL_Crlpath:
             Retrieved_Gtid_Set: 80544330-385c-11e6-8262-005056900269:32:34-41
              Executed_Gtid_Set: 80544330-385c-11e6-8262-005056900269:1-41
                  Auto_Position: 1
  1 row in set (0.00 sec)
  ```

- 开始搞破坏啦，制作GTID空洞

  ```sql
  mysql> stop slave;
  Query OK, 0 rows affected (0.02 sec)

  mysql> reset master;
  Query OK, 0 rows affected (0.08 sec)

  mysql> show global variables like '%gtid%';
  +--------------------------+-------+
  | Variable_name            | Value |
  +--------------------------+-------+
  | enforce_gtid_consistency | ON    |
  | gtid_executed            |       |
  | gtid_mode                | ON    |
  | gtid_owned               |       |
  | gtid_purged              |       |
  +--------------------------+-------+
  5 rows in set (0.00 sec)

  mysql> set global gtid_purged='80544330-385c-11e6-8262-005056900269:1-22:33-41';
  Query OK, 0 rows affected (0.05 sec)

  mysql> start slave;
  Query OK, 0 rows affected (0.01 sec)
  ```

- 由于主库上我将22-33之间的事物的binlog都已经purge掉了，因为每次`start slave;`，基于GTID复制会去找断点的GTID，尝试进行修复，所以此时，我这边的`show slave status\G`会报如下错误：

  ```sql
  mysql> show slave status\G
  *************************** 1. row ***************************
                 Slave_IO_State:
                    Master_Host: 192.168.0.101
                    Master_User: repl
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: mysql-bin.000003
            Read_Master_Log_Pos: 1167
                 Relay_Log_File: mysql-relay-log.000007
                  Relay_Log_Pos: 464
          Relay_Master_Log_File: mysql-bin.000003
               Slave_IO_Running: No
              Slave_SQL_Running: Yes
                Replicate_Do_DB:
            Replicate_Ignore_DB:
             Replicate_Do_Table:
         Replicate_Ignore_Table:
        Replicate_Wild_Do_Table:
    Replicate_Wild_Ignore_Table:
                     Last_Errno: 0
                     Last_Error:
                   Skip_Counter: 0
            Exec_Master_Log_Pos: 1167
                Relay_Log_Space: 5364
                Until_Condition: None
                 Until_Log_File:
                  Until_Log_Pos: 0
             Master_SSL_Allowed: No
             Master_SSL_CA_File:
             Master_SSL_CA_Path:
                Master_SSL_Cert:
              Master_SSL_Cipher:
                 Master_SSL_Key:
          Seconds_Behind_Master: NULL
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 1236
                  Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'The slave is connecting using CHANGE MASTER TO MASTER_AUTO_POSITION = 1, but the master has purged binary logs containing GTIDs that the slave requires.'
                 Last_SQL_Errno: 0
                 Last_SQL_Error:
    Replicate_Ignore_Server_Ids:
               Master_Server_Id: 330601
                    Master_UUID: 80544330-385c-11e6-8262-005056900269
               Master_Info_File: /data/mysql/3333/db_file/master.info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
             Master_Retry_Count: 86400
                    Master_Bind:
        Last_IO_Error_Timestamp: 160713 19:51:33
       Last_SQL_Error_Timestamp:
                 Master_SSL_Crl:
             Master_SSL_Crlpath:
             Retrieved_Gtid_Set: 80544330-385c-11e6-8262-005056900269:32:34-41
              Executed_Gtid_Set: 80544330-385c-11e6-8262-005056900269:1-22:33-41
                  Auto_Position: 1
  ```

- 关于这个error又要如何修复呢？让我门一起挑战吧～

  ```sql
  mysql> stop slave;
  Query OK, 0 rows affected (0.00 sec)

  mysql> reset master;
  Query OK, 0 rows affected (0.06 sec)

  mysql> show global variables like '%gtid%';
  +--------------------------+-------+
  | Variable_name            | Value |
  +--------------------------+-------+
  | enforce_gtid_consistency | ON    |
  | gtid_executed            |       |
  | gtid_mode                | ON    |
  | gtid_owned               |       |
  | gtid_purged              |       |
  +--------------------------+-------+
  5 rows in set (0.00 sec)

  mysql> set global gtid_purged='80544330-385c-11e6-8262-005056900269:1-41';
  Query OK, 0 rows affected (0.05 sec)

  mysql> start slave;
  Query OK, 0 rows affected (0.01 sec)

  mysql> show slave status\G
  *************************** 1. row ***************************
                 Slave_IO_State: Waiting for master to send event
                    Master_Host: 192.168.0.101
                    Master_User: repl
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: mysql-bin.000003
            Read_Master_Log_Pos: 1167
                 Relay_Log_File: mysql-relay-log.000008
                  Relay_Log_Pos: 464
          Relay_Master_Log_File: mysql-bin.000003
               Slave_IO_Running: Yes
              Slave_SQL_Running: Yes
                Replicate_Do_DB:
            Replicate_Ignore_DB:
             Replicate_Do_Table:
         Replicate_Ignore_Table:
        Replicate_Wild_Do_Table:
    Replicate_Wild_Ignore_Table:
                     Last_Errno: 0
                     Last_Error:
                   Skip_Counter: 0
            Exec_Master_Log_Pos: 1167
                Relay_Log_Space: 5881
                Until_Condition: None
                 Until_Log_File:
                  Until_Log_Pos: 0
             Master_SSL_Allowed: No
             Master_SSL_CA_File:
             Master_SSL_CA_Path:
                Master_SSL_Cert:
              Master_SSL_Cipher:
                 Master_SSL_Key:
          Seconds_Behind_Master: 0
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 0
                  Last_IO_Error:
                 Last_SQL_Errno: 0
                 Last_SQL_Error:
    Replicate_Ignore_Server_Ids:
               Master_Server_Id: 330601
                    Master_UUID: 80544330-385c-11e6-8262-005056900269
               Master_Info_File: /data/mysql/3333/db_file/master.info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
             Master_Retry_Count: 86400
                    Master_Bind:
        Last_IO_Error_Timestamp:
       Last_SQL_Error_Timestamp:
                 Master_SSL_Crl:
             Master_SSL_Crlpath:
             Retrieved_Gtid_Set: 80544330-385c-11e6-8262-005056900269:32:34-41
              Executed_Gtid_Set: 80544330-385c-11e6-8262-005056900269:1-41
                  Auto_Position: 1
  ```

- 好啦，大功告成啦，那我们去主库执行一个事物，看看同步是否正常～

  ```sql
  # master:
  mysql> show tables
      -> ;
  +--------------------+
  | Tables_in_bilibili |
  +--------------------+
  | tt                 |
  +--------------------+
  1 row in set (0.00 sec)

  mysql> insert into tt values();
  Query OK, 1 row affected (0.02 sec)

  # slave:
  mysql> show slave status\G
  *************************** 1. row ***************************
                 Slave_IO_State: Waiting for master to send event
                    Master_Host: 192.168.0.101
                    Master_User: repl
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: mysql-bin.000003
            Read_Master_Log_Pos: 1411
                 Relay_Log_File: mysql-relay-log.000008
                  Relay_Log_Pos: 708
          Relay_Master_Log_File: mysql-bin.000003
               Slave_IO_Running: Yes
              Slave_SQL_Running: Yes
                Replicate_Do_DB:
            Replicate_Ignore_DB:
             Replicate_Do_Table:
         Replicate_Ignore_Table:
        Replicate_Wild_Do_Table:
    Replicate_Wild_Ignore_Table:
                     Last_Errno: 0
                     Last_Error:
                   Skip_Counter: 0
            Exec_Master_Log_Pos: 1411
                Relay_Log_Space: 6125
                Until_Condition: None
                 Until_Log_File:
                  Until_Log_Pos: 0
             Master_SSL_Allowed: No
             Master_SSL_CA_File:
             Master_SSL_CA_Path:
                Master_SSL_Cert:
              Master_SSL_Cipher:
                 Master_SSL_Key:
          Seconds_Behind_Master: 0
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 0
                  Last_IO_Error:
                 Last_SQL_Errno: 0
                 Last_SQL_Error:
    Replicate_Ignore_Server_Ids:
               Master_Server_Id: 330601
                    Master_UUID: 80544330-385c-11e6-8262-005056900269
               Master_Info_File: /data/mysql/3333/db_file/master.info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
             Master_Retry_Count: 86400
                    Master_Bind:
        Last_IO_Error_Timestamp:
       Last_SQL_Error_Timestamp:
                 Master_SSL_Crl:
             Master_SSL_Crlpath:
             Retrieved_Gtid_Set: 80544330-385c-11e6-8262-005056900269:32:34-42
              Executed_Gtid_Set: 80544330-385c-11e6-8262-005056900269:1-42
                  Auto_Position: 1
  1 row in set (0.00 sec)
  ```

- 看完上面的步骤，是正常同步哦，在此我们就解决啦～