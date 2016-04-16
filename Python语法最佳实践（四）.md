---
title: Python语法最佳实践（四）
date: 2016-04-17 00:00:42
tags: Python
---

# 语法最佳实践

## 描述符和属性

Python没有private的关键字， 最接近的概念是“name mangling”， 就是在变量或者函数前面加上“\_\_”时，它就重命名。

```python
In [18]: class Class(object):
   ....:     __private_value = 1
   ....:     

In [19]: c = Class()

In [20]: c.__private_value
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
 in ()
----> 1 c.__private_value

AttributeError: 'Class' object has no attribute '__private_value'

```
这样就和private很相似，找不到这个attribute。

查看一下该类的属性：

```python
In [21]: dir(Class)
Out[21]: 
['_Class__private_value',
 '__class__',
 '__delattr__',
 '__dict__',
 '__doc__',
 '__format__',
 '__getattribute__',
 '__hash__',
 '__init__',
 '__module__',
 '__new__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__',
 '__weakref__']

```

属性有有**'\_Class\_\_private_value'**这么一项。和原来的属性类似。

```python
In [22]: c._Class__private_value
Out[22]: 1

In [23]: c._Class__private_value = 2

In [24]: c._Class__private_value
Out[24]: 2

```
依旧可以访问，修改，所以这个只是换了一个名字而已。它的真正作用是用来避免继承带来的命名冲突，特性被重命名为带有类名前缀的名称。

在实际中从不使用"\_\_",而是使用"\_"替代。这个不会改变任何东西，只是标明这是私有的而已。

### 描述符

描述符用来自定义在引用一个对象上的特性时所应该完成的事情。它们是定义一个另一个类特性可能的访问方式的类。换句话硕，一个类可以委托另一个类来管理其特性。

描述符基于三个必须实现的特殊方法：

>- \_\_set\_\_    在任何特性被设置的时候调用，在后面是实例中，将其称为setter；
- \_\_get\_\_    在任何特性被读取的时调用（被称为getter）
- \_\_delete\_\_    在特性请求del时调用

>这些方法在\_\_dict\_\_特性之前被调用。

在类特性定义并且有一个getter和一个setter方法时，平常的包含一个对象的实例的所有元素的\_\_dict\_\_映射都将被劫持。

>- 实现了\_\_get\_\_ 和\_\_set\_\_ 的描述符被称作数据描述符
- 只实现了\_\_get\_\_的描述符被称为非数据描述符

在Python中，访问一个属性的优先级顺序按照如下顺序:
1. 类属性
2. 数据描述符
3. 实例属性
4. 非数据描述符
5. \_\_getattr\_\_()方法

下面先创建一个描述符，并实例一个：

```python
In [1]: class UpperString(object):
   ...:     def __init__(self):
   ...:         self._value = ''
   ...:     def __get__(self, object, type):
   ...:         return self._value
   ...:     def __set__(self, object, value):
   ...:         self._value = value.upper()
   ...:         

In [2]: class MyClass(object):
   ...:     attribute = UpperString()
   ...:     
```

类UpperString是一个数据描述符,通过它实例化了一个attribute，也就是说对attribute的get和set操作都会被UpperString描述符劫持。

```python
In [25]: mc = MyClass()

In [26]: mc.attribute
Out[26]: ''

In [27]: mc.attribute = "my value"

In [28]: mc.attribute
Out[28]: 'MY VALUE'
```

如果给实例子中添加一个新的特性，它将被保存在\_\_dict\_\_中

```python
In [29]: mc.new_att = 1

In [30]: mc.__dict__
Out[30]: {'new_att': 1}
```

数据描述符将优先于实例的\_\_dict\_\_

```python
In [35]: MyClass.new_att = UpperString()

In [36]: mc.__dict__
Out[36]: {'new_att': 1}

In [37]: mc.new_att
Out[37]: ''

In [38]: mc.new_att = "other value"

In [39]: mc.new_att
Out[39]: 'OTHER VALUE'

In [40]: mc.__dict__
Out[40]: {'new_att': 1}

```

对于非数据描述符，实例将优先于描述符

```python
class Whatever(object):
    def __get__(self, object, type):
        return "whatever"
    

MyClass.whatever = Whatever()

mc.__dict__
{'new_att': 1}

mc.whatever
'whatever'

mc.whatever = 1

mc.__dict__
{'new_att': 1, 'whatever': 1}
```

描述符除了隐藏类的内容以外还有：

- 内省描述符——这种描述符将检查宿主类签名，以计算一些信息
- 元描述符——这种描述符时类方法本身完成值计算

#### 内省描述符

内省描述符就是一种用于自我检查的通用描述符，也就是可以在多个不同的类使用的一种描述符，书中给的例子是类似dir方法的描述符，也就是列出类的属性。
这种描述符就是检查了类中有什么东西和没有什么东西。我也模仿书中例子，写一个类似dir的自省描述符。

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

class API(object):
    def __get__(self, obj, type):
        if obj is not None:
            return self._print_doc(obj)
        else:
            print "No method in {0}".format(obj)
        
    def _print_doc(self, obj):
        method_list = [method for method in dir(obj) if not method.startswith('_')]
        for m in method_list:
            print "{0} : {1}".format(m, m.__doc__)
            print '-'*50
        return "Thank you for looking API"

class MyClass():

    __doc__ = API()
    
    def __init__(self):
        ''' init MyClass '''
        pass
    
    def method1(self):
        '''print the msg for method1'''
        print "i'm method1"
    
    def method2(self):
        '''print the msg for method2'''
        print "i'm method2"
        
if __name__ == '__main__':
    mc = MyClass()
    print mc.__doc__
```

非数据描述符API只有一个\_\_get\_\_方法，用于获取。在非数据描述符API中的\_print\_doc方法中，过滤掉内置方法，打印出每个方法的doc。

```
"""
method1 : str(object='') -> string

Return a nice string representation of the object.
If the argument is a string, the return value is the same object.
--------------------------------------------------
method2 : str(object='') -> string

Return a nice string representation of the object.
If the argument is a string, the return value is the same object.
--------------------------------------------------
Thank you for looking API
"""
```
#### 元描述符

略难，看看即可

### 属性

> 属性（Propetry）提供了一个内建的描述符类型，它知道如何将一个特性链接到以组方法上。属性采用fget参数和3个可选参数——fset、fdel、和doc。最后一个参数可以提供用来定义一个链接到特性的docstring，就像是个方法一样。

这个propetry函数和前面的描述符基本类似，用参数来实现了描述符中的\__get__、\__set__、\__del__方法而已。

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

class MyClass(object):
    def __init__(self):
        print 'init a value = 10'
        self.value = 10;
    
    def get(self):
        print 'get a value'
        return self.value
    
    def set(self, value):
        print 'set value '
        self.value = value
    
    def ddel(self):
        print 'bye bye'
        self.value = 0

    my_value = property(get, set, ddel, "i'm a value")
    
if __name__ == '__main__':
    mc = MyClass()
    mc.my_value
    mc.set(50)
    del mc.my_value
```
输出为如下：
```
'''
init a value = 10
get a value
set value 
bye bye
'''
```

和之前的描述符的现象一模一样，property为描述符提供了简单的接口。
继承的时候会产生混乱，所创建的特性使用当前类创建，不应该在子类中重载。
```python
class Base(object):
    def _get_price(self):
        return "$ 500"
    price = property(_get_price)
    
class SubClass(Base):
    def _get_price(self):
        return "$ 20"
 
money = SubClass()
money.price
```
取得的是父类的方法，而不是子类的，这不是我们预期的。重载父类属性不是好的做法，重载自身属性更好一些。

```python
class Base(object):
    def _get_price(self):
        return "$ 500"
    price = property(_get_price)
    
class SubClass(Base):
    def _get_price(self):
        return "$ 20"
    price = property(_get_price)
    
money = SubClass()
money.price
```

## 槽

>几乎从未被开发人员使用过的一种有趣的特性是槽

嗯，没人用过

## 元编程

可以通过\__new__和\__metaclass__这两个特殊方法在运行时候修改类和对象的定义

### \__new__方法

\__new__是一个元构造程序，当一个对象被工厂类实例化时候调用。
```python
lass MyClass(object):
   def __new__(cls):
       print '__new__'
       return object.__new__(cls)
   def __init__(self):
       print '__init__'
  
nstance = MyClass()
```
>- \__new__方法是继承object中的\__new__方法，所以该类必须先继承object。
- \__new__的作用是返回一个实例化完成的类，就像程序中的instance，也可以是别的类。
- \__new__中的cls参数是要实例化的类，就是MyClass，也就是类中self
- \__new__的执行在\__init__之前

在实例化类之前，\__new__总是在\__init__之前执行完成更低层次的初始化工作，例如网络套接字或者数据库初始化应该在\__new__中为不是\__init__中控制。它在类的工作必须完成这个初始化以及必须被继承的时候通知我们。

### \__metaclass__方法

[廖雪峰关于元类的理解(ORM例子有点难)](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386820064557c69858840b4c48d2b8411bc2ea9099ba000)
[深刻理解Python中的元类(metaclass)，强烈推荐](http://blog.jobbole.com/21351/)
元类提供了在类对象通过工厂方法在内存中创建时进行交互的能力，也就是动态的添加方法。内建类型type是内建的基本工厂，它用来生成指定名称、基类以及包含其特性的映射的任何类。

```python
klass = type('MyClass', (object,), {'method': method})

instance = klass()

instance.method()
```
Python中一切都是对象，包括创建对象的类，它实际上也是一个type对象。等价的类创建方法是：

```python

class MyClass(object):
    def method(self):
        return 1
    
instance = MyClass()
instance.method()
```

可以在类中显示的给__metaclass__赋值，所需要的是一个返回是type实例化后的类。如果某个类指定了__metaclass__，那么这个类将从__metaclass__中创建。
__metaclass__的特性必须被设置为:

- 接受和type相同的参数(类名， 一组基类，一个特性映射)
- 返回一个类对象

```python
def metaclass(classname, base_types, func_dicts):
    return type(classname, base_types, func_dicts)

class metaclass(object):
    __metaclass__ = metaclass
    def echo(self, str):
        print str

instance = MyClass()
instance.echo("Hello World")
```

元类的强大特性，可以动态的在已经实例化的类定义上创建许多不同的变化。原则上能不使用就不使用。


使用场景:

- 在框架级别，一个行为在许多类中是强制的时候
- 当一个特殊的行为被添加的目的不是诸如记录日志这样的类提供的功能交互时

引用一句话：
>当你需要动态修改类时，99%的时间里你最好使用上面这两种技术。当然了，其实在99%的时间里你根本就不需要动态修改类
 


 