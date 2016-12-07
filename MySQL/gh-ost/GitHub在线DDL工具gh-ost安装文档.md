## GitHub开源MySQL Online DDL工具gh-ost安装文档

查看GitHub开源的MySQL在线DDL工具gh-ost官方文档，以及google一圈都没有发现gh-ost的安装文档，于是，还是自己动手，丰衣足食吧==

Linux系统：Debian8.5

go版本：1.5

gh-ost版本：1.0.28

 注：gh-ost是基于go1.5编译的。

#### go源码安装

```shell
# 安装go依赖包
sudo apt-get install bison ed gawk gcc libc6-dev make

# 配置go环境变量，GOROOT为go源码目录，GOPATH为gh-ost这个工程的目录
vim ~/.bashrc
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=/usr/local/go/src/github.com/github/gh-ost

# 使环境变量生效
source ~/.bashrc

# 获取+解压go源码安装包，go下载地址：https://golang.org/dl/
wget https://storage.googleapis.com/golang/go1.5.linux-amd64.tar.gz
tar -zxvf go1.5.linux-amd64.tar.gz -C /usr/local/
# 此时是go的安装目录为/usr/local/go

# 验证go安装成功
go env
# 结果展示
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/usr/local/gh-ost/go"
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GO15VENDOREXPERIMENT=""
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0"
CXX="g++"
CGO_ENABLED="1"
```

#### gh-ost源码安装

```shell
# 在go安装目录下创建github.com/github/目录
mkdir -p /usr/local/go/src/github.com/github
# 别问我为什么要这样做，我这种go小白，只要安装出来就行，深入原因自己解决吧==

# 获取+解压gh-ost源码安装包，gh-ost下载地址：https://github.com/github/gh-ost/releases/tag/v1.0.28
wget https://codeload.github.com/github/gh-ost/tar.gz/v1.0.28
unzip gh-ost-1.0.28.zip -d /usr/local/go/src/github.com/github
cd /usr/local/go/src/github.com/github
mv gh-ost-1.0.28 gh-ost

# gh-ost源码安装
cd /usr/local/go/src/github.com/github/gh-ost
/bin/bash build.sh
# 结果展示
Building GNU/Linux binary
Binaries found in:
/tmp/gh-ost/gh-ost-binary-linux-20161201195143.tar.gz

tar -zxvf /tmp/gh-ost/gh-ost-binary-linux-20161201195143.tar.gz -C /usr/local
ln -s /usr/local/gh-ost /usr/bin/gh-ost

# 验证gh-ost安装成功
gh-ost -version
1.0.28
gh-ost --help
# 结果会输出一堆参数，gh-ost参数待以后详解
```

