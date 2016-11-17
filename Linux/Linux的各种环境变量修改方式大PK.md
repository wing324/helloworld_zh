## Linux的各种环境变量修改方式大PK

Linux下修改环境变量有好几种方式，到底应该用哪种方式修改呢？每种方式的区别又是什么呢？   
关于什么是环境变量，请自行google  

参考连接：http://www.powerxing.com/linux-environment-variable/   


查看环境变量
-----------
- 显示当前某个环境变量的指令为：echo $PATH  
  使修改PATH文件立即在当前窗口生效，可以使用source path_file。  

- 显示当前所有的环境变量：env

```shell
[root@localhost ~]# env
XDG_SESSION_ID=8
HOSTNAME=localhost.localdomain
SELINUX_ROLE_REQUESTED=
TERM=vt100
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=192.168.137.1 5538 22
SELINUX_USE_CURRENT_RANGE=
SSH_TTY=/dev/pts/0
USER=root
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:
MAIL=/var/spool/mail/root
PATH=/usr/java/jdk1.7.0_80//bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/python:/root/bin
PWD=/root
JAVA_HOME=/usr/java/jdk1.7.0_80/
LANG=zh_CN.UTF-8
SELINUX_LEVEL_REQUESTED=
HISTCONTROL=ignoredups
SHLVL=1
HOME=/root
LOGNAME=root
SSH_CONNECTION=192.168.137.1 5538 192.168.137.129 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
XDG_RUNTIME_DIR=/run/user/0
_=/usr/bin/env
```


全局环境变量
-----------
全局环境变量，即对每个用户都生效。  



-  /etc/profile文件，当用户登录时，该文件被执行，即生效；

```shell
[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@localhost ~]# vi /etc/profile
# 添加
export PATH=$PATH:/python

[root@localhost ~]# source /etc/profile

[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/python

#此时使用其他用户使用echo $PATH命令，可发现PATH变量中有新添加的/python路径。
```


-  /etc/bashrc文件，当新打开一个终端时，该文件被执行，即生效。

```shell
[root@localhost ~]# echo $PATH
usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@localhost ~]# vi /etc/bashrc
# 添加
export PATH=$PATH:/python

[root@localhost ~]# source /etc/profile

[root@localhost ~]# echo $PATH
usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/python

#此时使用其他用户使用echo $PATH命令，可发现PATH变量中有新添加的/python路径
```


-  etc/environment文件不做解释。



局部环境变量
-----------
局部环境变量，即对单个用户生效。  


-  ~/.bash_profile文件，当用户登录时，该文件仅执行一次，即生效一次；

```shell
[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@localhost ~]# vi ~/.bash_profile
PATH=$PATH:$HOME/bin:  >>  PATH=$PATH:$HOME/bin:/python

[root@localhost ~]# source ~/.bash_profile 

[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/root/bin:/python

#此时使用其他用户使用echo $PATH命令，并未发现PATH变量中有新添加的/python路径。
```


-  ~/.bashrc文件，当用户登录以及每次新打开终端时，该文件被执行，即生效。  

```shell
[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@localhost ~]# vi ~/.bashrc
# 添加如下两行
PATH=$PATH:/python
export PATH

[root@localhost ~]# source ~/.bashrc 

[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/python

#此时使用其他用户使用echo $PATH命令，并未发现PATH变量中有新添加的/python路径。
```


使用export命令设置临时变量
------------------------
直接使用export命令设置临时变量，仅对当前终端有效。

```shell
# 终端1
[root@localhost ~]# echo $PATH
usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/python

[root@localhost ~]# export PATH=usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@localhost ~]# echo $PATH
usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

#终端2
Last login: Thu Sep 24 05:50:31 2015 from 192.168.137.1
[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/python:/root/bin
```