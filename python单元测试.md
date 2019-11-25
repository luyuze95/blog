# python单元测试

[TOC]

本篇全面介绍一下python中很常用的单元测试框架unitest。

## 1、unitest主要功能模块介绍

unitest主要包含TestCase、TestSuite、TestLoader、TextTestRunner、TextTestResult这几个功能模块。

- TestCase：一个TestCase实例就是一个测试用例，一个测试用例就是一个完整的测试流程，包括测试前环境的搭建，测试代码的执行，以及测试后环境的还原或者销毁。元测试的本质也就在这里，一个测试用例是一个完整的测试单元，可以对某一具体问题进行检查验证。

- TestSuite：多个测试用例集合在一起就是TestSuite，TestSuite也可以嵌套TestSuite。

- TestLoader：TestLoader的作用是将Testcase加载到TestSuite中。

- TextTestRunner：TextTestRunner是用来执行测试用例的，其中的run(test)会执行TestSuite/TestCase中的run(result)方法。

- TextTestResult：TextTestResult用来保存测试结果，其中包括运行了多少测试用例，成功了多少，失败了多少等信息。

整个流程为：写好TestCase，然后由TestLoader加载TestCase到TestSuite，然后由TextTestRunner来运行TestSuite，运行的结果保存在TextTestResult中。

## 2、实例介绍

首先准备几个待测的方法，写在test_func.py中。

```python
def add(a, b):
    return a + b


def multi(a, b):
    return a * b


def lower_str(string):
    return string.lower()


def square(x):
    return x ** 2

```

准备好几个待测的方法之后，为这些方法写一个测试用例,写入our_testcase.py中。

```python
import unittest
from test_func import *


class TestFunc(unittest.TestCase):
    """Test test_func.py"""

    def test_add(self):
        """Test func add"""
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(1, 3))

    def test_multi(self):
        """Test func multi"""
        self.assertEqual(6, multi(2, 3))
        self.assertNotEqual(8, multi(3, 3))

    def test_lower_str(self):
        """Test func lower_str"""
        self.assertEqual("abc", lower_str("ABC"))
        self.assertNotEqual("Dce", lower_str("DCE"))

    def test_square(self):
        """Test func square"""
        self.assertEqual(17, square(4))  # 这里故意设计一个会出错的用例，测试4的平方等于17，实际上并不等于。
        self.assertNotEqual(35, square(6))


if __name__ == '__main__':
    unittest.main()

```

这里写好之后，进入命令行终端，执行python our_testcase.py，执行结果如下。

```python
...F
======================================================================
FAIL: test_square (__main__.TestFunc)
Test func square
----------------------------------------------------------------------
Traceback (most recent call last):
  File "our_testcase.py", line 27, in test_square
    self.assertEqual(17, square(4))
AssertionError: 17 != 16

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (failures=1)

```

这里分析一下这个执行结果。首先能够看到一共运行了4个测试用例，失败了1个，并且给出了失败原因，AssertionError: 17 != 16，这是我们故意留下的错误漏洞，被测试用例测试出来了。

第一行...F中，一个点.代表测试成功，F代表失败，我们的测试结果中，前三个成功了，第四个失败了，总共是四个测试，其余的符号中E代表出错，S代表跳过。

特别说明的一点是，测试的执行顺序跟方法的顺序没有关系，四个测试是随机先后执行的。

每个测试方法编写的时候，都要以test开头，比如test_square，否则是不被unitest识别的。

在unitest.main()中加上verbosity参数可以控制输出的错误报告的详细程序，默认是1，如果设为0，则不输出每一用例的执行结果，即上面的第一行的执行结果内容。如果设为2，则输出详细的执行结果。

修改our_testcase.py中主函数。

```python
if __name__ == '__main__':
    unittest.main(verbosity=2)

```

执行结果如下。

```python
test_add (__main__.TestFunc)
Test func add ... ok
test_lower_str (__main__.TestFunc)
Test func lower_str ... ok
test_multi (__main__.TestFunc)
Test func multi ... ok
test_square (__main__.TestFunc)
Test func square ... FAIL

======================================================================
FAIL: test_square (__main__.TestFunc)
Test func square
----------------------------------------------------------------------
Traceback (most recent call last):
  File "our_testcase.py", line 27, in test_square
    self.assertEqual(17, square(4))
AssertionError: 17 != 16

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (failures=1)

```

可以看到，每一个用例的详细执行情况以及用例名，用例描述均被输出了出来，在测试方法下加代码示例中的"""Doc String"""，在用例执行时，会将该字符串作为此用例的描述，加合适的注释能够使输出的测试报告更加便于阅读。

## 3、组织TestSuite

按照上面的测试方法，我们无法控制用例执行的顺序，这样显然是不合理的，因为在一些测试过程中，我们肯定需要控制先测试某些用例，再测试某些用例，这些用例有先后的因果关系。在这里，我们就需要用到TestSuite。我们添加到TestSuite中的case是会按照添加的顺序执行的。

还有一个问题是，我们现在只有一个测试文件，我们直接执行该文件即可，但如果有多个测试文件，怎么进行组织，总不能一个个文件执行吧，答案也在TestSuite中。

新建一个文件，test_suite.py。

```python
import unittest
from our_testcase import TestFunc

if __name__ == '__main__':
    suite = unittest.TestSuite()
    tests = [TestFunc("test_square"), TestFunc("test_lower_str"), TestFunc("test_multi")]
    suite.addTests(tests)
    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)

```

执行结果如下。

```python
test_square (our_testcase.TestFunc)
Test func square ... FAIL
test_lower_str (our_testcase.TestFunc)
Test func lower_str ... ok
test_multi (our_testcase.TestFunc)
Test func multi ... ok

======================================================================
FAIL: test_square (our_testcase.TestFunc)
Test func square
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/Users/luyuze/projects/test/our_testcase.py", line 27, in test_square
    self.assertEqual(17, square(4))
AssertionError: 17 != 16

----------------------------------------------------------------------
Ran 3 tests in 0.000s

FAILED (failures=1)

```

这样，用例执行的顺序就是按照我们添加进去的顺序来执行的了。

上面使用的是TestSuite的addTests()方法，并直接传入TestCase列表，也有一些其他的方法可以向TestSuite中添加用例。

```python
# 直接用addTest方法添加单个TestCase
suite.addTest(TestMathFunc("test_multi"))


# 使用loadTestFromName,传入模块名.TestCase名，下面俩方法效果相同
suite.addTests(unittest.TestLoader().loadTestsFromName('our_testcase.TestFunc'))
suite.addTests(unittest.TestLoader().loadTestsFromNames(['our_testcase.TestFunc']))


# loadTestsFromTestCase()，传入TestCase
suite.addTests(unittest.TestLoader().loadTestsFromTestCase(TestFunc))

```

用TestLoader的方法是无法对case进行排序的，同时，suite中也可以套suite。

## 4、输出文件

用例组织好了，但是结果只能输出到控制台，这样没办法查看之前的执行记录，我们想将结果输出到文件。

修改test_suite.py。

```python
import unittest
from our_testcase import TestFunc

if __name__ == '__main__':
    suite = unittest.TestSuite()
    tests = [TestFunc("test_square"), TestFunc("test_lower_str"), TestFunc("test_multi")]
    suite.addTests(tests)

    with open('UnitestTextReport.txt', 'a') as f:
        runner = unittest.TextTestRunner(stream=f, verbosity=2)
        runner.run(suite)

```

## 5、测试前后的处理

在之前的测试中，可能会存在这样的问题：如果要在测试之前准备环境，测试完成之后做一些清理怎么办？这里需要用到的是setUp()和tearDown()。

修改our_testcase.py。

```python
import unittest
from test_func import *


class TestFunc(unittest.TestCase):
    """Test test_func.py"""
    
    def setUp(self):
        print("do something before testcase")

    def test_add(self):
        """Test func add"""
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(1, 3))

    def test_multi(self):
        """Test func multi"""
        self.assertEqual(6, multi(2, 3))
        self.assertNotEqual(8, multi(3, 3))

    def test_lower_str(self):
        """Test func lower_str"""
        self.assertEqual("abc", lower_str("ABC"))
        self.assertNotEqual("Dce", lower_str("DCE"))

    def test_square(self):
        """Test func square"""
        self.assertEqual(17, square(4))
        self.assertNotEqual(35, square(6))
        
    def tearDownClass(self):
        print("do something after testcase")


if __name__ == '__main__':
    unittest.main(verbosity=2)


```

执行结果：

```python
test_add (__main__.TestFunc)
Test func add ... do something before testcase
do something after testcase
ok
test_lower_str (__main__.TestFunc)
Test func lower_str ... do something before testcase
do something after testcase
ok
test_multi (__main__.TestFunc)
Test func multi ... do something before testcase
do something after testcase
ok
test_square (__main__.TestFunc)
Test func square ... do something before testcase
do something after testcase
FAIL

======================================================================
FAIL: test_square (__main__.TestFunc)
Test func square
----------------------------------------------------------------------
Traceback (most recent call last):
  File "our_testcase.py", line 30, in test_square
    self.assertEqual(17, square(4))
AssertionError: 17 != 16

----------------------------------------------------------------------
Ran 4 tests in 0.001s

FAILED (failures=1)

```

可以发现setUp()和tearDown()在每个case前后都执行了一次。如果要在所有case执行之前和所有case执行之后准备和清理环境，我们可以使用setUpClass() 与 tearDownClass()。

```python
class TestFunc(unittest.TestCase):
    """Test test_func.py"""

    @classmethod
    def setUpClass(cls):
        print "This setUpClass() method only called once."

    @classmethod
    def tearDownClass(cls):
        print "This tearDownClass() method only called once too."

```

## 6、跳过case

如果我们临时想要跳过某个case不执行，unitest也有相应的方法。

- 1、skip装饰器

```python
# -*- coding: utf-8 -*-

import unittest
from test_func import *


class TestFunc(unittest.TestCase):
    """Test test_func.py"""

    @unittest.skip('do not run this case')
    def test_add(self):
        """Test func add"""
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(1, 3))

    def test_multi(self):
        """Test func multi"""
        self.assertEqual(6, multi(2, 3))
        self.assertNotEqual(8, multi(3, 3))

    def test_lower_str(self):
        """Test func lower_str"""
        self.assertEqual("abc", lower_str("ABC"))
        self.assertNotEqual("Dce", lower_str("DCE"))

    def test_square(self):
        """Test func square"""
        self.assertEqual(17, square(4))
        self.assertNotEqual(35, square(6))


if __name__ == '__main__':
    unittest.main(verbosity=2)

```

执行结果：

```python
test_add (__main__.TestFunc)
Test func add ... skipped 'do not run this case'
test_lower_str (__main__.TestFunc)
Test func lower_str ... ok
test_multi (__main__.TestFunc)
Test func multi ... ok
test_square (__main__.TestFunc)
Test func square ... FAIL

======================================================================
FAIL: test_square (__main__.TestFunc)
Test func square
----------------------------------------------------------------------
Traceback (most recent call last):
  File "our_testcase.py", line 28, in test_square
    self.assertEqual(17, square(4))
AssertionError: 17 != 16

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (failures=1, skipped=1)

```

结果显示为，总共执行4个测试，1个失败，1个被跳过。

skip装饰器一共有三个 unittest.skip(reason)、unittest.skipIf(condition, reason)、unittest.skipUnless(condition, reason)，skip无条件跳过，skipIf当condition为True时跳过，skipUnless当condition为False时跳过。

- 2、TestCase.skipTest()方法

```python
class TestFunc(unittest.TestCase):
    """Test test_func.py"""

    def test_add(self):
        """Test func add"""
        self.skipTest("do not run this case")
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(1, 3))

```

效果与第一种是一样的。