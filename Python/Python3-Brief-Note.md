#  Python3 Brief Note

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

#### 11. With expression

```python
# example without exception
class Testwith():
    def __enter__(self):
        print("run")
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit")
with Testwith():
    print("Test is running")
# result
run
Test is running
exit

# example with exception
class Testwith():
    def __enter__(self):
        print("run")
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_tb is None:
            print("Normal exit")
        else:
            print("exit with error: %s" %exc_tb)
with Testwith():
    print("Test is running")
    raise NameError("Test NameError")
# result
run
Test is running
exit with error: <traceback object at 0x030830D0>
Traceback (most recent call last):
  File "hellopython.py", line 108, in <module>
    raise NameError("Test NameError")
NameError: Test NameError
```

#### 12. Multi Threads

```python
# example while MainThread end first
import threading
import time
def myThread(arg1, arg2):
    print(threading.current_thread().getName(), "start")
    time.sleep(1)
    print("%s %s" %(arg1, arg1))
    print(threading.current_thread().getName(), "stop")
for i in range(1,6,1):
    t1 = threading.Thread(target=myThread, args=(i, i+1))
    t1.start()
print(threading.current_thread().getName(), "end")

# result
Thread-1 start
Thread-2 start
Thread-3 start
Thread-4 start
Thread-5 start
MainThread end
3 3
Thread-3 stop
2 2
Thread-2 stop
1 1
Thread-1 stop
4 4
Thread-4 stop
5 5
Thread-5 stop

# example while SubThread end first
import threading
class MyThread(threading.Thread):
    def run(self):
        print(threading.current_thread().getName(),"Start")
        print("Run")
        print(threading.current_thread().getName(), "Stop")
t1 = MyThread()
t1.start()
t1.join()
print(threading.current_thread().getName(), "End")
# result
Thread-1 Start
Run
Thread-1 Stop
MainThread End
```

#### 13. Queue

```python
>>> import queue
>>> q = queue.Queue()
>>> q.put(1)
>>> q.put(2)
>>> q.put(3)
>>> q.put(4)
>>> q.get()
1
>>> q.get()
2
>>> q.get()
3
>>> q.get()
4
```

#### 14. Producer & Consumer problem

```python
# basic code
from threading import Thread, current_thread
import time
import random
from queue import Queue
queue = Queue(5)
class ProducerThread(Thread):
    def run(self):
        name = current_thread().getName()
        nums = range(100)
        global queue
        while True:
            num = random.choice(nums)
            queue.put(num)
            print("Producer %s produces %s data" %(name, num))
            t = random.randint(1,3)
            time.sleep(t)
            print("Producer %s sleeps %s seconds" %(name, t))
class ConsumerThread(Thread):
    def run(self):
        name = current_thread().getName()
        global queue
        while True:
            num = queue.get()
            queue.task_done()
            print("Consumer %s consumers %s data" %(name, num))
            t = random.randint(1,5)
            time.sleep(t)
            print("Consumer %s sleeps %s seconds" %(name, t))

# Consumer more than Producer
p1 = ProducerThread(name="p1")
p1.start()
c1 = ConsumerThread(name="c1"prop)
c1.start()
c2 = ConsumerThread(name="c2")
c2.start()
# result
Producer p1 produces 73 data
Consumer c1 consumers 73 data
Producer p1 sleeps 1 seconds
Producer p1 produces 51 data
Consumer c2 consumers 51 data
Consumer c1 sleeps 2 seconds
Producer p1 sleeps 3 seconds
Producer p1 produces 7 data
Consumer c1 consumers 7 data
Consumer c1 sleeps 1 seconds
Producer p1 sleeps 1 seconds
Producer p1 produces 30 data
Consumer c1 consumers 30 data
Consumer c2 sleeps 5 seconds
Producer p1 sleeps 1 seconds

# Producer more than Consumer
p1 = ProducerThread(name="p1")
p1.start()
p2 = ProducerThread(name="p2")
p2.start()
p3 = ProducerThread(name="p3")
p3.start()
c1 = ConsumerThread(name="c1")
c1.start()
c2 = ConsumerThread(name="c2")
c2.start()
#result
Producer p1 produces 32 data
Producer p2 produces 4 data
Producer p3 produces 17 data
Consumer c1 consumers 32 data
Consumer c2 consumers 4 data
Producer p1 sleeps 2 seconds
Producer p1 produces 79 data
Producer p2 sleeps 2 seconds
Producer p2 produces 33 data
Consumer c1 sleeps 3 seconds
Consumer c1 consumers 17 data
Producer p1 sleeps 1 seconds
Producer p3 sleeps 3 seconds
Producer p3 produces 55 data
Producer p1 produces 1 data
Consumer c2 sleeps 4 seconds
Consumer c2 consumers 79 data
Producer p2 sleeps 3 seconds
Producer p2 produces 52 data
Producer p3 sleeps 2 seconds
Producer p3 produces 34 data
```

#### 15. Collect Data from Website

```python
# example 1
from urllib import request
url = "https://www.google.com/"
res = request.urlopen(url, timeout=1)
print(res.read())

# example 2
from urllib import request
from urllib import parse
# get
res1 = request.urlopen("http://httpbin.org/get", timeout=1)
print(res1.read())
# post
data = bytes(parse.urlencode({"world":"hello"}), encoding="utf8")
print(data)
res2 = request.urlopen("http://httpbin.org/post", data=data)
print(res2.read().decode("utf-8"))
```

