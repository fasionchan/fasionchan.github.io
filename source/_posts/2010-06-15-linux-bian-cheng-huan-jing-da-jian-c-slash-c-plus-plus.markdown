---
layout: post
title: "Linux编程环境搭建——C/C++"
date: 2010-06-15 17:17:42 +0800
comments: true
categories: Linux C C++
key words: Linux C C++ gcc g++ nano
---

> 本文是面向初学者的入门型教程，高手请忽略~

> 本文是面向初学者的入门型教程，高手请忽略~

> 本文是面向初学者的入门型教程，高手请忽略~

计算机专业都有开展编程课吧，但大部分院校都是基于`Windows`平台的，包括`VC`、`VS`等等。这样的`IDE`用多了，人容易“傻”——编程可不是非得要用`IDE`！再说了，`IDE`按钮按多了，都不知道编程的本质是啥。

很多人想学`Linux`，摆脱`Windows`的枷锁，可是老师不教呀，作业怎么办！

莫慌，我们先来看看，在`Linux`下，怎样进行`C`语言程序开发吧~

<!--more-->

## 工具

工欲善其事，便先利其器。在`Linux`编程，我们需要哪些工具呢？

编程其实无非就是几个步骤：

- 首先要能编辑代码并保存吧？也就是说需要一个编辑器，最简单的像记事本都可以。`Linux`下推荐用`vim`或者`emacs`，这两个都是属于学习曲线比较陡的利器(掌握后可以各种出神入化)。初学者也可以试试`nano`，简单易用。
- 代码写完后，怎么生成可执行程序呢(编译)？这时候，需要用到编译器。不同系统不同语言编译器也不尽相同。`Linux`下编译`C`代码，需要用到`gcc`；编译`C++`代码，需要用到`g++`。

|环节|可用工具|
|:----|:----|
|编辑|vim/emacs/nano|
|编译|gcc/g++|

下面，以`Ubuntu`为例，介绍一下如何安装这些工具：

### 安装编译器

`Ubuntu`下使用`apt`进行装包：

```
$ apt-get install gcc
```

## 编辑代码

`shell`下，运行`nano test.c`，开始编辑`test.c`文件：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2740477-4bde234e7ec313c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候出现以上界面，这时候便可以开始输入了。

代码编辑后如何保存呢？请注意下方操作提示条，这时可以清楚知道按`ctrl+x`保存文件；按`ctrl+x`退出`nano`程序。

接下来请输入以下代码，保存并退出：

```c
#include <stdio.h>

int main(int args, char *argv[])
{
    printf("Hello world\n");

    return 0;
}
```

这时候，在当前目录下可以看到`test.c`文件了：

```
$ ls
test.c
```

## 编译

接下来是编译环节，运行以下命令：

```
$ gcc -o test test.c
```

这个命令的意思是，运行`gcc`命令，编译`test.c`文件；`-o`表示将可执行文件保存为`test`。不出意外，在当前目录下可以看到一个名为`test`的可执行文件。

```
$ ls
test test.c
```

## 运行

直接运行可执行文件，就可以看到程序输出的`Hello world`了：

```
$ ./test
Hello world
```

## 下一步

看到这里，你已经掌握了`Linux`下`C`程序开发的过程！`C++`也是类似的，将`gcc`换成`g++`即可。

当然了，`Hello world`只是用来演示，并没有什么作用。万里长征算是迈出第一步，接下来更有挑战性的程序在等着你！

另外，调试也是开发中非常重要的一环，本文暂未介绍。`Linux`一般使用`gdb`进行调试，这是一个功能非常强大的工作，后续有机会818。


![欢迎加入玩转Linux](http://upload-images.jianshu.io/upload_images/2740477-ec903564f7bfd9be.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/200)
