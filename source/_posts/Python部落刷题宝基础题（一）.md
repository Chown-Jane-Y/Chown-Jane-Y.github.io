---
title: Python部落刷题宝基础题（一）
date: 2017-03-01 09:28:13
tags: Python
categories: Python部落刷题宝
description: Python部落刷题宝里的基础题。
---

下面是Python入门的题，打好基础还是很重要的~
https://python.freelycode.com/examination/exercise/do/

<!-- more -->

### len(bin(5))的值为\_____
>答案：5

解答：`bin(5)` -> 0b101 ; len('0b101') -> 5

**[文档解释]**

- **bin(x)**
Convert an integer number to a binary string. The result is a valid Python expression. If x is not a Python int object, it has to define an __index__() method that returns an integer.

**[Example]**
1. 将一个整型数字转换为二进制字符串
```python
>>> b = bin(3) 
>>> b
'0b11'
>>> type(b) #获取b的类型
<class 'str'>
```

2. 如果参数x不是个整数，则必须定义一个`__index()__`方法，并且方法返回值必须为整数。

3. 如果不是整数，则报错：
```
>>> class A:
    pass

>>> a = A()
>>> bin(a) 
Traceback (most recent call last):
  File "<pyshell#15>", line 1, in <module>
    bin(a)
TypeError: 'A' object cannot be interpreted as an integer

```


4.  如果对象定义了`__index__`方法，但返回值不是整数，报错：
```
>>> class B:
    def __index__(self):
        return "3"

>>> b = B()
>>> bin(b)
Traceback (most recent call last):
  File "<pyshell#21>", line 1, in <module>
    bin(b)
TypeError: __index__ returned non-int (type str)
```

5. 对象定义了`__index__`方法，且返回值是整数，将`__index__`方法返回值转换成二进制字符串：
```
>>> class C:
    def __index__(self):
        return 3

>>> c = C()
>>> bin(c)
'0b11'
```

### callable(1)的值为\_____

>答案：False

解答：`callable(object)`检查对象是否可以被调用，`1`不可被调用，所以是`False`。

**[文档解释]**
- **callable(object)**
Return True if the object argument appears callable, False if not. If this returns true, it is still possible that a call fails, but if it is false, calling object will never succeed. Note that classes are callable (calling a class returns a new instance); instances are callable if their class has a __call__() method.

**[Example]**
1. 方法用来检测对象是否可被调用，可被调用指的是对象能否使用()括号的方法调用。
```
>>> callable(callable)
True
>>> callable(1)
False
>>> 1()
Traceback (most recent call last):
File "<pyshell#5>", line 1, in <module>
1()
TypeError: 'int' object is not callable
```

2. 可调用对象，在实际调用也可能调用失败；但是不可调用对象，调用肯定不成功。
3. 类对象都是可被调用对象，类的实例对象是否可调用对象，取决于类是否定义了`__call__`方法。
```
>>> class A: #定义类A
    pass

>>> callable(A) #类A是可调用对象
True
>>> a = A() #调用类A
>>> callable(a) #实例a不可调用
False
>>> a() #调用实例a失败
Traceback (most recent call last):
  File "<pyshell#31>", line 1, in <module>
    a()
TypeError: 'A' object is not callable


>>> class B: #定义类B
    def __call__(self):
        print('instances are callable now.')

        
>>> callable(B) #类B是可调用对象
True
>>> b = B() #调用类B
>>> callable(b) #实例b是可调用对象
True
>>> b() #调用实例b成功
instances are callable now.
```

### 表达式 complex(1,2) == complex("1+2j") 的值为\____
>答案：True

解答：`complex([real[, imag]])`创建一个值为$real + imag * j$ 的复数或者转化一个字符串或数为复数。**如果第一个参数为字符串，则不需要指定第二个参数。**所以这两个表达式是等价的。

### 想查看某个模块中能够使用的属性和方法, 以sys模块为例, 填空: \____(sys)
>答案：dir

```
>>> import sys
>>> dir(sys)
['__displayhook__', '__doc__', '__excepthook__', '__interactivehook__', '__loader__'，
…………
'thread_info', 'version', 'version_info', 'warnoptions', 'winver']
```


### 如下表达式等价于(a // b, a % b)：\____(a, b)
>答案：divmod

解答：`//`在Python中表示整数除法，`divmod(x, y)`函数可以获得商和余数。

**[补充说明]**

通常C/C++中，"/ " 算术运算符的计算结果是根据参与运算的两边的数据决定的，比如：
```python
6 / 3 = 2      # 6,3都是整数，那么结果也就是整数2;
6.0 / 3.0 = 2.0 # 6.0,3.0是浮点数，那么结果也是浮点数2.0，跟精确的说，只要" / " 两边有一个数是浮点数，那么结果就是浮点数。
```
在Python2.2版本以前也是这么规定的，但是，Python的设计者认为这么做不符合Python简单明了的特性，于是乎就在Python2.2以及以后的版本中增加了一个算术运算符`// `来表示整数除法，返回不大于结果的一个最大的整数，而`/` 则单纯的表示浮点数除法。


### 遍历同时生成序号
```python
a = [2, 3, 4]
for index, item in ____(a):
    print(index)
    print(item)
# 填入合适函数
```
>答案：enumerate

**[解释]**
- **enumerate()**
对于一个可迭代的（iterable）/可遍历的对象（如列表、字符串），`enumerate`将其组成一个索引序列，利用它**可以同时获得索引和值**。

还有其他几种遍历列表序号和元素的方法：
```python
# -*- coding: utf-8 -*-
if __name__ == '__main__':
    list = ['0', '1', '2', '3', '4']

    # 方法1
    print('方法1：')
    for i in list:
        print ("序号：%s   值：%s" % (list.index(i) + 1, i))

    print('\n方法2：')
    # 方法2
    for i in range(len(list)):
        print ("序号：%s   值：%s" % (i + 1, list[i]))

    # 方法3
    print('\n方法3：')
    for i, val in enumerate(list):
        print ("序号：%s   值：%s" % (i + 1, val))

     # 方法3
    print('\n方法3 （设置遍历开始初始位置，只改变了起始序号）：')
    for i, val in enumerate(list, 2):
        print ("序号：%s   值：%s" % (i + 1, val))
        
	# 注意，enumerate(list, 2)中的第二个参数只改变起始序号，，并不会真的从第3个元素开始遍历。
```

<!-- more -->