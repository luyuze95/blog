# python内置模块collections介绍

[TOC]

collections是Python内建的一个集合模块，提供了许多有用的集合类。

## 1、namedtuple

python提供了很多非常好用的基本类型，比如不可变类型tuple，我们可以轻松地用它来表示一个二元向量。

```python
>>> v = (2,3) 
```

我们发现，虽然(2,3)表示出了一个向量的两个坐标，但是，如果没有额外说明，又很难直接看出这个元组是用来表示一个坐标的。

为此定义一个class又小题大做了，这时，namedtuple就派上用场了。

```python
>>> from collections import namedtuple
>>> Vector = namedtuple('Vector', ['x', 'y'])
>>> v = Vector(2,3)
>>> v.x
2
>>> v.y
3

```

namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素。

这样一来，我们用namedtuple可以很方便地定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用，使用十分方便。

我们可以验证创建的Vector对象的类型。

```python
>>> type(v)
<class '__main__.Vector'>

>>> isinstance(v, Vector)
True

>>> isinstance(v, tuple)
True 

```

类似的，如果要用坐标和半径表示一个圆，也可以用namedtuple定义：

```python
>>> Circle = namedtuple('Circle', ['x', 'y', 'r'])
# namedtuple('名称', [‘属性列表’])

```

## 2、deque

在数据结构中，我们知道队列和堆栈是两个非常重要的数据类型，一个先进先出，一个后进先出。在python中，使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低。

deque是为了高效实现插入和删除操作的双向链表结构，非常适合实现队列和堆栈这样的数据结构。

```python
>>> from collections import deque
>>> deq = deque([1, 2, 3])
>>> deq.append(4)
>>> deq
deque([1, 2, 3, 4])
>>> deq.appendleft(5)
>>> deq
deque([5, 1, 2, 3, 4])
>>> deq.pop()
4
>>> deq.popleft()
5
>>> deq
deque([1, 2, 3])

```

deque除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素。

## 3、defaultdict

使用dict字典类型时，如果引用的key不存在，就会抛出KeyError。如果希望Key不存在时，返回一个默认值，就可以用defaultdict。

```python
>>> from collections import defaultdict
>>> dd = defaultdict(lambda: 'defaultvalue')
>>> dd['key1'] = 'a'
>>> dd['key1']
'a'
>>> dd['key2'] # key2未定义，返回默认值
'defaultvalue'

```

注意默认值是调用函数返回的，而函数在创建defaultdict对象时传入。

除了在Key不存在时返回默认值，defaultdict的其他行为跟dict是完全一样的。

## 4、OrderedDict

使用dict时，key是无序的。在对dict做迭代时，我们无法确定key的顺序。

但是如果想要保持key的顺序，可以用OrderedDict。

```python
>>> from collections import OrderedDict
>>> d = dict([('a', 1), ('b', 2), ('c', 3)])
>>> d # dict的Key是无序的
{'a': 1, 'c': 3, 'b': 2}
>>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
>>> od # OrderedDict的Key是有序的
OrderedDict([('a', 1), ('b', 2), ('c', 3)])

```

注意，OrderedDict的key会按照插入的顺序排列，不是key本身排序

```python
>>> od = OrderedDict()
>>> od['z'] = 1
>>> od['y'] = 2
>>> od['x'] = 3
>>> list(od.keys()) # 按照插入的Key的顺序返回
['z', 'y', 'x']

```

OrderedDict可以实现一个FIFO（先进先出）的dict，当容量超出限制时，先删除最早添加的key。

```python
from collections import OrderedDict

class LastUpdatedOrderedDict(OrderedDict):

    def __init__(self, capacity):
        super(LastUpdatedOrderedDict, self).__init__()
        self._capacity = capacity

    def __setitem__(self, key, value):
        containsKey = 1 if key in self else 0
        if len(self) - containsKey >= self._capacity:
            last = self.popitem(last=False)
            print('remove:', last)
        if containsKey:
            del self[key]
            print('set:', (key, value))
        else:
            print('add:', (key, value))
        OrderedDict.__setitem__(self, key, value)

```

## 5、ChainMap

ChainMap可以把一组dict串起来并组成一个逻辑上的dict。ChainMap本身也是一个dict，但是查找的时候，会按照顺序在内部的dict依次查找。

什么时候使用ChainMap最合适？举个例子：应用程序往往都需要传入参数，参数可以通过命令行传入，可以通过环境变量传入，还可以有默认参数。我们可以用ChainMap实现参数的优先级查找，即先查命令行参数，如果没有传入，再查环境变量，如果没有，就使用默认参数。

下面的代码演示了如何查找user和color这两个参数。

```python
from collections import ChainMap
import os, argparse

# 构造缺省参数:
defaults = {
    'color': 'red',
    'user': 'guest'
}

# 构造命令行参数:
parser = argparse.ArgumentParser()
parser.add_argument('-u', '--user')
parser.add_argument('-c', '--color')
namespace = parser.parse_args()
command_line_args = { k: v for k, v in vars(namespace).items() if v }

# 组合成ChainMap:
combined = ChainMap(command_line_args, os.environ, defaults)

# 打印参数:
print('color=%s' % combined['color'])
print('user=%s' % combined['user'])

```

没有任何参数时，打印出默认参数：

```python
$ python3 use_chainmap.py 
color=red
user=guest

```

当传入命令行参数时，优先使用命令行参数：

```python
$ python3 use_chainmap.py -u bob
color=red
user=bob

```

同时传入命令行参数和环境变量，命令行参数的优先级较高：

```python
$ user=admin color=green python3 use_chainmap.py -u bob
color=green
user=bob
```

## 6、Counter

Counter是一个简单的计数器，例如，统计字符出现的个数：

```python
from collections import Counter
>>> s = 'abbcccdddd'
>>> Counter(s)
Counter({'d': 4, 'c': 3, 'b': 2, 'a': 1})

```

Counter实际上也是dict的一个子类。

## 7、小结

collections模块提供了一些有用的集合类，可以根据需要选用。