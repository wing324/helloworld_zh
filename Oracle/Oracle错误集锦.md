## Oracle错误集锦

收集Oracle的错误并记录，作为未来供自己参考。

Oracle-4031
-----------
##### 原因分析
大量的硬解析出现，产生大量的free chunk，然后突然出现大的SQL语句，找不到合适的free chunk。  
##### 解决方案
1. 将shared pool中的library cache和row cache全部清空(该方案治标不治本)  
```sql
alter system flush shared_pool;
```
2. 共享SQL以减少硬解析  
   开发时需要统一书写风格（要知道两个SQL语句多一个空格都是不一样的），其次使用绑定变量的方式实现SQL共享。  
   如果无法再程序开发层面修改，可以修改如下参数达到共享SQL的功能：  
```
# 该参数可以解决绑定变量的问题，但不能用于解决书写风格不统一的问题
alter system set cursor_sharing='force';
```
3. 保留区（该区域只用来缓存大对象，即较大的SQL语句）  
```sql
# 执行如下语句，查看在保留区中是否有存在请求chunk失败的SQL语句
select REQUEST_MISSES from v$shared_pool_reserved;
# 如果上述语句结果大于0，则需要调大shared_pool_reserved_size的大小
alter system shared_pool_reserved_size=？
```
4. 增加sharde pool的空间  
```sql
select COMMPONENT,CURRENT_SIZE from V$SGA_DYNAMIC_COMPONENTS;
show parameter sga_target;
show parameter sga_max_size;
alter system set shared_pool_size=150M scope=both;
```