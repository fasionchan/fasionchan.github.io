---
layout: post
title: "Python调试终端"
date: 2016-12-09 18:58:43 +0800
comments: true
categories: Python
---

后端服务程序开发运营过程中，难免会遇到一些BUG疑难杂症，是日志输出等调试手段无法定位的。
如有有一个工具，可以连上服务程序，查询服务中间状态或者修改程序变量，势必加速问题定位及解决。

本模块就是您想要的工具~

<!--more-->

# 安装

模块代码已经在`github`上开源：[libase.server.console](https://github.com/fasionchan/libase/blob/master/libase/server/console.py)，可以直接把代码`clone`下来安装。

```
$ git clone https://github.com/fasionchan/libase.git
$ cd libase
$ python setup.py install
```

当然了，`libase`也已经发布到[PyPI](https://pypi.python.org/pypi)上了。因此，更方便的安装方式是使用`pip`：

```
pip install libase
```

# 服务接入

需要远程调试的程序可以用`start_console_server`快捷启动一个远程调试终端服务。
然后，使用`pyconsole`命令便可以连上该程序，并初始化一个`Python`控制台用于调试了。

```
import time

from libase.server.console import start_console_server

counter = 0

start_console_server()

while True:
    print counter
    counter += 1
    time.sleep(1)
```

例子是一个计算程序，使用`start_console_server`接入远程调试终端服务，端口为默认值`4444`。

### 连接调试

`libase`提供一个用于连接远程调试终端的程序`pyconsole`，以访问上述程序为例：

``` html
$ pyconsole 4444
Python 2.7.3 (default, Jan  2 2013, 13:56:14)
[GCC 4.7.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
(ConsoleProxy)
>>> print main.counter
93
>>> main.counter = 0
>>> print main.counter
1
>>> print main.counter
2
```

注意到，默认已经将进程的`__main__`模块引入到当前名字空间，名为`main`，其效果等同于在`pyconsole`中执行`import __main__ as main`。

# 高级用法

## 接口文档

### start\_console\_server

**start\_console\_server(port=4444, addr='localhost', 'code'='')**

接入远程调试终端服务，返回一个`ConsoleServer`对象。

- port 远程调试终端服务监听的端口，默认为`4444`
- addr 远程调试终端服务监听的地址，可以为`IP`或者机器名，默认为`localhost`
- code 初始化代码，可用于预先引入一些对象到终端所在名字空间

### run\_console\_proxy

**run\_console\_proxy(port=4444, addr='localhost')**

以给定地址端口信息，连上接入远程调试终端的服务，并启动一个`Python`控制台。

- port 远程调试终端所在端口，默认为`4444`
- addr 远程调试中断所在机器地址，可以是`IP`或者机器名，默认为`localhost`

## 工具命令

### pyconsole

用法：pyconsole [port] [addr]

- port 远程调试终端所在端口，默认为`4444`
- addr 远程调试中断所在机器地址，可以是`IP`或者机器名，默认为`localhost`

该命令，只是把`run_console_proxy`命令化，实现如下：

```
import sys
import rlcompleter

from libase.server.console import run_console_proxy

if __name__ == '__main__':
    run_console_proxy(*sys.argv[1:])
```
