# python中@property装饰器的使用

[TOC]

## 1、引出问题

在为一个类实例绑定属性时，如果我们直接把属性暴露出去，虽然写起来很简单，但是，没办法检查参数，导致可以把成绩随便改，甚至类型错误都可以。

```python
class Student(object):

    def __init__(self, score):
        self.score = score


if __name__ == '__main__':
    s = Student(100)
    print(s.score)
    s.score = 50
    print(s.score)
    s.score = 'abc'
    print(s.score)

------------------------------

>>> 100
>>> 50
>>> abc

```

## 2、初步改善

上述例子显然不合逻辑，为了限制score的范围，可以通过一个set_score()方法来设置成绩，再通过一个get_score()方法来获取成绩，这样，在set_score()方法里就可以检查参数了。

```python
class Student(object):

    def set_score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer !')
        if value < 0 or value > 100:
            raise ValueError('score must between 0-100 !')
        self._score = value

    def get_score(self):
        return self._score


if __name__ == '__main__':
    s = Student()
    s.set_score(50)
    print(s.get_score())
    s.set_score('abc')

------------------------------

>>> 50
>>> Traceback (most recent call last):
  File "/Users/luyuze/projects/myflask/App/test.py", line 18, in <module>
    s.set_score('abc')
  File "/Users/luyuze/projects/myflask/App/test.py", line 6, in set_score
    raise ValueError('score must be an integer !')
ValueError: score must be an integer !

```

现在，对任意的Student实例进行操作，就不能随心所欲的设置score了。

## 3、使用@property

上面的调用方法虽然已经可以实现相关功能，但是使用起来略显复杂，设置和获取属性都需要通过调用方法来实现，没有直接用属性这么简洁明了。

那么，有没有既能检查参数，又可以用类似属性这样简单的方式来访问类的变量呢？对于追求完美的python来说，这是必须做到的！

下面，我们就使用python内置的装饰器@property来实现。

```python
class Student(object):

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer !')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 - 100 !')
        self._score = value


if __name__ == '__main__':
    s = Student()
    s.score = 50 # 实际转化为s.set_score()
    print(s.score) # 实际转化为s.get_score()
    s.score = 101

------------------------------

>>> 50
>>> Traceback (most recent call last):
  File "/Users/luyuze/projects/myflask/App/test.py", line 21, in <module>
    s.score = 101
  File "/Users/luyuze/projects/myflask/App/test.py", line 13, in score
    raise ValueError('score must between 0 - 100 !')
ValueError: score must between 0 - 100 !

```

## 4、解析@property

@property的实现比较复杂，我们先考察如何使用，把一个getter方法变成属性，只需要加上@property就可以了，此时，@property本身又创建了另一个装饰器@score.setter，负责把一个setter方法变成属性赋值，于是，我们就拥有了如上例子中的属性操作。

注意到这个神奇的@property，我们在对实例属性操作的时候，就知道该属性很可能不是直接暴露的，而是通过getter和setter方法来实现的。

我们还可以定义只读属性，只定义getter方法，不定义setter方法就是一个只读属性。

```python
import datetime


class Student(object):

    @property
    def birth(self):
        return self._birth

    @birth.setter
    def birth(self, value):
        if not isinstance(value, int):
            raise ValueError('birth must be an integer !')
        self._birth = value

    @property
    def age(self):
        return datetime.datetime.now().year - self._birth


if __name__ == '__main__':
    s = Student()
    s.birth = 1995
    print(s.age)
    s.age = 25

------------------------------

>>> 24
>>> Traceback (most recent call last):
  File "/Users/luyuze/projects/myflask/App/test.py", line 25, in <module>
    s.age = 25
AttributeError: can't set attribute

```

上面的birth是可读写属性，而age就是一个只读属性，因为可以根据birth和当前年份计算出来。

## 5、总结

@property广泛应用在类的定义中，可以让调用者写出简短的代码，同时保证对参数进行必要的检查，这样，程序运行时就减少了出错的可能性。
