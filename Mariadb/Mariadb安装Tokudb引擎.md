## Mariadb安装Tokudb引擎

zabbix数据库数据量之大真的让人非常非常的困扰，但是好在这个世界上有一种存储引擎叫做tokudb，这个tokudb压缩性能真的是好的不要不要的，也是我第一次接触到除InnoDB和MyISAM之外的第三种存储引擎：tokudb。


以二进制安装mariadb为例（注意mariadb 有两种二进制安装包，安装tokudb必须是glibc2.14版本以上，比如mariadb二进制安装包长得像这样：mariadb-10.1.18-linux-glibc_214-x86_64.tar.gz）

操作系统：Debian 8.5

Mariadb：10.1.18（二进制版本）

1. 如何确定操作系统glibc的版本为2.14以上

   ```shell
   dd --version
   ```

2. 二进制安装Mariadb
   参考官方文档：https://mariadb.com/kb/en/mariadb/installing-mariadb-binary-tarballs/

3. 安装tokudb

   ```sql
   # 安装依赖包
   apt-get install -y libjemalloc-dev
   #关闭操作系统HugePage
   cat /sys/kernel/mm/transparent_hugepage/enabled
   -- 显示always madvise [never]，则HugePage已经关闭

   # 否则关闭HugePage的方法
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
   echo never > /sys/kernel/mm/transparent_hugepage/defrag

   # 启动MySQL后，在MySQL
   ```


   # 卸载tokudb
   UNINSTALL SONAME 'ha_tokudb';
   ```

4. 配置tokudb参数

   ```shell 
   # 注意Tokudb的参数在mysql_install_db阶段，不能配置到my.cnf文件中，否则在数据库初始化阶段会报错
   # 在INSTALL SONAME 'ha_tokudb';之后，关闭数据库，在my.cnf文件中写上需要配置的参数，例如：
   tokudb-data-dir = /data/mysql/3306/tokudb_data
   tokudb-log-dir = /data/mysql/3306/tokudb_log
   tokudb_row_format = tokudb_fast
   tokudb_cache_size = 64G
   tokudb_commit_sync = 0
   tokudb_fsync_log_period = 5000
   tokudb_directio = 1
   tokudb_read_block_size = 128K
   tokudb_read_buf_size = 128K

   # 然后将INSTALL SONAME 'ha_tokudb';生成的部分文件拷贝到对应的tokudb目录中去
   # tokudb-data-dir中应有的文件：
   __tokudb_lock_dont_delete_me_data
   __tokudb_lock_dont_delete_me_temp
   # tokudb-log-dir应有的文件:
   log000000000001.tokulog29
   __tokudb_lock_dont_delete_me_logs
   __tokudb_lock_dont_delete_me_recovery

   # 此时重新启动MySQL，则相应的配置参数即可生效。
   ```

