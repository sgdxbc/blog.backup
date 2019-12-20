---
layout: post
title:  一种便于进行优化的程序表示形式
date:   2019-12-20 23:38:58 +0800
categories: weaver
tags: python 编译器
author: sgdxbc
---
> 涉及循环的程序（对我来说）太过于复杂，本文不做讨论。

条件分支使程序控制流形成有向无环图，例如以下代码中的菱形结构：

```c
int x = 42, y;  // `z` declared previously
if (x == z) {
    y = x - 1;
} else {
    y = z + 1;
}
y = y * 2;
```

在这段逻辑中，如果分支条件`x == z`为真，则`y`在最后拥有常量值`82`；否则`y`的值不固定。因此，虽然`y = y * 2`（以及其后续指令）为两条执行流共享，但将其复制并分别放在两个分支的上下文中进行分析，有可能得出其中一个分支专属的优化结果。

基于这一现象，我设计了一个简单的中间表示，以及一个被称为`BasicBlock`的便于优化的基本数据结构。

----

中间表示最少需要两条指令：`SetValue(reg, value)`将编号为`reg`的寄存器的值设置为`value`，而`If(cond, yes, no)`则创建一个条件分支，`cond`的值将会决定`yes`和`no`分别指定的一列指令中的哪一列会被执行。`value`和`cond`均为`Value`类型的对象，构造一个这样的对象需要指定一列寄存器编号，以及将这个寄存器内的值处理成最终结果的计算方式。

> `SetValue`的命令是为了避免与Python内建的集合类型`set`产生混淆。

假设`x`存放在0号寄存器，`y`和`z`分别存放在1号和2号寄存器，上面的C代码可以用这种中间表示写作：

```python
codes = [
    SetValue(0, Value([], "42")),
    If(Value([0, 2], "{0} == {1}"), [
        SetValue(1, Value([0], "{0} - 1")),
    ], [
        SetValue(1, Value([2], "{0} + 1")),
    ]),
    SetValue(1, Value([1], "{0} * 2")),
]
```

----

如果将上面的菱形结构重新展开成树，并且将相关的数据聚合在一起，我们可以得到如下的「标准」代码块：
* 一个代码块由（可空的）一列指令和一个（可选的）分支跳转组成
* 分支跳转包含一个`Value`类型的条件和两个后续代码块，分别在条件成立和不成立时进入
* 当分支跳转不存在时，控制流到达该代码块最后一条指令时程序结束。

以上定义便描述了`BasicBlock`数据结构。

考虑一个比上面的例子中更加复杂的程序的中间表示：

```python
b1 = BasicBlock.from_codes([
    SetValue(1, Value([], '1')),
    SetValue(2, Value([0, 1], '{0} + {1}')),
    If(EqualTest(2, Value([], '4')), [
        SetValue(2, Value([2, 1], '{0} - {1}')),
    ], []),
    If(EqualTest(2, Value([], '3')), [
        SetValue(2, Value([2, 1], '{0} - {1}')),
    ], [
        SetValue(2, Value([2, 1], '{0} + {1}')),
    ]),
])
```

将生成的代码块以及其涉及的所有其他代码块递归地打印出来：

```python
for block in b1.recursive():
    print(block)
    print()
```

可以得到如下打印结果

```
L6:
$1 = 1
$2 = $0 + $1
If $2 == 4 Goto L2 Else Goto L5

L2:
$2 = $2 - $1
If $2 == 3 Goto L0 Else Goto L1

L0:
$2 = $2 - $1

L1:
$2 = $2 + $1

L5:
If $2 == 3 Goto L3 Else Goto L4

L3:
$2 = $2 - $1

L4:
$2 = $2 + $1
```

在此基础上即可进行一些基本的优化操作。例如，以下代码打印进行了简单的常量传播后的代码块：

```python
for block in b1.eval_reduce().recursive():
    print(block)
    print()
```

得到的结果如下：

```
L9:
$1 = 1
$2 = $0 + $1
If $2 == 4 Goto L7 Else Goto L8

L7:
$2 = $2 - $1
$2 = $2 - $1

L8:
If $2 == 3 Goto L3 Else Goto L4

L3:
$2 = $2 - $1

L4:
$2 = $2 + $1
```

代码块的数量和代码长度都有了明显地缩减。
