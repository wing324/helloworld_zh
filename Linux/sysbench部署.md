## sysbench部署

操作系统：Debian8

sysbench版本：1.0

>- [获取sysbench源码](https://github.com/akopytov/sysbench)
>
>- 将sysbench源码放置在/usr/local目录下
>
>  ```shell
>  mv sysbench /usr/local
>  ```
>
>- 安装依赖包
>
>  ```shell
>  apt-get install m4  autoconf  automake libtool
>  ```
>
>-  切换至sysbench安装目录，运行autogen.sh脚本
>
>  ```shell
>  cd /usr/local/sysbench
>  ./autogen.sh
>  ```
>
>- 运行configure
>
>  ```shell
>  # /usr/local/mysql为MySQL安装目录
>  ./configure --with-mysql-includes=/usr/local/mysql/include --with-mysql-libs=/usr/local/mysql/lib
>  ```
>
>- 运行make
>
>  ```shell
>  make
>  ```
>
>- 软链sysbench
>
>  ```shell
>  ln -s /usr/local/sysbench/sysbench/sysbench /usr/bin/sysbench
>  ```

至此，sysbench安装完成啦～