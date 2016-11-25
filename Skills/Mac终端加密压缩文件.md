## Mac终端加密压缩文件

今天发现了在Mac如何给压缩文件加密，于是记录下来，留待后续需要的时候使用。



#### 命令

##### 加密文件

```zip -e test.zip test1.txt test2.txt test3.txt```

加密目录

```zip -e -r test.zip test/*```

#### 解释

- -e 为加密参数，添加该参数后，会有输入密码的提示。
- -r 为压缩地柜目录，不在该参数则不会压缩test目录的子目录内的内容。
- test/\*为压缩test目录下的所有文件+目录。
- test.zip为压缩后的文件名称。
- test1.txt test2.txt test3.txt为需要添加的压缩文件，即把这几个文件压缩起来。

