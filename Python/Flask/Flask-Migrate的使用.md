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





*本文整理参考文档：《Flask web开发》*