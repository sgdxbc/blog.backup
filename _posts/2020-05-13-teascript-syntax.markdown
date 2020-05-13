---
title: 体验至上的前端设计
date: 2020-05-13 14:17
categories: teascript
tags: 编译器
---

上一篇写着写着就写不动了，这个故事告诉我们不要一边写文章一边聊天，精力会耗尽的。根据之前得出的一些结论，定下来的语法有
* `StoreName('x', expr)` → `assign x to expr`
* `StoreState('x', expr)` → `update &x to expr`
* `Spawn(watcher, stateTable, dispatchTable)` → `spawn on watcher init &state1 to expr receive message -> ...`

其中关于`Spawn`的具体关键字选择还需要考虑一下，不过按理说应该可以交错地声明状态和方法，虽然还是推荐把状态都放在最前面就是了。

所有这些语法（包括传统的`if` `while`之类的）都有一个优点：看到第一个单词就能分辨节点类型（LL(1)），并且不需要向前看任何东西就能确定当前节点对应的代码是不是已经结束了。这两点除了降低编写前端难度以外，还可以降低阅读代码的心智负担。哦对了，上下文对象用`this`，不用`&&`。

那么最后就剩下`Dispatch`了。按理说它应该是`dispatch object with message(arg)`的形式，但是这样写有两个大问题。首先全世界的人都在用`.`，这么搞太陌生了。其次，这样写就没法链式调用了。因此还是得写成`object . message1(arg) . message2(arg)`这样。

不过想了想似乎`send object with message1(arg) then message2(arg) then ...`这样也是可以的嘛，所以还是要再想想了。

> Written with [StackEdit](https://stackedit.io/).
