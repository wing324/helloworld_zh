## getpass交互式密码模块

如今人们的安全意识真是越来越大了，密码泄露可是个大大的头疼事件，还好，python有个强大的库叫getpass，提供交互式输入密码，完全看不见密码长神马样子~


getpass模块提供了两个函数，分别为getpass.getpass([prompt[,stream]])和getpass.getuser()。  


getpass.getpass([prompt[,stream]])
----------------------------------
提示用户输入密码。  
```
>>> import getpass
>>> pwd1=getpass.getpass()
Password: 
>>> print pwd1
wing1

# 友好交互式输入
>>> pwd2=getpass.getpass('pls input ur password:')
pls input ur password:
>>> print pwd2
wing2
```


getpass.getuser()
------------------
获得登陆的用户名  
```
>>> user=getpass.getuser()
>>> print user
root
```