## 如何更新CentOS6.5的yum源

系统自带的Yum源太老了，很多东西都不是最新的。所以才会有更新yum源的需求。 



版本：CentOS6.5  
163yum源：  
http://mirrors.163.com/.help/CentOS6-Base-163.repo  


步骤
------

- 进入yum源配置目录  

```shell
cd /etc/yum.repos.d/
```



-  备份系统自带yum源  

```shell
mv CentOS-Base.repo CentOS-Base.repo.bk
```



-  下载163yum源  
```shell
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
```



-  然后标准化  
```shell
mv CentOS-Base-163.repo CentOS-Base.repo
```



-  然后执行如下命令，生效操作  

```shell
yum clean all （清除所有缓存）
yum makecache （加载元数据缓存）
```



-  更新系统到最新  

```shell
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-*
yum upgrade
```
