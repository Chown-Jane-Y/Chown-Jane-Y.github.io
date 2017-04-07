---
title: Iterable和Iterator不仅仅是形容词与名次的区别
date: 2017-04-07 14:53:37
tags: Python
categories: Python学习之路
description:
---


`Iterable`和`Iterator`乍一看一个是形容词，一个是名词形式，初学时很容易把它们看成一样的东西，其实不然。`Iterator`一定是`Iterable`的，但`Iterable`的不一定是`Iterator`哦。

<!--more-->
Python中主要的几种数据类型`list`, `tuple`, `str`, `dict`这些都可以被迭代(Iterable)，但他们并不是迭代器(Iterator)。为什么？

因为和迭代器相比有一个很大的不同，`list/tuple/map/dict`这些数据的大小在被定义后就是确定的，也就是说有多少是可知的。但迭代器不是，迭代器不知道要执行多少次，所以可以理解为不知道有多少个元素，每调用一次`next()`，才会往下走一步，是惰性的。


**判断是不是可以迭代，用Iterable**

```python
from collections import Iterable  
isinstance({}, Iterable) --> True  
isinstance((), Iterable) --> True  
isinstance(100, Iterable) --> False  
```

**判断是不是迭代器，用Iterator**
```python
from collections import Iterator  
isinstance({}, Iterator)  --> False  
isinstance((), Iterator) --> False  
isinstance( (x for x in range(10)), Iterator)  --> True  
```

---

**凡是可以for循环的，都是Iterable**

**凡是可以next()的，都是Iterator**

集合数据类型如`list`, `tuple`, `dict`, `str`，都是可迭代的但不是跌带起，但可以通过`iter()`函数来获得一个Iterator对象。

![IterableToIterator](http://i4.buimg.com/588729/b8e0e155ecd0440f.png)

**Python中的for循环就是通过next实现的**
```python
for x in [1,2,3,4,5]:  
    pass  
```

等价于
```python
#先获取iterator对象  
it = iter([1,2,3,4,5])  
while True:  
    try:  
        #获取下一个值  
        x = next(it);  
    except StopIteration:  
        # 遇到StopIteration就退出循环  
        break
```  
<!--more-->
