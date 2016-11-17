## Linux下替换Python新版本

目前Centos6.5的Python版本为2.6.6，可是python3和python2的区别真的是很大呢，现在想入手python3，当然要将python2.6.6替换成python3啦。



1.安装相关包：yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make  

2.下载python源码包，在官网上。  

3.安装Python3.4：  
tar -zxvf Python-3.4.0.tgz -C /usr/local/src/  
cd /usr/local/src/Python-3.4.0/  
./configure --prefix=/usr/local/python3.4  
make -j4 && make install  

4.添加python3.4命令到环境变量中：  
vi ~/.bash_profile  
PATH=$PATH:$HOME/bin   
改为： PATH=$PATH:$HOME/bin:/usr/local/python3.4/bin  
使环境变量生效：. ~/.bash_profile  

5.此时可以通过python3.4命令调用python，但很多时候我只希望用到python就可以调用python3.4，但centos6.5默认的python版本是2.6.6。所以我们可以进行接下来的步骤。  

6.先对系统默认版本进行操作：  
mv /usr/bin/python /usr/bin/python2.6  
ln -s /usr/local/python3.4/bin/python3.4 /usr/bin/python  

7.此时在使用Python命令调用的就是Python3.4的命令。  