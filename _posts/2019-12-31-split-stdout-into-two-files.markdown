---
title: 将标准输出「切」成两半
date: 2019-12-31 12:52
categories: develop
tags: make sed Shell
excerpt_separator: <!-- more -->
header:
  overlay_image: /assets/603018.jpg
  overlay_filter: rgba(0, 0, 0, 0.5)
  show_overlay_excerpt: false
---

## 需求是什么？

在[weaver](https://github.com/sgdxbc/weaver)项目中，运行`python3 -m weaver`可以生成一段C代码，这段C代码需要（经过修改以后）和其他一些已经存在的C源代码文件一起构建一个可执行程序。到底为止并没有什么难度——如果我想追求难度大概就用CMake了。（

<!-- more -->

```make
build: weaver_blackbox.c
	$(CC) # ...

gen:
	python3 -m weaver > weaver_blackbox.c
```

然而接下来，需求有变：运行`python3 -m weaver`需要生成两个源文件，原因是其中一个是需要用户修改的，另一个应该对用户保持距离——这也是这个项目存在的目的，把用户不需要碰的代码拿到远离用户的地方去。这样一来，通常的想法应该是给上面的命令提供两个命令行参数

```make
gen:
	python3 -m weaver weaver_blackbox.c weaver_whitebox.c
```

然后在Python脚本中直接写入文件。但是我还需要时不时肉眼检验一下Python输出的C代码是否正确，运行以后再去打开生成的文件实在有点麻烦（真的吗？真的吗？？？）。于是就有了今天的需求：**Python仍旧将两个文件的内容无脑`print`出来，通过命令行的方式将这些内容分别写入对应的文件中去。**

## 实现方式

[用`tee`将Python的标准输出复制成两份](https://unix.stackexchange.com/a/28519)

```
python3 -m weaver | tee >(command1 ...) | command2 ...
```

注意：这里的`>(...)`语法并不被`sh`支持，因此要在Makefile的开头添加一行`SHELL := /bin/bash`。

[用`sed`将标准输入的匹配行之前/之后的部分写入文件](https://stackoverflow.com/a/7104422)

```make
gen:
	python3 -m weaver | tee >(sed -e "/$(sep)/,\$$d" > $(wb)) | sed -n -e "/$(sep)/,\$$w $(bb)"
```

我将`$(sep)`的值赋为第二个文件开头的一行。那么，前一个`sed`命令的意思是，将从匹配行（包含）以后的所有行全部删除。`sed`默认会打印被处理过的行——如果没被删除的话，因此将它的输出重新定向到输出文件`$(wb)`即可。后一个`sed`命令的意思是，默认不要打印任何东西（`-n`），将匹配行（包含）以后的所有行全部写入文件`$(bb)`中。

这里要注意到这是一个在Makefile中使用的情形，这导致了两件事。第一件事是在上面的链接中作者有提到一中只需要一个`sed`指令（但是需要写两行）的方法，但是由于我没有找到把它合法地写入Makefile的同时表达正确的命令的方式，因此只能退而求其次；第二件事是上面奇怪的`\$$d`和`\$$w`序列。首先连续两个`$`是Makefile对`$`的转义，因为`make`在预处理命令的时候是会忽略所有的引号的，然后剩下的`\$`是shell对`$`的转义，否则shell会将`$d`替换为（并不存在的）环境变量`b`的值。当然，后一个转义可以通过使用单引号来避免，但是由于一直在写Python而一时没想到。（

## 总结

这篇文章介绍了一种方法，将一个程序的标准输出切成两个部分，并且视情况分别进行进一步的处理——这我的例子中就简单地将它们写入了不同的文件中。这种方法主要的缺点有两个：
* 依赖于`bash`对语法的支持。虽然要求用户有`bash`应该并不过分，但是考虑到这种语法`zsh`似乎并不支持，还是留下了一定的隐患。
* 性能问题。这种做法避免了重复执行`python3 -m weaver`，但是大量使用了管道和`sed`，因此在输出内容较多时性能难以预测。
