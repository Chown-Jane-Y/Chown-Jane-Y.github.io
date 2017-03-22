---
title: 如何理解Python中的可变对象与不可变对象
date: 2017-03-22 15:10:39
tags: Python
categories: Python学习之路
description:
---


学习Python接近一年，一直是当工具来使用，没有深入了解Python的一些机制。今天偶然看到可变与不可变对象的概念，发现与C/C++语言略有区别，遂梳理一遍。

<!--more-->
首先，要明确一点，Python中，**一切皆对象**。

Python在heap中分配的对象分成两类：**可变对象和不可变对象**。

**分类**
- 可变对象：字典型、列表型
- 不可变对象：整型、浮点型、字符串型、元组等(除了字典和列表之外都是不可变对象)

**概念**
- 所谓可变对象是指，对象的内容可变，而不可变对象是指对象的内容不可变。

---

下面用例子感受一下两者的区别。

```
i = 73
i += 2
```
![immutable](http://i1.piimg.com/588729/5e58d9c6b59a9150.jpg)

Python中一切是对象，数字、列表包括函数都是对象，而变量是对对象的引用，或者通俗的说，是对象的名字。对对象的操作都是通过引用来完成。Python中赋值操作`=`就是把一个名字绑定到一个对象上，相当于**给该对象贴了一个标签**。整型也是一个对象，`i=73`会在heap中创建一个整型对象：`PyIntObject`，然后把`i`绑定到这个变量上。

当需要对该对象内容进行修改，并不是对原来的对象进行修改，而是会新建一个对象(开辟一块新的内存空间)，然后**把原来的撕下来，贴到新的对象上**(原来的对象可能会被垃圾回收机制回收)。这和C/C++不同，C语言中`i`是指向73的内存地址，对变量`i`进行修改，就是在当前内存地址中的内容进行修改，不会开辟新的内存空间。

明白了这个原理，可变与不可变对象也就不难理解了。不可变，意思就是73这块内存空间内的内容是无法改变的，只能重新开辟空间，然后`i`指向新的地址。

那么相对的，可变对象就是，该空间内的内容允许被改变，而不用创建新的对象(开辟新的内存空间)。

也举一个简单的例子。

```
m=[5,9]
m.append(6)
```

![mutable](http://i1.piimg.com/588729/8adca063a2339934.jpg)

可以看到，对列表`m`增加元素或修改元素，是在当前`m`指向的地址进行修改，没有开辟新的内存空间。`m`指向的内存空间里的内容可以发生改变，因此说该对象是可变对象。

---

**实验代码**

```python
print('不可变对象')
print('==================')
a = 1
b = 1						# 代码注释部分为输出
print('a_id:  ', id(a))   	# a_id:   1384645104
print('b_id:  ', id(b))   	# b_id:   1384645104
# 可以看到a和b的id一样，即指向同一对象
print('a is b ? ', a is b)	# a is b ?  True
b += 1
print('执行b = b + 1之后')   
print('new a_id:  ', id(a))	# new a_id:   1384645104
print('new b_id:  ', id(b))	# new b_id:   1384645136
# b的id发生改变，因为b指向了新的对象
print('a is b ? ', a is b)	# a is b ?  False

print('-----------------------------')
print('可变对象')
print('==================')
list1 = ['a']
list2 = list1
print('list1_id ', id(list1))	# list1_id  18069640
print('list2_id', id(list2))	# list2_id 18069640
# list1和list2的id相同
print('list1 is list2 ? ', list1 is list2)
list2.append('b')
print('执行list2.append(\'b\')之后')
print('new list1_id ', id(list1))	# new list1_id  18069640
print('new list1_id ', id(list2))	# new list1_id  18069640
# list2修改过后，list2的id未发生改变
print('list1 is list2 ? ', list1 is list2)
```
