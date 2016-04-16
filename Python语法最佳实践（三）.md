---
title: Python语法最佳实践（三）
date: 2016-04-16 23:57:48
tags: Python
---

# 语法最佳实践


## 子类化内建类型

内建类型object是所有内建类型的公共祖先。

当需要实现一个与内建类型十分类似的类时，可以使用例如**list、tuple、dict**这样的内建类型进行子类化。

可以继承父类中的方法在子类中使用，可以让内建类型适用在更多场景下,例如下面这个不能有重复元素的list：

```python
In [17]: class ListError(Exception):
   ....:     pass
   ....: 

In [18]: class SingleList(list):
   ....:     def append(self, elem):
   ....:         if elem in self:
   ....:             raise ListError("{0} aleady in SingleList")
   ....:         else:
   ....:             super(SingleList, self).append(elem)
   ....:             

In [19]: sl = SingleList()

In [20]: sl.append('a')

In [21]: sl.append('b')

In [22]: sl.append('c')

In [23]: sl.append('a')
-----------------------------------------------------------------------
ListError                                 Traceback (most recent call l
 in ()
----> 1 sl.append('a')

 in append(self, elem)
      2     def append(self, elem):
      3         if elem in self:
----> 4             raise ListError("{0} aleady in SingleList")
      5         else:
      6             super(SingleList, self).append(elem)

ListError: {0} aleady in SingleList
```

----

## 访问超类中的方法

**super**是一个内建类型，用来访问属于某个对象的超类中的特性(attribute)

访问父类的方法有两种：
>1. super(子类名, self) . 父类方法()
2. 父类名 . 父类方法(self)

暂且称为super法和self法

先定义一个父类**Animal**

```python
In [35]: class Animal(object):
   ....:     def walk(self):
   ....:         print "Animal can walk"
   ....:     
```


**super**使用父类方法

```python
In [36]: class People(Animal):
   ....:     def fly(self):
   ....:         super(People, self).walk()
   ....:         print "People can fly"
   ....:         

In [37]: person = People()

In [38]: person.fly()
Animal can walk
People can fly

```

**self**

```python
In [40]: class People(Animal):
   ....:     def run(self):
   ....:         Animal.walk(self)
   ....:         print "People can run"
   ....:         

In [41]: person = People()

In [42]: person.run()
Animal can walk
People can run

```

其中super是新方法，对比下两种方法，关键的不同之处在于调用时候**self法使用父类名字，super法使用子类名字**。这就使得在多继承的时候super变的极其难用。

### 理解Python中方法解析顺序（MRO）

>Python2.3中添加了基于Dylan构建的MRO，即C3的一个新的MRO，它描述了C3构建一个类的线性化（也称优先级，即祖先的一个排序列表）的方法。这个列表被用于特性的查找


书中描述的很理论化，这个本身就是可很理想的假设，因为没有人会这么设计。

说白了，以前的MRO是深度优先，现在的是广度优先。

### super的缺陷

在Python中子类不会自动调用父类的**\_\_init\_\_**，所以手动的调用。

#### 混用super和传统调用

定义两个基类A、B：

```python
In [1]: class A(object):
   ...:     def __init__(self):
   ...:         print 'A'
   ...:         super(A, self).__init__()
   ...:         

In [2]: class B(object):
   ...:     def __init__(self):
   ...:         print 'B'
   ...:         super(B, self).__init__()
   ...:         

```

定义一个C类继承A、B：

```python
In [3]: class C(A, B):
   ...:     def __init__(self):
   ...:         print 'C'
   ...:         A.__init__(self)
   ...:         B.__init__(self)
   ...:   

```

这里在C类中使用super和基类使用传统调用，输出结果

```python
In [4]: print "MRO", [x.__name__ for x in C.__mro__]
MRO ['C', 'A', 'B', 'object']

In [5]: C()
C
A
B
B
Out[5]: <__main__.C at 0x7fd2457df810>

In [6]: 

```

输出了两次B，这不是我们想要的。把C类中改成super方法后就正常了

```python
In [6]: class C(A, B):
   ...:     def __init__(self):
   ...:         print 'C'
   ...:         super(C, self).__init__()
   ...:         

In [7]: print "MRO", [x.__name__ for x in C.__mro__]
MRO ['C', 'A', 'B', 'object']

In [8]: C()
C
A
B
Out[8]: <__main__.C at 0x7fd2457df390>

In [9]: 

```

#### 不同类型的参数

super的一个缺陷是，每个基类\_\_init\_\_的参数个数不同的话，怎么办？

```python
class A(object):
     def __init__(self):
         print 'A'
         super(A, self).__init__()
   
 class B(object):
     def __init__(self, arg):
         print 'B'
         super(B, self).__init__()        

class C(A, B):
     def __init__(self, arg):
         print 'C'
         super(C, self).__init__(arg)         
In [12]: c = C(10)
C
```

```
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
 in ()
----> 1 c = C(10)

 in __init__(self, arg)
      2     def __init__(self, arg):
      3         print 'C'
----> 4         super(C, self).__init__(arg)
      5 

TypeError: __init__() takes exactly 1 argument (2 given)

```

从报出的错误中可以看出是多了一个参数，导致**\_\_init\_\_**调用失败。

一个妥协的方法就是使用\*args和\*\*kw,这样无论是一个还是两个，还是没有参数，都可以成功。但是这么做导致代码变的脆落。另一个解决的办法就是使用传统的**\_\_init\_\_**，这又会导致第一个问题。

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

class Base(object):
    def __init__(self, *args, **kw):
        print 'Base'
        super(Base, self).__init__()

class A(Base):
    def __init__(self, *args, **kw):
        print 'A'
        super(A, self).__init__(*args, **kw)
class B(Base):
    def __init__(self, *args, **kw): 
        print 'B'
        super(B, self).__init__(*args, **kw)
class C(A , B):
    def __init__(self, arg):
        print 'C'
        super(C, self).__init__(arg)

if __name__ == '__main__':
    C(10)

---------------------------------------
C
A
B
Base
```

书中的例子已经不能运行成功了，可能是版本更新的原因, object的\_\_init\_\_必须为空，否则会报出参数不对的错误。

## 最佳实践

- 应该避免多重继承， 使用一些设计模式替代
- super的使用必须一致， 在类层次结构中， 应该在所有地方使用super或者彻底不使用它。混用super和传统调用是一种混乱的方法， 人们倾向于避免使用super， 这样使代码更清晰。
- 不要混用老式和新式的类， 两者都具备的代码库将导致不同的MRO表现。
- 调用父类时必须检查类层次， 避免出现任何代码问题， 每次调用父类时， 必须查看一下所涉及的MRO（使用__mro__）



