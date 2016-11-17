## Oralce中VARCHAR2()和NVARCHAR2()的区别

Oralce中VARCHAR2()和NVARCHAR2()的区别。  



VARCHAR2()和NVARCHAR2()的官方定义
--------------------------------
官方文档定义如下：  
###### VARCHAR2(size [BYTE | CHAR])  
Variable-length character string having maximum length size bytes or characters. Maximum size is 4000 bytes or characters, and minimum is 1 byte or 1 character. You must specify size for VARCHAR2.  
BYTE indicates that the column will have byte length semantics. CHAR indicates that the column will have character semantics.  
###### NVARCHAR2(size)  
Variable-length Unicode character string having maximum length size characters. The number of bytes can be up to two times size for AL16UTF16 encoding and three times size for UTF8 encoding. Maximum size is determined by the national character set definition, with an upper limit of 4000 bytes. You must specify size for NVARCHAR2.  

中文翻译：  
###### VARCHAR2(size [BYTE | CHAR])  
具有最大长度的字节数（bytes）或字符数（char）的可变长度的字符类型。最大长度为4000字节/字符，最小长度是1字节/字符。你必须为VARCHAR2()类型指定大小。  
BYTE代表该列以字节计算长度，CHAR代表该列以字符计算长度。  
###### NVARCHAR2(size)  
具有最大长度的带有字符集属性的可变长度的字符类型。它的长度是AL16UTF16字符集的2倍，UTF8字符集的三倍。它的最大长度取决于字符集，上限位4000字节。您必须为NVARCHAR2()类型指定大小。  

总结：  
NVARCHAR2(size)与VARCHAR2(size CHAR)相似，唯一的区别是NVARCHAR2(size)的最大长度是4000**字节**（实验测试结果是，在utf8的字符集下，最大长度为2000字符），而VARCHAR2(size CHAR）的最大长度是4000**字符**。  


实战演练
-------
使用字符集为UTF8。  
```sql
# 验证NVARCHAR2(size)与VARCHAR2(size CHAR)相似
SQL> create table t_varchar2(name varchar2(6 CHAR));
Table created

SQL> insert into t_varchar2 values('中国');
1 row inserted

SQL> insert into t_varchar2 values('中华人民共和');
1 row inserted

SQL> insert into t_varchar2 values('中华人民共和国');
insert into t_varchar2 values('中华人民共和国')
ORA-12899: 列 "SCOTT"."T_VARCHAR2"."NAME" 的值太大 (实际值: 7, 最大值: 6)

SQL> create table t_nvarchar2(name nvarchar2(6));
Table created

SQL> insert into t_nvarchar2 values('中国');
1 row inserted

SQL> insert into t_nvarchar2 values('中华人民共和');
1 row inserted

SQL> insert into t_nvarchar2 values('中华人民共和国');
insert into t_nvarchar2 values('中华人民共和国')
ORA-12899: 列 "SCOTT"."T_NVARCHAR2"."NAME" 的值太大 (实际值: 7, 最大值: 6)

# 验证NVARCHAR2(sie)与VARCHAR(size CHAR)存在最大长度不同
SQL> create table t_varchar2(name varchar2(4000 CHAR));
Table created

SQL> create table t_nvarchar2(name nvarchar2(4000));
create table t_nvarchar2(name nvarchar2(4000))
ORA-00910: 指定的长度对于数据类型而言过长
```
