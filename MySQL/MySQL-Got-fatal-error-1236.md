## MySQL:Got fatal error 1236

记一次MySQL replication error:Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'的解决方案  

问题
----
从库出现的问题如下：　
```sql
root@localhost : (none) 11:05:47> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.200.157
                  Master_User: repl
                  Master_Port: 3307
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000136
          Read_Master_Log_Pos: 41648
               Relay_Log_File: mysql-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000136
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
          Exec_Master_Log_Pos: 41648
              Relay_Log_Space: 120
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
                Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 330757
                  Master_UUID: d4772941-70e1-11e5-ad9c-fa163e7860dd
             Master_Info_File: /data/mysql/mysqldata3307/mydata/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 160117 23:01:49
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)
```


原因分析
--------
查看show slave status\G,发现Master_Log_File: mysql-bin.000136和Read_Master_Log_Pos: 41648，于是去主库查看mysql-bin.index发现唯独缺少了mysql-bin.000136，所以原因就在这，但是我还是没明白我的mysql-bin.000136为什么会缺失，之前存在磁盘空间不够，添加更多的磁盘空间，我想跟这个可能性有关吧。  


解决方法
-------
master:  
1. 在mysql-bin.index中添加mysql-bin.000136;  
2. flush logs;  
   slave：  
3. stop slave;  
4. change master to master_log_file='mysql-bin.000136',master_log_pos=41648;  
5. start slave;  
   此时show slave status\G即可看到主从复制正常。  