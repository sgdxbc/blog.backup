---
title: Python「属性」类的最佳实践
date: 2020-01-27 11:41
categories: python
tag: Python 不记下来明天就忘了
excerpt_separator: <!-- more -->
---

遇到了一个「仔细想想还有点奇怪」的需求

```python
obj = AttrObject()
obj.x = 42
print(obj.x)  # => Wrapper(42)
print(obj.name_map)  # => {'x': 42}
```

其中`x`是一个用户随便取的名字。事实上，`name_map`并不是公共 API 的一部分，但是会在其他 API 的实现中被用到，比如

```python
class AttrObject:
    # ...
    def print_pairs(self):
        print(self.name_map.items())
```

<!-- more -->

直截了当的实现思路是劫持读写属性

```python
class AttrObject:
    def __init__(self):
        self.name_map = {}

    def __setattr__(self, name, value):
        self.name_map[name] = value

    def __getattr__(self, name):
        return Wrapper(self.name_map[name])
```

但是这段代码并不能工作，因为对读写方法的截取同样会影响到`__init__`中对`name_map`的写和下面两个方法中对`name_map`的读。我们需要将所有的「内部」读写都手动「转义」一下

```python
class AttrObject:
    def __init__(self):
        super().__setattr__(self, 'name_map', {})

    def __setattr__(self, name, value):
        getattr(super(), 'name_map')[name] = value

    # __getattr__类似
```

> 还要格外注意，对象默认是没有`__getattr__`方法的所以要用内建函数`getattr`。

当类的功能变得复杂起来，读写属性的操作变多时，这种方案用起来极其痛苦。

---

通过对[Example setattr & getattr overloading (Python recipe) by Michael Foord](http://code.activestate.com/recipes/389916-example-setattr-getattr-overloading/)进行少许修改，可以得到下面这种方案

```python
class NameMapMixin:
    def __init__(self):
        self.init = True

    def __setattr__(self, name, value):
        if not hasattr(self, "init") or hasattr(self, name):
            super().__setattr__(name, value)
        else:
            self.name_map[name] = value

    def __getattr__(self, name):
        try:
            return getattr(super(), name)
        except AttributeError:
            if name == "init":
                raise
            return self.name_map[name]


class AttrObject(NameMapMixin):
    def __init__(self, init_name_map):
        self.name_map = init_name_map
        self.other_attr = 42
        super().__init__()
```

在使用`NameMapMixin`时，必须满足一个要求：**所有的内部属性必须在`__init__`函数中进行创建**。

在`__setattr__`中分三种情况分析。如果`init`属性尚未存在，说明子类的构造过程尚未完成，因此对于写属性不进行干涉；如果要写的属性在`self`中存在，说明这次写入同样是对内部属性的更新，也不进行干涉；除此之外的写入按照 API 调用进行处理。

在`__getattr__`中，不能使用`hasattr`，因为`hasattr`的实现调用了`getattr`，从而会导致无限递归。`__getattr__`同样分为三种情况进行讨论，不再赘述。在`__getattr__`中没有对 API 返回值进行包裹操作，如果有这个需求可以通过调用虚方法的方式轻松实现。

这种方案需要进一步加强的功能主要是为用户提供友好的报错信息，避免用户在尝试设置与内部属性同名的属性是出现难以排查的 bug。
