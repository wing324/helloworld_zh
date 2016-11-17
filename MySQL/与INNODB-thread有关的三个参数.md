## 与INNODB thread有关的三个参数

对于INNODB并发线程数的见解。。   



三个参数
-------
##### innodb_thread_concurrency  
INNODB存储引擎中允许的最大的线程并发数。  
##### innodb_thread_sleep_delay  
单位为毫秒；thread未能进入INNODB存储引擎后，需要等待innodb_thread_sleep_delay毫秒再次尝试进入。  
##### innodb_concurrency_tickets  
thread进入INNODB中，会获得innodb_concurrency_tickets次数通行证，该线程在接下来的innodb_concurrency_tickets次进入到INNODB中不需要再进行检查，可直接进入。  


三个参数关系
-----------
当一个thread需要进入到INNODB存储引擎层（以下简称INNODB），INNODB会检查已经进入到INNODB存储引擎层的thread总数是否超过**innodb_thread_concurrency**，如果超过了，那么该thread需要等待**innodb_thread_sleep_delay**（单位：毫秒）毫秒再次进行尝试，如果这次尝试失败后，该thread将会进入到FIFO的队列中进行等待唤醒（此时状态为sleeping）。一旦该thread进入到INNODB中，该thread将会获得**innodb_concurrency_tickets**的通行证，即该thread将会在接下来的innodb_concurrency_tickets次进入到INNODB中都不需要再进行检查，可直接进入。  

**注意**
thread尝试两次进入INNODB存储引擎层的目的是，减少等待线程的数量以及减少上下文切换。  
