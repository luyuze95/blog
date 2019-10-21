# python类方法@classmethod与@staticmethod

[TOC]

## 一、@classmethod

### 介绍

与普通的类方法有所不同的是，用@classmethod修饰的类方法不传入self实例本身，而是传入cls，代表这个类自身，可以来调用类的属性，类的方法，实例化对象等。

### 语法

使用的语法也非常简单，直接在类方法上加上装饰器@classmethod即可，另外传入cls参数作为方法的第一个参数。

```python
class A(object):

    @classmethod
    def func(cls):
        pass

```

### 举例

```python
class A(object):
    num = 1

    def func1(self):
        print('func1')

    @classmethod
    def func2(cls):
        print('func2')
        print(cls.num)
        cls().func1()


if __name__ == '__main__':
    A.func2()

------------------------------
>>> func2
>>> 1
>>> func1

```

## 二、@staticmethod

### 介绍

使用@staticmethod修饰的类方法也被称为静态方法，此方法不传入代表实例对象的self参数，并且不强制要求传递任何参数，可以被类直接调用，当然实例化的对象也可以调用。

### 语法

使用时直接在类方法上加上装饰器@staticmethod，参数不需要self，其他参数也是可选。

```python
class B(object):

    @staticmethod
    def func()
        pass

```

### 举例

```python
class B(object):

    @staticmethod
    def func1():
        print('func1')

    @staticmethod
    def func2(a, b):
        print('func2')
        print('a=', a)
        print('b=', b)


if __name__ == '__main__':
    B.func1()
    b = B()
    b.func1()
    B.func2(1, 2)

------------------------------
>>> func1
>>> func1
>>> func2
>>> a= 1
>>> b= 2

```

