---
title: Python语法最佳实践（二）
date: 2016-04-16 23:57:41
tags: Python
---

# 语法最佳实践


## 装饰器
书缺一页。。。。

### 2.3.1 如何编写装饰器

无参数的通用模式：

```python
def mydecorator(function):
    def _mydecorator(**args, **kw):
        #do something
     
        res = function(*args, **kw)
        
        #do something
        return res
    return _mydecorator
```

有参数的通用模式：

```python
def mydecorator(arg1, arg2):
    def _mydecorator(function):
        def __mydecorator(*args, **kw)
            #do something
     
            res = function(*args, **kw)
        
            #do something
            return res
        return  __mydecorator
    return _mydecorator
```

#### 参数检查

```python
from itertools import izip

def check_args(in_=(), out=(type(None),)):
    def _check_args(function):
        def _check_in(in_args):
            chk_in_args = in_args
            if len(chk_in_args) != len(in_):
                raise  TypeError("argument(in) count is wrong")
            
            for type1, type2 in izip(chk_in_args, in_) :
                if type(type1) != type2:
                    raise TypeError("argument's(in) type is't matched")
        
        def _chech_out(out_args):
            if type(out_args) == tuple:
                if len(out_args) != len(out):
                    raise TypeError("argument(out) count is wrong")
            else:
                if len(out) != 1:
                    raise TypeError("argument(out) count is wrong")
                if not type(out_args) in out:
                    raise TypeError("argument's(out) type is't matched")        
        def __chech_args(*args):
            
            _check_in(args)
            
            res = function(*args)
            
            _chech_out(res)
            
            return res
        return __chech_args
    return _check_args


@check_args((int, int), )
def meth1(i, j):
    print i,j
 
@check_args((str, list), (dict, ))    
def meth2(v_str, v_list):
    return {v_str : 1}
if __name__ == "__main__":
    
    meth1(1, 1)
    meth2("1", [1,2,3])
```

#### 缓存

没看懂

```python
import time
import hashlib
import pickle


cache = {}

def is_obssolete(entry, duration):
    return time.time() - entry["time"] > duration

def compute_key(function, args, kw):
    key = pickle.dumps(function.func_name, args, kw)
    return hashlib.sha1(key).hexdigest()

def memoize(duration=10):
    def _memoize(function):
        def __memoize(*args, **kw):
            key = compute_key(function, args, kw)
            
            if(key in cache and not is_obssolete(cache[key], duration)):
                print "we got a winner"
                return cache[key]["value"]
            
            result =  function(*args, **kw)
        
            cache[key] = {"value":result,
                          "time":time.time()}
            
            return result
        return __memoize
    return _memoize

```

#### 代理

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

class User(object):
    def __init__(self, roles):
        self.roles = roles

class Unauthorized(Exception):
    pass
        

def protect(role):
    def _protect(function):
        def __protect(*args, **kw):
            
            user = globals().get("user")
            if user is None or role not in user.roles:
                raise Unauthorized("I won't tell you")
            return function(*args, **kw)
        return __protect
    return _protect        

@protect("admin")
def premession():
    print "premession ok"

if __name__ == '__main__':

    jack = User(("admin", "user"))
    bill = User(("user",))
```

#### 上下文提供者

在函数运行前后执行一些其他的代码，例如：写一个线程安全的程序。

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

from threading import RLock
lock = RLock()
def synchronized(function):
    def _synchronized(*args, **kw):
        lock.acquire()
        try:
            return function(*args, **kw)
        finally:
            lock.release()
    return _synchronized

if __name__ == '__main__':
    pass
```
----

## with 和 contextlib

程序中很多地方使用try...except...finally来实现异常安全和一些清理代码，应用场景有：
>- 关闭一个文件
- 释放一个锁
- 创建一个烂事的代码补丁
- 在特殊的环境中运行受保护的代码

with语句覆盖和这些场景，可以完美替代try...except...finally。

with协议中依赖的是**\_\_enter\_\_**和**\_\_exit\_\_**两个方法，任何类都一个实现with协议。

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

class File(object):
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
    
    def __enter__(self):
        print 'entering this File class'
        self.fd = open(self.filename, self.mode)
        return self.fd
    
    def __exit__(self, exception_type, 
                 exception_value, 
                 exception_traceback):
        if exception_type is None:
            print 'without error'
        else:
            print "with a error({0})".format(exception_value) 
            
        self.fd.close()
        print 'exiting this File class'

```
**\_\_enter\_\_**和**\_\_exit\_\_**分别对应进入和退出时候的方法，**\_\_exit\_\_**还可以对异常进行捕捉，和try...except...finally的功能一模一样。

```python
if __name__ == '__main__':
    with File("1.txt", 'r') as f:
        print "Hello World"
```

输出信息为：

```python
'''
entering this File class
Hello World
without error
exiting this File class
'''
```

加上异常：

```python
if __name__ == '__main__':
    with File("1.txt", 'r') as f:
        print "Hello World"
        raise TypeError("i'm a bug")
```

输出信息为：

```
'''
entering this File class
Hello World
with a error(i'm a bug)
exiting this File class
Traceback (most recent call last):
  File "E:\mycode\Python\ѧϰ\src\with.py", line 28, in 
    raise TypeError("i'm a bug")
TypeError: i'm a bug
'''
```

**as**的值关键字取决于**\_\_enter\_\_**的返回值，例子中也就是打开的文件描述符。

### contextlib模块

标准库中contextlib模块就是用来增强with的上下文管理。其中最有用的是contextmanager，
这是一个装饰器，被该装饰器修饰的必须是以yield语句分开的**\_\_enter\_\_**和**\_\_exit\_\_**两部分的生成器。
yield之前的是**\_\_enter\_\_**中的内容，之后是**\_\_exit\_\_**中的内容。

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

import contextlib

@contextlib.contextmanager
def talk():
    print "entering"
    try:
        yield 
    except Exception, e:
        print e;
    finally:
        print "leaving"


if __name__ == '__main__':
    with talk():
        print "do something."
        print "do something.."
        print "do something..."
```

输出是：

```python
'''
entering
do something.
do something..
do something...
leaving
'''
```

如果要使用**as**关键字，那么yield要返回一个值，就和**\_\_enter\_\_**的返回值一样。

在contextlib模块还有closing和nested：
有些函数或者类不支持with协议，句柄之类的并不会自己关闭，可以使用contextlib模块中的closing方法，自动调用close()方法

```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

import contextlib

class Work(object):
    def __init__(self):
        print "start to work"
    def close(self):
        print "close and stop to work"

if __name__ == '__main__':
    print "没有异常"
    with contextlib.closing(Work()) as w:
        print "working"
    
    print '-'*30
    print "有异常"
    try:
        with contextlib.closing(Work()) as w:
            print "working"
            raise RuntimeError('error message')
    except Exception, e:
        print e
```

输出为:

```python
'''
没有异常
start to work
working
close and stop to work
------------------------------
有异常
start to work
working
close and stop to work
error message
'''
```
可以看出无论是否执行成功都会调用close()方法。

nested的作用是可以同时打开多个with协议的语法。

在2.7之前：

```python
with contextlib.nested(open('fileToRead.txt', 'r'),
                       open('fileToWrite.txt', 'w')) as (reader, writer):
    writer.write(reader.read())
```

在2.7之后可以直接使用：

```python
with open('fileToRead.txt', 'r') as reader, \
        open('fileToWrite.txt', 'w') as writer:
        writer.write(reader.read())
```

