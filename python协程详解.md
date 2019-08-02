# python协程详解

[TOC]

## 一、什么是协程

协程又称为微线程，协程是一种用户态的轻量级线程

协程拥有自己的寄存器和栈。协程调度切换的时候，将寄存器上下文和栈都保存到其他地方，在切换回来的时候，恢复到先前保存的寄存器上下文和栈，因此：协程能保留上一次调用状态，每次过程重入时，就相当于进入上一次调用的状态。

协程的好处：

- 1.无需线程上下文切换的开销（还是单线程）

- 2.无需原子操作（一个线程改一个变量，改一个变量的过程就可以称为原子操作）的锁定和同步的开销

- 3.方便切换控制流，简化编程模型

- 4.高并发+高扩展+低成本：一个cpu支持上万的协程都没有问题，适合用于高并发处理

缺点：

- 1.无法利用多核的资源，协程本身是个单线程，它不能同时将单个cpu的多核用上，协程需要和进程配合才能运用到多cpu上（协程是跑在线程上的）

- 2.进行阻塞操作时会阻塞掉整个程序：如io

## 二、了解协程的过程

### 1、yield工作原理

从语法上来看，协程和生成器类似，都是定义体中包含yield关键字的函数。
yield在协程中的用法：

- 在协程中yield通常出现在表达式的右边，例如：datum = yield,可以产出值，也可以不产出--如果yield关键字后面没有表达式，那么生成器产出None。
- 在协程中yield也可能从调用方接受数据，调用方是通过send(datum)的方式把数据提供给协程使用，而不是next(...)函数，通常调用方会把值推送给协程。
- 协程可以把控制器让给中心调度程序，从而激活其他的协程。

所以总体上在协程中把yield看做是控制流程的方式。

先通过一个简单的协程的例子理解：

```python
def simple_demo():
    print("start")
    x = yield
    print("x:", x)

sd = simple_demo()
next(sd)
sd.send(10)

---------------------------

>>> start
>>> x: 10
>>> Traceback (most recent call last):
>>>   File "D:/python_projects/untitled3/xiecheng1.py", line 9, >>> in <module>
>>>     sd.send(10)
>>> StopIteration
```

对上述例子的分析：

yield 的右边没有表达式，所以这里默认产出的值是None
刚开始先调用了next(...)是因为这个时候生成器还没有启动，没有停在yield那里，这个时候也是无法通过send发送数据。所以当我们通过next(...)激活协程后，程序就会运行到x = yield，这里有个问题我们需要注意，x = yield这个表达式的计算过程是先计算等号右边的内容，然后在进行赋值，所以当激活生成器后，程序会停在yield这里，但并没有给x赋值。

当我们调用send方法后yield会收到这个值并赋值给x,而当程序运行到协程定义体的末尾时和用生成器的时候一样会抛出StopIteration异常

如果协程没有通过next(...)激活(同样我们可以通过send(None)的方式激活)，但是我们直接send，会提示如下错误：

```python
def simple_demo():
    print("start")
    x = yield
    print("x:", x)

sd = simple_demo()
# next(sd)
sd.send(10)

---------------------------

>>> Traceback (most recent call last):
>>>   File "D:/python_projects/untitled3/xiecheng1.py", line 9, >>> in <module>
>>>     sd.send(10)
>>> TypeError: can't send non-None value to a just-started generator
```

关于调用next(...)函数这一步通常称为”预激(prime)“协程，即让协程向前执行到第一个yield表达式，准备好作为活跃的协程使用

协程在运行过程中有四个状态：

- GEN_CREATE:等待开始执行
- GEN_RUNNING:解释器正在执行，这个状态一般看不到
- GEN_SUSPENDED:在yield表达式处暂停
- GEN_CLOSED:执行结束

通过下面例子来查看协程的状态：

```python
>>> from inspect import getgeneratorstate
>>> def simple_demo(a):
    print("start: a = ", a)
    b = yield a
    print("b = ", b)
    c = yield a + b
    print("c = ", c)

    
>>> sd = simple_demo(2)
>>> print(getgeneratorstate(sd))
GEN_CREATED
>>> next(sd)  # 预激协程，使它走到第一个yield处，因为第一个yield处有yield值a，所以返回a的值，然后在此yield处阻塞
start: a =  2
2
>>> print(getgeneratorstate(sd))
GEN_SUSPENDED
>>> sd.send(3) # 发送3，进入协程接着上一次阻塞的yield处执行，yield接收参数3赋值给b，到下一个yield处返回a+b的值，然后在此yield处再次阻塞，等待下次send值
b =  3
5
>>> sd.send(4) # 同上一次send过程，到此结束抛异常
c =  4
Traceback (most recent call last):
  File "<pyshell#8>", line 1, in <module>
    sd.send(4)
StopIteration
>>> print(getgeneratorstate(sd))
GEN_CLOSED
```

可以通过注释理解这个例子。

接着再通过一个计算平均值的例子来继续理解：

```python
>>> def averager():
	total = 0.0
	count = 0
	average = None
	while True:
		term = yield average
		total += term
		count += 1
		average = total/count

		
>>> avg = averager()
>>> next(avg)
>>> avg.send(10)
10.0
>>> avg.send(30)
20.0
>>> avg.send(40)
26.666666666666668
```

这里是一个死循环，只要不停send值给协程，可以一直计算下去。
通过上面的几个例子我们发现，我们如果想要开始使用协程的时候必须通过next(...)方式激活协程，如果不预激，这个协程就无法使用，如果哪天在代码中遗忘了那么就出问题了，所以有一种预激协程的装饰器，可以帮助我们干这件事。

### 2、预激协程的装饰器

下面是预激装饰器的演示例子：

```python
from functools import wraps

def coroutine(func):
    @wraps(func)
    def primer(*args,**kwargs):
        gen = func(*args,**kwargs)
        next(gen)
        return gen
    return primer

@coroutine
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield average
        total += term
        count += 1
        average = total/count

coro_avg = averager()
from inspect import getgeneratorstate
print(getgeneratorstate(coro_avg))
print(coro_avg.send(10))
print(coro_avg.send(30))
print(coro_avg.send(5))

---------------------------

>>> GEN_SUSPENDED
>>> 10.0
>>> 20.0
>>> 15.0
```

关于预激，在使用yield from句法调用协程的时候，会自动预激活，这样其实与我们上面定义的coroutine装饰器是不兼容的，在python3.4里面的asyncio.coroutine装饰器不会预激协程，因此兼容yield from

### 3、终止协程和异常处理

协程中未处理的异常会向上冒泡，传给 next 函数或 send 方法的调用方（即触发协程的对象）。

继续使用上面averager的例子
```python
>>> coro_avg = averager()
>>> coro_avg.send(40)
40.0
>>> coro_avg.send(50)
45.0
>>> coro_avg.send('spam')
Traceback (most recent call last):
...
TypeError: unsupported operand type(s) for +=: 'float' and 'str'
>>> coro_avg.send(60)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
StopIteration
```

由于在协程内没有处理异常，协程会终止。如果试图重新激活协程，会抛出StopIteration 异常。

 

从 Python 2.5 开始，客户代码可以在生成器对象上调用两个方法：throw 和 close，显式地把异常发给协程。

- 1：generator.throw(exc_type[, exc_value[, traceback]])

使生成器在暂停的 yield 表达式处抛出指定的异常。如果生成器处理了抛出的异常，代码会向前执行到下一个 yield 表达式，而产出的值会成为调用 generator.throw方法得到的返回值。如果生成器没有处理抛出的异常，异常会向上冒泡，传到调用方的上下文中。

- 2：generator.close()

使生成器在暂停的 yield 表达式处抛出 GeneratorExit 异常。如果生成器没有处理这个异常，或者抛出了 StopIteration 异常（通常是指运行到结尾），调用方不会报错。如果收到 GeneratorExit 异常，生成器一定不能产出值，否则解释器会抛出RuntimeError 异常。生成器抛出的其他异常会向上冒泡，传给调用方。

示例如下：

```python
from inspect import getgeneratorstate
class DemoException(Exception):
    """为这次演示定义的异常类型。"""
    pass
    
def demo_exc_handling():
    print('-> coroutine started')
    while True:
        try:
            x = yield
        except DemoException:
            print('*** DemoException handled. Continuing...')
        else:
            print('-> coroutine received: {!r}'.format(x))
    raise RuntimeError('This line should never run.')
    
>>> exc_coro = demo_exc_handling()
>>> next(exc_coro)
-> coroutine started
>>> exc_coro.send(11)
-> coroutine received: 11
>>> exc_coro.send(22)
-> coroutine received: 22

>>> exc_coro.throw(DemoException)
*** DemoException handled. Continuing...
>>> getgeneratorstate(exc_coro)
'GEN_SUSPENDED'
>>> exc_coro.close()
>>> getgeneratorstate(exc_coro)
'GEN_CLOSED'
```

### 4、让协程返回值

在Python2中，生成器函数中的return不允许返回附带返回值。在Python3中取消了这一限制，因而允许协程可以返回值：

```python
from collections import namedtuple
Result = namedtuple('Result', 'count average')

def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count += 1
        average = total/count
    return Result(count, average)
    
>>> coro_avg = averager()
>>> next(coro_avg)
>>> coro_avg.send(10)
>>> coro_avg.send(30)
>>> coro_avg.send(6.5)
>>> coro_avg.send(None)
Traceback (most recent call last):
...
StopIteration: Result(count=3, average=15.5)    
```

发送 None 会终止循环，导致协程结束，返回结果。一如既往，生成器对象会抛出StopIteration 异常。异常对象的 value 属性保存着返回的值。

注意，return 表达式的值会偷偷传给调用方，赋值给 StopIteration 异常的一个属性。这样做有点不合常理，但是能保留生成器对象的常规行为——耗尽时抛出StopIteration 异常。如果需要接收返回值，可以这样：

```python
>>> try:
...    coro_avg.send(None)
... except StopIteration as exc:
...    result = exc.value
...
>>> result
Result(count=3, average=15.5)
```

获取协程的返回值要绕个圈子，可以使用Python3.3引入的yield from获取返回值。yield from 结构会在内部自动捕获 StopIteration 异常。这种处理方式与 for 循环处理 StopIteration 异常的方式一样。对 yield from 结构来说，解释器不仅会捕获 StopIteration 异常，还会把value 属性的值变成 yield from 表达式的值。

### 5、yield from的使用

yield from 是 Python3.3 后新加的语言结构。在其他语言中，类似的结构使用 await 关键字，这个名称好多了，因为它传达了至关重要的一点：在生成器 gen 中使用 yield from subgen() 时，subgen 会获得控制权，把产出的值传给 gen 的调用方，即调用方可以直接控制 subgen。与此同时，gen 会阻塞，等待 subgen 终止。

yield from 可用于简化 for 循环中的 yield 表达式。例如：

```python
>>> def gen():
... for c in 'AB':
...     yield c
... for i in range(1, 3):
...     yield i
...
>>> list(gen())
['A', 'B', 1, 2]
```

可以改为

```python
>>> def gen():
...     yield from 'AB'
...     yield from range(1, 3)
...
>>> list(gen())
['A', 'B', 1, 2]
```

yield from x 表达式对 x 对象所做的第一件事是，调用 iter(x)，从中获取迭代器。因此，x 可以是任何可迭代的对象。
 
如果 yield from 结构唯一的作用是替代产出值的嵌套 for 循环，这个结构很有可能不会添加到 Python 语言中。

yield from 的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来，这样二者可以直接发送和产出值，还可以直接传入异常，而不用在位于中间的协程中添加大量处理异常的样板代码。有了这个结构，协程可以通过以前不可能的方式委托职责。

PEP 380 使用了一些yield from使用的专门术语：

- 委派生成器：包含 yield from <iterable> 表达式的生成器函数；

- 子生成器：从 yield from 表达式中 <iterable> 部分获取的生成器；

- 调用方：调用委派生成器的客户端代码；

委派生成器在 yield from 表达式处暂停时，调用方可以直接把数据发给子生成器，子生成器再把产出的值发给调用方。子生成器返回之后，解释器会抛出StopIteration 异常，并把返回值附加到异常对象上，此时委派生成器会恢复。

下面是一个求平均身高和体重的示例代码:

```python
from collections import namedtuple

Result = namedtuple('Result', 'count average')

# 子生成器
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        # main 函数发送数据到这里 
        print("in averager, before yield")
        term = yield
        if term is None: # 终止条件
            break
        total += term
        count += 1
        average = total/count

    print("in averager, return result")
    return Result(count, average) # 返回的Result 会成为grouper函数中yield from表达式的值


# 委派生成器
def grouper(results, key):
     # 这个循环每次都会新建一个averager 实例，每个实例都是作为协程使用的生成器对象
    while True:
        print("in grouper, before yield from averager, key is ", key)
        results[key] = yield from averager()
        print("in grouper, after yield from, key is ", key)


# 调用方
def main(data):
    results = {}
    for key, values in data.items():
        # group 是调用grouper函数得到的生成器对象
        group = grouper(results, key)
        print("\ncreate group: ", group)
        next(group) #预激 group 协程。
        print("pre active group ok")
        for value in values:
            # 把各个value传给grouper 传入的值最终到达averager函数中；
            # grouper并不知道传入的是什么，同时grouper实例在yield from处暂停
            print("send to %r value %f now"%(group, value))
            group.send(value)
        # 把None传入groupper，传入的值最终到达averager函数中，导致当前实例终止。然后继续创建下一个实例。
        # 如果没有group.send(None)，那么averager子生成器永远不会终止，委派生成器也永远不会在此激活，也就不会为result[key]赋值
        print("send to %r none"%group)
        group.send(None)
    print("report result: ")
    report(results)


# 输出报告
def report(results):
    for key, result in sorted(results.items()):
        group, unit = key.split(';')
        print('{:2} {:5} averaging {:.2f}{}'.format(result.count, group, result.average, unit))


data = {
    'girls;kg':[40, 41, 42, 43, 44, 54],
    'girls;m': [1.5, 1.6, 1.8, 1.5, 1.45, 1.6],
    'boys;kg':[50, 51, 62, 53, 54, 54],
    'boys;m': [1.6, 1.8, 1.8, 1.7, 1.55, 1.6],
}

if __name__ == '__main__':
    main(data) 
```

grouper 发送的每个值都会经由 yield from 处理，通过管道传给 averager 实例。grouper 会在 yield from 表达式处暂停，等待 averager 实例处理客户端发来的值。averager 实例运行完毕后，返回的值绑定到 results[key] 上。while 循环会不断创建 averager 实例，处理更多的值。

外层 for 循环重新迭代时会新建一个 grouper 实例，然后绑定到 group 变量上。前一个 grouper 实例（以及它创建的尚未终止的 averager 子生成器实例）被垃圾回收程序回收。

代码结果如下：

```python
create group:  <generator object grouper at 0x7f34ce8458e0>
in grouper, before yield from averager, key is  girls;kg
in averager, before yield
pre active group ok
send to <generator object grouper at 0x7f34ce8458e0> value 40.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 41.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 42.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 43.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 44.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 54.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> none
in averager, return result
in grouper, after yield from, key is  girls;kg
in grouper, before yield from averager, key is  girls;kg
in averager, before yield

create group:  <generator object grouper at 0x7f34ce845678>
in grouper, before yield from averager, key is  girls;m
in averager, before yield
pre active group ok
send to <generator object grouper at 0x7f34ce845678> value 1.500000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845678> value 1.600000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845678> value 1.800000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845678> value 1.500000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845678> value 1.450000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845678> value 1.600000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845678> none
in averager, return result
in grouper, after yield from, key is  girls;m
in grouper, before yield from averager, key is  girls;m
in averager, before yield

create group:  <generator object grouper at 0x7f34ce845620>
in grouper, before yield from averager, key is  boys;kg
in averager, before yield
pre active group ok
send to <generator object grouper at 0x7f34ce845620> value 50.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845620> value 51.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845620> value 62.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845620> value 53.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845620> value 54.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845620> value 54.000000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce845620> none
in averager, return result
in grouper, after yield from, key is  boys;kg
in grouper, before yield from averager, key is  boys;kg
in averager, before yield

create group:  <generator object grouper at 0x7f34ce8458e0>
in grouper, before yield from averager, key is  boys;m
in averager, before yield
pre active group ok
send to <generator object grouper at 0x7f34ce8458e0> value 1.600000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 1.800000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 1.800000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 1.700000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 1.550000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> value 1.600000 now
in averager, before yield
send to <generator object grouper at 0x7f34ce8458e0> none
in averager, return result
in grouper, after yield from, key is  boys;m
in grouper, before yield from averager, key is  boys;m
in averager, before yield
report result: 
 6 boys  averaging 54.00kg
 6 boys  averaging 1.68m
 6 girls averaging 44.00kg
 6 girls averaging 1.58m
```

这个试验想表明的关键一点是，如果子生成器不终止，委派生成器会在yield from 表达式处永远暂停。如果是这样，程序不会向前执行，因为 yield from（与 yield 一样）把控制权转交给客户代码（即，委派生成器的调用方）了。

### 6、yield from的意义

把迭代器当作生成器使用，相当于把子生成器的定义体内联在 yield from 表达式中。此外，子生成器可以执行 return 语句，返回一个值，而返回的值会成为 yield from 表达式的值。

PEP 380 在“Proposal”一节（https://www.python.org/dev/peps/pep-0380/#proposal）分六点说明了 yield from 的行为。这里几乎原封不动地引述，不过把有歧义的“迭代器”一词都换成了“子生成器”，还做了进一步说明。上面的示例阐明了下述四点：

子生成器产出的值都直接传给委派生成器的调用方（即客户端代码）；

使用 send() 方法发给委派生成器的值都直接传给子生成器。如果发送的值是None，那么会调用子生成器的 __next__() 方法。如果发送的值不是 None，那么会调用子生成器的 send() 方法。如果子生成器抛出 StopIteration 异常，那么委派生成器恢复运行。任何其他异常都会向上冒泡，传给委派生成器；

生成器退出时，生成器（或子生成器）中的 return expr 表达式会触发StopIteration(expr) 异常抛出；

yield from 表达式的值是子生成器终止时传给 StopIteration 异常的第一个参数。

yield from 的具体语义很难理解，尤其是处理异常的那两点。在PEP 380 中阐述了 yield from 的语义。还使用伪代码（使用 Python 句法）演示了 yield from 的行为。

若想研究那段伪代码，最好将其简化，只涵盖 yield from 最基本且最常见的用法：yield from 出现在委派生成器中，客户端代码驱动着委派生成器，而委派生成器驱动着子生成器。为了简化涉及到的逻辑，假设客户端没有在委派生成器上调用throw(...) 或 close() 方法。而且假设子生成器不会抛出异常，而是一直运行到终止，让解释器抛出 StopIteration 异常。上面示例中的脚本就做了这些简化逻辑的假设。

下面的伪代码，等效于委派生成器中的 RESULT = yield from EXPR 语句（这里针对的是最简单的情况：不支持 .throw(...) 和 .close() 方法，而且只处理 StopIteration 异常）：

```python
_i = iter(EXPR) 
try:
    _y = next(_i)
except StopIteration as _e:
    _r = _e.value
else:
    while 1:
        _s = yield _y
    try:
        _y = _i.send(_s)
    except StopIteration as _e:
        _r = _e.value
        break
RESULT = _r
```

但是，现实情况要复杂一些，因为要处理客户对 throw(...) 和 close() 方法的调用，而这两个方法执行的操作必须传入子生成器。此外，子生成器可能只是纯粹的迭代器，不支持 throw(...) 和 close() 方法，因此 yield from 结构的逻辑必须处理这种情况。如果子生成器实现了这两个方法，而在子生成器内部，这两个方法都会触发异常抛出，这种情况也必须由 yield from 机制处理。调用方可能会无缘无故地让子生成器自己抛出异常，实现 yield from 结构时也必须处理这种情况。最后，为了优化，如果调用方调用 next(...) 函数或 .send(None) 方法，都要转交职责，在子生成器上调用next(...) 函数；仅当调用方发送的值不是 None 时，才使用子生成器的 .send(...) 方法。

下面的伪代码，是考虑了上述情况之后，语句：RESULT = yield from EXPR的等效代码：

```python
_i = iter(EXPR)
try:
    _y = next(_i)
except StopIteration as _e:
    _r = _e.value
else:
    while 1:
        try:
            _s = yield _y
        except GeneratorExit as _e:
            try:
                _m = _i.close
            except AttributeError:
                pass
            else:
                _m()
            raise _e
        except BaseException as _e:
            _x = sys.exc_info()
            try:
                _m = _i.throw
            except AttributeError:
                raise _e
            else:
                try:
                    _y = _m(*_x)
                except StopIteration as _e:
                    _r = _e.value
                    break
        else:
            try:
                if _s is None:
                    _y = next(_i)
                else:
                    _y = _i.send(_s)
            except StopIteration as _e:
                _r = _e.value
                break
RESULT = _r
```

上面的伪代码中，会预激子生成器。这表明，用于自动预激的装饰器与 yield from 结构不兼容。

## 三、greenlet的使用

python中为实现协程封装了一些非常好用的包，首先介绍greenlet的使用。

Greenlet是python的一个C扩展，旨在提供可自行调度的‘微线程’， 即协程。generator实现的协程在yield value时只能将value返回给调用者(caller)。 而在greenlet中，target.switch（value）可以切换到指定的协程（target）， 然后yield value。greenlet用switch来表示协程的切换，从一个协程切换到另一个协程需要显式指定。

以下例子：

```python
from greenlet import greenlet
def test1():
    print(12)
    gr2.switch()
    print(34)

def test2():
    print(56)
    gr1.switch()
    print(78)

gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()

---------------------------

>>> 12
>>> 56
>>> 34
```

当创建一个greenlet时，首先初始化一个空的栈， switch到这个栈的时候，会运行在greenlet构造时传入的函数（首先在test1中打印 12）， 如果在这个函数（test1）中switch到其他协程（到了test2 打印34），那么该协程会被挂起，等到切换回来（在test2中切换回来 打印34）。当这个协程对应函数执行完毕，那么这个协程就变成dead状态。

对于greenlet，最常用的写法是 x = gr.switch(y)。 这句话的意思是切换到gr，传入参数y。当从其他协程（不一定是这个gr）切换回来的时候，将值付给x。

```python
import greenlet
def test1(x, y):
    z = gr2.switch(x+y)
    print 'test1 ', z

def test2(u):
    print 'test2 ', u
    gr1.switch(10)

gr1 = greenlet.greenlet(test1)
gr2 = greenlet.greenlet(test2)
print gr1.switch("hello", " world")

---------------------------

>>> 'test2 ' 'hello world'
>>> 'test1 ' 10
>>> None
```

上面的例子，第12行从main greenlet切换到了gr1，test1第3行切换到了gs2，然后gr1挂起，第8行从gr2切回gr1时，将值（10）返回值给了 z。

使用greenlet需要注意一下三点：
- 第一：greenlet创生之后，一定要结束，不能switch出去就不回来了，否则容易造成内存泄露
- 第二：python中每个线程都有自己的main greenlet及其对应的sub-greenlet ，不能线程之间的greenlet是不能相互切换的
- 第三：不能存在循环引用，这个是官方文档明确说明

## 四、gevent的使用

gevent可以自动捕获I/O耗时操作，来自动切换协程任务。

```python
import gevent

def f1():
    for i in range(5):
        print('run func: f1, index: %s ' % i)
        gevent.sleep(1)

def f2():
    for i in range(5):
        print('run func: f2, index: %s ' % i)
        gevent.sleep(1)

t1 = gevent.spawn(f1)
t2 = gevent.spawn(f2)
gevent.joinall([t1, t2])

------------------------------

>>> run func: f1, index: 0 
>>> run func: f2, index: 0 
>>> run func: f1, index: 1 
>>> run func: f2, index: 1 
>>> run func: f1, index: 2 
>>> run func: f2, index: 2 
>>> run func: f1, index: 3 
>>> run func: f2, index: 3 
>>> run func: f1, index: 4 
>>> run func: f2, index: 4 
```

由图中可以看出，f1和f2是交叉打印信息的，因为在代码执行的过程中，我们人为使用gevent.sleep(0)创建了一个阻塞，gevent在运行到这里时就会自动切换函数切换函数。也可以在执行的时候sleep更长时间，可以发现两个函数基本是同时运行然后各自等待。

关于协程，首先要充分理解协程的实现原理，然后使用现有的轮子greenlet和gevent时才能更加得心应手！
