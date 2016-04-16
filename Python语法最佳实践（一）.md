---
title: Python语法最佳实践（一）
date: 2016-04-16 23:57:16
tags: Python
---

# 语法最佳实践

## 列表推导

在Python中总是要透着一种极简主义，这样才能显示出Python牛逼哄哄。所以Python语法中总是能写的短就写的短，这是Python的精髓。

列表推导就是为了简化列表生成，换一种说法就是用一行代码生成复杂的列表。

在C语言中你要生成一个0~100中偶数构成的数组你只能这么写：

```c
int array[50] = {};
for(int i = 0; i < 100; i++)
{
    if(0 == i % 2)
        array[i] = i;
}
```
一共用了5行，当然更多时候是6行。

Python也有这种土逼的写法，当然也有更牛逼的写法，你可以这样写：

```python
[i for i in xrange(100) if (i % 2) is 0]

```

Python的enmuerate内建函数十分有用，不仅能取元素还可以取序号
```python
>>> l = ['a', 'b', 'c', 'd']
>>> for i, e in enumerate(l):
...   print "{0} : {1}".format(i,e)
... 
0 : a
1 : b
2 : c
3 : d
>>> 

```

结合列表推导，可以写出很简洁的代码：
```python
>>> def handle_something(pos, elem):
...     return "{0} : {1}".format(pos, elem)
... 
>>> something = ["one", "two", "three", "four"]
>>> [handle_something(p, e) for p, e in enumerate(something)]
['0 : one', '1 : two', '2 : three', '3 : four']
>>>
```
**当要对序列中的内容进行循环处理时， 就应该尝试使用List comprehensions**

----

## 迭代器和生成器

迭代器是一种高效率的迭代方式，基于两个方法
- next  返回容器的下一个项目
- \_\_iter\_\_  返回迭代器本身

```python
>>> i = iter("abc")
>>> i.next()
'a'
>>> i.next()
'b'
>>> i.next()
'c'
>>> i.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> 

```

在迭代器的最后，也就是没有对象可以返回的时候，就会抛出StopIteration异常，这个异常会被for捕获用作停止的条件。


要实现一个自定义的类迭代器，必须实现上面的两个两个方法，next用于遍历，\_\_iter\_\_用于返回。

```python
>>> class MyIterator(object):
...   def __init__(self, step):
...     self.step = step
...   def next(self):
...     if self.step == 0:
...         raise StopIteration
...     self.step -= 1
...     return self.step
...   def __iter__(self):
...     return self
... 
>>> for el in MyIterator(4):
...     print el
... 
3
2
1
0
>>> 

```
书中没有对这段程序详细说明，以下是我的理解
>流程是这样子的：
1. 首先初始化类传入4这个参数
2. 调用next方法， step - 1,返回 3，输出
3. 调用\_\_iter\_\_方法，返回self，也就是把self.step当作参数传入
4. 如此循环

### 2.2.1 生成器

对于yield的使用可以是程序更简单、高效。yield指令可以暂停一个函数返回结果，保存现场，下次需要时候继续执行。这个有点像函数的入栈和出栈的过程。但只是形式上相同罢了。

```python
>>> def fibonacii():
...     a, b = 0, 1
...     while True:
...         yield b
...         a, b = b, a + b
... 
>>> fib = fibonacii()
>>> fib.next()
1
>>> fib.next()
1
>>> fib.next()
2
>>> fib.next()
3
>>> [fib.next() for i in range(10)]
[5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
>>> 
```
带有yield的函数不再是一个普通的函数，而是一个generator对象，可以保存环境，每次生成序列中的下一个元素。

>1. 书中没有说明生成器的原理，网上也没有很确定的说法，查阅资料后，个人理解是：当带有yield的函数生成的同时也会生成一个协程，这个协程用于保存yield的上下文，当再次代用该函数时候，切换到协程中取值。
>2. 书中提到“不必须提供使函数可停止的方法”，事实上说的也就是return。在yield函数中只能是**return**而且立马抛出StopIteration，不能是**return a**这样子的，否则抛出SyntaxError

这是抛出SyntaxError：

```python
>>> def thorwstop():
...     a = 0
...     while True:
...         if a == 3:
...             return a
...         a += 1
...         yield b
... 
  File "", line 7
SyntaxError: 'return' with argument inside generator

```

这是抛出StopIteration：

```python
>>> def thorwstop():
...     a = 0
...     while True:
...         if a == 3:
...             return
...         yield a
...         a += 1
>>>
>>> ts = thorwstop()
>>> ts.next()
0
>>> ts.next()
1
>>> ts.next()
2
>>> ts.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> 
```

yield最大的用处的就要提供一个值的时候，不必事前得到全部元素，这样的好处不言而喻，省内存，效率高。这看起来和迭代器有些类似，但是要明确两点：

1. 迭代器必须要知道下一个迭代的对象是谁，也就是迭代的是一个已经知道的值
2. yield每次返回的可以是我们不知道的值，这个值可能是通过某个函数计算得出的


现在有这么一个应用场景：现在正在写一个股票软件，要获取某一只股票实时状态，涨了多少，跌了多少。这都是无法事先知道的，要通过函数实时获取，然后绘制的一个图上。
```python
>>> from random import randint
>>> def get_info():
...     while True:
...         a = randint(0, 100)
...         yield a
... 
>>> def plot(values):
...     for value in values:
...         value = str(value) + "%"
...         yield value
... 
>>> p = plot(get_info())
>>> p.next()
'88%'
>>> p.next()
'61%'
>>> p.next()
'0%'
>>> p.next()
'7%'
>>> p.next()
'6%'
>>> p.next()
'19%'
>>>
```

yield不仅能传出也可以传入，程序也是停在yield出等待参数传入，使用send方法可以传入：

```python
>>> def send_gen():
...     while True:
...         str = (yield)
...         print str
... 
>>> sg = send_gen()
>>> sg.next()
>>> sg.send("hello")
hello
>>> sg.send("python")
python
>>> sg.send("generator")
generator
>>> 
```

>send机制和next一样，但是yield将编程能够返回的传入的值。因而，这个函数可以根据客服端代码来改变行为。同时还添加了throw和close两个函数，以完成该行为。它们将向生成器抛出一个错误：
-  throw允许客户端代码传入要抛出的任何类型的异常
-  close的工作方式是相同的，但是会抛出一个特定的GeneratorExit

一个生成器模版应该由try、except、finally组成：
```python
>>> def fuck_generator():
...     try:
...         i = 0
...         while True:
...             yield "fuck {0}".format(i)
...             i += 1
...     except ValueError:
...         yield "catch your error"
...     finally:
...         print "fuck everthing, bye-bye"
... 
>>> gen = fuck_generator()
>>> gen.next()
'fuck 0'
>>> gen.next()
'fuck 1'
>>> gen.next()
'fuck 2'
>>> gen.throw(ValueError("fuck, fuck, fuck"))
'catch your error'
>>> gen.close()
fuck everthing, bye-bye
>>> gen.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> 

```
----
### 协同程序
没看懂

----
### 生成器表达式

生成器表达式就是用列表推导的语法来替代yield，把[]替换成()。

```python
>>> iter = (i for i in range(10) if i % 2 == 0)
>>> iter.next()
0
>>> iter.next()
2
>>> iter.next()
4
>>> iter.next()
6
>>> iter.next()
8
>>> iter.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> 

```
对于创建简单的生成器，使用生成器表达式，可以减少代码量。

----
### itertools模块

#### islice：窗口迭代器

返回一个子序列的迭代器，超出范围不执行。


>**itertools.islice(iterable, [start,] stop[, step])**
- iterable一个可迭代对象
- start开始位置
- stop结束位置
- step步长


```python
>>> from itertools import islice
>>> def fuck_by_step():
...   str = "fuck"
...   for c in islice(str, 1, None):
...     yield c
... 
>>> fuck = fuck_by_step()
>>> fuck.next()
'u'
>>> fuck.next()
'c'
>>> fuck.next()
'k'
>>> fuck.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> 
```
islice函数就是从特定位置开始迭代，可以指定结束位置，范围为[), 可以指定步长。

#### tee： 往返式的迭代器

书中的例子，不能清楚表示tee的作用，有点看不懂的感觉。其实就是tee可以返回同一个迭代对象的多个迭代器，默认的2个。

```python
>>> from itertools import tee
>>> iter_fuck = [1, 2, 3, 4]
>>> fuck_a, fuck_b = tee(iter_fuck)
>>> fuck_a.next()
1
>>> fuck_a.next()
2
>>> fuck_a.next()
3
>>> fuck_a.next()
4
>>> fuck_a.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> fuck_b.next()
1
>>> fuck_b.next()
2
>>> fuck_b.next()
3
>>> fuck_b.next()
4
>>> fuck_b.next()
Traceback (most recent call last):
  File "", line 1, in 
StopIteration
>>> 
```
----
#### groupby：uniq迭代器

groupby是对可迭代对象重复元素进行分组。

>itertools.groupby(iterable[, key])
- iterable一个可迭代的对象
- key处理函数，默认为标识处理

```python
>>>from itertools import groupby

>>>def compress(data):
...    return ((len(list(group)), name) for name, group in groupby(data))
...
>>>def combainator(iter_data):
...    string = ""
...    for t in iter_data:
...        string = string + str(t[0]) + str(t[1])
...    return string
...
>>>raw_str = "get uuuuuuuuuuuuup"
>>>com_str = combainator(compress(raw_str))
>>>print com_str
1g1e1t1 13u1p
>>>
```
