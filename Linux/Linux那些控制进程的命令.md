## Linux那些控制进程的命令

操作系统：Debian8.5

**注意：下述中的n，代表的是jobs中的序号**

1. 查看Linux中有哪些在后台运行的进行命令：`jobs`

   ```shell
   root@wing-01:~ # jobs
   [1]+  Stopped                 python backup_mysql.py wing 1.2.3.4 8888 wing
   [2]-  Running                 python backup_mysql.py wing 1.2.3.5 8888 wing &
   root@wing-01:~ #
   ```

2. 让进程后台运行：`cmd &`

   ```shell
   root@wing-01:~ # python backup_mysql.py wing 1.2.3.5 8888 wing &
   [2] 27716
   root@wing-01:~ #
   ```

3. 让后台进程n到前台运行: `fg %n`

   ```shell
   root@wing-01:~ # jobs
   [1]+  Stopped                 python backup_mysql.py wing 1.2.3.4 8888 wing
   [2]-  Running                 python backup_mysql.py wing 1.2.3.5 8888 wing &
   root@wing-01:~ # fg %2
   python backup_mysql.py wing 1.2.3.5 8888 wing
   ```

4. 让前台n到后台运行：`bg %n`

   ```shell
   # 该命令适用于通过ctrl-z暂停的进程
   # 如下面job中的job1是通过ctrl-z暂停的，该程序将不在占用CPU，暂停执行，可以通过top查看其占用CPU率为0%，此时不仅可以通过fg %n让其前台继续运行，也可以通过bg %n让其后台继续运行
   root@wing-01:~ # jobs
   [1]+  Stopped                 python backup_mysql.py wing 1.2.3.4 8888 wing
   [2]-  Running                 python backup_mysql.py wing 1.2.3.5 8888 wing &
   root@wing-01:~ #  bg %1
   [1]+ python backup_mysql.py wing 1.2.3.4 8888 wing &
   root@wing-01:~ #
   # 此时再通过top,发现它开始占用CPU,CPU使用率不在为0，说明其已经在后台运行了。
   ```

5. 暂停当前程序运行：ctrl-z

   此时程序是处于不适用CPU执行任何任务状态，即程序是暂停的状态，等待使用其他命令将其唤醒.

   ```shell
   root@wing-01:~ # python backup_mysql.py yumin 172.16.33.227 3333 yumin platform test
   ^Z
   [1]+  Stopped                 python backup_mysql.py yumin 172.16.33.227 3333 yumin platform test
   root@wing-01:~ #
   ```

6. 通过PID将程序暂停：kill -STOP pid

   ```shell
   root@wing-01:~ # kill -STOP 28021

   [1]+  Stopped                 python backup_mysql.py yumin 172.16.33.227 3333 yumin
   root@wing-01:~ # 
   # 此时可以通过top查看其占用CPU率为0%，即进程已经停止。
   ```

7. 通过PID将程序恢复到后台运行：kill -CONT pid

   ```shell
   root@wing-01:~ # kill -CONT 28021
   root@wing-01:~ #
   # 此时再通过top,发现它开始占用CPU,CPU使用率不在为0，说明其已经在后台运行了。
   ```

   ​



[本文参考文档请点击此处](http://blog.163.com/clevertanglei900@126/blog/static/1113522592011618113059409/)

