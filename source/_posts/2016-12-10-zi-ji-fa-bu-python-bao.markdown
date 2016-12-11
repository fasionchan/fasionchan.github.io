---
layout: post
title: "自己发布Python包"
date: 2016-10-23 18:50:04 +0800
comments: true
categories: Python
---

# 注册

先在`Python`官网注册一个账号，地址是：[https://pypi.python.org/pypi](https://pypi.python.org/pypi)。

<!--more-->

# 配置

账号注册后，就可以用其来登记新建的`Python`包，以及上传包更新。上传新包一般由`twine`命令完成，`twine`需要知道`Python`仓库地址以及账号信息。因此，可以将这些信息写在配置文件`~/.pypirc`里，这样运行命令时便不需要再次输入了。配置如何写呢？以`foo`为用户名，`bar`为密码为例：

```
[distutils]
index-servers=pypi

[pypi]
repository = https://pypi.python.org/pypi
username = foo
password = bar
```

其中，密码也可以不写进配置，这样每次运行`twine`时，将提示输入密码。

# 准备

编写`Python`包的过程，这里就不细说了。本文的重点是怎么将自己的`Python`包发布出去，然后可以用`pip`命令安装。

`Python`包的实现方式可以参考。其实无非就是规划好目录结构，然后编写`setup.py`文件：

```
#!/usr/bin/env python
# -*- encoding=utf8 -*-

'''
FileName:   setup.py
Author:     Fasion Chan
@contact:   fasionchan@gmail.com
@version:   $Id$

Description:

Changelog:

'''

VERSION = '1.0'

from setuptools import (
    setup,
    )

setup(
    name='libase',
    version=VERSION,
    author='Fasion Chan',
    author_email='fasionchan@gmail.com',
    packages=[
        'libase',
        ],
    scripts=[
        ],
    package_data={
        },
    install_requires=[
        ],
    )
```

# 登记

运行命令`python setup.py egg_info`生成包信息，找到`xxxx.egg_info`目录下`PKG-INFO`文件，在`Python`官网[提交表单](https://pypi.python.org/pypi?%3Aaction=submit_form)上传即可。

# 构建

运行命令`python setup.py sdist`进行构建并打包。完成之后，在`dist`目录下可以看到形如`xxxx-1.0.tar.gz`的压缩包。

# 上传

运行命令`twine upload dist/*`便可将构建的所有包上传，这时便大功告成了！

# 应用

运行命令`pip install xxxx`便可以安装你发布的`Python`包了！你可以发布任何你想发布的东西，任何人也可以安装任何你发布的东西，成就感杠杠的有木有！

我们提倡自由分享的精神，有用的代码无私奉献出来可以节约很多人很多时间！`Life is short, use Python!`

当然了，努力提升自己，分享高质量代码，不要坑别人哦~
