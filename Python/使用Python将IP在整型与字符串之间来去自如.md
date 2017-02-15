## 使用Python将IP在整型与字符串之间来去自如

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

