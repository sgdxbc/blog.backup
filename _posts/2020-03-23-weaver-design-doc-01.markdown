---
title: 「Weaver设计文档」一、需求概述和管线模型
date: 2020-03-23 16:35
categories: weaver
tag: Python 文档
excerpt_separator: <!-- more -->
---

程序员都知道，最想写文档的那一刻，必然是代码最写不下去的那一刻。

那么，怎样的情况会让此刻的我恨不得把weaver的方方面面都事无巨细地记录下来呢？这是怎样的一种「万策尽」的心情呢？

> 无论如何切不可让李昊知道这回事。

写在2020年3月23日，距离NSDI投稿截止（4.17）还有大半个月。

<!-- more -->

----

weaver项目是我对RUBIK论文的实现。weaver这个名字是我自作主张并且遭到论文作者的抵制。也许这个项目会以另一个名字公之于众，但在系列文档中仍然被称为weaver。

weaver项目致力于生成高度可定制的网络中间件（middlebox）。用户可以用Python编写一个寥寥数行的配置文件，使用weaver根据这个配置文件生成数万行C代码，并且进一步将其与用户（用C）编写的应用代码一起编译成可执行程序。

配置文件主要由三部分构成：结构、管线和协议栈。结构是用户定义协议头和声明变量的工具，例如一个定义了以太网协议头的结构

```python
class eth_hdr(layout):
    src_mac1 = Bit(16)
    src_mac2 = Bit(16)
    src_mac3 = Bit(16)
    dst_mac1 = Bit(16)
    dst_mac2 = Bit(16)
    dst_mac3 = Bit(16)
    protocol = UInt(16)
```

可以注意到两点：结构中的字段长度要么小于一个字节（位字段），要么是1、2、4或8个字节长，因此6个字节长的MAC地址不能用一个单独的字段来定义；除了`Bit`以外，还有一种特殊的`UInt`类型字段，当它出现在表达式中的时候，它所包含的值会先通过`ntohs`或`ntohl`函数进行大小端调整后再参与运算。`Bit`字段的长度可以动态计算

```python
class tcp_blank(layout):
    blank_type = Bit(8)
    blank_len = Bit(8)
    blank_value = Bit((blank_len - 2) << 3)
```

`UInt`的长度只能是`8`或`16`。

以下是对临时变量和持久变量的定义示例

```python
class tcp_temp(layout):
    wnd = Bit(32)
    wnd_size = Bit(32)
    data_len = Bit(32)

class tcp_data(layout):
    active_lwnd = Bit(32, init=0)
    passive_lwnd = Bit(32, init=0)
    active_wscale = Bit(32, init=0)
    passive_wscale = Bit(32, init=0)
    active_wsize = Bit(32, init=(1 << 32) - 1)
    passive_wsize = Bit(32, init=(1 << 32) - 1)
    fin_seq1 = Bit(32, init=0)
    fin_seq2 = Bit(32, init=0)
```

临时变量在一个数据包到来时分配，在数据包处理完毕时销毁；持久变量随着一条流的创建而创建，随着流的销毁而销毁。

关于使用结构和字段的一些要求：
* 长度大于等于一个字节的字段必须与字节边界对齐
* 临时/持久变量的长度必须是静态的并且不能小于一个字节，并且不能是`UInt`类型
* 持久变量必须通过`init`参数指定初值

## 六阶段管线模型

处理一个协议的过程被限定为顺序固定的六个阶段：
1. 「header」解析包头
2. 「selector/instance」获取/创建流
3. 「prep(preprocess)」用户自定义的准备工作
4. 「seq(sequence)」维护数据队列
5. 「PSM(protocol state machine)」运行协议自动机
6. 「event」触发事件

除了header以外的阶段都是可选的，省略时有以下要求：
* 如果有seq或PSM则必须有selector
* 如果使用了持久变量则必须有selector
* 如果使用了`current_state`则必须有PSM
* 如果使用了`sdu`则必须有seq

一般来说一个协议的管线定义在一个函数中，比如最简单的以太网协议

```python
def eth_parser():
    eth = Connectionless()
    eth.header = ethernet_hdr
    return eth
```

定义在函数中的好处是可以在定义管线时接受来自其他层的额外信息，比如对TCP协议的定义

```python
def tcp_parser(ip):
    tcp = ConnectionOriented()

    # 省略若干

    tcp.selector = (
        [ip.header.saddr, tcp.header.sport],
        [ip.header.daddr, tcp.header.dport],
    )
```

在后续的文档中，将这样的函数返回的`Connectionless`或`ConnectionOriented`类型的对象称为原型（prototype）。事实上在目前的实现中，`Connectionless`和`ConnectionOriented`都是`Prototype`类的别名。

----

一个完整的协议栈由一到多个原型、它们之间的跳转关系和可选的用户事件组成。比如一个完整的TCP/IP协议栈

```python
stack = Stack()
stack.eth = eth_parser()
stack.ip = ip_parser()
stack.tcp = tcp_parser(stack.ip)
stack.udp = udp_parser()

stack += (stack.eth >> stack.ip) + Predicate(1)
stack += (stack.ip >> stack.tcp) + Predicate(stack.ip.header.protocol == 6)
stack += (stack.ip >> stack.udp) + Predicate(stack.ip.header.protocol == 17)
```

通过`stack.xx = ...`的方式进行的赋值是被劫持了的，最终生成的`stack.xx`属性不是一个原型，而是一个层（layer）。层与原型的区别是层是拥有「资源」的，而原型只是一张图纸而已。因此，`tcp_parse`接受的参数必须是一个层而是一个原型，因为只有层当中的包头字段和变量字段才对应着生成的C代码中的变量。

在定义协议栈时，可以通过层所提供的`stack.xx.event.yy = ...`接口向某一层追加用户自定义的事件。这样的事件和原型自带的事件没有区别，甚至两者执行的顺序都可以打乱。

》写不下去了，日后补充吧。