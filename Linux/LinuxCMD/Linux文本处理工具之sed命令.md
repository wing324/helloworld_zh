## Linux文本处理工具之sed常用命令

测试文本：

```shell
# test.txt
test 1
test2
testtest
XtestX
BBtest
```

- 显示test.txt文件的第2-4行

  ```shell
  # sed -n '2,4p' test.txt
  test2
  testtest
  XtestX
  ```

- 显示文件的第4-最后一行

  ```shell
  # sed -n '4,$p' test.txt
  XtestX
  BBtest
  ```

- 将包含B的行删除

  ```shell
  # sed '/B/d' test.txt
  test 1
  test2
  testtest
  XtestX
  ```

- 将所有开头以t或者X的行中的e替换成E

  ```shell
  # sed '/^[tX]/s/e/E/g' test.txt
  tEst 1
  tEst2
  tEsttEst
  XtEstX
  BBtest
  ```

- 删除每一行的前两个字符

  ```shell
  # sed 's/..//' test.txt
  st 1
  st2
  sttest
  estX
  test
  ```

- 删除每一行的后两个字符

  ```shell
  # sed 's/..$//' test.txt
  test
  tes
  testte
  Xtes
  BBte
  ```

- 给匹配BB行的后面加上helloworld

  ```shell
  # sed 's/BB.*/&helloword/g' test.txt
  test 1
  test2
  testtest
  XtestX
  BBtesthelloword

  # sed 's/te.*/&helloword/g' test.txt
  test 1helloword
  test2helloword
  testtesthelloword
  XtestXhelloword
  BBtesthelloword
  ```

- 使用-e指定多个命令项

  ```shell
  # 输出文件的第2-3行，以及第4行
  # sed -n -e '2,3p' -e '4p' test.txt
  test2
  testtest
  XtestX
  ```

  ​