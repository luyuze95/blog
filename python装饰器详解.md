# python 装饰器详解
[toc]

## 1、闭包

要想了解装饰器，首先要了解一个概念，闭包。什么是闭包，一句话说就是，在函数中再嵌套一个函数，并且引用外部函数的变量，这就是一个闭包了。光说没有概念，直接上一个例子。

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

print(outer(6)(5))
-----------------------------
>>>11
```
如代码所示，在outer函数内，又定义了一个inner函数，并且inner函数又引用了外部函数outer的变量x，这就是一个闭包了。在输出时，outer(6)(5),第一个括号传进去的值返回inner函数，其实就是返回6 + y，所以再传第二个参数进去，就可以得到返回值，6 + 5。

## 2、装饰器
接下来就讲装饰器，其实装饰器就是一个闭包，装饰器是闭包的一种应用。什么是装饰器呢，简言之，python装饰器就是用于拓展原来函数功能的一种函数，这个函数的特殊之处在于它的返回值也是一个函数，使用python装饰器的好处就是在不用更改原函数的代码前提下给函数增加新的功能。使用时，再需要的函数前加上@demo即可。
```python
def debug(func):
    def wrapper():
        print("[DEBUG]: enter {}()".format(func.__name__))
        return func()
    return wrapper

@debug
def hello():
    print("hello")

hello()
-----------------------------
>>>[DEBUG]: enter hello()
>>>hello
```
例子中的装饰器给函数加上一个进入函数的debug模式，不用修改原函数代码就完成了这个功能，可以说是很方便了。

## 3、带参数的装饰器
上面例子中的装饰器是不是功能太简单了，那么装饰器可以加一些参数吗，当然是可以的，另外装饰的函数当然也是可以传参数的。
```python
def logging(level):
    def outwrapper(func):
        def wrapper(*args, **kwargs):
            print("[{0}]: enter {1}()".format(level, func.__name__))
            return func(*args, **kwargs)
        return wrapper
    return outwrapper

@logging(level="INFO")
def hello(a, b, c):
    print(a, b, c)

hello("hello,","good","morning")
-----------------------------
>>>[INFO]: enter hello()
>>>hello, good morning
```
如上，装饰器中可以传入参数，先形成一个完整的装饰器，然后再来装饰函数，当然函数如果需要传入参数也是可以的，用不定长参数符号就可以接收，例子中传入了三个参数。

## 4、类装饰器
装饰器也不一定只能用函数来写，也可以使用类装饰器，用法与函数装饰器并没有太大区别，实质是使用了类方法中的__call__魔法方法来实现类的直接调用。
```python
class logging(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("[DEBUG]: enter {}()".format(self.func.__name__))
        return self.func(*args, **kwargs)

@logging
def hello(a, b, c):
    print(a, b, c)

hello("hello,","good","morning")
-----------------------------
>>>[DEBUG]: enter hello()
>>>hello, good morning
```
类装饰器也是可以带参数的，如下实现
```python
class logging(object):
    def __init__(self, level):
        self.level = level

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print("[{0}]: enter {1}()".format(self.level, func.__name__))
            return func(*args, **kwargs)
        return wrapper
        
@logging(level="TEST")
def hello(a, b, c):
    print(a, b, c)

hello("hello,","good","morning")
-----------------------------
>>>[TEST]: enter hello()
>>>hello, good morning
```

好了，如上就是装饰器的一些概念和大致的用法啦，想更深入的了解装饰器还是需要自己在平时的练习和应用中多体会，本篇只是给出一个概念，谢谢~
