## TPCC-MySQL工具

TPCC-MySQL是一款MySQL基准测试工具，能测试出当前MySQL的每分钟事务数（TPmC）。本文介绍TPCC-MySQL工具的安装以及如何使用。  


TPCC-MySQL安装
--------------
1. 安装bazaar客户端  
   yum install -y bzr  
2. 至官网注册帐号  
   https://launchpad.net/bzr  
3. 在Linux下生成SSH密钥  
   ssh-keygen -t rsa  
4. 将SSH密钥提交至bazaar  
   less /root/.ssh/id_rsa.pub  
   将内容复制到 https://launchpad.net/~username/+editsshkeys  
   注：*username*为用户名  
5. 创建tpcc-mysql目录  
   mkdir /tpcc-mysql  
6. 使用bazaar下载tpcc-mysql源码  
   bzr launchpad-login username  
   bzr branch lp:~percona-dev/perconatools/tpcc-mysql  
7. 编译安装  
   cd /tpcc-mysql/tpcc-mysql/src  
   make  
8. 安装成功的标志为在/tpcc-mysql/tpcc-mysql目录下生成tpcc命令行工具tpcc_load、tpcc_start  

**注意事项**  
1. 安装过程遇到如下错误：  
   /usr/bin/ld: cannot find -lmysqlclient_r  
   collect2: ld 返回 1  
   make: *** [../tpcc_load] 错误 1  
   解决方法：  
   export PATH=/usr/local/mysql/bin:$PATH  
   export LD_LIBRARY_PATH=/usr/local/mysql/lib  
2. 启用tpcc_load时,遇到如下错误:  
   ./tpcc_load: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory  
   解决方法：  
   ln -s /usr/local/mysql-5.6.24-linux-glibc2.5-x86_64/lib/libmysqlclient.so.18 /usr/lib64/libmysqlclient.so.18  


TPCC-MySQL相关数据表及用途
--------------------------
customer:客户表  
district:地区表  
history:历史订单表  
item:商品表  
new_orders:新订单表  
orders:订单表  
stock:库存表  
warehouse:仓库表  


TPCC-MySQL测试
--------------
1. 创建测试数据库  
   mysqladmin  -uroot --socket=/data/mysqldata3306/sock/mysql.sock create tpcc  
2. 创建相关数表结构  
   1). 创建表  
   source /tpcc-mysql/tpcc-mysql/create_table.sql  
   2). 同种方法创建其他条件,TPCC-MySQL提供如下脚本  
   add_fkey_idx.sql  
   count.sql  
   create_table.sql  
   drop_cons.sql  
3. 使用tpcc_load加载测试数据  
   **语法**  
   ./tpcc_load  --help  
   tpcc_load [server] [DB] [user] [pass] [warehouse]  
   tpcc_load [server] [DB] [user] [pass] [warehouse] [part] [min_wh] [max_wh]  
   [part]: 1=ITEMS 2=WAREHOUSE 3=CUSTOMER 4=ORDERS  
   此处连接的socket为/var/lib/mysql/mysql.sock,如果需要连接指定的socket,将[server]改写成host:port即可;  
   [part]为只创建数据到[part]对应的表中;  
   [min_wh] [max_wh]为min_wid max_wid。  
   **测试用例**  
   ./tpcc_load  127.0.0.1:3306 tpcc root '' 1   
   该测试用例为创建一个仓库,表数据如下：

```sql
root@localhost : tpcc 08:51:20> select count(*) from customer;
+----------+
| count(*) |
+----------+
|    30000 |
+----------+
1 row in set (0.14 sec)

root@localhost : tpcc 08:51:29> select count(*) from district;
+----------+
| count(*) |
+----------+
|       10 |
+----------+
1 row in set (0.00 sec)

root@localhost : tpcc 08:51:39> select count(*) from history;
+----------+
| count(*) |
+----------+
|    30000 |
+----------+
1 row in set (0.04 sec)

root@localhost : tpcc 08:51:47> select count(*) from item ;
+----------+
| count(*) |
+----------+
|   100000 |
+----------+
1 row in set (0.04 sec)

root@localhost : tpcc 08:51:53> select count(*) from new_orders;
+----------+
| count(*) |
+----------+
|     9000 |
+----------+
1 row in set (0.01 sec)

root@localhost : tpcc 08:52:00> select count(*) from order_line;
+----------+
| count(*) |
+----------+
|   299616 |
+----------+
1 row in set (0.10 sec)

root@localhost : tpcc 08:52:08> select count(*) from orders;
+----------+
| count(*) |
+----------+
|    30000 |
+----------+
1 row in set (0.02 sec)

root@localhost : tpcc 08:52:14> select count(*) from stock;
+----------+
| count(*) |
+----------+
|   100000 |
+----------+
1 row in set (0.06 sec)

root@localhost : tpcc 08:52:20> select count(*) from warehouse;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)
```
 ./tpcc_load  127.0.0.1:3306 tpcc root '' 1  3 5 9  
 该测试用例为创建5<WID<9的数据到customer表中。  
4. 使用tpcc_start进行基准测试  
   **语法**  
   ./tpcc_start --help  
   tpcc_start -h server_host -P port -d database_name -u mysql_user -p mysql_password -w warehouses -c connections -r warmup_time -l running_time -i report_interval -f report_file -t trx_file  
   **参数含义**  
   -w 指定仓库数量  
   -c 指定并发连接数  
   -r 指定开始测试前的warmup时间,预热后测试效果会更好  
   -l 指定测试持续时间  
   -i 指定生成报告间隔时长,默认为10s  
   -f 指定生成的报告文件名  
   **测试用例**  
   ./tpcc_start -h 127.0.0.1 -P3306 -d tpcc -u root -w 1 -c 5 -r 10 -l 60   
   模拟对一个仓库(-w 1),并发5个线程(-c 5),预热10s(-r 10),持续压测60s(-l 60)  
   **测试结果** 

```sql
option h with value '127.0.0.1'
option P with value '3306'
option d with value 'tpcc'
option u with value 'root'
option w with value '1'
option c with value '5'
option r with value '10'
option l with value '60'
<Parameters>
     [server]: 127.0.0.1
     [port]: 3306
     [DBname]: tpcc
       [user]: root
       [pass]: 
  [warehouse]: 1
 [connection]: 5
     [rampup]: 10 (sec.)
    [measure]: 60 (sec.)

RAMP-UP TIME.(10 sec.)

MEASURING START.

  10, 66(47):19.999|66.298, 64(2):12.187|14.665, 7(0):2.450|8.299, 7(0):19.999|55.631, 8(7):19.999|144.925
  20, 398(106):19.999|54.874, 403(12):9.354|91.049, 39(1):1.691|14.304, 39(0):17.983|19.232, 40(8):19.999|73.468
  30, 371(85):19.999|58.431, 369(12):19.999|33.536, 38(1):4.278|16.276, 36(0):19.999|37.133, 38(7):19.999|49.256
  40, 555(94):19.999|60.288, 551(5):3.294|6.692, 55(0):1.188|1.403, 57(0):12.233|45.886, 54(1):18.236|58.101
  50, 394(81):19.999|56.410, 395(9):8.006|37.376, 39(0):1.008|1.441, 39(0):19.999|58.982, 41(8):19.999|39.037
  60, 518(65):19.478|32.064, 517(5):4.041|18.009, 52(2):5.731|11.318, 52(0):19.999|25.891, 51(3):19.999|24.224
# 每10s输出一次压测数据,以逗号分隔,总共6列
# 第一列：第N次10s
# 第二列：总成功执行压测的次数(总推迟执行压测的次数):90%事务的响应时间|本轮测试最大响应时间
# 第三列：创建订单业务执行成功次数(推迟执行次数):90%事务的响应时间|本轮测试最大响应时间
# 第四列：支付订单业务执行成功次数(推迟执行次数):90%事务的响应时间|本轮测试最大响应时间
# 第五列：发货业务执行成功次数(推迟执行次数):90%事务的响应时间|本轮测试最大响应时间
# 第六列：查询库存业务执行成功次数(推迟执行次数):90%事务的响应时间|本轮测试最大响应时间

STOPPING THREADS.....

<Raw Results>
  [0] sc:1825  lt:478  rt:0  fl:0 
  [1] sc:2254  lt:45  rt:0  fl:0 
  [2] sc:226  lt:4  rt:0  fl:0 
  [3] sc:230  lt:0  rt:0  fl:0 
  [4] sc:198  lt:34  rt:0  fl:0 
 in 60 sec.
# 第一次结果统计
# [0]:创建订单业务统计成功次数(success),延迟次数(late),重试次数(retry),失败次数(failure)
# [1]:支付订单业务统计成功次数(success),延迟次数(late),重试次数(retry),失败次数(failure)
# [2]:查询订单状态业务统计成功次数(success),延迟次数(late),重试次数(retry),失败次数(failure)
# [3]:发货业务统计成功次数(success),延迟次数(late),重试次数(retry),失败次数(failure)
# [4]:查询库存业务统计成功次数(success),延迟次数(late),重试次数(retry),失败次数(failure)

<Raw Results2(sum ver.)>
  [0] sc:1826  lt:478  rt:0  fl:0 
  [1] sc:2254  lt:45  rt:0  fl:0 
  [2] sc:226  lt:4  rt:0  fl:0 
  [3] sc:230  lt:0  rt:0  fl:0 
  [4] sc:198  lt:34  rt:0  fl:0 
# 第二次结果统计,与第一次结果相同

<Constraint Check> (all must be [OK])   -- 所有的都必须为OK才能通过
 [transaction percentage]
        Payment: 43.43% (>=43.0%) [OK]    -- 支付成功次数(上述结果中sc+lt>43%,才为OK,否则为NG)
   Order-Status: 4.34% (>= 4.0%) [OK]   -- 订单状态(上述结果中sc+lt>4%,才为OK,否则为NG)
       Delivery: 4.34% (>= 4.0%) [OK]   -- 发货业务(上述结果中sc+lt>4%,才为OK,否则为NG)
    Stock-Level: 4.38% (>= 4.0%) [OK]   -- 库存业务(上述结果中sc+lt>4%,才为OK,否则为NG)
 [response time (at least 90% passed)]    -- 响应耗时指标必须通过90%才为OK,否则NG
      New-Order: 79.24%  [NG] *       -- 新订单
        Payment: 98.04%  [OK]       -- 支付
   Order-Status: 98.26%  [OK]       -- 订单状态
       Delivery: 100.00%  [OK]        -- 发货
    Stock-Level: 85.34%  [NG] *       -- 库存

<TpmC>                    -- 最终的测试结果
                 2303.000 TpmC        -- TpmC结果值,即每分钟的事务数
```


参考链接
--------
http://imysql.cn/2012/08/04/tpcc-for-mysql-manual.html  
http://mp.weixin.qq.com/s?__biz=MjM5NzAzMTY4NQ==&mid=200754078&idx=1&sn=91fd3b9ea7d4a20ea90c4eafe4f9fa3c&3rd=MzA3MDU4NTYzMw==&scene=6#rd  
叶金荣修改过的TPCC-MySQL版本  https://github.com/yejr/tpcc-mysql  