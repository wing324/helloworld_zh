## MySQL的SHOW PROFILE详解

MySQL中的profiling功能为MySQL的优化提供了另一条路径，可以根据profiling功能查看一条SQL到底是在哪里损失了性能。  




SHOW PROFILE语法
----------------
```sql
# 语法
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY n]
    [LIMIT row_count [OFFSET offset]]

type:
    ALL
  | BLOCK IO
  | CONTEXT SWITCHES
  | CPU
  | IPC
  | MEMORY
  | PAGE FAULTS
  | SOURCE
  | SWAPS

# 举个栗子
mysql> show profiles;
+----------+------------+---------------------------------------+
| Query_ID | Duration   | Query                                 |
+----------+------------+---------------------------------------+
|        1 | 0.01588975 | show variables like "profiling"       |
|        2 | 0.00014025 | SELECT DATABASE()                     |
|        3 | 0.00026475 | show databases                        |
|        4 | 0.00014150 | show tables                           |
|        5 | 0.00046300 | show variables like "profiling_hist%" |
|        6 | 0.00909950 | set profiling_history_size=100        |
|        7 | 0.00045875 | show variables like "profiling_hist%" |
+----------+------------+---------------------------------------+
7 rows in set, 1 warning (0.03 sec)

mysql> show profile for query 7;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000040 |
| checking permissions | 0.000007 |
| Opening tables       | 0.000039 |
| init                 | 0.000010 |
| System lock          | 0.000007 |
| optimizing           | 0.000005 |
| statistics           | 0.000012 |
| preparing            | 0.000010 |
| executing            | 0.000247 |
| Sending data         | 0.000016 |
| end                  | 0.000005 |
| query end            | 0.000004 |
| closing tables       | 0.000003 |
| removing tmp table   | 0.000007 |
| closing tables       | 0.000003 |
| freeing items        | 0.000031 |
| cleaning up          | 0.000016 |
+----------------------+----------+
17 rows in set, 1 warning (0.07 sec)

mysql> show profile for query 7 limit 3;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000040 |
| checking permissions | 0.000007 |
| Opening tables       | 0.000039 |
+----------------------+----------+
3 rows in set, 1 warning (0.00 sec)

mysql> show profile for query 7 limit 3 offset 5;
+------------+----------+
| Status     | Duration |
+------------+----------+
| optimizing | 0.000005 |
| statistics | 0.000012 |
| preparing  | 0.000010 |
+------------+----------+
3 rows in set, 1 warning (0.00 sec)

mysql> show  profile all for query 7;
+----------------------+----------+----------+------------+-------------------+---------------------+--------------+---------------+---------------+-------------------+-------------------+-------------------+-------+-----------------------+------------------+-------------+
| Status               | Duration | CPU_user | CPU_system | Context_voluntary | Context_involuntary | Block_ops_in | Block_ops_out | Messages_sent | Messages_received | Page_faults_major | Page_faults_minor | Swaps | Source_function       | Source_file      | Source_line |
+----------------------+----------+----------+------------+-------------------+---------------------+--------------+---------------+---------------+-------------------+-------------------+-------------------+-------+-----------------------+------------------+-------------+
| starting             | 0.000040 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | NULL                  | NULL             |        NULL |
| checking permissions | 0.000007 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | check_access          | sql_parse.cc     |        5386 |
| Opening tables       | 0.000039 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | open_tables           | sql_base.cc      |        5011 |
| init                 | 0.000010 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_prepare_select  | sql_select.cc    |        1050 |
| System lock          | 0.000007 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_lock_tables     | lock.cc          |         304 |
| optimizing           | 0.000005 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | optimize              | sql_optimizer.cc |         138 |
| statistics           | 0.000012 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | optimize              | sql_optimizer.cc |         362 |
| preparing            | 0.000010 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | optimize              | sql_optimizer.cc |         485 |
| executing            | 0.000247 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | exec                  | sql_executor.cc  |         110 |
| Sending data         | 0.000016 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | exec                  | sql_executor.cc  |         190 |
| end                  | 0.000005 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_execute_select  | sql_select.cc    |        1105 |
| query end            | 0.000004 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_execute_command | sql_parse.cc     |        5085 |
| closing tables       | 0.000003 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_execute_command | sql_parse.cc     |        5133 |
| removing tmp table   | 0.000007 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | free_tmp_table        | sql_tmp_table.cc |        1868 |
| closing tables       | 0.000003 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | free_tmp_table        | sql_tmp_table.cc |        1897 |
| freeing items        | 0.000031 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_parse           | sql_parse.cc     |        6564 |
| cleaning up          | 0.000016 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | dispatch_command      | sql_parse.cc     |        1776 |
+----------------------+----------+----------+------------+-------------------+---------------------+--------------+---------------+---------------+-------------------+-------------------+-------------------+-------+-----------------------+------------------+-------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之ALL  
显示所有性能信息。  
```sql
wing@3303>select count(*) from customer;
+----------+
| count(*) |
+----------+
|  1000000 |
+----------+
1 row in set (0.26 sec)

wing@3303>select * from customer order by telephone limit 2;
+---------+-------+-------------+------------+----------+--------+----------------+
| id      | name  | telephone   | provinceid | province | city   | address        |
+---------+-------+-------------+------------+----------+--------+----------------+
| 1000000 | TVLJC | 13198765432 |          8 | Guangxi  | City X | Street Y No. Z |
|       2 | FLKZS | 13198765432 |          2 | Shandong | City X | Street Y No. Z |
+---------+-------+-------------+------------+----------+--------+----------------+
2 rows in set (0.65 sec)

wing@3303>show profiles;
+----------+------------+---------------------------------------------------+
| Query_ID | Duration   | Query                                             |
+----------+------------+---------------------------------------------------+
|        1 | 0.00132425 | show variables like 'profiling_%'                 |
|        2 | 0.26232200 | select count(*) from customer                     |
|        3 | 0.00023575 | show create tabel customer                        |
|        4 | 0.00021000 | show create table customer                        |
|        5 | 0.64640025 | select * from customer order by telephone limit 2 |
+----------+------------+---------------------------------------------------+
5 rows in set, 1 warning (0.01 sec)

wing@3303>show profile all for query 5;
+----------------------+----------+----------+------------+-------------------+---------------------+--------------+---------------+---------------+-------------------+-------------------+-------------------+-------+-----------------------+------------------+-------------+
| Status               | Duration | CPU_user | CPU_system | Context_voluntary | Context_involuntary | Block_ops_in | Block_ops_out | Messages_sent | Messages_received | Page_faults_major | Page_faults_minor | Swaps | Source_function       | Source_file      | Source_line |
+----------------------+----------+----------+------------+-------------------+---------------------+--------------+---------------+---------------+-------------------+-------------------+-------------------+-------+-----------------------+------------------+-------------+
| starting             | 0.000088 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | NULL                  | NULL             |        NULL |
| checking permissions | 0.000015 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | check_access          | sql_parse.cc     |        5297 |
| Opening tables       | 0.000027 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | open_tables           | sql_base.cc      |        5025 |
| init                 | 0.000031 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_prepare_select  | sql_select.cc    |        1050 |
| System lock          | 0.000015 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_lock_tables     | lock.cc          |         304 |
| optimizing           | 0.000013 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | optimize              | sql_optimizer.cc |         138 |
| statistics           | 0.000022 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | optimize              | sql_optimizer.cc |         362 |
| preparing            | 0.000018 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | optimize              | sql_optimizer.cc |         485 |
| Sorting result       | 0.000023 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | make_tmp_tables_info  | sql_select.cc    |        5307 |
| executing            | 0.000009 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | exec                  | sql_executor.cc  |         110 |
| Sending data         | 0.000017 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | exec                  | sql_executor.cc  |         190 |
| Creating sort index  | 0.645990 | 0.661899 |   0.001999 |               106 |                   0 |            0 |           272 |             0 |                 0 |                 0 |                 0 |     0 | sort_table            | sql_executor.cc  |        2504 |
| end                  | 0.000035 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_execute_select  | sql_select.cc    |        1105 |
| query end            | 0.000014 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_execute_command | sql_parse.cc     |        4996 |
| closing tables       | 0.000020 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_execute_command | sql_parse.cc     |        5044 |
| freeing items        | 0.000032 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | mysql_parse           | sql_parse.cc     |        6433 |
| cleaning up          | 0.000034 | 0.000000 |   0.000000 |                 0 |                   0 |            0 |             0 |             0 |                 0 |                 0 |                 0 |     0 | dispatch_command      | sql_parse.cc     |        1778 |
+----------------------+----------+----------+------------+-------------------+---------------------+--------------+---------------+---------------+-------------------+-------------------+-------------------+-------+-----------------------+------------------+-------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之BLOCK IO  
显示块IO（块的输入输出）的次数。  
```sql
wing@3303>show profile block io for query 5;
+----------------------+----------+--------------+---------------+
| Status               | Duration | Block_ops_in | Block_ops_out |
+----------------------+----------+--------------+---------------+
| starting             | 0.000088 |            0 |             0 |
| checking permissions | 0.000015 |            0 |             0 |
| Opening tables       | 0.000027 |            0 |             0 |
| init                 | 0.000031 |            0 |             0 |
| System lock          | 0.000015 |            0 |             0 |
| optimizing           | 0.000013 |            0 |             0 |
| statistics           | 0.000022 |            0 |             0 |
| preparing            | 0.000018 |            0 |             0 |
| Sorting result       | 0.000023 |            0 |             0 |
| executing            | 0.000009 |            0 |             0 |
| Sending data         | 0.000017 |            0 |             0 |
| Creating sort index  | 0.645990 |            0 |           272 |
| end                  | 0.000035 |            0 |             0 |
| query end            | 0.000014 |            0 |             0 |
| closing tables       | 0.000020 |            0 |             0 |
| freeing items        | 0.000032 |            0 |             0 |
| cleaning up          | 0.000034 |            0 |             0 |
+----------------------+----------+--------------+---------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之CONTEXT SWITCHES  
显示自动和被动的上下文切换数量。  
```sql
wing@3303>show profile context switches for query 5;
+----------------------+----------+-------------------+---------------------+
| Status               | Duration | Context_voluntary | Context_involuntary |
+----------------------+----------+-------------------+---------------------+
| starting             | 0.000088 |                 0 |                   0 |
| checking permissions | 0.000015 |                 0 |                   0 |
| Opening tables       | 0.000027 |                 0 |                   0 |
| init                 | 0.000031 |                 0 |                   0 |
| System lock          | 0.000015 |                 0 |                   0 |
| optimizing           | 0.000013 |                 0 |                   0 |
| statistics           | 0.000022 |                 0 |                   0 |
| preparing            | 0.000018 |                 0 |                   0 |
| Sorting result       | 0.000023 |                 0 |                   0 |
| executing            | 0.000009 |                 0 |                   0 |
| Sending data         | 0.000017 |                 0 |                   0 |
| Creating sort index  | 0.645990 |               106 |                   0 |
| end                  | 0.000035 |                 0 |                   0 |
| query end            | 0.000014 |                 0 |                   0 |
| closing tables       | 0.000020 |                 0 |                   0 |
| freeing items        | 0.000032 |                 0 |                   0 |
| cleaning up          | 0.000034 |                 0 |                   0 |
+----------------------+----------+-------------------+---------------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之CPU  
显示用户和系统的CPU使用情况。  
```sql
wing@3303>show profile cpu for query 5;
+----------------------+----------+----------+------------+
| Status               | Duration | CPU_user | CPU_system |
+----------------------+----------+----------+------------+
| starting             | 0.000088 | 0.000000 |   0.000000 |
| checking permissions | 0.000015 | 0.000000 |   0.000000 |
| Opening tables       | 0.000027 | 0.000000 |   0.000000 |
| init                 | 0.000031 | 0.000000 |   0.000000 |
| System lock          | 0.000015 | 0.000000 |   0.000000 |
| optimizing           | 0.000013 | 0.000000 |   0.000000 |
| statistics           | 0.000022 | 0.000000 |   0.000000 |
| preparing            | 0.000018 | 0.000000 |   0.000000 |
| Sorting result       | 0.000023 | 0.000000 |   0.000000 |
| executing            | 0.000009 | 0.000000 |   0.000000 |
| Sending data         | 0.000017 | 0.000000 |   0.000000 |
| Creating sort index  | 0.645990 | 0.661899 |   0.001999 |
| end                  | 0.000035 | 0.000000 |   0.000000 |
| query end            | 0.000014 | 0.000000 |   0.000000 |
| closing tables       | 0.000020 | 0.000000 |   0.000000 |
| freeing items        | 0.000032 | 0.000000 |   0.000000 |
| cleaning up          | 0.000034 | 0.000000 |   0.000000 |
+----------------------+----------+----------+------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之IPC  
显示发送和接收的消息数量。  
```sql
wing@3303>show profile ipc for query 5;
+----------------------+----------+---------------+-------------------+
| Status               | Duration | Messages_sent | Messages_received |
+----------------------+----------+---------------+-------------------+
| starting             | 0.000088 |             0 |                 0 |
| checking permissions | 0.000015 |             0 |                 0 |
| Opening tables       | 0.000027 |             0 |                 0 |
| init                 | 0.000031 |             0 |                 0 |
| System lock          | 0.000015 |             0 |                 0 |
| optimizing           | 0.000013 |             0 |                 0 |
| statistics           | 0.000022 |             0 |                 0 |
| preparing            | 0.000018 |             0 |                 0 |
| Sorting result       | 0.000023 |             0 |                 0 |
| executing            | 0.000009 |             0 |                 0 |
| Sending data         | 0.000017 |             0 |                 0 |
| Creating sort index  | 0.645990 |             0 |                 0 |
| end                  | 0.000035 |             0 |                 0 |
| query end            | 0.000014 |             0 |                 0 |
| closing tables       | 0.000020 |             0 |                 0 |
| freeing items        | 0.000032 |             0 |                 0 |
| cleaning up          | 0.000034 |             0 |                 0 |
+----------------------+----------+---------------+-------------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之MEMORY  
MySQL5.6中还未实现，只是计划实现。  
##### type之PAGE FAULTS  
显示主要的和次要的页面故障。  
```sql
wing@3303>show profile page faults for query 5;
+----------------------+----------+-------------------+-------------------+
| Status               | Duration | Page_faults_major | Page_faults_minor |
+----------------------+----------+-------------------+-------------------+
| starting             | 0.000088 |                 0 |                 0 |
| checking permissions | 0.000015 |                 0 |                 0 |
| Opening tables       | 0.000027 |                 0 |                 0 |
| init                 | 0.000031 |                 0 |                 0 |
| System lock          | 0.000015 |                 0 |                 0 |
| optimizing           | 0.000013 |                 0 |                 0 |
| statistics           | 0.000022 |                 0 |                 0 |
| preparing            | 0.000018 |                 0 |                 0 |
| Sorting result       | 0.000023 |                 0 |                 0 |
| executing            | 0.000009 |                 0 |                 0 |
| Sending data         | 0.000017 |                 0 |                 0 |
| Creating sort index  | 0.645990 |                 0 |                 0 |
| end                  | 0.000035 |                 0 |                 0 |
| query end            | 0.000014 |                 0 |                 0 |
| closing tables       | 0.000020 |                 0 |                 0 |
| freeing items        | 0.000032 |                 0 |                 0 |
| cleaning up          | 0.000034 |                 0 |                 0 |
+----------------------+----------+-------------------+-------------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之SOURCE  
显示源代码的函数名称，以及在源码文件名称与行数（即源码中的位置）。  
```sql
wing@3303>show profile source for query 5;
+----------------------+----------+-----------------------+------------------+-------------+
| Status               | Duration | Source_function       | Source_file      | Source_line |
+----------------------+----------+-----------------------+------------------+-------------+
| starting             | 0.000088 | NULL                  | NULL             |        NULL |
| checking permissions | 0.000015 | check_access          | sql_parse.cc     |        5297 |
| Opening tables       | 0.000027 | open_tables           | sql_base.cc      |        5025 |
| init                 | 0.000031 | mysql_prepare_select  | sql_select.cc    |        1050 |
| System lock          | 0.000015 | mysql_lock_tables     | lock.cc          |         304 |
| optimizing           | 0.000013 | optimize              | sql_optimizer.cc |         138 |
| statistics           | 0.000022 | optimize              | sql_optimizer.cc |         362 |
| preparing            | 0.000018 | optimize              | sql_optimizer.cc |         485 |
| Sorting result       | 0.000023 | make_tmp_tables_info  | sql_select.cc    |        5307 |
| executing            | 0.000009 | exec                  | sql_executor.cc  |         110 |
| Sending data         | 0.000017 | exec                  | sql_executor.cc  |         190 |
| Creating sort index  | 0.645990 | sort_table            | sql_executor.cc  |        2504 |
| end                  | 0.000035 | mysql_execute_select  | sql_select.cc    |        1105 |
| query end            | 0.000014 | mysql_execute_command | sql_parse.cc     |        4996 |
| closing tables       | 0.000020 | mysql_execute_command | sql_parse.cc     |        5044 |
| freeing items        | 0.000032 | mysql_parse           | sql_parse.cc     |        6433 |
| cleaning up          | 0.000034 | dispatch_command      | sql_parse.cc     |        1778 |
+----------------------+----------+-----------------------+------------------+-------------+
17 rows in set, 1 warning (0.00 sec)
```
##### type之SWAPS  
显示swap的次数。  
```sql
wing@3303>show profile swaps for query 5;
+----------------------+----------+-------+
| Status               | Duration | Swaps |
+----------------------+----------+-------+
| starting             | 0.000088 |     0 |
| checking permissions | 0.000015 |     0 |
| Opening tables       | 0.000027 |     0 |
| init                 | 0.000031 |     0 |
| System lock          | 0.000015 |     0 |
| optimizing           | 0.000013 |     0 |
| statistics           | 0.000022 |     0 |
| preparing            | 0.000018 |     0 |
| Sorting result       | 0.000023 |     0 |
| executing            | 0.000009 |     0 |
| Sending data         | 0.000017 |     0 |
| Creating sort index  | 0.645990 |     0 |
| end                  | 0.000035 |     0 |
| query end            | 0.000014 |     0 |
| closing tables       | 0.000020 |     0 |
| freeing items        | 0.000032 |     0 |
| cleaning up          | 0.000034 |     0 |
+----------------------+----------+-------+
17 rows in set, 1 warning (0.00 sec)
```


SHOW PROFILE详解
----------------
通过设置profiling参数，开启MySQL的profiling功能；  
通过设置profiling_history_size参数，设置保留的SQL语句数量；  
如果将profiling_history_size参数设置为0，同样具有关闭MySQL的profiling效果。  
```sql
mysql> show variables like "profiling";  
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show variables like "profiling";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | ON    |
+---------------+-------+
1 row in set (0.02 sec)

mysql> show variables like "profiling_hist%";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| profiling_history_size | 15    |
+------------------------+-------+
1 row in set (0.00 sec)

mysql> set profiling_history_size=100;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> show variables like "profiling_hist%";
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| profiling_history_size | 100   |
+------------------------+-------+
1 row in set (0.00 sec)
```
##### SHOW PROFILE各个Status详解
详见show processlist https://github.com/wing324/helloworld/blob/master/MySQL/SHOW-PROCESSLIST%E5%92%8CSHOW-PROFILE%E7%9A%84Status%E6%95%B4%E7%90%86%EF%BC%88%E8%8B%B1%E6%96%87%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E7%89%88%EF%BC%89.md。
