## 基于CentOS6.5安装Inception

Inception是去哪儿网开源的一款SQL审核工具，功能丰富，便于使用。  
Inception官方文档：  
http://mysql-inception.github.io/inception-document/    
由于官方文档中没有对该工具的安装写详细教程，故自己写一个便于不了解编译安装的人参考。    



inception编译安装
------------------------

```
1. 至github中获取inception源码  
github地址：https://github.com/mysql-inception/inception  
最简便的方法是点击该页面的Dowload ZIP,然后在CentOS安装unzip工具，以下安装过程均基于获得源码的zip压缩包。
  
2. 安装unzip工具  

yum install -y unzip

3. 解压inception源码包  

unzip inception-master.zip -d /

4. 安装相关依赖包和编译工具  

yum install -y openssl-devel
yum install -y gcc-c++
yum install -y ncurses-devel
yum install -y bison
yum install -y cmake

5. 编译安装  

sh inception_build.sh inception [linux]
#上述命令中inception为编译安装的目录，[linux]为当前操作系统平台

6. 此时等待一大堆密密麻麻的英文数字跑完以后，亲测是完成安装了。  
关于运行和使用，请移步inception官方文档。  
```

