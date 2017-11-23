## Hive常用正则表达式语法总结

```sql
# 本篇文章dual表中使用的测试数据(由于介绍的是Hive，所以该表一定是存在Hive中的)
hive (default)> select * from dual;
1
abc123def
123456
zxcv
av
A
```

#### 一、Hive的四种正则表达式

1. rlike

   ```sql
   A rlike B
   ```

2. regexp

   ```java
   A regexp B
   ```

3. regexp_extract

   ```sql
   regexp_extract(string subject,string pattern, int index)
   ```

4. regexp_replace

   ```sql
   regexp_replace(string subject,string replacepattern,string wantpattern)
   ```

#### 二、常用正则命令

| 实际命令                   | 快捷命令 | 含义             |
| ---------------------- | ---- | -------------- |
| [0-9]                  | \d   | 匹配纯数字          |
| \[a\-z\]\[0\-9\]\[\_\] | \w   | 匹配数字/字母/下划线    |
|                        | .    | 匹配任意一个字符，除了换行符 |
|                        | \*   | 匹配前面一个正则0次或者多次 |
|                        | +    | 匹配前面一个正则1次     |
|                        | ?    | 定义前面括号内是可选的    |
|                        | ^    | 以什么开头          |
|                        | $    | 以什么结尾          |

#### 三、常用的正则语法

- 以"a"或者"z"开头的

  ```sql
  hive (default)> select * from dual where name regexp '^(a|z)';
  abc123def
  zxcv
  av
  ```

- 以"f"或着"6"结尾的

  ```sql
  hive (default)> select * from dual where name regexp '(f|6)$';
  abc123def
  123456
  ```

- 匹配指定次数的字符串("{}"：前面的表达式出现的次数)

  ```sql
  hive (default)> select * from dual where name regexp '[a-z]{3}\d{3}[a-z]{3}';
  hive (default)> select * from dual where name regexp '[a-z]{3}\\d{3}[a-z]{3}';
  abc123def
  ```

- 匹配开头不在[b-z]之间的字符串

  ```sql
  hive (default)> select * from dual where name regexp '^[^b-z]';
  1
  abc123def
  123456
  av
  A
  ```

- \*和+的区别

  ```sql
  # 匹配前面正则0／多次
  hive (default)> select * from dual where name regexp '([1-3])\\w\*';
  OK
  dual.name
  1
  abc123def
  123456
  Time taken: 0.108 seconds, Fetched: 3 row(s)

  # 匹配前面正则至少1次，所以1这个值就没有被匹配出来
  hive (default)> select * from dual where name regexp '([1-3])\\w+';
  OK
  dual.name
  abc123def
  123456
  ```

  ​