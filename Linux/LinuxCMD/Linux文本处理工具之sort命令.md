## Linux文本处理工具之sort命令

sort命令将每一行作为一个单位进行比较，比较原则是从首字符向后，依次按ASCII码值进行比较，最后将他们按一定的顺序进行输出。

#### 一、sort实战演练

- **sort.txt**

  ```
  root,x,0,0,root,/root,/bin/bash
  daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  bin,x,2,2,bin,/bin,/usr/sbin/nologin
  bin,x,2,2,bin,/bin,/usr/sbin/nologin
  sys,x,3,3,sys,/dev,/usr/sbin/nologin
  ```

- **number.txt**

  ```
   9
  8
  90
  82
  2
  ```

- **human_readable.txt**

  ```
  2k
  2G
  2M
  20M
  ```

- **实战演练**

  > 1. 输出升序排序文件
  >
  >    ```shell
  >    root@wing:~/wing # sort sort.txt 
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  >    root,x,0,0,root,/root,/bin/bash
  >    sys,x,3,3,sys,/dev,/usr/sbin/nologin
  >    ```
  >
  > 2. 输出降序排序文件
  >
  >    ```shell
  >    root@wing:~/wing # sort -r sort.txt 
  >    sys,x,3,3,sys,/dev,/usr/sbin/nologin
  >    root,x,0,0,root,/root,/bin/bash
  >    daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    ```
  >
  > 3. 将sort文件按照第三列升序输出
  >
  >    ```
  >    root@wing:~/wing # sort -t , -k 3 sort.txt 
  >    root,x,0,0,root,/root,/bin/bash
  >    daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    sys,x,3,3,sys,/dev,/usr/sbin/nologin
  >    ```
  >
  > 4. 去掉重复行降序输出
  >
  >    ```shell
  >    root@wing:~/wing # sort -ur sort.txt 
  >    sys,x,3,3,sys,/dev,/usr/sbin/nologin
  >    root,x,0,0,root,/root,/bin/bash
  >    daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
  >    bin,x,2,2,bin,/bin,/usr/sbin/nologin
  >    ```



二、sort常用参数

**-b, --ignore-leading-blanks**

> 忽略每一行开头的空格，从第一个不是空格的字符开始比较。

**-c, --check, --check=diagnose-first**

>检查文件是否已经排序，如果没有排序，则输出第一个未排序的行，如果已经排序，则返回1.
>
>```shell
>root@wing:~/wing # sort -c sort.txt 
>sort: sort.txt:2: disorder: daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
>```

**-C, --check=quiet, --check=silent **

> 与-c参数功能一致，不同的是，如果没有排序，该参数并不会输出第一个未排序的行，二是什么都不会输出。
>
> ```
> root@wing:~/wing # sort -c sort.txt 
> sort: sort.txt:2: disorder: daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
> root@wing:~/wing # 
> ```

**-f, --ignore-case**

> 忽略大小写，将所有的小写字母转换为大写字母进行比较。

**-h, --human-numeric-sort**

> 以人类可读的方式排序，如对k,G进行排序。
>
> ```shell
> root@wing:~/wing # sort human_readable.txt 
> 20M
> 2G
> 2k
> 2M
> root@wing:~/wing # sort -h human_readable.txt 
> 2k
> 2M
> 20M
> 2G
> ```

**-M, --month-sort**

> 按月份排序，如JAN,MAR等等。

**-n, --numeric-sort**

> 将数字转换为数值的方式排序。
>
> ```shell
> root@wing:~/wing # sort number.txt 
> 2
> 8
> 82
> 9
> 90
> root@wing:~/wing # sort -n number.txt 
> 2
> 8
> 9
> 82
> 90
> ```

**-R, --random-sort**

> 以随机的方式进行排序。
>
> ```shell
> root@wing:~/wing # sort -R sort.txt 
> daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
> sys,x,3,3,sys,/dev,/usr/sbin/nologin
> root,x,0,0,root,/root,/bin/bash
> bin,x,2,2,bin,/bin,/usr/sbin/nologin
> bin,x,2,2,bin,/bin,/usr/sbin/nologin
>
> root@wing:~/wing # sort -R sort.txt 
> root,x,0,0,root,/root,/bin/bash
> sys,x,3,3,sys,/dev,/usr/sbin/nologin
> daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
> bin,x,2,2,bin,/bin,/usr/sbin/nologin
> bin,x,2,2,bin,/bin,/usr/sbin/nologin
>
> root@wing:~/wing # sort -R sort.txt 
> root,x,0,0,root,/root,/bin/bash
> bin,x,2,2,bin,/bin,/usr/sbin/nologin
> bin,x,2,2,bin,/bin,/usr/sbin/nologin
> daemon,x,1,1,daemon,/usr/sbin,/usr/sbin/nologin
> sys,x,3,3,sys,/dev,/usr/sbin/nologin
> ```

**-r, --reverse**

>默认是升序排序，加上-r参数是降序排序。

**-o, --output=FILE**

> 将sort命令的结果输出到另一个文件中。

**-u, --unique**

> 和-c参数一起时，并没有什么用;
>
> 不与-c参数一起时，将所有的行去重后排序输出。

**-t, --field-separator**

> 分隔符。

**-k, --key**

> 根据key去排序，可以是列的位置或者类型。