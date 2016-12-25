## Linux文本处理工具之uniq命令

如果你相对文本进行去重操作，那么uniq是你必不可少的工具。uniq常与sort命令一起使用，因为uniq只能对相邻的行做去重操作，如果重复的两行不是相邻的，uniq是没法去重的哦，[请点击此处可以查看sort命令的用法哦](https://github.com/wing324/helloworld_zh/blob/master/Linux/LinuxCMD/Linux%E6%96%87%E6%9C%AC%E5%A4%84%E7%90%86%E5%B7%A5%E5%85%B7%E4%B9%8Bsort%E5%91%BD%E4%BB%A4.md)。

#### 一、实战演练

- **uniq.txt**

  ```
  root,x,0,0,root,/root,/bin/bash
  daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  bin,x,2,2,bin,/bin,/usr/sbin/nologin
  bin,x,2,2,bin,/bin,/usr/sbin/nologin
  root,x,0,0,root,/root,/bin/bash
  sys,x,3,3,sys,/dev,/usr/sbin/nologin
  root,x,0,0,root,/root,/bin/bash
  ```

  ​