---
title:  "Python字典的高级应用 - Counter"
date:   2019-04-22 11:33:43 +0800
categories: Python
tags: Python 字典 计数器
---
[上篇](http://mr-ping.com/python/2019/04/19/defaultdict-in-python/)关于`defaultdict`的介绍里，我们拿网站计数器举了个例子。  
实际上`python`的`collections`模块还提供了一个专用于计数的`Counter`类，它本身的特性和内置的许多方法可以更便捷的实现计数器功能。

## 首先看看`Counter`类的介绍
```python
>>> from collections import Counter
>>> help(Counter)
Help on class Counter in module collections:

class Counter(builtins.dict)
 |  Counter(*args, **kwds)
 |  
 |  Dict subclass for counting hashable items.  Sometimes called a bag
 |  or multiset.  Elements are stored as dictionary keys and their counts
 |  are stored as dictionary values.
 ...
```

所以，`Counter`跟`defaultdict`一样，也是`dict`的子类。

`Counter`存在的主要目的是对对象进行计数，并将结果以哈希值的形式记录下来。  
记录的时候把计数的对象存储为字典的*key*，把计数结果存储为字典的*value*。

## 如何使用

下面还是拿**网站计数器**为例来介绍如何使用`Counter`。

### 创建计数器

```python
>>> from collections import Counter

>>> cnt_page1 = Counter()  # 实例化计数器对象

# 如果有不可言说的理由，也可以给计数器设定初始值。以下两种传参方式效果一样
>>> cnt_page1 = Counter(('IP1', 'IP7', 'IP1'))
>>> print(cnt_page1)
Counter({'IP1': 2, 'IP7': 1})

>>> cnt_page1 = Counter(IP1=2, IP7=1)
>>> print(cnt_page1)
Counter({'IP1': 2, 'IP7': 1})
```

### 如何为访问计数

```python
>>> cnt_page1['IP1'] += 1  # IP1访问计数加1
>>> print(cnt_page1)
Counter({'IP1': 3, 'IP7': 1})

>>> cnt_page1['IP2'] += 1  # 自动创建IP2元素，并赋值int(0)+1
>>> print(cnt_page1)
Counter({'IP1': 2, 'IP7': 1, 'IP2': 1})
```
结果可见，`Counter`实例本身是一个*有序字典*。  
关于有序字典，我们会在下一篇博客中介绍。

### 查看访问次数最多的两个IP

```python
>>> most_access = cnt_page1.most_common(2)
>>> print(most_access)
[('IP1', 2), ('IP7', 1)]

>>> most_access = cnt_page1.most_common()  # 不指定个数，则依次返回所有元素
>>> print(most_access)
[('IP1', 3), ('IP7', 1), ('IP2', 1)]
```
`Counter.most_common`方法以k, v元组对的形式返回计数器内值最大的两个元素。  
因为`Counter`实例对象是有序的，如果多个元素值相同，按顺序依次返回。

### 查看访问计数总量

```python
>>> total_access = sum(cnt_page1.values())
>>> print(total_access)
5
```

### 列出所有访问者IP

```python
>>> all_access = list(cnt_page1)
>>> print(all_access)
['IP1', 'IP7', 'IP2']
```

### 查看某个IP访问次数

```python
>>> print(cnt_page1['IP1'])
3
>>> print(cnt_page1['IP22'])  # 访问不存在的元素，不会报错，而是返回into(0)
0
```

### 合并多个页面访问量

```python
>>> cnt_page2 = Counter(IP7=5, IP8=3, IP9=1)
>>> print(cnt_page2)
Counter({'IP7': 5, 'IP8': 3, 'IP9': 1})

>>> cnt_pages = Counter()  # 创建整站计数对象实例
>>> cnt_pages.update(cnt_page2)  # 将page2计数对象直接作为update参数
>>> print(cnt_pages)
Counter({'IP7': 5, 'IP8': 3, 'IP9': 1})
>>> cnt_pages.update(cnt_page1)  # 将page1的计数结果累加到pages中
>>> print(cnt_pages)
Counter({'IP7': 6, 'IP8': 3, 'IP9': 1, 'IP1': 3, 'IP2': 1})

>>> print(cnt_pages.most_common())
[('IP7', 6), ('IP8', 3), ('IP1', 3), ('IP9', 1), ('IP2', 1)]
```
`Counter`中的`update`方法不同于`dict`中的`update`，后者处理重复元素会做替代(`replace`)动作，而前者是做合计(`sum`)动作。

当然，你也可以直接把两个页面的计数器对象相加，结果是一样的。
```python
>>> cnt_pages = cnt_page1 + cnt_page2
>>> print(cnt_pages.most_common())
[('IP7', 6), ('IP8', 3), ('IP1', 3), ('IP9', 1), ('IP2', 1)]
```

### 减去某个页面的访问计数

如果想从总的站点计数中将某个页面的计数减掉，可以使用`Counter.subtract`方法：
```python
>>> print(cnt_pages)
Counter({'IP7': 6, 'IP8': 3, 'IP9': 1, 'IP1': 3, 'IP2': 1})
>>> print(cnt_page2)
Counter({'IP7': 5, 'IP8': 3, 'IP9': 1})

>>> cnt_pages.subtract(cnt_page2)  # 直接将cnt_page2计数对象当做参数
>>> print(cnt_pages)
Counter({'IP7': 1, 'IP8': 0, 'IP9': 0, 'IP1': 3, 'IP2': 1})
```
`subtract`方法从总访问计数中依次将page2对象中每个IP计数的值从总计数值中减去。

### `Counter.elements`方法

这个方法在我们的例子中貌似没有什么用处，但因为它是`Counter`不同于`dict`的一个特殊方法，还是介绍下：

```python
>>> elements_pages = list(cnt_pages.elements())
>>> print(elements_pages)
['IP7', 'IP1', 'IP1', 'IP1', 'IP2']
```
`Counter.elements`方法根据每个元素的计数，迭代将元素返回相对应次数。


## 接下来

`Counter`其余的函数跟`dict`并无二致，就不一一罗列了。

接下来要介绍一下，Python collection中的有序字典。
