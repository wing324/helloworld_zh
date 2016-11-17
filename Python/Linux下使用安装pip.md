## Linux下使用安装pip

Linux下安装pip，python2.7.9之前的版本，python是没有内置pip的，所以需要我们自己安装哦~  


1、首先得有安装工具此处推荐使用pip，python2.7.9版本之后已经内置该模块管理工具。安装pip模块管理工具之前得有setuptools
Setuptools最新版本：https://pypi.python.org/pypi/setuptools
Pip最新版本：https://pypi.python.org/pypi/pip
得到Setuptools和Pip最新版本，并在Linux下解压。

2、安装setuptools:
编译：python setup.py build
安装：python setup.py install

3、安装pip:
编译：python setup.py build
安装：python setup.py install

4、利用pip安装python第三方模块包：
pip install 模块包

```shell
# pip安装模块包实例
pip install prettytable
```