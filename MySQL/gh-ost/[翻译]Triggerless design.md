## 不使用Trigger的设计理念

介绍gh-ost不使用Trigger背后的逻辑和算法，其次是这样设计的含义，优点，以及缺点。

#### 基于触发器迁移的理念

这里有两个比较流行的online DDL工具：

- [pt-online-schema-change](https://www.percona.com/doc/percona-toolkit/2.2/pt-online-schema-change.html)
- [Facebook OSC](https://www.facebook.com/notes/mysql-at-facebook/online-schema-change-for-mysql/430801045932/)

前者使用同步设计原理：它在原表上创建三个触发器(AFTER INSERT，AFTER UPDATE，AFTER DELETE)。每个触发器将原表上的操作重播到ghost表中。因此，原表上的每一个UPDATE都会被重播到gh-ost表中，INSERT／DELETE也是如此。触发器和原表的操作存在与同一个事务空间里。

后者使用异步设计原理：它在原表上创建三个触发器(AFTER INSERT，AFTER UPDATE，AFTER DELETE)。它也会创建一个changelog表。触发器不会直接将原表操作重播到ghost表中。取而代之的是，它会在changelog中添加一条记录。原表上的UPDATE操作在changelog表中插入一条记录像"原表上有一个UPDATE操作，将这个值变为了另外一个值"，INSERT／DELETE也是如此。一个后台的线程会实时去查看changelog表的变化，并应用所有的变更应用到ghost表上。这是一个异步的操作，因为gh-ost重播操作的过程与原表的操作不在相同的事务空间中了，并且gh-ost的重播操作可能会延迟。值得注意的是，changelog的记录写入操作还是与原表操作存在于相同的事务空间中。



#### 不基于触发器的异步迁移理念

gh-os使用不基于触发器的异步迁移方案。但是它不需要触发器，因为它不像FB工具那样需要一个changelog表。因为它不是基于changelog表做数据迁移，而是基于MySQL二进制日志(binlog)。

gh-ost基于binlog_format=row(RBR)获取来源于原表的记录，你也可以基于binlog_format=statement(SBR)获取记录，[详情请点击](https://github.com/github/gh-ost/blob/master/doc/migrating-with-sbr.md)。

RBR格式的二进制日志将复杂的SQL语句(可能存在多表)分解为不同的单表单行记录，这样更易于将binlog应用到gh-ost上。

gh-ost将自己伪装成MySQL从库：gh-ost连上主库，并且从主库获取binlog。因此，它获取binlog日志，并且通过过滤得到原表的操作事件。

gh-ost可以直接连接到主库上，但是gh-ost更喜欢连接到一个从库上。但是该从库需要设置参数`log_slave_updates=1`和`binlog_format=row`(binlog_format也可以通过调整gh-ost参数来进行自动设置)。

读取二进制日志，特别是在在从库上读取二进制，进一步强调了算法的异步性。当有一个事物写入到了binlog中，它依旧需要等gh-ost伪装成一个从库之后，才开始获取binlog并且应用到gh-ost中。

异步设计也意味着有很多值得注意的因素，稍后将做讨论。



#### 工作流程概览

整个工作流程包括：原表上读取表数据，binlog读取操作事件，检查主从延迟以及其它的节流参数，将变更应用到ghost表中(通常是master上的ghost表)，通过二进制日志流发送提示等等。

**工作流程如下：**

> 1. 初始化设置&验证
>
>    初始化设置不是一个并发操作。
>
>    - 连接从库(推荐)／主库，检查主库标志
>    - 预验证ALTER语句
>    - 初始化验证：权限以及表是否存在
>    - 创建changelog和ghost表
>    - 在ghost表上执行ALTER语句
>    - 对比原表和ghost表的结构，检查共享列，共享唯一键，验证是否有外键，选择共享的唯一键，这个键用于处理表的唯一标识，比如数据迁移等
>    - 开始监听binlog，监听changelog表的事件
>    - 在changelog表上注入"good to go"的记录(被二进制日志拦截)
>    - 开始监听原表DML的binlog事件
>    - 获取之前原表和ghost表的共享唯一键在原表上的最小值和最大值
>
> 2. 数据复制流程
>
>    该步骤包括多个移动部分，所有操作相互协调并发执行。
>
>    - 设置一个心跳机制：频繁的写入到changelog表(这是一个低负载的操作)
>    - 心跳机制不断的更新状态
>    - 定期(频繁)检查潜在的节流信息或者提示
>    - 在原表上通过行范围控制，将原表数据拆分为一个chunk一个chunk，并且添加到数据迁移任务队列中
>    - 通过binlog获取原表的DML语句，并且添加到binlog重播任务队列中
>    - 处理数据迁移任务队列和binlog重播任务队列，并将其顺序的应用到ghost表上(当遇到节流操作或者hint提示时，将会暂停该操作)
>    - 当数据迁移与binlog重播完成后，将会在changelog表上注入／拦截"copy all done"的记录
>    - 当`-postpone-cut-over-flag-file`参数设置的文件存在时，将会推迟接下来的cut-over操作(但是原表的DML操作依旧会通过binlog应用到ghost表上)
>
> 3. 结束操作：交换表流程
>
>    - 将原表加上写锁，binlog事件会继续应用在gh-ost上(该步骤是个异步操作，因此即使表被锁住，gh-ost仍然可以处理队列中未处理完的binlog事件)
>
>    - 将原表rename为\_tablename\_del表，ghost则rename为tablename表
>
>      ```sql
>      rename /* gh-ost */ table `wing`.`t` to `wing`.`_t_del`, `wing`.`_t_gho` to `wing`.`t`
>      ```
>
>    - 清理工作：删除需要被清除的表



#### 异步设计优点

> 1. Cut-over阶段
>
>    异步设计最复杂的地方在于cut-over阶段：原表和ghost表的交换。在同步设计中，由于原表操作与触发器操作是在同一个事物空间中的，所以原表和ghost表的数据始终是同步的，因此一个简单的原表和ghost表交换(Rename)是可以存在的。
>
>    在异步设计中，即使我们对原表加锁，管道中仍然会存在一些事件，binlog日志依旧会将来自原表的事件重播到ghost表上。采用同步设计中交换表的方式是不可取的，因为这意味着，即使还没有对来自原表的事件重播完毕，就开始使用重命名后的ghost表了，这将造成数据不一致。
>
>    Facebook的使用"中断机制"，二步重命名法：
>
>    - 锁住原表，处理积压的事件(backlog)
>    - 将原表重命名
>    - 将ghost重命名为原表的名字
>
>    在两个表交换的时候，会有一个表不存在的阶段，因此会存在"表中断"的问题。
>
>    gh-ost通过"two-step"算法解决这个问题，"two-step"算法会阻塞表写入，然后将两表交换。它使操作很安全，要么成功，要么失败则回滚到cut-over阶段之前。
>
>    更多的信息请阅读[cut-over](https://github.com/github/gh-ost/blob/master/doc/cut-over.md)
>
> 2. 分离去耦
>
>    不使用触发器的异步设计最大的影响在于工作负载的去耦。使用触发器的设计，不管是同步还是异步方法，原表上的每一个写入意味着需要立刻在另外一张表上写入。
>
>    We will break down the meaning of workload decoupling, shortly. But it is important to understand that gh-ost interprets the situation in its own time and acts in its own time, yet still makes this an online operation.
>
>    The decoupling is important not only as the tool's logic goes, but very importantly as the master server sees it. As far as the master knows, write to the table and writes to the ghost table are unrelated
>
> 3. 写负载
>
>    不使用触发器意味着主库不需要有过多的写负载用在存储过程上，以及对gh-ost表的锁争用上。
>
>    将原表操作重播到ghost表上完全由gh-ost工具完成。因此gh-ost可以决定何时将数据写入到ghost表中。为了分解原表的写负载，gh-ost工具选择使用一个单独的线程将原表操作重播到ghost表上。
>
>    MySQL对一个表大量并发写入的时候性能不是很好，这个时候锁变成了一个很大的问题。This is why we choose to alternate between the massive row-copy and the ongoing binlog events backlog such that the server only sees writes from a single connection。
>
>    更有趣的是，gh-ost是唯一写入到ghost表的程序，没有人意识到ghost表的存在。因此，gh-ost工具不存在由触发器产生的高并发性问题以及资源高争用问题。
>
> 4. 可暂停性
>
>    当gh-ost暂停工作(节流导致)，此时将会没有任何数据写入到ghost表中。因为gh-ost不存在触发器，写负载是与gh-ost写负载分开的。并且由于gh-ost使用异步设计的方法，gh-ost算法已经处理了主库写入与ghost表写入的时间差。对于gh-ost来说，几毫秒的时间差与几小时的时间差并没有任何区别，对于gh-ost的运行并没有任何的影响。
>
>    当gh-ost进行节流操作的时候，不管是因为主从复制延迟，还是达到`max-load`设置阀值等，原表还是正常操作。仅仅只是对ghost表没有任何的写操作。除了changelog表上的heartbeat在不断的更新，但这个操作带来的性能影响是可以忽略不计的。
>
> 5. 可测试性
>
>    我们甚至可以测试数据迁移(migration)步骤：就像我们将数据迁移的操作与主库的负载分开一样，我们不在主库上应用所有的变更，我们选择一个从库，这样我们可以在从库上进行表的数据迁移(migration)。
>
>    这本身是一个很不错的功能；它为我们提供了测试的可能性：正如我们完成数据迁移一样，我们从库的复制(stop slave)。gh-ost会进行cut-over阶段，但是gh-ost也会回滚回去。gh-ost测试时不会删除任何的表。结果是从库上的原表和ghost表都会存在，也不会做进一步的变更操作(因为此时主从复制已经停止)。我们可以对两张表进行对比。
>
>    这个方法可以用于验证gh-ost工具的正确性：在多个生产从库上不断的重复的做"数据迁移"(实际上并不会修改列)。每次数据迁移后都对原表和ghost表做一次数据校验。gh-ost预计是所有的表数据校验都是一致的。
>
> 6. 多表并发操作
>
>    gh-ost可以运行多个不同的表并发操作(当然不是在相同的表上并发操作)。gh-ost异步设计的方法是支持多个不同的表并发操作的。事实上没有触发器的存在，多个不同的表并发操作gh-ost对于主库来说，只是并发的多个连接而已。每个gh-ost都可以控制自己的节流，或者全部一次性控制它们的节流操作。
>
> 7. Going outside the server space
>
>    More to come as we make progress



#### 异步设计缺点

>1. 增加流量
>
>   现有的工具都是通过触发器来重播原表上的操作。gh-ost是通过自己来获取原表操作，然后重播到gh-ost表上。gh-ost当然更喜欢在从库上获取原表操作，然后在主库上重播到gh-ost表上。这也意味着，主库机器和从库机器之间存在数据的传输。并且gh-ost使用的MySQL客户端不支持压缩功能， and so during a migration you can expect the full volume of a table to transfer on the wire。
>
>2. 增加代码复杂性
>
>   基于触发器同步方法的online DDL工具相对来说代码较少。大量的数据迁移是基于触发器完成的。回滚，数据类型以及cut-over阶段都是由数据库隐式处理的。gh-ost的异步方法让它的代码变的更复杂。它分别连接到主库和从库，伪装成从库，向主库写入心跳事件，在从库上读取binlog事件并写入到主库上，它还需要管理连接失败，主从复制延迟等等。
>
>   因此，gh-ost拥有更大的代码库以及更复杂的异步并发逻辑。

