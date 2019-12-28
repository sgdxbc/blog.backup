---
title: 从WSL外面使用里面的命令
date: 2019-12-28 12:45
categories: develop
tags: 搭环境 WSL Windows 不记下来明天就忘了
excerpt_separator: <!-- more -->
---

## 需求是什么？

我在Windows和Debian中都需要用Git。安装两个Git将会带来（起码）两个问题：在Debian中两个Git都是可见且可运行的（虽然Windows版本的命令基本都会带上奇怪的扩展名），有潜在的冲突隐患；以及两个Git所用到的ssh会分别在两个家目录下维护公私钥。（后者的一个（我不能接受的）解决方案自然是创建两对公私钥。）由于WSL只是一个普通的应用，我不太愿意从其内部软连接到外部——这会在我删除WSL时失效。但是我在这个场景下又不能把Windows家目录下的`.ssh`目录软连接到WSL内部，因为似乎Windows文件系统（`/mnt/c/`）内的文件权限是不受我控制的777，而这又是ssh完全不能接受的。

<!-- more -->

> 对于这个需求，其实还有一种解决方案是在WSL中使用Windows版本的`git`。在我逐渐不能忍受`git.exe add .`，又发现[1903版本中引入的bug][4]导致无法创建一个「能够正常工作的」从WSL内部指向外部可执行程序的软连接后，我放弃了这个方案。

除此之外，由于[PyCharm的Makefile插件][1]的存在，我有机会在Windows中使用`make`以及它会调用的`cc`、`rm`等一系列命令。想要在Windows下获得它们通常需要通过MSYS2之类的技术，但是我都已经有WSL了啊……

> 总感觉谁嘀咕了一句「明明是我先来的……」

幸运的是，Windows提供了一个简单好用的`wsl`命令。我们只需要稍微给它一点帮助就可以完成心愿了。

## 实现过程

在任何你喜欢的地方创建一个文件，`git.cmd`

```batchfile
@echo off
setlocal enabledelayedexpansion
set command=%*
set find=C:\
set replace=/mnt/c/
call set command=%%command:!find!=!replace!%%
call set command=%%command:\=/%%
echo | wsl git %command%
```

> Rouge居然在2019年8月才刚刚[支持了Windows批处理脚本的高亮][2]，好险啊……

在这个脚本中，我们首先将所有的命令行参数取出并存放在`command`中，然后对它执行两次全局替换：将`C:\`替换为`/mnt/c/`，再将反斜杠都替换为正斜杠。这样一来，绝大多数的Windows路径应该都被替换成的等价的WSL路径（到目前为止它工作正常）。最后通过`wsl`命令执行WSL内的`git`。这段脚本主要参考自[这里][3]。

如果你需要从cmd或者PowerShell中执行`git`的话，可以考虑将这个脚本所在的路径加入环境变量。但是我只是需要在PyCharm中调用它们，因此只要在PyCharm中设置好就行了。

## 总结

其实，利用类似的创建脚本伪装成可执行程序的方式，（在软连接不能用的时候）同样可以用来在WSL中不加拓展名调用Windows下的可执行程序。可是，正如我对`make`的需求一样，相比之下WSL中有而Windows中没有的命令要比反过来的多太多了。

这种方法的缺点是要为每一个要（被从Windows直接）执行的命令都创建一个脚本文件（虽说可以复制就是了）。此外，不知道是PyCharm还是Makefile插件的问题，在PyCharm中执行`make`命令不能正确的显示Unicode字符，因此要[在`Makefile`中添加一行`LANG=en_US.ISO-8859-1`来避免`gcc`在输出中包含Unicode字符][5]。

[1]: https://plugins.jetbrains.com/plugin/9333-makefile-support/
[2]: https://github.com/rouge-ruby/rouge/pull/1286
[3]: https://stackoverflow.com/a/43878654
[4]: https://github.com/microsoft/WSL/issues/3999
[5]: https://stackoverflow.com/a/2446963