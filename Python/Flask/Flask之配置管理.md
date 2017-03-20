## Flask之配置管理

本文主要讲解的是app.config的五种加载方式。

1. 设置项较少时，直接硬编码到主体代码中。

   ```python
   app = Flask(__name__)
   app.config['DEBUG'] = True
   ```

2. 通过配置文件加载，传入字符串的模块名字。

   ```python
   # 假设配置文件为settings.py
   app.config.from_object('settings')
   ```

3. 通过配置文件加载，引用后直接传入模块对象。

   ```python
   # 假设配置文件为settings.py
   import settings
   app.config.from_object(settings)
   ```

4. 通过文件名字加载。

   ```python
   # 假设配置文件为settings.py
   app.config.from_pyfile('settings.py',silent=True)
   # 默认当配置文件不存在时会抛出异常，使用silent=True时只会返回False,而不会抛出异常
   ```

5. 通过环境变量加载。

   ```python
   # 假设配置文件为settings.py
   shell> export SETTINGS='settings.py'
   app.config.from_envvar('SETTINGS')
   ```

   ​

参考书籍：《Flask Web开发》 --[董伟明](https://github.com/dongweiming)