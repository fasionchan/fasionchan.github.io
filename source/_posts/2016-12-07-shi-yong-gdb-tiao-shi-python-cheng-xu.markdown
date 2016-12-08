---
layout: post
title: "使用gdb调试Python程序"
date: 2016-12-07 22:51:29 +0800
comments: true
categories: gdb Python
---

最近在为一个监控系统开发`agent`，需要支持`Linux`、`FreeBSD`及`Windows`等操作系统。复杂的线上环境，带来了一系列诡异的问题，尽管代码上线前在为数不少的测试机器验证过。

`Python`程序吐`coredump`文件怎么办？很多人都会想到`gdb`加载`coredump`文件，然后查看信号及堆栈信息，以此分析原因。堆栈信息在调试中非常有用，但是别忘了，你写的是`Python`代码，但是`gdb`给你的是`C`堆栈信息！似乎没啥鸟用！难道要撸`Python`源码然后分析各种核心数据结构吗？有什么方式可以查看到`Python`堆栈信息吗？

还遇到过另一个问题，一个`Python`进程突然间陷入死循环，所有其他线程都调度不到。遇到这种情况，首先可能需要知道死循环到底在干什么。如何获悉呢？可能用`strace`跟一下系统调用可以看出一点端倪。但是一个堆栈信息更为具体更有说服力，就算是只有`C`堆栈信息有时也是足以说明问题的。

`gdb`就可以解决以上难题(其实远不止)，接下来，我们一起看看具体要怎么操作吧~

<!--more-->

# 准备

首先得有`gdb`吧，这个就不细说了，`debian`系发行版上运行以下命令完成安装：

```
apt-get install gdb
```

其次，还需要装一个包——`python-dbg`。这个包有什么作用呢？前面不是抱怨过`C`堆栈对于调试一个`Python`有何用？我们更需要的是`Python`堆栈信息，`python-dbg`就是为了完成这个使命。

# 运行

全新启动一个`Python`程序并进行调试，可以采用交互式方式，先启动`gdb`然后在`gdb shell`中启动`Python`程序：

```
$ gdb python
...
(gdb) run <programname>.py <arguments>
```

当然了，也可以一步到位，一条命令搞定这两步：

```
gdb -ex r --args python <programname>.py <arguments>
```

遗憾的是，现实中往往是这样的情景——一个正在运行的程序突然异常了，你需要调试它！这时为之奈何？

有一种方法你可以给它发一个信号，出一个`coredump`文件，然后用`gdb`来调试`coredump`文件：

```
gdb <coredump_file>
```

显然易见，这并不是一种很好的方式，那么有没有什么办法可以捕获进程并调试呢？你想得到的很有可能都有人实现了——

```
gdb python <process id>
```

```
gdb attach <process id>
```

这两种方式都可以让`gdb`捕获一个进程。因此，我们需要做的只是确定问题进程的`pid`，这个总该没有难度了吧——`top`、`ps`等等一系列命令都可以做到。

# 堆栈查看

查看`C`堆栈信息，用过`gdb`命令的估计都知道怎么做：

```
(gdb) bt
#0 0x0000002a95b3b705 in raise () from /lib/libc.so.6
#1 0x0000002a95b3ce8e in abort () from /lib/libc.so.6
#2 0x00000000004c164f in posix_abort (self=0x0, noargs=0x0) at ../Modules/posixmodule.c:7158
#3 0x0000000000489fac in call_function (pp_stack=0x7fbffff110, oparg=0) at ../Python/ceval.c:3531
#4 0x0000000000485fc2 in PyEval_EvalFrame (f=0x66ccd8) at ../Python/ceval.c:2163
...
```

那么，怎么查看`Python`堆栈呢。安装`python-gdb`之后，`gdb`会提供若干相关的操作。其中`py-bt`就是用来查看`Python`堆栈的：

```
(gdb) py-bt
```

# 线程查看

调试多线程程序，首先总得搞清楚到底有哪些线程吧：

```
(gdb) info threads
 Id Target Id Frame
  37 Thread 0xa29feb40 (LWP 17914) "NotificationThr" 0xb7fdd424 in __kernel_vsyscall ()
  36 Thread 0xa03fcb40 (LWP 17913) "python2.7" 0xb7fdd424 in __kernel_vsyscall ()
  35 Thread 0xa0bfdb40 (LWP 17911) "QProcessManager" 0xb7fdd424 in __kernel_vsyscall ()
  34 Thread 0xa13feb40 (LWP 17910) "python2.7" 0xb7fdd424 in __kernel_vsyscall ()
  33 Thread 0xa1bffb40 (LWP 17909) "python2.7" 0xb7fdd424 in __kernel_vsyscall ()
  31 Thread 0xa31ffb40 (LWP 17907) "QFileInfoGather" 0xb7fdd424 in __kernel_vsyscall ()
  30 Thread 0xa3fdfb40 (LWP 17906) "QInotifyFileSys" 0xb7fdd424 in __kernel_vsyscall ()
  29 Thread 0xa481cb40 (LWP 17905) "QFileInfoGather" 0xb7fdd424 in __kernel_vsyscall ()
  7  Thread 0xa508db40 (LWP 17883) "QThread" 0xb7fdd424 in __kernel_vsyscall ()
  6  Thread 0xa5cebb40 (LWP 17882) "python2.7" 0xb7fdd424 in __kernel_vsyscall ()
  5  Thread 0xa660cb40 (LWP 17881) "python2.7" 0xb7fdd424 in __kernel_vsyscall ()
  3  Thread 0xabdffb40 (LWP 17876) "gdbus" 0xb7fdd424 in __kernel_vsyscall ()
  2  Thread 0xac7b7b40 (LWP 17875) "dconf worker" 0xb7fdd424 in __kernel_vsyscall ()
* 1  Thread 0xb7d876c0 (LWP 17863) "python2.7" 0xb7fdd424 in __kernel_vsyscall ()
```

请注意`*`号哦——它标识的是当前线程。

那么如何切换线程呢？

```
(gdb) thread 37
```

这样就将37号线程设置为当前线程进行调试了。

好，那么怎么查看当前线程的相关信息呢？上节中，`py-bt`可以帮上忙——至少知道线程的执行堆栈。还有一个操作`py-list`，可以清楚看到当前执行到代码的第几行，还有前后若干行的代码可以对照哦：

```
(gdb) py-list
2025         # Open external files with our Mac app
2026         if sys.platform == "darwin" and 'Spyder.app' in __file__:
2027             main.connect(app, SIGNAL('open_external_file(QString)'),
2028                          lambda fname: main.open_external_file(fname))
2029
>2030        app.exec_()
2031         return main
2032
2033
2034     def __remove_temp_session():
2035         if osp.isfile(TEMP_SESSION_PATH):
```

还有更6的，查看所有进程执行位置，非常方便有木有：

```
(gdb) thread apply all py-list
...
 200
 201         def accept(self):
>202             sock, addr = self._sock.accept()
 203             return _socketobject(_sock=sock), addr
 204         accept.__doc__ = _realsocket.accept.__doc__
 205
 206         def dup(self):
 207             """dup() -> socket object

Thread 35 (Thread 0xa0bfdb40 (LWP 17911)):
Unable to locate python frame

Thread 34 (Thread 0xa13feb40 (LWP 17910)):
 197             for method in _delegate_methods:
 198                 setattr(self, method, dummy)
 199         close.__doc__ = _realsocket.close.__doc__
 200
 201         def accept(self):
>202             sock, addr = self._sock.accept()
 203             return _socketobject(_sock=sock), addr...
```

# 参考资料

[https://wiki.python.org/moin/DebuggingWithGdb](https://wiki.python.org/moin/DebuggingWithGdb)
