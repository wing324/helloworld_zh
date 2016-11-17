## 关于MySQL异常捕捉那些事儿

本系列文章主要介绍如何捕捉处理MySQL异常以及实战演练展示，参考于MySQL5.6官方文档(http://dev.mysql.com/doc/refman/5.6/en/index.html) 和 ZHDBA官网之MySQL数据库的例外处理测试(http://www.zhdba.com/mysqlops/2013/08/31/mysql-handler-2/)。



#### 异常捕捉原因
存储程序执行过程中对于一些异常需要特殊处理，如退出当前执行程序或者继续执行等，而这些异常需要先捕捉而后处理。  


#### 异常捕捉准备知识之错误信息的认识
1. 当客户端线程向MySQL服务器提交命令后，执行是否成功，服务器端都会返回其相关判断性数据。若是执行过程中出错了，MySQL会返回两类信息：  
   A. MySQL特殊的错误编码（如1146），是一个整数类型的代码，这些错误编码并不适用于其他的数据库产品；  
   B. MySQL的SQL状态值，由5个字符组成（如42S02），该值支持ANSI SQL、ODBC或其他标准协议，并不是所有的MySQL返回的错误信息都有SQL状态值，对于没有SQL状态值使用‘HY000’代替。  

2. 对于执行过程中的错误返回的错误编码、SQL状态值以及更详细的错误描述信息，MySQL提供C API函数，如下：  
   A. 返回MySQL错误编码的API函数：mysql_errno()；  
   B. 返回MySQL SQL状态值的API函数：mysql_sqlstate()；  
   C. 返回MySQL错误描述信息的API函数：mysql_error()。  

3. 对于预处理语句，相应的错误函数有mysql_stmt_errno()、mysql_stmt_sqlstate()、mysql_stmt_error()。  

4. Errors/Warnings/Notes的数量可以通过调用mysql_warning_count()函数获得。  

5. 对于MySQL执行过程中的错误返回的SQL状态值前两个字符的含义如下：  
   A. ‘00’代表MySQL执行成功；  
   B. ‘01’代表MySQL执行成功，但是存在警告信息；  
   C. ‘02’代表MySQL没有获得结果，游标移动到记录结果集的最后或者SELECT ... INTO var_list没有取到数据，就会触发此类错误信息；  
   D. 前两个字符大于‘02’的SQL状态值统称为异常，使用EXCEPTION标识。  

6. MySQL的错误编码、SQL状态值及信息  
   http://dev.mysql.com/doc/refman/5.6/en/error-messages-server.html  


#### 定义异常
1. 语法  
   ![](/img/MySQL_exception1.png)  

2. 特别说明  
   A. 异常的定义必须出现在游标或异常处理定义之前；
   B. 不必要处理mysql_error_code为0或者sqlstate_value以‘00’开头的情况，因为它们标识着MySQL执行成功。


#### 定义异常处理
1. 语法  
   ![](/img/MySQL_exception2.png)  

2. 特别说明  
   A. statement可以是简单的SQL语句如SET val_name=value，也可以由BEGIN 和END组成的复合语句；  
   B. 异常处理的定义必须出现在变量或异常定义之后；  
   C. handler_action表明触发异常后如何处理：  
   CONTINUE:触发异常后，待执行完处理异常部分的语句体之后，回到触发异常的下一个语句位置，继续向下执行；  
   EXIT:触发异常后，待执行完处理异常部分的语句体之后，退出当前程序；  
   UNDO:不支持，创建时会出现语法错误。  
   D. condition_value表示需要处理的异常，包括如下异常值：  
   MySQL错误编码或SQL状态值  
   DECLARE ... CONDITION定义的异常名称  
   SQLWARNING，表示以‘01’开头的SQL状态值  
   NOT FOUND,表示以‘02’开头的SQL状态值  
   SQLEXCEPTION，表示以大于‘02’开头的SQL状态值  

3. 对于没有定义异常处理的异常，MySQL采取的措施如下：  
   A. 对于SQLEXCEPTION的异常，则程序在触发异常的位置终止；  
   B. 对于SQLWARNING的异常，则程序在触发异常之后继续执行；  
   C. 对于NOT FOUND的异常，正常情况下程序在触发异常之后继续执行，当使用SIGNAL/RESIGNAL处理之后，程序则在触发异常的位置终止。  


#### 伪装错误信息之SIGNAL
1. 语法  
   ![](/img/MySQL_EXCEPTION3.png)  

2. 特别说明  
   A. SIGNAL语句的执行不需要任何特殊权限；  
   B. condition_value可以是SQL状态值或DECLARE ... CONDITION定义的异常名称，注意此处不可使用错误编号，否则会导致ERROR 1646 (HY000): SIGNAL/RESIGNAL can only use a CONDITION defined with SQLSTATE；  

3. SIGNAL语句实战演练  
   A. 存储过程示例  
   ![](/img/MySQL_exception4.png)  
   B. 执行结果  
   ![](/img/MySQL_exception5.png)  
   C. 结果分析  
   当divisor=1时，根据其执行结果红框中的SQLSTATE值可知触发的是第一个my_error异常，是由最后一条SIGNAL my_error;语句触发的；  
   当divisor=0时，根据其执行结果红框中的SQLSTATE值可知触发的是第二个my_error异常，是由第一个SIGNAL my_error;语句触发的；  


#### 伪装错误信息之RESIGNAL
1. 语法  
   ![](/img/MySQL_exception6.png)  

2. 特别说明  
   A. RESINGAL语句和SIGNAL语句作用相似；  
   B. RESIGNAL语句的执行不需要任何特殊的权限；  
   C. condition_value可以是SQL状态值或DECLARE ... CONDITION定义的异常名称，注意此处不可使用错误编号，否则会导致ERROR 1646 (HY000): SIGNAL/RESIGNAL can only use a CONDITION defined with SQLSTATE；  

3. RESIGNAL的几种用法  
   A. 仅使用RESIGNAL作为单独的SQL语句  
   B. RESIGNAL+SET语句一起使用  
   C. RESIGNAL+condition_value+SET语句一起使用  


#### 异常捕捉实战演练
1. 特别说明  
   A. 以定义异常处理中的三种处理方式CONTINUE/EXIT/UNDO为不同点，演示如何定义异常和异常处理；  
   B. 实战演练以存储过程为示例；  
   C. 示例中的异常触发条件为：SELECT * FROM t;在执行示例的数据库中并不存在t表。  

2. 实战演练  

A. 异常处理方式为CONTINUE  
存储过程示例：  
![](/img/MySQL_exception7.png)  

存储过程结果：  
![](/img/MySQL_exception8.png)  

结果分析：  
call sp_test的结果中包含三条语句，第一条语句为SELECT b的输出；然后‘SELECT * FROM t’触发错误编号为1146的异常，此时存储过程中存在对错误编号为1146的异常处理，所以第二条语句为异常处理语句块中SELECT a的输出；由于错误编号为1146的异常处理为CONTINUE，故异常处理后，回到触发异常的下一个语句位置，继续执行，所以第三条语句为SELECT c的输出。  

B. 异常处理方式为EXIT  
存储过程示例：  
![](/img/MySQL_exception9.png)  

存储过程结果：  
![](/img/MySQL_exception10.png)

结果分析：  
call sp_test的结果中包含两条语句，第一条语句和第二条语句的输出与‘异常处理为CONTINUE’的第一条和第二条语句输出分析一致；由于错误编号为1146的异常处理为EXIT，故异常处理完毕后，跳出当前程序，即该示例中SELECT c并不会被执行。  

C. 异常处理方式为UNDO  
存储过程示例：  
![](/img/MySQL_exception11.png)  

存储过程结果：  
![](/img/MySQL_exception13.png)  

结果分析：  
由于异常处理UNDO方式不被支持，故创建时会报错。  
