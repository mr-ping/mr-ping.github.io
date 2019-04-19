---
title:  "Python中字典的高级应用"
date:   2019-04-15 10:43:43 +0800
categories: Python
tags: Python 字典
---
怎样使用Python字典这种数据类型呢？  
Well, 知道创建字典、给元素赋值、对元素取值、删除元素、遍历元素，就OK啦！能走遍天下了！那了解什么高级特性干嘛？  
让你更牛X啊！人家10分钟写完的东西你10秒就能交货啊！还不够你牛X啊！

**先从可以大幅提高生产效率的`collections.defaultdict`函数说起**

## 首先看看`defaultdict`的文档介绍：

```python
>>> from collections import defaultdict

>>> help(defaultdict)

Help on class defaultdict in module collections:

class defaultdict(builtins.dict)
 |  defaultdict(default_factory[, ...]) --> dict with default factory
 |  
 |  The default factory is called without arguments to produce
 |  a new value when a key is not present, in __getitem__ only.
 |  A defaultdict compares equal to a dict with the same items.
 |  All remaining arguments are treated the same as if they were
 |  passed to the dict constructor, including keyword arguments.
 ...
```

用人类语言解释为：  
1. 这个`collections.defaultdict`是我们常用的`dict`的子类。
2. 它跟`dict`的区别在于当你访问`defaultdict`实例对象中某个不存在的元素时，它不会跟`dict`实例对象一样给你抛出`KeyError`异常，而是在`defaultdict`实例对象中添加此元素，并给他赋个默认值。
3. `defaultdict`其他属性、方法的应用跟`Python`内置的`dict`一毛一样。

### 换个角度总结：

**`defaultdict`可以让你创建一个无限大的，其中包含无限个可能的元素的字典，而且其中每个元素都有一个由你指定的统一默认值。** 并且因为它类似生成器思想的lazy模式使得代价及其之小。  
Amazing!!!

## 使用方法

### `defaultdict`实例对象：

实例化`defaultdict`的时候有两个可选参数。  
- 第一个是`defaultdict.default_factory`，它是任何可以被以无参形式调用的函数或Python内置数据类型；  
- 第二个是实例对象的初始元素，数据类型为`dict`或者`defaultdict`。

#### 第一个传参：`default_factory`

我们可以指定`default_factory`为一个内置数据类型：
```python
>>> dfdict_obj = defaultdict(int)  # 指定default_factory元素默认值为int()的返回值

>>> print(dfdict_obj)
defaultdict(<class 'int'>, {})

>>> dfd_obj['x']  # 当我们访问一个不存在的元素时，会自动生成此元素并将int()的返回值赋给它
0

>>> print(dfdict_obj)
defaultdict(<class 'int'>, {'x': 0})
```

参数`default_factory`也可以是我们自定义的可调用的无参函数:
```python
>>> def my_pi(pi=3.14):  # 自定义一个可无参化调用的函数
    return pi

>>> dfd_obj_1 = defaultdict(my_pi)  # 将定义的函数作为default_factory

>>> print(dfd_obj_1)
defaultdict(<function my_pi at 0x7f2e90029f28>, {})

>>> dfd_obj_1['x']  # 访问尚不存在的元素时，自动创建元素并赋值
3.14

>>> print(dfd_obj_1)
defaultdict(<function my_pi at 0x7f2e90029f28>, {'x': 3.14})
```

#### 第二个传参

第二个参数是实例对象的初始元素，数据类型可以是`dict`或者`defaultdict`类型。
```python
>>> dfd_obj_2 = defaultdict(my_pi, {'a': 1, 'b': 2})

>>> print(dfd_obj_2)
defaultdict(<function my_pi at 0x7f2e90029f28>, {'a': 1, 'b': 2})

>>> print(dfd_obj_2['b'])
2
```

### 应用场景
