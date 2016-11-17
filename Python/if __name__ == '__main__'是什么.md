## if __name__ ==  '__main__是什么'

在实验楼学习Python的时候，被一个奇怪的代码弄得很不知所措，if __name__ == '__main__'代表啥意思呢？通过google,终于在stackoverflow找到了我能理解的答案，通过实验让自己更深入的理解和牢记。
参考链接：http://stackoverflow.com/questions/419163/what-does-if-name-main-do  

实验开始吧~  

- 首先准备一段代码：  
```python
#! /usr/bin/env python
# coding:utf-8

# A.py

def t():
	print 'I AM IN FUNCTION!'

print 'I AM I A!'

if __name__ == '__main__':
	print 'FIRST TIME IN A!'
else:
	print 'SECOND TIME IN A!'
	

# B.py

import A
A.t()

print 'I AM IN B!'

if __name__ == '__main__':
	print 'FIRST TIME IN B!'
else:
	print 'SECOND TIME IN B'
```

- 看看代码的执行结果吧
```python
[root@localhost python]# python A.py 
I AM I A!
FIRST TIME IN A!
[root@localhost python]# python B.py 
I AM I A!
SECOND TIME IN A!
I AM IN FUNCTION!
I AM IN B!
FIRST TIME IN B!
```
- 执行结果能理解吗？还是先说下if __name__ == '__main__'的含义吧
  当包含if __name__ == '__main__'的程序直接被执行时，如python A.py，此时__name__与__main__相等，即if __name__ == '__main__'成立，执行的是if成立的代码块；
  当包含if __name__ == '__main__'的程序被import到其他的程序中，如B.py程序中的import A，此时A中的__name__与__main__不相等，即if __name__ == '__main__'不成立，执行的是if不成立的代码块，即else的代码块。

- 好了，原理知道了，那我们来分析下代码的执行结果吧
```python
# A程序的执行结果
[root@localhost python]# python A.py 
I AM I A! #这是程序中if语句之外的语句块print 'I AM I A!'输出的
FIRST TIME IN A! #这是程序自己执行时，__name__==__main__时，if成立的语句块输出的


[root@localhost python]# python B.py 
I AM I A! #这是import A时，if语句之外的语句块print 'I AM I A!'输出的
SECOND TIME IN A! #这是import A时，__name__！=__main__时，if不成立的语句块else语句输出的
I AM IN FUNCTION! #这是B程序调用A程序中的函数A.t()输出的
I AM IN B! #这是程序中if语句之外的语句块print 'I AM I B!'输出的
FIRST TIME IN B! #这是程序自己执行时，__name__==__main__时，if成立的语句块输出的
```