## flask-sqlalchemy的MySQL数据库操作

**安装命令**

```python
pip install mysql-python
pip install flask-sqlalchemy
```

**相关示例Python代码**

```python
# 文件名hello.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
app = Flask(__name__)
app.config['SECRET_KEY'] = 'Fianna'
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://user:password@host:port/dbname'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
db = SQLAlchemy(app)
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, nullable=False, primary_key=True, autoincrement=True)
    name = db.Column(db.String(16), nullable=False, server_default='', unique=True)
    def __repr__(self):
        return '<Role %r>' % self.name
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, nullable=False, primary_key=True, autoincrement=True)
    username = db.Column(db.String(32), nullable=False, unique=True, server_default='', index=True)
    role_id = db.Column(db.Integer, nullable=False, server_default='0')
    def __repr__(self):
        return '<User %r,Role id %r>' %(self.username,self.role_id)
```

#### 一、连接数据库

```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://user:password@host:port/dbname'
```

#### 二、创建所有表

```
python hello.py shell
>>> from hello import db,Role,User
>>> db.create_all()
```

#### 三、删除所有表

```python
python hello.py shell
>>> from hello import db,Role,User
>>> db.drop_all()
```

#### 四、插入行

```python
# 插入单行
python hello.py shell
>>> from hello import db,Role,User
>>> db.session.add(Role(name='Admin'))
>>> db.session.commit()
>>> db.session.add(Role(name='Moderator'))
>>> db.session.add(Role(name='User'))
>>> db.session.commit()

# 插入多行
python hello.py shell
>>> from hello import db,Role,User
>>> db.session.add_all([User(username='john',role_id=1),User(username='susan',role_id=3),User(username='david',role_id=3)])
>>> db.session.commit()
```

#### 五、更新行

```python
python hello.py shell
>>> from hello import db,Role,User
>>> admin = Role.query.filter_by(name='Admin').first()
>>> admin.name='Administrator'
>>> db.session.commit()
```

#### 六、删除行

```python
python hello.py shell
>>> from hello import db,Role,User
>>> mod = Role.query.filter_by(name='Moderator').first()
>>> db.session.delete(mod)
>>> db.session.commit()
```

#### 七、查询表

- 查询表中全部数据

  ```python
  # 注意，此处的查询结果完全取决于代码示例中的
  # def __repr__(self)
  # 这段函数，请各位小主们，好好斟酌下～
  python hello.py shell
  >>> from hello import db,Role,User
  >>> Role.query.all()
  [<Role u'Administrator'>, <Role u'User'>]
  >>> User.query.all()
  [<User u'john',Role id 1L>, <User u'susan',Role id 3L>, <User u'david',Role id 3L>]
  ```

- 按照一个条件过滤数据记录(where)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> Role.query.filter_by(name='Administrator').first()
  <Role u'Administrator'>
  >>> User.query.filter_by(role_id=3).all()
  [<User u'susan',Role id 3L>, <User u'david',Role id 3L>]
  >>> User.query.filter_by(role_id=3).first()
  <User u'susan',Role id 3L>
  ```

- 按照两个条件过滤数据记录(where and)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> User.query.filter_by(role_id=3,username='susan').first()
  <User u'susan',Role id 3L>
  >>> User.query.filter_by(role_id=3,username='susan').all()
  [<User u'susan',Role id 3L>]
  ```

- 聚合(count)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> User.query.filter_by(role_id=3,username='susan').count()
  1L
  >>> User.query.filter_by(role_id=3).count()
  2L
  >>> User.query.count()
  3L
  ```

- 求和(sum)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> from sqlalchemy.sql import func
  >>> User.query.with_entities(func.sum(User.id)).all()
  [(Decimal('6'),)]
  >>> User.query.with_entities(func.sum(User.role_id)).all()
  [(Decimal('7'),)]
  ```

- 平均数(avg)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> from sqlalchemy.sql import func
  >>> User.query.with_entities(func.avg(User.role_id)).all()
  [(Decimal('2.3333'),)]
  >>> User.query.with_entities(func.avg(User.id)).all()
  [(Decimal('2.0000'),)]
  ```

- 排序(order by)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  # 升序(asc)
  >>> User.query.order_by(User.role_id).all()
  [<User u'john',Role id 1L>, <User u'susan',Role id 3L>, <User u'david',Role id 3L>]
  # 降序(desc)
  >>> User.query.order_by(User.role_id.desc()).all()
  [<User u'susan',Role id 3L>, <User u'david',Role id 3L>, <User u'john',Role id 1L>]
  ```

- 分组(group by)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> User.query.group_by(User.role_id).all()
  [<User u'john',Role id 1L>, <User u'susan',Role id 3L>]
  ```

- 限制(limit)

  ```python
  python hello.py shell
  >>> from hello import db,Role,User
  >>> User.query.all()
  [<User u'john',Role id 1L>, <User u'susan',Role id 3L>, <User u'david',Role id 3L>]
  # limit 1
  >>> User.query.limit(1).all()
  [<User u'john',Role id 1L>]
  # limit 2,1
  >>> User.query.limit(1).offset(2).all()
  [<User u'david',Role id 3L>]
  >>> User.query.filter_by(role_id=3).all()
  [<User u'susan',Role id 3L>, <User u'david',Role id 3L>]
  # limit 1
  >>> User.query.filter_by(role_id=3).limit(1).all()
  [<User u'susan',Role id 3L>]
  # limit 1,1
  >>> User.query.filter_by(role_id=3).limit(1).offset(1).all()
  [<User u'david',Role id 3L>]
  ```

#### 八、将Flask-SQLAlchemy的查询语句转换为SQL

```python
python hello.py shell
>>> from hello import db,Role,User
>>> User.query.all()
[<User u'john',Role id 1L>, <User u'susan',Role id 3L>, <User u'david',Role id 3L>]
>>> str(User.query)
'SELECT users.id AS users_id, users.username AS users_username, users.role_id AS users_role_id \nFROM users'
>>> User.query.limit(1).all()
[<User u'john',Role id 1L>]
>>> str(User.query.limit(1))
'SELECT users.id AS users_id, users.username AS users_username, users.role_id AS users_role_id \nFROM users \n LIMIT %s'
>>> User.query.limit(1).offset(2).all()
[<User u'david',Role id 3L>]
>>> str(User.query.limit(1).offset(2))
'SELECT users.id AS users_id, users.username AS users_username, users.role_id AS users_role_id \nFROM users \n LIMIT %s, %s'
```



*本文示例参考文档：《Flask web开发》*