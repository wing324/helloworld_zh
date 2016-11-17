## SQL语句在Oracle数据库中的漫游之路

本文档为甲骨论视频笔记。  
讲述SQL语句在Oracle下的执行过程，由于初学Oracle,存在不完善的地方，以后会日渐完善。  


1. SQL语句通过客户端与Oracle之间的连接（即网络），来到Oracle实例（后称oracle）面前，被Oracle的server process（server对process的进程）接收；  
2. server process拿着sql语句会首先进入shared pool中，检查该SQL语句有没有在shared pool中缓存，如果存在shared pool中，那么该sql语句将跳过解析过程，如果没有，则该sql语句将会被解析；  
3. Oracle需要将SQL语句解析成执行计划才能被Oracle执行；  
4. 解析过程：判断sql语法是否存在问题，判断用户是否有权限执行该SQL语句，判断sql语句涉及的表列等元数据信息是否存在，判断sql语句使用哪种执行方案等；（sql执行计划可以缓存到shared pool中，shared pool用来缓存sql的执行计划使用的）  
5. 解析完之后，server process首先到buffer cache中查找是否有该SQL语句存在的数据，如果存在，则通过网络返回给client，如果没有，则继续到dbf文件中获取数据至buffer pool中；（buffer cache用来缓存dbf的数据使用的）  
6. 如果需要对表进行修改，那么server process在buffer pool中修改表的数据，然后server process产生修改日志，这些修改日志会写到redolog buffer中，DBWR进程将server process修改后的数据写会磁盘上的dbf文件中，LGWR将redolog buffer中的redolog写到磁盘上的redolog文件中；（server process只负责读不负责写，由后台进程进行写。）

**图解SQL语句在Oracle数据库中的漫游之路**  
![](http://i.imgur.com/OMN1E5k.png)
