## GitHub开源MySQL Online DDL工具gh-ost参数解析

gh-ost版本：1.0.28

**-allow-master-master**

> 允许gh-ost运行在双主复制架构中，一般与`-assume-master-host`参数一起使用。

**-allow-nullable-unique-key**

> 允许gh-ost在数据迁移(migrate)依赖的唯一键可以为NULL，默认为不允许为NULL的唯一键。如果数据迁移(migrate)依赖的唯一键允许NULL值，则可能造成数据不正确，请谨慎使用。

**-allow-on-master**

> 允许gh-ost直接运行在主库上。默认gh-ost连接的从库。

**-alter string**

> ALTER语句的body部分，如"ALTER TABLE wing ADD COLUMN id int not null default 0"，使用gh-ost的`-alter`参数时，写成`-alter ADD COLUMN id int not null default 0 `即可。

**-approve-renamed-columns ALTER**

> 如果你修改一个列的名字(如change column)，gh-ost将会识别到并且需要提供重命名列名的原因，默认情况下gh-ost是不继续执行的，除非提供`-approve-renamed-columns ALTER`。
>
> e.g:
>
> **没有**添加`-approve-renamed-columns ALTER`参数，并且修改列名：
>
> `gh-ost -user="wing" -host="127.0.0.1" -port=3306 -database="wing" -table="t" -password="wing" -alter="change column col1 c1 int not null default 0"  -assume-rbr -execute`
> 2016-12-07 16:39:55 FATAL gh-ost believes the ALTER statement renames columns, as follows: map[col1:col11]; as precation, you are asked to confirm gh-ost is correct, and provide with `--approve-renamed-columns`, and we're all happy. Or you can skip renamed columns via `--skip-renamed-columns`, in which case column data may be lost
>
> **没有**添加`-approve-renamed-columns ALTER`参数，并且修改列名为相同名称:
>
> `gh-ost -user="wing" -host="127.0.0.1" -port=3306 -database="wing" -table="t" -password="wing" -alter="change column c1 c1 int not null default 0"  -assume-rbr -execute`
>
> Migrating `wing`.`t`; Ghost table is `wing`.`_t_gho`
> Migrating  wing-01:3306; inspecting  wing-02:3306; executing on  wing-02
> Migration started at Wed Dec 07 17:03:22 +0800 2016
> chunk-size: 1000; max-lag-millis: 1500ms; max-load: ; critical-load: ; nice-ratio: 0.000000
> throttle-additional-flag-file: /tmp/gh-ost.throttle
> Serving on unix socket: /tmp/gh-ost.wing.t.sock
> Copy: 0/3 0.0%; Applied: 0; Backlog: 0/100; Time: 0s(total), 0s(copy); streamer: mysql-bin.000005:62874; State: migrating; ETA: N/A
> Copy: 0/3 0.0%; Applied: 0; Backlog: 0/100; Time: 1s(total), 1s(copy); streamer: mysql-bin.000005:64686; State: migrating; ETA: N/A
> Copy: 3/3 100.0%; Applied: 0; Backlog: 0/100; Time: 1s(total), 1s(copy); streamer: mysql-bin.000005:64686; State: migrating; ETA: due
> Copy: 3/3 100.0%; Applied: 0; Backlog: 1/100; Time: 2s(total), 1s(copy); streamer: mysql-bin.000005:69035; State: migrating; ETA: due
> Migrating `wing`.`t`; Ghost table is `wing`.`_t_gho`
> Migrating  wing-01:3306; inspecting  wing-02:3306; executing on  wing-02
> Migration started at Wed Dec 07 17:03:22 +0800 2016
> chunk-size: 1000; max-lag-millis: 1500ms; max-load: ; critical-load: ; nice-ratio: 0.000000
> throttle-additional-flag-file: /tmp/gh-ost.throttle
> Serving on unix socket: /tmp/gh-ost.wing.t.sock
> Copy: 3/3 100.0%; Applied: 0; Backlog: 0/100; Time: 2s(total), 1s(copy); streamer: mysql-bin.000005:69759; State: migrating; ETA: due
>
> 添加`-approve-renamed-columns ALTER`参数，并且修改列名：
>
> `gh-ost -user="wing" -host="127.0.0.1" -port=3306 -database="wing" -table="t" -password="wing" -alter="change column col1 c1 int not null default 0"  -assume-rbr -execute -approve-renamed-columns ALTER`
>
> Migrating `wing`.`t`; Ghost table is `wing`.`_t_gho`
> Migrating  wing-01:3306; inspecting wing-02:3306; executing on wing-02
> Migration started at Wed Dec 07 16:42:16 +0800 2016
> chunk-size: 1000; max-lag-millis: 1500ms; max-load: ; critical-load: ; nice-ratio: 0.000000
> throttle-additional-flag-file: /tmp/gh-ost.throttle
> Serving on unix socket: /tmp/gh-ost.wing.t.sock
> Copy: 0/3 0.0%; Applied: 0; Backlog: 0/100; Time: 0s(total), 0s(copy); streamer: mysql-bin.000005:41767; State: migrating; ETA: N/A
> Copy: 0/3 0.0%; Applied: 0; Backlog: 0/100; Time: 1s(total), 1s(copy); streamer: mysql-bin.000005:43576; State: migrating; ETA: N/A
> Copy: 3/3 100.0%; Applied: 0; Backlog: 0/100; Time: 1s(total), 1s(copy); streamer: mysql-bin.000005:43576; State: migrating; ETA: due
> Copy: 3/3 100.0%; Applied: 0; Backlog: 1/100; Time: 2s(total), 1s(copy); streamer: mysql-bin.000005:47927; State: migrating; ETA: due
> Migrating `wing`.`t`; Ghost table is `wing`.`_t_gho`
> Migrating  wing-01:3306; inspecting  wing-02:3306; executing on  wing-02
> Migration started at Wed Dec 07 16:42:16 +0800 2016
> chunk-size: 1000; max-lag-millis: 1500ms; max-load: ; critical-load: ; nice-ratio: 0.000000
> throttle-additional-flag-file: /tmp/gh-ost.throttle
> Serving on unix socket: /tmp/gh-ost.wing.t.sock
> Copy: 3/3 100.0%; Applied: 0; Backlog: 0/100; Time: 2s(total), 1s(copy); streamer: mysql-bin.000005:48651; State: migrating; ETA: due

**-assume-master-host string**

> 为gh-ost指定一个主库，格式为"ip:port"或者"hostname:port"。默认推荐gh-ost连接从库。

**-assume-rbr**

> 确认gh-ost连接的数据库实例的binlog_format=ROW的情况下，可以指定`-assume-rbr`，这样可以禁止从库上运行`stop slave`,`start slave`,执行gh-ost用户也不需要SUPER权限。

**-chunk-size int**

> 在每次迭代中处理的行数量(允许范围：100-100000)，默认值为1000。

**-concurrent-rowcount**

> 该参数如果为True(默认值)，则进行row-copy之后，估算统计行数(使用explain select count(*)方式)，并调整ETA时间，否则，gh-ost首先预估统计行数，然后开始row-copy。

 **-conf string**

> gh-ost的配置文件路径。

**-critical-load string**

> 一系列逗号分隔的status-name=values组成，当MySQL中status超过对应的values，gh-ost将会退出。
>
> e.g:
>
> `-critical-load Threads_connected=20,Connections=1500`
>
> 指的是当MySQL中的状态值Threads_connected>20,Connections>1500的时候，gh-ost将会由于该数据库严重负载而停止并退出。

**-critical-load-interval-millis int**

> 当值为0时，当达到`-critical-load`，gh-ost立即退出。当值不为0时，当达到`-critical-load`，gh-ost会在`-critical-load-interval-millis`秒数后，再次进行检查，再次检查依旧达到`-critical-load`，gh-ost将会退出。

**-cut-over string**

> 选择cut-over类型:atomic/two-step，atomic(默认)类型的cut-over是github的算法，two-step采用的是facebook-OSC的算法。

**-cut-over-lock-timeout-seconds int**

> gh-ost在cut-over阶段最大的锁等待时间，当锁超时时，gh-ost的cut-over将重试。(默认值：3)

**-database string**

> 数据库名称。

**-debug**

> debug模式。

**-default-retries int**

> 各种操作在panick前重试次数。(默认为60)

**-discard-foreign-keys**

> 很危险的参数，慎用！
>
> 该参数针对一个有外键的表，在gh-ost创建ghost表时，并不会为ghost表创建外键。该参数很适合用于删除外键，除此之外，请谨慎使用。

**-exact-rowcount**

> 准确统计表行数(使用select count(*)的方式)，得到更准确的预估时间。

**-execute**

> 实际执行alter&migrate表，默认为不执行，仅仅做测试并退出，如果想要ALTER TABLE语句真正落实到数据库中去，需要明确指定`-execute`。

**-force-named-cut-over**

> When true, the 'unpostpone|cut-over' interactive command must name the migrated table。

**-heartbeat-interval-millis int**

> gh-ost心跳频率值，默认为500。

**-help**

> 显示gh-ost的帮助信息。

**-hooks-hint string**

> arbitrary message to be injected to hooks via GH_OST_HOOKS_HINT, for your convenience。

**-hooks-path string**

> hook文件存放目录(默认为empty，即禁用hook)。hook会在这个目录下寻找符合约定命名的hook文件来执行。

**-host string**

> 数据库的ip/hostname。(默认值：127.0.0.1)。

**-initially-drop-ghost-table**

> gh-ost操作之前，检查并删除已经存在的ghost表。该参数不建议使用，请手动处理原来存在的ghost表。默认不启用该参数，gh-ost直接退出操作。

**-initially-drop-old-table**

> gh-ost操作之前，检查并删除已经存在的旧表。该参数不建议使用，请手动处理原来存在的ghost表。默认不启用该参数，gh-ost直接退出操作。

**-initially-drop-socket-file**

> gh-ost强制删除已经存在的socket文件。该参数不建议使用，可能会删除一个正在运行的gh-ost程序，导致DDL失败。

**-max-lag-millis int**

> 主从复制最大延迟时间，当主从复制延迟时间超过该值后，gh-ost将采取节流(throttle)措施，默认值：1500s。

**-max-load string**

> 一系列逗号分隔的status-name=values组成，当MySQL中status超过对应的values，gh-ost将采取节流(throttle)措施。
>
> e.g:
>
> `-max-load Threads_connected=20,Connections=1500`
>
> 指的是当MySQL中的状态值Threads_connected>20,Connections>1500的时候，gh-ost将采取节流(throttle)措施。

**-migrate-on-replica**

> gh-ost的数据迁移(migrate)运行在从库上，而不是主库上。
> Have the migration run on a replica, not on the master. This will do the full migration on the replica including cut-over (as opposed to --test-on-replica)

**-nice-ratio float**

> 每次chunk时间段的休眠时间，范围[0.0...100.0]。
>
> e.g:
>
> 0：每个chunk时间段不休眠，即一个chunk接着一个chunk执行；
>
> 1：每row-copy 1毫秒，则另外休眠1毫秒；
>
> 0.7：每row-copy 10毫秒，则另外休眠7毫秒。

**-ok-to-drop-table**

> gh-ost操作结束后，删除旧表，默认状态是不删除旧表，会存在_tablename_del表。

**-panic-flag-file string**

> 当这个文件被创建，gh-ost将会立即退出。

**-password string**

> MySQL密码。

**-port int**

> MySQL端口。

**-postpone-cut-over-flag-file string**

> 当这个文件存在的时候，gh-ost的cut-over阶段将会被推迟，直到该文件被删除。

**-quiet**

> 静默模式。

**-replication-lag-query string**

> 检查主从复制延迟的SQL语句，默认gh-ost通过show slave status获取Seconds_behind_master作为主从延迟时间依据。如果使用pt-heartbeat工具，检查主从复制延迟的SQL语句类似于:
>
> `SELECT ROUND(UNIX_TIMESTAMP() - MAX(UNIX_TIMESTAMP(ts))) AS delay FROM my_schema.heartbeat;`

**-serve-socket-file string**

> gh-ost的socket文件绝对路径。

**-serve-tcp-port int**

> gh-ost使用端口，默认为关闭端口。

**-skip-renamed-columns ALTER**

> 如果你修改一个列的名字(如change column)，gh-ost将会识别到并且需要提供重命名列名的原因，默认情况下gh-ost是不继续执行的。该参数告诉gh-ost跳该列的数据迁移，让gh-ost把重命名列作为无关紧要的列。该操作很危险，你会损失该列的所有值。
>
> e.g:
>
> 原始表数据：
>
> ```sql
> mysql> select * from t;
> +----+------+----+------+------+------+-------+
> | id | name | c1 | col2 | col3 | col4 | col11 |
> +----+------+----+------+------+------+-------+
> |  1 |      | 22 |    0 |    0 |    0 |     0 |
> |  2 |      | 22 |    0 |    0 |    0 |     0 |
> |  3 |      | 22 |    0 |    0 |    0 |     0 |
> +----+------+----+------+------+------+-------+
> ```
>
> 执行命令：
>
> `gh-ost -user="wing" -host="127.0.0.1" -port=3306 -database="wing" -table="t" -password="wing" -alter="change column c1 col1 int not null default 0"  -assume-rbr -execute -skip-renamed-columns ALTER` 
>
> 操作后表数据:
>
> ```sql
> mysql> select * from t;
> +----+------+------+------+------+------+-------+
> | id | name | col1 | col2 | col3 | col4 | col11 |
> +----+------+------+------+------+------+-------+
> |  1 |      |      |    0 |    0 |    0 |     0 |
> |  2 |      |      |    0 |    0 |    0 |     0 |
> |  3 |      |      |    0 |    0 |    0 |     0 |
> +----+------+------+------+------+------+-------+
> 3 rows in set (0.00 sec)
> ```

**-stack**

> 添加错误堆栈追踪。

**-switch-to-rbr**

> 让gh-ost自动将从库的binlog_format转换为ROW格式。

**-table string**

> 表名称。

**-test-on-replica**

> 在从库上测试gh-ost，包括在从库上数据迁移(migration)，数据迁移完成后stop slave，原表和ghost表立刻交换而后立刻交换回来。继续保持stop slave，使你可以对比两张表。

**-test-on-replica-skip-replica-stop**

>  当`-test-on-replica`执行时，该参数表示该过程中不用stop slave。

**-throttle-additional-flag-file string**

> 当该文件被创建后，gh-ost操作立即停止。该参数可以用在多个gh-ost同时操作的时候，创建一个文件，让所有的gh-ost操作停止，或者删除这个文件，让所有的gh-ost操作恢复。

**-throttle-control-replicas string**

> 列出所有需要被检查主从复制延迟的从库。
>
> e.g:
>
> `-throttle-control-replica=192.16.12.22:3306,192.16.12.23:3307,192.16.13.12:3308`

**-throttle-flag-file string**

> 当该文件被创建后，gh-ost操作立即停止。该参数适合控制单个gh-ost操作。`-throttle-additional-flag-file string`适合控制多个gh-ost操作。

**-throttle-query string**

> 节流查询。每秒钟执行一次。当返回值=0时不需要节流，当返回值>0时，需要执行节流操作。该查询会在数据迁移(migrated)服务器上操作，所以请确保该查询是轻量级的。

**-tungsten**

> 告诉gh-ost你正在运行的是一个tungsten-replication拓扑结构。

**-user string**

> MySQL用户。

**-verbose**

> gh-ost运行时输出详细信息。

**-version**

> 输出gh-ost版本信息并退出。