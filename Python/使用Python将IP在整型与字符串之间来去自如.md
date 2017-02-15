## 使用Python将IP在整型与字符串之间来去自如

对于字符串型的IP存入数据库中，实在是个即浪费空间又浪费性能的家伙，所以可爱的人们想出来将IP转换为整型存储。MySQL中存在INET_ATON()、INET_NTOA()函数进行IP整型和字符串之间的转换，那么Python中存在什么方法可以实现MySQL中INET_ATON()、INET_NTOA()的功能呢？方法肯定是有的～

```python
# 导入相关模块包
import socket
import struct

# 将IP从字符串转为整型
>>> int(socket.inet_aton('127.0.0.1').encode('hex'),16)
2130706433

# 将IP从整型转为字符串
>>> socket.inet_ntoa(struct.pack("!I",2130706433))
'127.0.0.1'
```

