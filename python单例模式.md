#python 单例模式
[TOC]

## 1、什么是单例模式

单例模式（Singleton Pattern）是一种常用的软件设计模式，该模式的主要目的是确保某一个类只有一个实例存在。当你希望在整个系统中，某个类只能出现一个实例时，单例对象就能派上用场。

比如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个 AppConfig 的类来读取配置文件的信息。如果在程序运行期间，有很多地方都需要使用配置文件的内容，也就是说，很多地方都需要创建 AppConfig 对象的实例，这就导致系统中存在多个 AppConfig 的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似 AppConfig 这样的类，我们希望在程序运行期间只存在一个实例对象。

在 Python 中，我们可以用多种方法来实现单例模式。

## 2、__new__方法实现

首先，我最推荐使用__new__方法来实现单例模式，因为我觉得这种方法是最容易理解的。我们知道__new__方法是类在实例化过程中调用的一个方法，该方法还要在__init__方法之前调用，用于创建类的实例化对象。那么我们是不是可以重写__new__ ，让他判断是否已经调用过该方法了，如果已经调用过了，直接返回上次实例化的对象，那么该类不就一直只存在一个实例化对象了吗，就实现了单例，例子如下。
```python
class Singleton(object):
    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        rerurn cls._instance

class A(Singleton):
    pass

# 类A即为单例类
```

## 3、装饰器实现

我们可以写一个装饰器，来装饰我们需要指定的单例类，这样也可以实现。
```python
def singleton(cls):
    instance = {}
    def wapper():
        if cls not in instance:
            instance[cls] = cls(*args, **kwargs)
        return instance[cls]
    return wapper

@singleton
class C:
    pass

# 类C即为单例类
```

## 4、模块实现

我们知道，在python中，我们可以自己写一个py文件，里面的类或者方法就可以通过模块导入。在python中，用模块导入的类就是一个天然的单例。
```python
# 作为Python模块时是天然的单例模式

#创建一个sington.py文件，内容如下：
class Singleton(object):
    def foo(self):
        pass

mysington = Singleton()

# 运用
from sington import mysington
```

## 5、共享属性实现

创建实例时把所有实例的__dict__指向同一个字典,这样它们都具有相同的属性和方法(类的__dict__存储对象属性)

```python
class Singleton(object):
    _state = {}
    def __new__(cls, *args, **kwargs):
        ob = super(Singleton,cls).__new__(cls, *args, **kwargs)
        ob.__dict__ = cls._state
    return ob

class B(Singleton):
    pass

# 类B即为单例类
```

## 6、元类实现

最后一种方法使用元类实现单例模式，这种方法比较不容易理解，首先需要理解元类是什么，这里就不展开细讲了，有兴趣的同学可以自行学习元类的用法。

```python
class Singleton(type):
    _instacne = {}
    def __call__(cls,*args,**kwargs):
        if cls not in cls._instance:
            cls._instances[cls] = super(Singleton, cls).__call__(*args,**kwargs)
            #以cls为key，cls(*args, **kwargs) 为值放入盛放单例的字典
        return cls._instance[cls]

class MyClass(metaclass=Singleton):
    pass
#MyClass即为单例类
```

好啦，单例模式的实现方法大致就是以上几种了，大家可以学习学习。
