#python 迭代器和生成器
[TOC]

## 一、迭代器

### 1、什么是迭代器

说迭代器之前有个相关的名词需要介绍：
可迭代对象：只要定义了__iter__()方法，我们就说该对象是可迭代对象，并且可迭代对象能提供迭代器。

在Python中，for循环可以用于Python中的任何类型，包括列表、元祖等等，实际上，for循环可用于任何“可迭代对象”，这其实就是迭代器。

迭代器是一个实现了迭代器协议的对象，Python中的迭代器协议就是有__next__方法的对象会前进到下一结果，而到一系列结果的末尾，则会引发StopIteration。任何这类的对象在Python中都可以用for循环或其他遍历工具迭代，迭代工具内部会在每次迭代时调用next方法，并且捕捉StopIteration异常来确定何时离开。

### 2、为什么要用迭代器
使用迭代器一个显而易见的好处就是：每次只从对象中读取一条数据，不会造成内存的过大开销。

比如要逐行读取一个文件的内容，利用readlines()方法，我们可以这么写：

```python
for line in open("test.txt").readlines():
    print line
```

这样虽然可以工作，但不是最好的方法。因为他实际上是把文件一次加载到内存中，然后逐行打印。当文件很大时，这个方法的内存开销就很大了。

利用file的迭代器，我们可以这样写：

```python
for line in open("test.txt"):   #use file iterators
    print line
```

这是最简单也是运行速度最快的写法，他并没显式的读取文件，而是利用迭代器每次读取下一行。

### 3、如何使用迭代器

使用内建的工厂函数iter(iterable)可以获取迭代器对象（对象包含__iter__方法即可迭代，__iter__方法返回一个迭代器）：

```python
>>> lst = range(5)
>>> it = iter(lst)
>>> it
<listiterator object at 0x0000000001E43390>
```

使用next()方法访问下一个元素

```python
>>> it.next()
0
>>> it.next()
1
>>> it.next()
2
```

python处理迭代器越界是抛出StopIteration异常

```python
>>> it.next()
3
>>> it.next
<method-wrapper 'next' of listiterator object at 0x01A63110>
>>> it.next()
4
>>> it.next()
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
StopIteration
```

了解了StopIteration，可以使用迭代器进行遍历了：

```python
lst = range(5)
it = iter(lst)
try:
    while True:
        val = it.next()
        print val
except StopIteration:
    pass
```

for语法糖：

```python
>>> lst = range(5)
>>> for i in lst:
...     print i
...
0
1
2
3
4
```

## 二、生成器

### 1、什么是生成器

生成器函数在Python中与迭代器协议的概念联系在一起。简而言之，包含yield语句的函数会被特地编译成生成器。当函数被调用时，他们返回一个生成器对象，这个对象支持迭代器接口。函数也许会有个return语句，但它的作用是用来yield产生值的。

### 2、为什么要使用生成器

Python使用生成器对延迟操作提供了支持。所谓的延迟操作，是指在需要的时候才产生结果，而不是立即产生结果。所以生成器也有了如下的好处：
- 1、节省资源消耗，和声明序列不同的是生成器在不使用的时候几乎不占内存，也没有声明计算过程！
- 2、使用的时候，生成器是随用随生成，用完即刻释放，非常高效！
- 3、可在单线程下实现并发运算处理效果。

### 3、如何使用生成器

使用斐波那契数列的例子来说明一下：

```python
def fab(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n+ 1
#执行
for n in fab(5):
    print n
    
1
1
2
3
5
```

该函数通过yield关键字来返回结果，每一次迭代就停止于yield语句处，一直到下一次迭代。

生成器也是一种迭代器，简单地讲，yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator，调用 fab(5) 不会执行 fab 函数，而是返回一个 iterable 对象！在 for 循环执行时，每次循环都会执行 fab 函数内部的代码，执行到 yield b 时，fab 函数就返回一个迭代值，下次迭代时，代码从 yield b 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到 yield。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

也可以手动调用 fab(5) 的 next() 方法（因为 fab(5) 是一个 generator 对象，该对象具有 next() 方法），这样我们就可以更清楚地看到 fab 的执行流程：

```python
>>> f = fab(3)
>>> f.next()
1
>>> f.next()
1
>>> f.next()
2
>>> f.next()
 
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
StopIteration
```

### 4、return的作用

在一个生成器中，如果没有return，则默认执行到函数完毕；如果遇到return,如果在执行过程中 return，则直接抛出 StopIteration 终止迭代。例如：

```python
def read_file(fpath): 
    BLOCK_SIZE = 1024 
    with open(fpath, 'rb') as f: 
        while True: 
            block = f.read(BLOCK_SIZE) 
            if block: 
                yield block 
            else: 
                return
```

如果直接对文件对象调用 read() 方法，会导致不可预测的内存占用。好的方法是利用固定长度的缓冲区来不断读取文件内容。通过 yield，我们不再需要编写读文件的迭代类，就可以轻松实现文件读取。