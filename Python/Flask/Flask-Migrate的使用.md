## Flask-Migrate的使用

**安装命令**

```python
pip install flask-migrate
```

**注意⚠️**

相关的表结构示例，[请点击该链接。](https://github.com/wing324/helloworld_zh/blob/master/Python/Flask/%E4%BD%BF%E7%94%A8flask-sqlalchemy%E7%8E%A9%E8%BD%ACMySQL.md)

#### 一、配置flask-migrate

```python
# 文件名 hello.py
from flask_migrate import Migrate,MigrateCommand
migrate = Migrate(app,db)
manager.add_command('db',MigrateCommand)
```

#### 二、使用init命令创建迁移仓库

```python
python hello.py db init
  Creating directory /Users/domain/mycoding/web/migrations ... done
  Creating directory /Users/domain/mycoding/web/migrations/versions ... done
  Generating /Users/domain/mycoding/web/migrations/alembic.ini ... done
  Generating /Users/domain/mycoding/web/migrations/env.py ... done
  Generating /Users/domain/mycoding/web/migrations/env.pyc ... done
  Generating /Users/domain/mycoding/web/migrations/README ... done
  Generating /Users/domain/mycoding/web/migrations/script.py.mako ... done
  Please edit configuration/connection/logging settings in '/Users/domain/mycoding/web/migrations/alembic.ini' before proceeding.
teardown request
http://localhost/
```

#### 三、使用migrate命令创建迁移脚本

```python
python hello.py db migrate -m "initial migrateion"
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'roles'
INFO  [alembic.autogenerate.compare] Detected added table 'users'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_users_username' on '['username']'
  Generating /Users/domain/mycoding/web/migrations/versions/e89e2c8b3174_initial_migrateion.py ... done
```

该步骤生成的迁移脚本，可在./migrations/versions/的文件夹下面查看，迁移脚本中的upgrade()和downgrade()函数，它们的作用分别是：

> upgrade()：将迁移中的改动应用到数据库中。
>
> downgrade()：将迁移中的改动从数据库中删除，即具有回滚到某个迁移点的功能。

该步骤还会在数据库中生成`alembic_version`表，该表作为flask-migrateion的迁移记录表。

#### 四、使用upgrade命令将迁移中的改动应用到数据库中

```python
python hello.py db upgrade
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> e89e2c8b3174, initial migrateion
```

五、使用downgrade命令回滚迁移中的数据库改动

```python
# 首先在之前flaks-sqlalchemy表结构的user表里面添加一个email字段
# class User(db.Model):
# ...
# email = db.Column(db.String(32), nullable=False, server_default='')
# ...
# 使用python hello.py db migrate -m 'add email column to user table'命令生成迁移脚本
python hello.py db migrate -m 'add email column to user table'
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added column 'users.email'
  Generating /Users/domain/mycoding/web/migrations/versions/c779bf595f79_add_email_column_to_user_table.py ... done
# 使用python hello.py db upgrade将迁移脚本中的改动应用到
python hello.py db upgrade
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade d303dfaaefba -> c779bf595f79, add email column to user table
# 此时发现并不想做这个变更，想要回滚到上一个版本咋办呢，来来来，我教你个大招
python hello.py db downgrade d303dfaaefba
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running downgrade c779bf595f79 -> d303dfaaefba, add email column to user table
# d303dfaaefba为上一个版本号，可以从数据库中的alembic_version表中查询到，也可以从上面python hello.py db upgrade的最后一个INFO信息中查看到，当然也可以用./migrations/versions/的文件夹下面查看到，总之看到版本号的地方很多，就看你是否是个有心人了～
```

至此，Flask-Migrateion的用法就整理完毕了，当然Flask-Migration还有很多用法，但是上面这些是最常用的用法，更相信的用法请参考[Flask-Migrationg官方文档](https://flask-migrate.readthedocs.io/en/latest/)。
当然也请小伙伴们放心，在整个flask-migration迁移过程中，你的数据也会跟着迁移的，所以完全不用担心哈～

*本文整理参考文档：《Flask web开发》*