# python中对多态的理解

[TOC]

## 一、多态

多态是指一类事物有多种形态，比如动物类，可以有猫，狗，猪等等。（一个抽象类有多个子类，因而多态的概念依赖于继承）

```python
import abc
class Animal(metaclass=abc.ABCMeta): #同一类事物:动物
    @abc.abstractmethod
    def talk(self):
        pass

class Cat(Animal): #动物的形态之一:猫
    def talk(self):
        print('say miaomiao')

class Dog(Animal): #动物的形态之二:狗
    def talk(self):
        print('say wangwang')

class Pig(Animal): #动物的形态之三:猪
    def talk(self):
        print('say aoao')
```

## 二、多态性

**注意**：多态与多态性是两种概念

多态性是指具有不同功能的函数可以使用相同的函数名，这样就可以用一个函数名调用不同内容的函数。在面向对象方法中一般是这样表述多态性：向不同的对象发送同一条消息，不同的对象在接收时会产生不同的行为（即方法）。也就是说，每个对象可以用自己的方式去响应共同的消息。所谓消息，就是调用函数，不同的行为就是指不同的实现，即执行不同的函数。

```python
import abc
class Animal(metaclass=abc.ABCMeta): #同一类事物:动物
    @abc.abstractmethod
    def talk(self):
        pass

class Cat(Animal): #动物的形态之一:猫
    def talk(self):
        print('say miaomiao')

class Dog(Animal): #动物的形态之二:狗
    def talk(self):
        print('say wangwang')

class Pig(Animal): #动物的形态之三:猪
    def talk(self):
        print('say aoao')

c = Cat()
d = Dog()
p = Pig()

def func(obj):
    obj.talk()

func(c)
func(d)
func(p)

------------------------------

>>> say miaomiao
>>> say wangwang
>>> say aoao
```

综上可以说，多态性是 : **一个接口,多种实现**

**多态性的好处:**

- 增加了程序的灵活性,以不变应万变，不论对象千变万化，使用者都是同一种形式去调用，如func(obj)
- 增加了程序额可扩展性,通过继承animal类创建了一个新的类，使用者无需更改自己的代码，还是用func(obj)去调用

## 三、鸭子类型

调用不同的子类将会产生不同的行为，而无须明确知道这个子类实际上是什么，这是多态的重要应用场景。而在python中，因为鸭子类型(duck typing)使得其多态不是那么酷。

鸭子类型是动态类型的一种风格。在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由"当前方法和属性的集合"决定。这个概念的名字来源于由James Whitcomb Riley提出的鸭子测试，“鸭子测试”可以这样表述：“当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。”

在鸭子类型中，关注的不是对象的类型本身，而是它是如何使用的。例如，在不使用鸭子类型的语言中，我们可以编写一个函数，它接受一个类型为"鸭子"的对象，并调用它的"走"和"叫"方法。在使用鸭子类型的语言中，这样的一个函数可以接受一个任意类型的对象，并调用它的"走"和"叫"方法。如果这些需要被调用的方法不存在，那么将引发一个运行时错误。任何拥有这样的正确的"走"和"叫"方法的对象都可被函数接受的这种行为引出了以上表述，这种决定类型的方式因此得名。

鸭子类型通常得益于不测试方法和函数中参数的类型，而是依赖文档、清晰的代码和测试来确保正确使用。

Duck typing 这个概念来源于美国印第安纳州的诗人詹姆斯·惠特科姆·莱利（James Whitcomb Riley,1849- 1916）的诗句：”When I see a bird that walks like a duck and swims like a duck and quacks like a duck, I call that bird a duck.”

先上代码，也是来源于网上很经典的案例：

```python
class Duck():
    def walk(self):
         print('I walk like a duck')
    def swim(self):
         print('i swim like a duck')
 
class Person():
    def walk(self):
     　　print('this one walk like a duck') 
    def swim(self):
     　　print('this man swim like a duck')
```

可以很明显的看出，Person类拥有跟Duck类一样的方法，当有一个函数调用Duck类，并利用到了两个方法walk()和swim()。我们传入Person类也一样可以运行，函数并不会检查对象的类型是不是Duck，只要他拥有walk()和swim()方法，就可以正确的被调用。 

再举例，如果一个对象实现了__getitem__方法，那python的解释器就会把它当做一个collection，就可以在这个对象上使用切片，获取子项等方法；如果一个对象实现了__iter__和next方法，python就会认为它是一个iterator，就可以在这个对象上通过循环来获取各个子项。