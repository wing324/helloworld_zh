## glob路径下查找文件

Python中通过输入路径和通配符查找到相关的文件或目录的库。



通配符
-----
\* ： 匹配无限个字符  
? ： 匹配单个字符  
[] ： 匹配指定范围内的数字  


glob.glob
---------
返回的是一个列表，所有在路径下匹配的文件或目录。  

```python
# 返回/root目录下所有以MySQL-开头，以.rpm结尾的文件或目录
>>> import glob
>>> glob.glob('/root/MySQL-*.rpm')
['/root/MySQL-client-5.6.26-1.el6.x86_64.rpm', '/root/MySQL-devel-5.6.26-1.el6.x86_64.rpm', '/root/MySQL-server-5.6.26-1.el6.x86_64.rpm', '/root/MySQL-shared-5.6.26-1.el6.x86_64.rpm', '/root/MySQL-shared-compat-5.6.26-1.el6.x86_64.rpm']

# 验证glob.glob返回的是列表list
>>> type(glob.glob('/root/MySQL-*.rpm'))
<type 'list'>
```


glob.iglob
----------
功能与glob.glob相似，但glob.glob输出为列表，而glob.iglob为各个输出。  

```python
>>> import glob
>>> f = glob.iglob('/root/MySQL-*.rpm')
>>> f
<generator object iglob at 0x7f7206ab63c0>
>>> for i in f:
...     print i
... 
/root/MySQL-client-5.6.26-1.el6.x86_64.rpm
/root/MySQL-devel-5.6.26-1.el6.x86_64.rpm
/root/MySQL-server-5.6.26-1.el6.x86_64.rpm
/root/MySQL-shared-5.6.26-1.el6.x86_64.rpm
/root/MySQL-shared-compat-5.6.26-1.el6.x86_64.rpm
```

