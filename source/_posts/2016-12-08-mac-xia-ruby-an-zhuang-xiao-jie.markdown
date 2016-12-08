---
layout: post
title: "Mac下Ruby安装小结"
date: 2016-12-08 22:29:43 +0800
comments: true
categories: Mac Ruby
---

最近，重新拿起博客，发现已有一年多没有动过的`Octopress`跑步起来了，囧。并不意外，每次`Mac`一升级，总有一些东西会挂掉，久而久之习惯了。

如何让`Octopress`重新跑起来呢？重新看安装文档，大头只是安装一个`ruby`而已。但是，由于对`ruby`并不熟悉，中间还是踩了个大坑。因此，有必要记一下，免得以后再犯。

<!--more-->

# 安装rbenv

`rbenv`是什么鬼？

我们知道，系统本来已经有一个`ruby`了，不信命令行运行`which ruby`看看。这个是系统自带的，一般版本都是极其老的。然而，我们在用一些软件环境时，却是需要某个特定版本的`ruby`的。这时，你可以选择重新编译一个，然后进行安装。问题是，安装到系统这个动作太暴力了，影响范围略大，可能会搞挂其他东西。如果可以安装多个版本的`ruby`，与系统独立，按需使用，那敢情好。`rbenv`就是这样的工具~

好吧，那么要怎么安装呢？`Mac`下，一般用`brew`安装软件，`rbenv`也不例外：

```
brew update
brew install rbenv
brew install ruby-build
```

# 安装ruby

`rbenv`完成安装后，`ruby`的安装就没有任何难度了：

```
rbenv install 1.9.3-p0
rbenv local 1.9.3-p0
rbenv rehash
```

好吧，这么说来，写这篇文章有什么意义呢？别急，下面不是还有**坑**一节嘛~

# 坑

上面的安装步骤正确操作完，我发现`ruby`缺不是用`rbenv local`选定的版本！我运行`which ruby`看到还是用系统的，什么鬼！

确定`rbenv`没生效，但暂时不知道为什么。我猜`rbenv`应该是通过`PATH`环境变量生效的，但是看了一下并没有什么变化。`rbenv help`也看不到任何关于生效的内容，无奈只能`Google`了。

`F*ck!`确实需要做点什么才能生效，`rbenv init`就是需要做的内容，但是`rbenv help`一个字也没提？

```
$ rbenv init
# Load rbenv automatically by appending
# the following to ~/.bashrc:

eval "$(rbenv init -)"

```

`shell`下运行`eval "$(rbenv init -)"`后，再运行`which ruby`就看到确实生效了！这个操作最好根据指引放到`.bashrc`里，这样就不要要每次都要运行一遍。
