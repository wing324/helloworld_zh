## Liunx系统命令之tar命令

我经常混淆tar命令的各种选项，为了不用每次使用tar命令的时候，去google搜索tar的选项详解，主要还是FQ，所以我还是默默的把tar选项整理一遍，便于日后自己查找方便吧~  



tar选项详解
----------------
-c 建立压缩文件  
-x 解压一个压缩文件  
-t 查看tarfile里面的文件  
-z 是否具有gzip属性？或者是否要用gzip属性压缩  
-j 是否具有bzip2属性？或者是否需要用bzip2属性压缩  
-v 压缩的过程中显示文件夹  
-f 使用f后紧跟的文件名  
-p 使用原文件属性，属性不会根据使用者而改变  
-P 使用绝对路径来压缩  


实战演练
-------------
实战演练一：压缩  
tar -cf new_file.tar old_file         仅打包文件不压缩  
tar -zcvf new_file.tar.gz old_file     打包文件且以gzip属性压缩  
tar -jcvf new_file.tar.bz2 old_file    打包文件且以bzip2属性压缩  

实战演练二：查看tarfile中包含哪些文件  
tar -ztvf file.tar.gz    查看gzip属性的压缩文件  

实战演练三：解压缩文件  
tar -zxvf new_file old_file.tar.gz    解压缩gzip属性的文件  
tar -jxvf new_file old_file.tar.bz2   解压缩bzip2属性的文件  
