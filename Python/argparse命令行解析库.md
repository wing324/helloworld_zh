## argparse命令行解析库

argparse为命令行解析库，可以轻松的写出交互友好式的命令行。  



开胃菜
-----
以官方一个简单例子展示argparse模块的用处：  
```python
#!/usr/bin/env python
#coding:utf-8

import argparse

parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                   help='an integer for the accumulator')
parser.add_argument('--sum', dest='accumulate', action='store_const',
                   const=sum, default=max,
                   help='sum the integers')

args = parser.parse_args()
print args.accumulate(args.integers)
```
运行结果如下：
```python
[root@ip-172-31-1-8 python]# python argparse_demo.py -h
usage: argparse_test.py [-h] [--sum] N [N ...]

Process some integers.

positional arguments:
  N           an integer for the accumulator

optional arguments:
  -h, --help  show this help message and exit
  --sum       sum the integers
```


创建解析
-------
通过argparse.ArgumentParser()创建解析。  
```python
ArgumentParser(prog='', usage=None, description='Process some integers.', version=None, formatter_class=<class 'argparse.HelpFormatter'>, conflict_handler='error', add_help=True)

# 例如
parser = argparse.ArgumentParser(description='Process some integers.')
```
##### 解释
- prog  
  程序名称(默认sys.argv[0])  
```python
# 变更参数
parser = argparse.ArgumentParser(prog='sum or max',description='Process some integers.')

#运行结果
#原始
[root@ip-172-31-1-8 python]# python argparse_test.py -h
usage: argparse_test.py [-h] [--sum] N [N ...]

#变更后
[root@ip-172-31-1-8 python]# python argparse_test.py -h
usage: sum or max [-h] [--sum] N [N ...]
```
- usage  
  用于描述程序的使用用法（默认为添加到解析器中的参数）    
```python
# 变更参数
parser = argparse.ArgumentParser(usage='python argparse_demo.py arguments',description='Process some integers.')

#运行结果
#原始
[root@ip-172-31-1-8 python]# python argparse_demo.py -h
usage: argparse_demo.py [-h] [--sum] N [N ...]

#变更后
[root@ip-172-31-1-8 python]# python argparse_demo.py -h
usage: python argparse_demo.py arguments
```
- description  
  参数帮助前的显示文本，可以通过该参数来描述该程序的功能  
```python
# 变更参数
parser = argparse.ArgumentParser(description='Process some integers.')

# 运行结果
[root@ip-172-31-1-8 python]# python argparse_demo.py -h
usage: python argparse_demo.py argument

Process some integers.
```
- epilog  
  参数选项帮助后的显示文本  
```python
# 变更参数
epilog='And What can I help U?'

# 运行结果
optional arguments:
  -h, --help  show this help message and exit
  --sum       sum the intergers

And What can I help U?
```
- parents  
  共享同一个父类解析器，由ArgumentParser对象组成的列表，它们的arguments选项会被包含到新ArgumentParser对象中。
- formatter_class  
  help信息输出格式  
  共有三种形式：argparse.RawDescriptionHelpFormatter、argparse.RawTextHelpFormatter、argparse.ArgumentDefaultsHelpFormatter  
  argparse.RawDescriptionHelpFormatter:description以输入格式输出，并不将其合并为一行  
  argparse.RawTextHelpFormatter:所有信息以输入格式输出，并不将其合并为一行  
  argparse.ArgumentDefaultsHelpFormatter:输出参数的defalut值  
```python
# 变更参数
formatter_class=argparse.ArgumentDefaultsHelpFormatter

# 运行结果
optional arguments:
  -h, --help  show this help message and exit
  --sum       sum the intergers (default: max)
```
- prefix_chars  
  参数前缀，默认为'-'  
- fromfile_prefix_chars  
  前缀字符，放在文件名之前  
  当参数过多时，可以将参数放在文件中读取。  
- argument_default  
  参数的全局默认值  
- conflict_handler  
  解决冲突的策略  
```python
# 没有conflict_handler冲突策略
>>> parser=argparse.ArgumentParser(prog='TEST')
>>> parser.add_argument('-f','--foo',help='old foo help')
_StoreAction(option_strings=['-f', '--foo'], dest='foo', nargs=None, const=None, default=None, type=None, choices=None, help='old foo help', metavar=None)
>>> parser.add_argument('-f','--foo',help='new foo help')   
Traceback (most recent call last):
......
argparse.ArgumentError: argument -f/--foo: conflicting option string(s): -f, --foo

#有conflict_handler策略
>>> parser=argparse.ArgumentParser(prog='TEST',conflict_handler='resolve')
>>> parser.add_argument('-f','--foo',help='old foo help')
_StoreAction(option_strings=['-f', '--foo'], dest='foo', nargs=None, const=None, default=None, type=None, choices=None, help='old foo help', metavar=None)
>>> parser.add_argument('-f','--foo',help='new foo help')   
_StoreAction(option_strings=['-f', '--foo'], dest='foo', nargs=None, const=None, default=None, type=None, choices=None, help='new foo help', metavar=None)
>>> parser.print_help()
usage: TEST [-h] [-f FOO]

optional arguments:
  -h, --help         show this help message and exit
  -f FOO, --foo FOO  new foo help
```
- add_help  
  设为False时，不再显示-h,--help信息，建议使用True  


添加参数
-------
通过argparse.ArgumentParser.add_argument()添加参数  
```python
# 例如
parser.add_argument('intergers',metavar='N',type=int,nargs='+',help='an interger for the accumulator')
parser.add_argument('--sum',dest='accumulate',action='store_const',const=sum,default=max,help='sum the intergers (default:find the max)')
args = parser.parse_args()
```
- name or flag  
  optional arguments以'-'为前缀的参数,其他的为positional arguments  
```python
# 添加optional argument
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('-t','--test')
_StoreAction(option_strings=['-t', '--test'], dest='test', nargs=None, const=None, default=None, type=None, choices=None, help=None, metavar=None)
# 添加position argument
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('test')
_StoreAction(option_strings=[], dest='test', nargs=None, const=None, default=None, type=None, choices=None, help=None, metavar=None)
```
- action  
  命令行参数的操作  
  store:仅仅存储参数的值（默认）  
```python
>>> parser=argparse.ArgumentParser(conflict_handler='resolve')
>>> parser.add_argument('-t')
_StoreAction(option_strings=['-t'], dest='t', nargs=None, const=None, default=None, type=None, choices=None, help=None, metavar=None)
>>> parser.parse_args('-t 1'.split())  
Namespace(t='1')
```
store_const:存储const关键字指定的值  
```python
>>> import argparse
>>> parser=argparse.ArgumentParser(conflict_handler='resolve')
>>> parser.add_argument('-t',action='store_const',const=7)
_StoreConstAction(option_strings=['-t'], dest='t', nargs=0, const=7, default=None, type=None, choices=None, help=None, metavar=None)
>>> parser.parse_args('-t'.split())
Namespace(t=7)
```
store_true/store_false:值为True/False  
append:值追加到list中  
append_const:将const的值追加到list中  
count:统计参数出现的次数  
help:显示help信息  
version:显示version信息  
- nrgs  
  参数的数量  
  N：N个参数  
  ?:首先从命令行中获取，若没有则从const中获取，仍然没有则从default中获取  
  */+:任意多个参数  
- const  
  保存为一个常量  
- default  
  默认值  
- type  
  参数类型  
- choices  
  可供选择的值，该值适合于所有类型，如list、set等  
- required  
  是否为必选参数  
- desk  
  参数别名  
- help  
  参数的帮助信息，即解释信息  
```python
# 添加help参数
parser.add_argument('intergers',metavar='N',type=int,nargs='+',help='an interger for the accumulator')
parser.add_argument('--sum',dest='accumulate',action='store_const',const=sum,default=max,help='sum the intergers')

# 运行结果
[root@ip-172-31-1-8 python]# python argparse_demo.py -h
usage: python argparse_demo.py arguments

Process some integers.

positional arguments:
  N           an interger for the accumulator

optional arguments:
  -h, --help  show this help message and exit
  --sum       sum the intergers (default: max)
```
- metavar  
  相当于参数别名  


解析参数
-------
通过argparse.ArgumentParser.parse_args()解析参数。  
**注意**  
argparse.ArgumentParser.parse_args()中的参数可以模糊匹配，如下：  
```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('-bacon')
>>> parser.add_argument('-badger')
>>> parser.parse_args('-bac MMM'.split())
Namespace(bacon='MMM', badger=None)
>>> parser.parse_args('-bad WOOD'.split())
Namespace(bacon=None, badger='WOOD')
>>> parser.parse_args('-ba BA'.split())
usage: PROG [-h] [-bacon BACON] [-badger BADGER]
PROG: error: ambiguous option: -ba could match -badger, -bacon
```


参数组
------
使用ArgumentParser.add_argument_group(title=None, description=None)添加参数组  
```python
# 执行脚本
#!/usr/bin/env python
#coding:utf-8

import argparse
# 参数解析器重命名
parser = argparse.ArgumentParser(usage='python argparse_demo.py arguments',description='Process some integers.')
# 添加参数
parser.add_argument('--sum',dest='accumulate',action='store_const',const=sum,default=max,help='sum the intergers (default:find the max)')
# 添加参数组1
group1 = parser.add_argument_group('group1','group test 1')
#为参数组添加参数
group1.add_argument('intergers',metavar='N',type=int,nargs='+',help='an interger for the accumulator')
group1.add_argument('lists',metavar='L',type=list,nargs='+',help='a list for the accumulator')
args = parser.parse_args()
print args.accumulate(args,intergers)

# 运行结果
[root@ip-172-31-1-8 python]# python argparse_demo.py --help
usage: python argparse_demo.py arguments

Process some integers.

optional arguments:
  -h, --help  show this help message and exit
  --sum       sum the intergers (default:find the max)

group1:
  group test 1

  N           an interger for the accumulator
  L           a list for the accumulator
```


设置默认值
---------
使用ArgumentParser.set_defaults(**kwargs)设置参数默认值,ArgumentParser.get_default(dest)获取参数默认值。  
**注意**  
解释器级别的默认值会覆盖参数级别的默认值。  
```python
>>> import argparse
>>> prser=argparse.ArgumentParser()
>>> parser=argparse.ArgumentParser()
# 参数级别默认值
>>> parser.add_argument('--foo',default='7')   
_StoreAction(option_strings=['--foo'], dest='foo', nargs=None, const=None, default='7', type=None, choices=None, help=None, metavar=None)
# 解析器级别默认值
>>> parser.set_defaults(foo='77')
# 解析器级别的默认值覆盖参数级别的默认值
>>> parser.parse_args([])
Namespace(foo='77')
# 获取参数默认值
>>> parser.get_default('foo')
'77'
```

