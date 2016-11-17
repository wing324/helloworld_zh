## pt-online-schema-change使用

看了一遍pt-osc文档，也用了几天，并且会持续用下去，所以做了一些笔记，方便以后工作吧＝＝


### 原理

1. 根据原表结构创建一个新表；
2. 按照pt-osc的alter语句修改新表；
3. 将原表中的数据copy到新表中去；
4. 将原表copy数据期间的数据更新应用到新表中去（通过触发器实现）；
5. 将原表重命名，将新表重命名到原表名称，默认情况下删除原表。

```shell
# USAGE
pt-online-schema-change --host=xxx.xx.xx.xx --port=xxxx --user=xxxx --ask-pass --charset=utf8 --alter "add column id int not null default 0" D=wing,t=t --execute
```

### 优点

执行Alter阶段不阻塞读和写。

### 限制

1. 原表不能存在触发器，因为pt-osc需要通过触发器将原表copy数据阶段产生的数据应用到新表去。
2. 表必须具有primary key/unique key，为了准确应用原表copy数据期间产生的数据到新表中去。通过primary key/unique key定位数据是否存在新表中。
3. 原表不能是其他外键的父表，需要添加—alter-foreign-keys-method参数即可。
4. 字段属性为NOT NULL时，必须有DEFAULT属性，否则会报错。

### 常用参数

- —alter-foreign-keys-method
  如果原表为其他外键的父表，需要添加该参数。该参数存在如下几个方法：
  auto：自动选择rebuild_constraints/drop_swap方式；
  rebuild_constraints：使用alter table 删除字表的外键约束，在父表结束pt-osc操作后，再在表上创建新的约束（个人不推荐，特别是当子表很大的时候）;
  drop_swap：关闭外键检查约束，即FOREIGN_KEY_CHECKS=0(存在数据不一致的可能性)；
  none：会使foreign key失效，极度不推荐。
- —dry-run
  创建以及修改新表，但是不创建触发器，数据复制以及重命名原表。
- —execute
  执行alter语句。


- —print
  输出pt-osc中copy data的详细过程。

- —quiet
  静默方式输出，即不输出STDOUT信息，只输出STDERR信息。

- —statistics
  输出pt-osc中copy data中INSERT执行的总次数。

- PTDEBUG
  以debug的方式输出pt-osc的输出信息

  ```shell
  PTDEBUG=1 pt-online-schema-change --host=xxx.xx.xx.xx --port=xxxx --user=xxxx --ask-pass --charset=utf8 --alter "add column id int not null default 0" D=wing,t=t --execute
  ```

### 使用pt-osc之前需要做的检查

1. 原表是否存在触发器，存在：no，不存在：yes；
2. 原表是否具有pk/uk，不存在：no,存在：yes；
3. 原表是否为其他外键的父表，是：—alter-foreign-keys-method='xxx'，不是：yes；
4. 字段是否为NOT NULL 属性，不是：yes,是：DEFAULT；

![](/img/pt-osc使用前检查.png)

