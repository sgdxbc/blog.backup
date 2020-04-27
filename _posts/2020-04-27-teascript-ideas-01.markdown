---
title: 关于TeaScript的原始想法
date: 2020-04-27 14:27
categories: thoughts
tags: TeaScript JavaScript 脑内编程
excerpt: <!-- more -->
---

TeaScript是一门编译到JavaScript，心智模型类似于简化的JavaScript（ES5）的脚本语言。如果说CoffeeScript是企图将JavaScript「Ruby化」，那么TeaScript
就是将JavaScript「纯化」，类似于C#之于Java。

> 拓展名是`.ts`吗？不不，拓展名是`.tea`。

<!-- more -->

> weaver终于告一段落以后将WSL逐出我的系统，结果发现想用上WSL2还得一段日子，又不想在虚拟机里装中文输入法，所以先用GitHub网页编辑顶一阵子。

----

关于选择基于类的继承还是基于原型的继承我在上一篇文章里阐述了后者优胜的理由。从JavaScript的发展史来看这属于文艺复兴，但也没什么不好的不是。

TeaScript是一门基于对象的的语言。对象的定义包括两部分：
* 拥有指导者（supervisor）并支持动态分发（dispatch）
* 拥有内部状态（state）

第一点是绝大多数设计模式的基石。任何一个对象都以某个对象作为指导者，除了`object`。`object`是系统初始化时自动创建的「祖先」对象，它没有指导者，且任何
对象沿着指导者链条向上追溯最终都会到达`object`。每一个对象都有一个动态分发表，其中的值是方法。在一个对象上调用一个方法时，会在它的分发表中寻找这个方法，
如果没有找到则继续在它的指导者的分发表中寻找，并调用找到的第一个方法，并且在调用中以最初被调用的对象作为上下文对象`this`。这一套语义和原生JavaScript
可以说是完全一致，事实上和Python/Ruby的「本质」也是很相近的，只是这两门语言（故意）明显地区别了「创建类对象（继承）」和「创建实例对象」，而TeaScript
则统一地使用`spawn`关键字来强调两者本质上的无差别。

```
; 创建广义上的猫
cat = object spawn
  ; 分发方法
  sayHi -> 'Meow, I\'m ' + this.name + '.'  ; 访问状态的语法需要设计一下
  new(name) -> 
    ; 创建一只具体的猫
    cat spawn
      ; 内部状态
      name = name
      
  newExcited(name) ->
    cat spawn
      name = 'excited ' + name
      ; 重新分发sayHi覆盖上面的版本
      sayHi -> 'Meow, I\'m ' + this.name + '!'

; 用点语法分发方法
aCat = cat . new('catsay')
aCat . sayHi  ; Meow, I'm catsay.
anExcitedCat = cat . newExcited('catsay')
anExcitedCat . sayHi  ; Meow, I'm excited catsay!
```

其中的`anExcitedCat.sayHi`看起来比较不可思议。事实上用Python也能写出来等价的代码，只是写法相当隐晦而已。如果这种写法看起来很不妙，那么只要简单地
避免在所有非全局作用域中的`spawn`中出现分发，就可以退化到和类与对象一样的模型了。

内部状态是TeaScript与其他脚本语言区别比较大的地方。大部分类似模型的脚本语言是不区分状态和方法的——两者被统称为「属性」。但是TeaScript的状态是不参与
多态分发的，是对象的私有物，因此我在用语上反复强调它是「内部」状态。我希望通过这种途径来限制一个类的接口……

……只能通过方法分发而不是暴露状态本身来实现，写到这里我感到有些困惑，下次再接着写。
