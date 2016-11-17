## Linux如何将eth1改为eth0

在工作中，新开启一个Linux服务器，发现我居然找不到eth0,使用ifconfig居然没有发现我认识的eth0，然后通过查询资料，找到原因及解决方法，故记录之。  




现象
----
ifconfig时，并未发现和ethN相关的信息，ifconfig -a时，发现eth1,并未发现eth0。


原因
----
很多Linux distribution使用udev动态管理设备文件，并根据设备的信息对其进行持久化命名。udev会在系统引导的过程中识别网卡，将mac地址和网卡名称对应起来记录在udev的规则脚本中。而对于新的虚拟机，VMware会自动为虚拟机的网卡生成MAC地址，当你克隆或者重装虚拟机软件时，由于你使用的是以前系统虚拟硬盘的信息，而该系统中已经有eth0的信息，对于这个新的网卡，udev会自动将其命名为eth1（累加的原则），所以在你的系统启动后，你使用ifconfig看到的网卡名为eth1。
（摘自参考链接）


解决方案
-------
```
vi /etc/udev/rules.d/70-persistent-net.rules

注释掉其中的eth0行，将原来的eth1名称修改为eth0
```


参考链接
-------
http://www.cnblogs.com/swblog/p/3251935.html 








