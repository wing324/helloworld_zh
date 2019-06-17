title: Python3 Brief Note

date: 2019-06-16 20:29:06

categories:

- Python

tags:

- Note

------

Some knowledge hard to understand in Python.
<!-- more -->

#### 1. List Comprehension

```python
# if item in list is even, return item*item
[ i*i for i in range(1,11) if(i%2==0)]

# result
[4, 16, 36, 64, 100]
```

#### 2. Dict Comprehension

```python
# code without dict comprehension
z_name = {"A", "B","C"}
z_num = {}
for i in z_name:
    z_num[i]=0
    
# code with dict comprehension
z_name = {"A","B","C"}
z_num = [ i:0 for i in z_name]
```

#### 3. Exception

```python
# concept
try:
    <monitor code>
except Exception:
    <exception code>
finally:
    <excute code whether or not there is an exception>
    
# example 1
try:
    year = int(input("PLS enter a year"))
except ValueError:
    print("Please enter an INT for the input box.")
    
# example 2
try:
    fo = open("name.txt")
except Exception as e:
    print(e)
finally:
    fo.close()
```

#### 4. Variable Parameters for Function

```python
# example
def howlong(first, *others):
    return len(first) + len(others)
print(howlong("a", "def", "ghijklmn", "opq"))

# result
4
```

#### 5. Variable Scope

```python
# example without global
x = 123
def func():
    x = 456
    print("Inside X is: %d" %x)
func()
print("Outside X is: %d" %x)

# result
Inside X is: 456
Outside X is: 123
    
# example with global
x = 123
def func():
    global x
    x = 456
    print("Inside X is: %d" %x)
func()
print("Outside X is: %d" %x)

# result
Inside X is: 456
Outside X is: 456
```

#### 6. lambda expression

```python
# example
>>> g = lambda x:x+1
>>> g(3)
4
>>> g(5)
6

# filter example
a = [1,2 ,3,4,5,6,7]
print(list(filter(lambda x:x>4, a)))
# result
[5, 6, 7]

# map example 1
a = [1,2 ,3,4,5,6,7]
print(list(map(lambda x:x*x, a)))
# result
[1, 4, 9, 16, 25, 36, 49]

# map example 2
a = [1,2 ,3,4,5,6,7]
b = [11,22,33,44,55,66,77]
print(list(map(lambda x,y:x*y, a, b)))
# result
[11, 44, 99, 176, 275, 396, 539]

# reduce example
>>> from functools import reduce
>>> reduce(lambda x,y:x+y, [7,8,9],1)
25
>>> 1+7+8+9
25
>>> reduce(lambda x,y:x+y, [7,8,9],2)
26
>>> 2+7+8+9
26

# zip example
```

#### 7. Closure

```python
# example
def func():
    a = 1
    b = 2
    return a+b
def sum(a):
    def add(b):
        return a+b
    return add
num1 = func()
num2 = sum(1)
print(type(num1))
print(type(num2))
print(num1)
print(num2(2))

# result
<class 'int'>
<class 'function'> # this is "add" function
3
3
```

#### 8. Decorator

```python
# example
import time
def timer(func):
    def wrapper():
        start_time = time.time()
        func()
        stop_time = time.time()
        print("Total time is: %s second." %(stop_time - start_time))
    return wrapper
@timer
def i_can_sleep():
    time.sleep(3)
i_can_sleep()

# result
Total time is: 3.0007171630859375 second.
```

#### 9. Class

```python
# example
class Player():
    def __init__(self, name, life):
        self.__name = name
        self.life = life
    def print_role(self):
        print("name is: %s, life is %d" %(self.__name, self.life))
p1 = Player("Boss",100)
p1.print_role()

# result
name is: Boss, life is 100
```

#### 10. Inheritance

```python
# example
class Animal():
    def __init__(self, name):
        self.name = name
    def run(self):
        print("%s is running." %(self.name))
class Dog(Animal):
    def __init__(self, name):
        # self.name = name
        super().__init__(name)
class Cat(Animal):
    pass
t1 = Animal("animal")
print(t1.name)
print(t1.run())
t2 = Dog("dog")
print(t2.name)
print(t2.run())
t3 = Cat("cat")
print(t3.name)
print(t3.run())

# result
animal
animal is running.
dog
dog is running.
cat
cat is running.
```

