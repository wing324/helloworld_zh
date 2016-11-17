解决UnicodeEncodeError: 'ascii' codec can't encode characters in position问题
在文件中读取select \* form *table_name*语句中的'*',总是读取不出来，报出error为：UnicodeEncodeError: 'ascii' codec can't encode characters in position，然后通过google,找到了解决方法啦~  


解决方法
-------
在程序的开头添加如下代码：
```python
import sys
reload(sys)
sys.setdefaultencoding( "utf-8" )
```