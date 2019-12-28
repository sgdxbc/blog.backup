---
layout: single
title: 关于我和我的博客
permalink: /about/
date: 2019-12-28 14:19
last_modified_at: 2019-12-28 14:19
author_profile: true
---

## 关于我

2014年春天在一本名字类似于《Windows系统基础入门》的最后一章中偶遇了C语言教程（然而前一章还是一张张地截图教你怎么用Outlook的级别），看了一个中午以后向家长索要K&R C并在老爸的笔记本上安装Dev-C++，从此以后一直走在编程路上。

目前最得意的项目是[Shattuck](https://github.com/whoiscc/shattuck)，这是一个用Rust实现「类Python但是真并行」脚本语言的宏伟企划，目前实现了大概10%——一个垃圾收集器，并且随时可能会完全推翻从头来过。

在计算机网络方向（2018.2起）有若干科研经历。论文永远在「在投」和「被拒」两个状态之间跳转。为了科研创建过若干代码仓库，其中比较重要的有
* [sidfam-v3](https://github.com/whoiscc/sidfam-v3)，由于最初实现的严重效率问题怒而狂写Cython的产物，也正是因为这次开发经历认识到一门脚本语言的真并行性有多重要
* [rubik](https://github.com/ants-xjtu/rubik)，根据基于Python的DSL编写的配置生成C代码，中途进行一定的优化。目前正在绝赞重写中。

个人认为曾经写出来过的最天才的代码是[standarize6.cpp](https://github.com/lifta42/lib-never/blob/dev/proof/standarize6.cpp)和[standarize7.cpp](https://github.com/lifta42/lib-never/blob/dev/proof/standarize7.cpp)。假如你有一个这样的基于回调的异步函数

```js
add_async(3, 5, result => cosole.log(result));  // print '8'
```

使用这两篇代码中的`stand`函数后……

```js
stand(add_async, stand_add_async => {
  stand_add_async(3, add_3_async => {
    add_3_async(5, result => console.log(result));  // print '8'
  });
});
```

……它就被柯里化了。那么问题来了，用C++编写的`stand`，它的返回值类型是什么呢？

----

我也很希望上面这个「丰功伟业」列表能够再长一些。

多年来我一直不懂得一个道理，写代码并不能完全以塑造工艺品的心态去写。或者说，就算是工艺品，也没有能够真正脱离现实的工艺品存在。如果一篇代码没有任何存在的现实意义，纯粹是基于「我想要写着开心」的心态被创作出来的话，那它就像海浪退去时的沙滩城堡一样脆弱。「多去写能留存下来的代码」，这是我早就想要实现的目标，却一直没有找到实现的方法；如今我也不敢说「多写解决实际问题的代码」就一定能解决它，还是需要更多的时间来检验解决的效果。

最后一提我的生平最爱——开小号。GitHub曾用账号包括但不限于：
* [whoiscc](https://github.com/whoiscc)（这个大概还会继续用下去），这个账号的[博客](https://whoiscc.github.io)虽然不如下面那个宝藏，但是列举了我大半大学时光中（还能回想起来的）若干奇奇怪怪的项目企划
* [lifta42](https://github.com/lifta42)
* [oziroe](https://github.com/oziroe)，稍微看了一下这个账号的[博客](https://oziroe.github.io)简直是宝藏级别的……我自己都不记得自己写过这些东西……

所幸我的[知乎账号](https://www.zhihu.com/people/cowsay/activities)似乎没有被换掉过，我为自己的项目写了不少文章，也发表过一些感慨受到了一定的认可，比如这篇[《我为什么对Rust常怀敬畏之心》](https://zhuanlan.zhihu.com/p/88478551)。

## 关于这个博客

作为这个博客的一个常用标签，同时也是整个博客的真实主题，

> 不记下来明天就忘了

大概就是这么回事吧。

此外，我给自己定下了「每天只能发表一篇文章」的规矩，防止自己用力过猛（又一次）将热情消耗殆尽了。