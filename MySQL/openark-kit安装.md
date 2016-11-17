## openark-kit安装

作为MySQL另一个工具集openark-kit，内部包含很多小工具，会在将来一一介绍~


官方网站 http://code.openark.org/forge/openark-kit  

本文以CentOS为操作系统，且默认操作系统中已经安装Python环境。  

1. 安装Python模块包之MySQL-python,用于使用Python连接操作MySQL使用。  
   yum install -y MySQL-python  

2. 安装openark-kit工具包  
   1) RPM安装方式  
   获得RPM包 https://code.google.com/p/openarkkit/downloads/detail?name=openark-kit-196-1.noarch.rpm  
   执行命令 rpm -ivh openark-kit-196-1.noarch.rpm  
   2) tar包安装方式
   获取tar包 https://code.google.com/p/openarkkit/downloads/detail?name=openark-kit-196.tar.gz  
   解压tar包 tar -zxvf openark-kit-196.tar.gz -C */usr/local/openark-kit/*  
   安装openark-kit工具 python setup.py install  

**特别说明**  
openark-kit工具的前缀为oak-,相关工具有如下，详细介绍后续将会使用并记录。  
oak-apply-ri  
oak-get-slave-lag  
oak-modify-charset  
oak-purge-master-logs  
oak-show-limits
oak-block-account  
oak-hook-general-log  
oak-online-alter-table  
oak-repeat-query  
oak-show-replication-status
oak-chunk-update  
oak-kill-slow-queries  
oak-prepare-shutdown  
oak-security-audit