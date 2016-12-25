## Linux文本处理工具之cut命令

指定输出一行中的选取部分。

#### 一、cut实战演练

- **cut.txt文件**

  ```
  root,x,0,0,root,/root,/bin/bash
  daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  bin,x,2,2,bin,/bin,/usr/sbin/nologin
  sys,x,3,3,sys,/dev,/usr/sbin/nologin
  ```


- **实战演练**

  > 1. 输出每一行的第二个字节内容
  >
  >    ```shell
  >    root@wing:~/wing # cut -b 2 cut.txt 
  >    o
  >    a
  >    i
  >    y
  >    ```
  >
  > 2. 输出每一行的第二个字符内容
  >
  >    ```shell
  >    root@OPS-yumin-02:~/wing # cut -c 2 cut.txt 
  >    o
  >    a
  >    i
  >    y
  >    ```
  >
  > 3. 输出每一行第一列内容
  >
  >    ```shell
  >    root@OPS-yumin-02:~/wing # cut -d , -f 1 cut.txt 
  >    root
  >    daemon
  >    bin
  >    sys
  >    ```
  >
  > 4. 输出每一行第一、二列内容
  >
  >    ```shell
  >    root@OPS-yumin-02:~/wing # cut -d , -f 1,2 cut.txt 
  >    root,x
  >    daemon,x
  >    bin,x
  >    sys,x
  >    ```

## 二、cut常用参数详解

**-b,--bytes**

> 选取字节的列表，即选取每行的第N个字节。

**-c,--characters**

> 选取字符的列表，即选取每个的第N个字符。(英文字符下与-b没有区别，中文字符下，一个中文占据2-3个字节，所以存在中文的时候更倾向于用-c)。

**-d,--delimiter**

> 分隔符，默认为TAB。

**-f,--field**

> 选取列的列表，即选取每行的第N列。