# Five underscores in Python

#### 1. \_var(single underscore)

This is a hint to programmer, it means the variable for internal use. It is just a hint, not enforce by Python.

```python
>>> class Test:
...     def __init__(self):
...             self.foo = 11
...             self._bar = 23
...
>>> t = Test()
>>> t.foo
11
>>> t._bar
23
```

From the example, we can see that although \_bar for internal use, we can also get it from outside. Because it is just a hint, not enforce by Python.

#### 2. var\_(single underscore)

Sometimes, we already use a suitable name for others or some keyword we can not use as a variable name, so we can user `var_` instead of it.

```python
>>> def test(class_):
...     print("%s"  %class_)
...
>>> test("hello class_")
hello class_
```

For the instance, we can not use `class` as a variable name, but we can use `class_` as a variable name.

#### 3. __var(double underscore)

A double underscore prefix cause Python rewrite the attribute name. This is also called *name mangling*, the Python interpreter change variable name in order to make it harder create collisions when the class is extended later, it also looks like private attribute.

```python
>>> class Test:
...     def __init__(self):
...             self.foo=11
...             self._bar = 23
...             self.__baz = 23
...
>>> t1 = Test()
>>> dir(t1)
['_Test__baz', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_bar', 'foo']
>>> class ExtendedTest(Test):
...     def __init__(self):
...             super().__init__()
...             self.foo = "overrideden"
...             self._bar = "overrideden"
...             self.__baz = "overrideden"
...
>>> t2 = ExtendedTest()
>>> dir(t2)
['_ExtendedTest__baz', '_Test__baz', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_bar', 'foo']
>>> t2.foo
'overrideden'
>>> t2._bar
'overrideden'
>>> t2.__baz
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'ExtendedTest' object has no attribute '__baz'
>>> t2._ExtendedTest__baz
'overrideden'
```

From the example, we can see that variables name with double underscore prefix changed to another name inside, and we can not get the variable with the name we assigned to it, we should use the name which the Python interpreter assign to it. But, **the amazing thing is our code can use the name we assigned inside of the class**. Let\'s get another example:

```python
>>> class ManglingTest:
...     def __init__(self):
...             self.__mangled = "hello"
...     def get_mangled(self):
...             return self.__mangled
...
>>> mtest = ManglingTest()
# we can not get __mangled outside of ManglingTest class
>>> mtest.__mangled
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'ManglingTest' object has no attribute '__mangled'
# ManglingTest method can get __mangled inside of ManglingTest class
>>> mtest.get_mangled()
'hello'
```

#### 4. \_\_var\_\_(double underscore)

*Name Mangling* doesn\'t apply for variable name starts and ends with double underscore. Variables name start and end with double underscore means the variable is constructor(like \_\_init\_\_)  or can be call by outside(like public attribute). Normally, we don\'t use start and end double underscore to name our variables.

```python
class PrefixPostfixTest:
    def __init__(self):
        self.__bam__ = 42

>>> PrefixPostfixTest().__bam__
42
```

#### 5. \_ (just a single underscore)

\_ is just temporary variables, and we don't actually use it. it looks like placeholder variables.

```python
# temporary variables example 
>>> for _ in range(32):
...     print('Hello, World.')

# placeholder variables example
>>> car = ('red', 'auto', 12, 3812.4)
>>> color, _, _, mileage = car

>>> color
'red'
>>> mileage
3812.4
>>> _
12
```



Reference: [The Meaning of Underscores in Python](https://dbader.org/blog/meaning-of-underscores-in-python)