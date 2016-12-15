## python的requirements.txt文件生成与安装

总是别人项目中看到requirements.txt文件，打开一看，全是项目依赖包的名字，后来通过google发现这东西居然可以安装依赖包。于是便有了这篇文章。

#### 生成requirements.txt

`pip freeze > requirements.txt`

#### 安装requirements.txt

`pip install -r requirements.txt`