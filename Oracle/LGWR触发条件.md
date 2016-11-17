LGWR触发条件

Oracle中LGWR进程和MySQL的redo log刷新方式很相似。所以记录下来。  
 

1. 用户commit；  
2. 有1/3的redolog buffer未被写入磁盘  
3. 有大于1M的redolog buffer未被写入磁盘  
4. 每隔3s触发一次LGWR  
5. DBWR需要写入的数据的SCN大于LGWR记录的SCN,DBWR触发LGWR写入


LGWR特点
--------
1. 写频繁  
2. 一次写的量较小  
3. 顺序写  

redo log需要放在写性能比较好的磁盘上，如固态盘、raid10、raid01（raid5/6写性能较差）