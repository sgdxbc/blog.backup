---
title: Python奇技淫巧三则
date: 2019-12-25 14:38
categories: python
tags: Python 不记下来明天就忘了
excerpt_separator: <!-- more -->
header:
  overlay_image: /assets/Izumi.Chiaki.full.911114.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  show_overlay_excerpt: false
---

[如何获取列表中第一个满足某个条件的值](https://stackoverflow.com/a/8534381)

```python
first_42 = next((x for x in my_list if x == 42), None)
# 如果不存在42，first_42取值为None
```

<!-- more -->

----

[如何在匿名函数中抛异常](https://stackoverflow.com/a/8294654)

```python
x = lambda: (_ for _ in ()).throw(Exception)
# 如果想要给出具体的异常对象
y = lambda: (_ for _ in ()).throw(Exception, Exception('something wrong'))
# 这样写也行，但是不符合标准库的类型声明
z = lambda: (_ for _ in ()).throw(Exception('something wrong'))
```

----

如何优雅的将一段代码放进花括号，这一个是我自己发明的，所以看起来比较粗糙。

{% raw %}
```python
if code:
    code = ('\n' + code).replace('\n', '\n  ') + '\n'
wrapped = f'{{{code}}}'
```
{% endraw %}

什么，你是左花括号换行党？出门左转不送，再见。
