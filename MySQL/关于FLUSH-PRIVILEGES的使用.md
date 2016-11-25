## 关于FLUSH PRIVILEGES的使用

很多人在MySQL里面乱用FLUSH PRIVILEGES，你知道吗？包括从前的我，但是FLUSH PRIVILEGES你真的知道应该在什么场合使用么？  


根据官方文档的如下链接内容可知：  
https://dev.mysql.com/doc/refman/5.6/en/privilege-changes.html   


**当mysqld启动时，所有的权限都会被加载到内存中。**  
如果使用GRANT/REVOKE/SET PASSWORD/RENAME USER命令来更改数据库中的权限表，mysqld服务器将会注意到这些变化并立即加载更新后的权限表至内存中，即权限生效；  
如果使用INSERT/UPDATE/DELETE语句更新权限表(如mysql.user表等)，则内存中的权限表不会感知到数据库中权限的更新，必须重启服务器或者使用FLUSH PRIVILEGES命令使更新的权限表加载到内存中，即权限需在重启服务器或者FLUSH PRIVILEGES之后方可生效。  

**权限生效的含义：**  
1、表级别/列级别的权限，当更新后的权限加载至内存表中，已存在的会话下一次请求时可使用该权限，在修改权限后的建立的会话则立即生效；  
2、数据库级别的权限，当更新后的权限加载至内存表中，已存在会话下一次使用USE db_name后，可使用该权限，在修改权限后的建立的会话则立即生效；  
3、全局权限或者修改密码，当更新后的权限加载至内存表中，需要在下一次登录mysqld后，可使用该权限或密码，对已存在会话不起作用。  

