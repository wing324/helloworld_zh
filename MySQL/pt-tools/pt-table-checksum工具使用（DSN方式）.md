## pt-table-checksum工具使用（DSN方式）

pt-table-checksum是校验主从是否一致的校验工具。

本文为什么使用的是DSN方式呢，因为喜欢自由，DSN方式可以按照自己指定的方式发现从库，并且查阅网上教程很难发现完整的DSN使用方式，导致我研究了好久，所以总结下自己的研究成果。



##### 一、原理

pt-table-checksum在基于binlog_format=row的模式下分别主库和从库上执行checksum的SQL语句，分别计算主库和从库表数据块的checksum值。而后比较主从的checksum值是否一致来判断主从数据是否一致。数据块来自于根据主键／唯一索引将表的记录(row)切分成单个数据库(chunck)，同时也降低了数据库的负载。



##### 二、安全性

1. pt-table-checksum一次检查一个表，并且将每个表的数据切分成多个单个的数据块，从而降低数据库的压力，并且在pt-table-checksum的执行过程中，它会自动检测数据库的负载、主从复制延迟，如果负载过大或者主从延迟过大，此时pt-table-checksum会自动暂停，进而降低数据库的负载或者主从复制延迟；
2. pt-table-checksum自动设置session级别的 `innodb_lock_wait_timeout` 为1s，避免影响业务SQL的lock wait timeout；
3. pt-table-checksum使用`--max-load`参数控制数据库负载，当数据库负载超过设置的阀值时，pt-table-checksum将会暂停自己，降低数据库压力。默认情况下，`--max-load`的主从复制延迟阀值为1s，并发线程(Threads_running)为25；
4. 当用 Ctrl+C 停止任务后，工具会正常的完成当前 chunk 检测，下次使用 `--resume` 选项启动可以恢复继续下一个 chunk。



##### 三、工作原理

1. pt-table-checksum连接到主库，并执行一系列变更参数语句，如

   ```sql
   SET SESSION innodb_lock_wait_timeout=1;
   SET @@SQL_QUOTE_SHOW_CREATE = 1/*!40101, @@SQL_MODE='NO_AUTO_VALUE_ON_ZERO,NO_ENGINE_SUBSTITUTION'*/;
   SET SQL_MODE='NO_AUTO_VALUE_ON_ZERO,NO_ENGINE_SUBSTITUTION'/*!50108 SET @@binlog_format := 'STATEMENT'*/;
   SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
   ```

2. 从主库上查找它的从库，默认通过show full processlist方式，还可以通过show slave hosts以及DSN方式发现从库；

3. 查找从库是否有复制过滤规则，可以通过`--nocheck-replication-filters`跳过该项检查；

4. 检查／创建checksums数据表，在确认数据表已经创建的前提下，可以通过`--[no]create-replicate-table`跳过该项检查；

5. 获取一个个数据表，并依次检查；

6. 如果是表的第一个chunk，那么chunk-size一般为1000；如果不是表的第一个chunk，那么采用19步中分析出的结果；

7. 检查表结构，进行数据类型转换等，生成checksum的sql语句；

8. 根据表上的索引和数据的分布，选择最合适的split表的方法；

9. 开始对表做checksum操作；

10. 删除该表的上一次checksum值，除非使用`--resume`参数；

11. 进入需要做checksum的数据库中，USE database;

12. 获取下一个chunk的数据记录；

13. explain该chunk的checksum语句；

14. 根据explain的结果判断该chunk的大小是否超过自定义的`--chunk-size`的大小，如果超过，将会忽略计算出来的chunk-size，使用自定义的`--chunk-size`；

    ```sql
    EXPLAIN SELECT COUNT(*) AS cnt, COALESCE(LOWER(CONV(BIT_XOR(CAST(CRC32(CONCAT_WS('#', `id`, CONCAT(ISNULL(`id`)))) AS UNSIGNED)), 10, 16)), 0) AS crc FROM `xxxx`.`xxxx` /*explain checksum table*/
    ```

15. 执行该chunk的checksum语句，该过程中会把该chunk上的记录加上for update锁；

    ```sql
    REPLACE INTO `percona`.`checksums` (db, tbl, chunk, chunk_index, lower_boundary, upper_boundary, this_cnt, this_crc) SELECT 'wing', 't', '1', NULL, NULL, NULL, COUNT(*) AS cnt, COALESCE(LOWER(CONV(BIT_XOR(CAST(CRC32(CONCAT_WS('#', `id`, CONCAT(ISNULL(`id`)))) AS UNSIGNED)), 10, 16)), 0) AS crc FROM `xxxx`.`xxxx` /*checksum table*/
    ```

16. show warnings查看是否有警告信息等；

17. 获取chunk的checksum值

    ```sql
    SELECT this_crc, this_cnt FROM `percona`.`checksums` WHERE db = 'xxx' AND tbl = 'xxx' AND chunk = '1'
    ```

18. 更新主库chunk的checksum值

    ```sql
    UPDATE `percona`.`checksums` SET chunk_time = 'xxx', master_crc = 'xxxx', master_cnt = 'xxx' WHERE db = 'xxx' AND tbl = 'xxx' AND chunk = '1'
    ```

19. 调整下一个chunk的大小；

20. 等待从库追上主库，此时如果主从延迟时间超过`--max-load`的设置值，pt工具将会暂停自己；

21. 如果发现主库的max-load超过某个阈值，pt工具在这里将暂停；

22. 继续下一个chunk，直到这个table被chunk完毕；

23. 等待从库执行完checksum，便于生成汇总的统计结果；

24. 发现并输出该表的主从是否存在差异；

25. 循环每张表，直到所有表被检查完毕；

26. 如果有warn/err/diff，则退出代码为1，正常退出为0。



##### 四、使用示例

```shell
# 本示例为DSN方式检查
# 主库上创建percona数据库
CREATE DATABASE percona;

# 主库上创建checksums和dsns表
CREATE TABLE checksums (
   db             CHAR(64)     NOT NULL,
   tbl            CHAR(64)     NOT NULL,
   chunk          INT          NOT NULL,
   chunk_time     FLOAT            NULL,
   chunk_index    VARCHAR(200)     NULL,
   lower_boundary TEXT             NULL,
   upper_boundary TEXT             NULL,
   this_crc       CHAR(40)     NOT NULL,
   this_cnt       INT          NOT NULL,
   master_crc     CHAR(40)         NULL,
   master_cnt     INT              NULL,
   ts             TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   PRIMARY KEY (db, tbl, chunk),
   INDEX ts_db_tbl (ts, db, tbl)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
CREATE TABLE `dsns` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) DEFAULT NULL,
  `dsn` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);

# 主库上创建一个用户，从pt测试的服务器上既可以通过该用户连接主库，也可以通过该用户连接从库
GRANT ALL PRIVILEGEES on percona.* to 'percona_user'@'%' IDENTIFIED BY 'percona_pass';
GRANT SELECT,LOCK TABLES,PROCESS,SUPER on *.* to 'percona_user'@'%';

# 主库上dsns表插入从库信息
INSERT INTO dsns(dsn) VALUES('h=repl_host,P=repl_port')

# 连接主库执行pt工具
pt-table-checksum h='master_host',u='percona_user',p='percona_pass',P=master_port --databases=xxx --tables=xx,xxx --nocheck-replication-filters --nocreate-replicate-table --no-check-binlog-format --recursion-method dsn=h=repl_host,P=repl_port,D='percona',t='dsns'
# 返回结果
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
03-08T20:31:22      0      1       xx       xx       0   1.058 xxx.xx
03-08T20:32:22      0      0       xx       xx       0   1.058 xxx.xxx
```



##### 五、结果解析

```shell
#  pt-table-checksum工具检测返回结果
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
03-08T20:31:22      0      1       xx       xx       0   1.058 xxx.xx
03-08T20:32:22      0      0       xx       xx       0   1.058 xxx.xxx
```

TS：pt工具结束执行的时间；

ERRORS：checksum表的时候出现的error和warn的次数；

DIFFS：当为0时，该表主从不存在差异，当为1时，该表主从存在差异；

ROWS：进行checksum的表的记录数；

CHUNKS：表被split为N个chunk，N为chunk的数量；

SKIPPED：由于以下原因，pt跳过的chunk数；

>```
>* MySQL not using the --chunk-index
>* MySQL not using the full chunk index (--[no]check-plan)
>* Chunk size is greater than --chunk-size * --chunk-size-limit
>* Lock wait timeout exceeded (--retries)
>* Checksum query killed (--retries)
>```

TIME：checksum表的时间长度；

TABLE：checksum相应的表名。



##### 六、发现差异

那么如何通过checksums表发现差异呢，在每一个从库上执行如下SQL，即可选出有差异的表：

```sql
SELECT db, tbl, SUM(this_cnt) AS total_rows, COUNT(*) AS chunks
FROM percona.checksums
WHERE (
 master_cnt <> this_cnt
 OR master_crc <> this_crc
 OR ISNULL(master_crc) <> ISNULL(this_crc))
GROUP BY db, tbl;
```



##### 七、常用参数

**--nocheck-binlog-format**

不检查所有数据库的binlog_format是否一致，请在确认所有数据库的binlog_format一致的情况下，使用该参数。

**--chunk-index=s**

使用该索引将表split到多个chunk中。

**--chunk-size=z**

指定chunk的大小，默认为1000，即一个chunk容纳1000条数据记录。

**--chunk-time=f**

动态调整--chunk-size，当一个chunk的checksum时间超过该--chunk-time的时间。

**--nocreate-replicate-table**

不创建`--replicate`指定的database和table。

**--function=s**

指定checksum的hash函数，选择项：FNV1A_64,MURMUR_HASH, SHA1, MD5, CRC32等。

**--recursion-method=a**

指定发现slave的方式，选择项：

**--replicate=s**

将checksum的指定结果写入到该表中，默认为percona.checksums。

**--resume**

从上一次中断的已完成的chunk开始继续执行下一个chunk，最终完成整个表的checksum。

**--where=s**

仅仅校验该where条件匹配的行数。

**--ask-pass**

交互式输入数据库验证码，命令行推荐使用。

**--host**

数据库host。

**--port**

数据库port。

**--password**

数据库密码。

**--user**

数据库用户。

**--socket**

数据库的socket文件。

**--columns=a**

仅仅校验指定的列。

**--databases=h**

仅仅校验指定的数据库。

**--databases-regex=s**

仅仅校验与该参数匹配的数据库。

**--engines=h**

仅仅校验指定engine的表。

**--tables=h**

仅仅校验指定的表。

**--tables-regex=s**

仅仅校验与该参数匹配的表。























参考文档：[用pt-table-checksum校验数据一致性](http://www.nettedfish.com/blog/2013/06/04/check-replication-consistency-by-pt-table-checksum/)