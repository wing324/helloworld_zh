## SSH免密码登录操作步骤

每次SSH免密码登录都要去Google上搜索，其实也就是那么几条命令嘛。所以我决定最后一次Google怎么SSH免密码登录，然后记录下来。

原文链接：

[http://blog.csdn.net/leexide/article/details/17252369](很详细的SSH免密码登录教程和原理)


### 准备机器

server1：192.168.0.101

server2：192.168.0.102

server3：291.168.0.103

### SSH目标

server1可以通过SSH免密码登录的方式登录到server2和server3上。

### SSH步骤

- server1上使用ssh-keygen创建公钥

  ```shell
  ssh-keygen -t rsa -N "xxxx"
  # 而后按下一系列的Enter键即可,-N为登陆时候输入一个ssh加密，等于给ssh-key私钥加密
  ```

  ​说明：

  ​	-t 指定算法

- 查看server1钥匙

  ```shell
  ls -l ~/.ssh  
  # 存在两个文件id_rsa(私钥)、id_rsa.pub(公钥)
  ```


- 将server1的公钥(id_rsa.pub)的内容复制粘贴到server2和server3的~/.ssh/	authorized_keys中

  ```shell
  vim ~/.ssh/	authorized_keys
  # 每个公钥占用一行

  # 也可以使用ssh-copy-id
  # server1上
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.102
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.103
  ```


- 设置server2和server3的相关权限

  ```shell
  chmod 600 authorized_keys
  chmod 700 -R .ssh

  # 如果使用ssh-copy-id工具，此处权限设置已经由ssh-copy-id自动完成，无须再手动操作
  ```


- 此时可以实现server1通过SSH免密码登录server2和server3了

  ```
  [root@192.168.0.101 .ssh]# ssh 192.168.0.102
  [root@192.168.0.101 .ssh]# ssh 192.168.0.103
  ```

