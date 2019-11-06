# python高阶函数的使用

[TOC]

## 1、map

Python内建了map()函数，map()函数接受两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每一个元素上，并把结果作为新的Iterator返回。

举例说明，比如我们有一个函数f(x)=x*2，要把这个函数作用在一个list[1, 2, 3, 4, 5, 6, 7, 8, 9]上，就可以用map()实现。

```python
>>> def f(x):
...     return x*2
... 
>>> r = map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> list(r)
[2, 4, 6, 8, 10, 12, 14, 16, 18]

```

map()传入的第一个参数是f，即函数对象本身。由于结果r是一个Iterator，Iterator是惰性序列，因此通过list()函数让它把整个序列都计算出来并返回一个list。

你可能会想，不需要map()函数，写一个循环，也可以计算出结果：

```python
L = []
for i in [1, 2, 3, 4, 5, 6, 7, 8, 9]:
    L.append(f(i))
print(L)

```

的确也可以，但是，从上面的循环代码，能一眼看明白”把f(x)作用在list的每一个元素并把结果生成一个新的list“吗？

所以，map()作为高阶函数，事实上它把运算规则抽象了，因此，我们不但可以计算简单的f(x)=x*2，还可以计算任意复杂的函数，比如把这个list所有的数字转为字符串：

```python
>>> list(map(str,[1, 2, 3, 4, 5, 6, 7, 8, 9]))
['1', '2', '3', '4', '5', '6', '7', '8', '9']

```

只需要一行代码就可以搞定。

## 2、reduce

再看reduce的用法。reduce是把一个函数作用在一个序列[x1, x2, x3……]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累计计算。简单来说，就是先计算x1和x2的结果，再拿结果与x3计算，依次类推。

```python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1,x2), x3), x4)

```

比如说一个序列求和，就可以用reduce实现。

```python
>>> from functools import reduce
>>> def add(x, y):
...     return x + y
... 
>>> reduce(add, [1, 3, 5, 7, 9])
25

```

当然求和运算可以直接使用python内建函数sum()，没必要动用reduce。

但是如果要把序列[1, 3, 5, 7, 9]变换为整数13579，reduce就可以派上用场：

```python
>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
... 
>>> reduce(fn, [1, 3, 5, 7, 9])
13579

```

这个例子本身没多大用处，但是，如果考虑到字符串str也是一个序列，对上面的例子稍加改动，配合map，我们就可以写出把str转换为int的函数：

```python
>>> from functools import reduce
>>> def fn(x, y):
...     return x * 10 + y
...
>>> def char2num(s):
...     digits = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}
...     return digits[s]
...
>>> reduce(fn, map(char2num, '13579'))
13579

```

整理成一个str2int的函数就是：

```python
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def str2int(s):
    def fn(x, y):
        return x * 10 + y
    def char2num(s):
        return DIGITS[s]
    return reduce(fn, map(char2num, s))

```

还可以用lambda函数进一步简化成：

```python
from functools import reduce

DIGITS = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}

def char2num(s):
    return DIGITS[s]

def str2int(s):
    return reduce(lambda x, y: x * 10 + y, map(char2num, s))

```

也就是说，假设python没有提供int()函数，你完全可以自己写一个把字符串转化为整数的函数，而且只需要几行代码。

## 3、filter

python内建的filter()函数用于过滤序列。

和map()类似，filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每一个元素，然后根据返回值是True还是False决定保留还是丢弃该元素。

例如，在一个list中，删掉偶数，只保留奇数，可以这么写：

```python
def is_odd(n):
    return n % 2 == 1

list(filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15]))
# 结果: [1, 5, 9, 15]

```

把一个序列中的空字符串删掉，可以这么写：

```python
def not_empty(s):
    return s and s.strip()

list(filter(not_empty, ['A', '', 'B', None, 'C', '  ']))
# 结果: ['A', 'B', 'C']

```

可见用filter()这个高阶函数，关键在于正确实现一个筛选函数。

注意到filter()函数返回的是一个Iterator，也就是一个惰性序列，所有要强迫filter()完成计算结果，需要用list()函数获得所有结果并返回list。

## 4、sorted

排序也是在程序中经常用到的算法。无论使用冒泡排序还是快速排序，排序的核心是比较两个元素的大小。如果是数字，我们可以直接比较，但如果是字符串或者两个dict呢？直接比较数学上的大小是没有意义的，因此，比较的过程必须通过函数抽象出来。

Python内置的sorted()函数就可以对list进行排序：

```python
>>> sorted([36, 5, -12, 9, -21])
[-21, -12, 5, 9, 36]

```

此外，sorted()函数也是一个高阶函数，它还可以接收一个key函数来实现自定义的排序，例如按绝对值大小排序：

```python
>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]

```

key指定的函数将作用于list的每一个元素上，并根据key函数返回的结果进行排序。

我们再看一个字符串排序的例子：

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'])
['Credit', 'Zoo', 'about', 'bob']

```

默认情况下，对字符串排序，是按照ASCII的大小比较的，由于'Z' < 'a'，结果，大写字母Z会排在小写字母a的前面。

现在，我们提出排序应该忽略大小写，按照字母序排序。要实现这个算法，不必对现有代码大加改动，只要我们能用一个key函数把字符串映射为忽略大小写排序即可。忽略大小写来比较两个字符串，实际上就是先把字符串都变成大写（或者都变成小写），再比较。

这样，我们给sorted传入key函数，即可实现忽略大小写的排序：

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
['about', 'bob', 'Credit', 'Zoo']

```

要进行反向排序，不必改动key函数，可以传入第三个参数reverse=True：

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']

```

## 5、小结

高阶函数的抽象能力是非常强大的，在代码中善于利用这些高阶函数，可以使我们的代码变得简洁明了。