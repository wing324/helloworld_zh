## 不听话的timestamp类型

##### 一、前因后果

今天开发提交了一个这样的表结构(做了无用信息的清理)给我：

```sql
create table t(c1 timestamp not null,
               c2 timestamp not null)
```

但是发现，操作完毕之后，数据库中的表结构变成了这样：

```sql
create table t(c1 timestamp not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
               c2 timestamp not null DEFAULT '0000-00-00 00:00:00')
```

于是我百思不得其解，这是为什么为什么为什么呢？。。

##### 二、动手实验

创建如下表结构：

```sql
create table t1(c1 timestamp not null,
                c2 timestamp not null,
                c3 timestamp not null,
                c4 timestamp null);

 create table t2(c1 datetime not null,
                 c2 datetime not null,
                 c3 datetime not null,
                 c4 datetime null);

create table t3(c1 timestamp not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                c2 timestamp not null,
                c3 timestamp not null,
                c4 timestamp null);

create table t4(c1 timestamp not null,
                c2 timestamp not null,
                c3 timestamp not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
                c4 timestamp null);

create table t5(c1 timestamp not null DEFAULT CURRENT_TIMESTAMP,
                c2 timestamp not null,
                c3 timestamp not null,
                c4 timestamp null);

create table t6(c1 timestamp not null,
                c2 timestamp not null,
                c3 timestamp not null DEFAULT CURRENT_TIMESTAMP,
                c4 timestamp null);
```

操作完毕后，数据库实际存在的表结构：

```sql
-- 参数explicit_defaults_for_timestamp=off
CREATE TABLE `t1` (
  `c1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `c2` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c3` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c4` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `t2` (
  `c1` datetime NOT NULL,
  `c2` datetime NOT NULL,
  `c3` datetime NOT NULL,
  `c4` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `t3` (
  `c1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `c2` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c3` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c4` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `t4` (
  `c1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `c2` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c3` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `c4` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `t5` (
  `c1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `c2` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c3` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c4` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8

CREATE TABLE `t6` (
  `c1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `c2` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `c3` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `c4` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

##### 三、得出结论

- timestamp类型的not null 必须和default值共存，datetime类型的not null不需要default值共存；(t1 VS t2)
- 如果timestamp类型的not null没有和default值共存的情况下，MySQL/MariaDB处理方式如下：
  第一个timestamp not null类型自动变更为：timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP，
  第一个timestamp not null类型之后的所有该类型，自动变更为：timestamp NOT NULL DEFAULT '0000-00-00 00:00:00'；(t1)
- 如果timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP类型位于第一个timestamp not null类型字段，那么其他的timestamp not null类型字段自动处理为timestamp NOT NULL DEFAULT '0000-00-00 00:00:00'，否则第一个timestamp not null类型字段依旧会自动处理为timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP类型；（t3 VS t4）
- 如果timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP 类型位于第一个timestamp not null类型字段，那么其他的timestamp not null类型字段自动处理为timestamp NOT NULL DEFAULT '0000-00-00 00:00:00'，否则第一个timestamp not null类型字段依旧会自动处理为timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP类型；(t5 VS t6)