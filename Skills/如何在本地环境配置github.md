## 如何在本地环境配置github

#### 一、安装git

Windows/Mac/Linux安装方式各不相同，但是google上有一大把的资料可以查看。故此处不详细给出方案。

#### 二、配置账号信息

```shell
git config --global user.name xxx
git config --global user.emai xxxxxx
```

请将上述的xxx和xxxxxx分别换成github用户名以及邮箱。

#### 三、生成github公钥

```shell
ssh-keygen -t rsa -C "xxxxxx"
```

同样，xxxxxx换成github邮箱。

然后复制上述命令生成的一个文件id_rsa.pub文件内容。

打开github网页，点击用户名-->"setting"-->"SSH and GPG keys"-->"NEW ssh key"，将上述复制的id_rsa.pub文件粘贴到出现的文本框中。

#### 四、验证是否配置成功

```shell
ssh -T git@github.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```

xxx为github用户名，出现上述提示，则表示本地环境配置github成功。现在就可以在本地电脑操作自己的github仓库了。

