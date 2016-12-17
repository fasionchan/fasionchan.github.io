---
layout: post
title: "打造炫酷的开发环境"
date: 2016-12-17 21:20:34 +0800
comments: true
categories: Mac Vim
---

无图无真相，这篇文章的主要目的是打造如下所示的终端开发环境，涉及`terminal`、`shell`以及`editor`等，着重于配色。

{% img /images/zsh.png %}

<!--more-->

`terminal`不必多说，`iTerm2`是`Mac`下的不二选择，不解释了。如果你在`windows`上开发，推荐使用`Putty`，当然了跟`iTerm2`不是同个级别的东西。

关于`shell`，我之前一直在`bash`上瞎折腾，认为最普适的就是最好的。毕竟在绝大多数机器环境上，`bash`唾手可得，不需要额外安装。但是，`bash`没有足够丰富的现成配置方案可用，自己山寨要花很多精力而且效果也不是很好。`zsh`就不一样，名字就透着`shell`终结者的意思。`zsh`比`bash`要好用，而且有现成的配置方案，特别是`oh-my-zsh`，非常好用。

{% img /images/vim.png %}

关于`editor`，向来都有`vim`或`emacs`的争议。我只使用过`vim`，也没法评论。争议是没有必要的，如果都没用过，选一个学一下就是；如果用过其中之一，维持现状即可；想两个都体验一下也行，找到适合自己的才是最重要的。我习惯使用`vim`，下文便以`vim`为例好了。

# iTerm2

`iTerm2`是`Mac`下一款优秀的终端软件，功能比内置的`iTerm`强很多。关键是，这款应用还是免费的！赶紧到[www.iterm2.com](https://www
.iterm2.com/)下载吧！安装后，它是这个样子的：

{% img /images/iterm2-default.png %}

呃，颜色货不对版呀！对的，这是`iTerm2`默认的颜色，别心急，接下来我们来调整配色方案。

## 配色

[solarized](http://ethanschoonover.com/solarized)是一套好看又不伤眼的配置方案，很多人推荐，目前托管在[github](https://github.com/altercation/solarized)上。接下来，我们看看怎么给`iTerm2`加上`solarized`配色。

首先，将`solarized`下载到本地：

```
git clone git@github.com:altercation/solarized.git
```

可以看到，里面包罗万象，这里我们要使用的是`iterm2-colors-solarized`目录下的，包括`Solarized Dark.itermcolors`和`Solarized Light.itermcolors`两个配置文件。

{% img /images/iterm2-color.png %}

打开`Preferences->Profiles->Color`面板，在`Color Presets`中将以上两个配置方案导入，然后选择`Solarized Dark`或者`Solarized Light`即可。一般推荐使用`Solarized Dark`，`Solarized Light`有种亮瞎的感觉。配色搞定后，终端是这样子的：

{% img /images/iterm2-solarized.png %}

看着这样柔和的颜色，眼睛就不会很容易就累了。

# zsh

`shell`提示符高亮及`git`代码状态展示是怎么实现的呢？这就要说到强大的`zsh`了。不废话，`Mac`下应该是内置的了；`Linux`下运行`apt-get install zsh`即可安装；`Windows`下烫烫烫烫烫。

## 配置

接下来，用[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)来武装`zsh`，一行命令搞定：

```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

## 主题

`oh-my-zsh`中提供了多套主题可供选择，有不同的输出样式及配色。默认应该是[robbyrussell](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes#robbyrussell)，感觉中规中矩没啥亮点。翻看了一下，发现了[agnoster](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes#agnoster)主题，感觉非常入眼。

接下来，编辑`~/.zshrc`，找到变量`ZSH_THEME`将其赋值改为`agnoster`即可。操作完毕，发现离目标很近了：

{% img /images/zsh-font-fault.png %}

除了有些字体显示异常，什么鬼？

## 字体

为了显示正常，需要安装`powerline`字体，方法如下：

```
git clone git@github.com:powerline/fonts.git
cd fonts
./install
```

然后，在`iTerm2->Preferences->Profiles->Text`面板中将`Non-ASCII Font`改成`Roboto Mono Powerline`，显示就正常了！

{% img /images/iterm2-text.png %}

# vim

 `vim`配置这个话题有点大，包括：代码静态检查、高亮、文件树等等。理论上，通过插件，可以将`vim`打造成一个`IDE`，后续有空专门介绍一下。这节聚焦在：`vim`配色主题。

这里还是使用`solarized`配色方案：

```
mkdir -p ~/.vim/colors
cd solarized
cp vim-colors-solarized/colors/solarized.vim ~/.vim/colors/
```

编辑`~/.vimrc`文件，加上：

```
syntax enable
set background=dark
colorscheme solarized
```

这样，打开`vim`看到的就是文件开头图片显示的效果了。

# 自动化配置

如果只在自己的电脑上配置，手工操作一遍是没有问题的。但是，现实是，每个人可能都有多个不同的开发环境：自己的`PC`、公司的`PC`、线上的机器等等。每台机器都这样手工配置一遍，繁琐而且容易出错，感觉不敢想。

程序猿是一种有懒癌的动物，一件事情绝不重复做第二遍。如果，可以将以上配置操作脚本化，在一个新的机器环境，跑一下脚本就搞定一切，那多好！这就是我维护一个`conf`代码仓库的初衷。

`conf`提供一个脚本，`install`用来按照各种配置。比如，运行以下命令，`zsh`和`vim`的配置就安装完毕了。

```
cd conf
./install zsh
./install vim
```

当然了，`conf`完全是按照我个人的喜好定制的，可能并不适合你。尽管如此，你可以参考它打造属于自己的自动化配置工具。**时间应该用来做有趣的事情，而不是浪费在简单的重复中**。
