## zabbix使用python脚本发送报警邮件

此处是用外部脚本python实现zabbix的报警机制的。对于zabbix3.0此处存在一个小改动，让本宝宝忙活了一天才找到原因哒。。。。


1、编辑zabbix_server.conf文件，修改AlertScriptsPath参数,该参数用于指定外部脚本的绝对路径。  
```shell
vim /etc/zabbix/zabbix_server.conf
AlertScriptsPath=/usr/lib/zabbix/alertscripts
```

2、上传新增py脚本至AlertScriptsPath参数指定的绝对路径下,py文件如下：  
```python
#! /usr/bin/env python
# coding:utf-8

'''
[INFORMATION]
Zabbix Send Email With Python
AUTHOR : Wing
GitHub : https://github.com/wing324
Email : wing324@126.com
'''

from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.utils import parseaddr, formataddr
import smtplib
import sys

def send_mail(_to_email,_subject,_message):
# 定义邮件发送
	smtp_host = 'smtp.xxx.xx'
	from_email = 'xxx@xxx.xx'
	passwd = 'xxxxxx'
	msg = MIMEText(_message,'plain','utf-8')
	msg['Subject'] = _subject
	smtp_server = smtplib.SMTP(smtp_host,25)
	smtp_server.login(from_email,passwd)
	smtp_server.sendmail(from_email,[_to_email],msg.as_string())
	smtp_server.quit()

if __name__ == '__main__':
	send_mail(sys.argv[1],sys.argv[2],sys.argv[3])
```

3、修改python脚本的权限  
```shell
chown -R zabbix:zabbix zabbix_send_email.py
chmod 755 zabbix_send_email.py
```

4、zabbix web端配置  
- Administration --> Media types --> Create media type  
  ![](http://i.imgur.com/CEG66ZN.png)
- 创建一个测试用户Administration --> Users --> Create user  
  ![](http://i.imgur.com/3EUzsyJ.png)
- 为新创建的user指定media:Administration --> Users --> Create user --> Media  
  ![](http://i.imgur.com/IbFuab6.png)
- 创建action实现邮件报警Configuration --> Actions --> Create action  
  ![](http://i.imgur.com/c67Jb5e.png)
  ![](http://i.imgur.com/JI2HXGJ.png)
  ![](http://i.imgur.com/yFT9alq.png)

5、zabbix测试发送邮件
找一个test的zabbix_agentd，kill掉，查看是否收到报警邮件。再将其恢复，查看是否收到恢复后的邮件。如果一切如预期所想，那么至此就完成了使用python脚本完成zabbix的报警邮件了。如果没有如预期所想。

**TIPS：**  
如果你用的zabbix3.0，请注意Administration --> Media types --> Create media type这一步的配置如下：  
![](http://i.imgur.com/0vqpbhN.png)
（今天被这个坑了一天。。。。。。）

