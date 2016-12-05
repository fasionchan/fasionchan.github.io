---
layout: post
title: "SSH高级用法"
date: 2015-03-22 10:21:52 +0800
comments: true
categories: 
---

## Keep Alive

在某些网络环境下，SSH会话空闲一小段时间后就会断掉。可能是因为中间有个NAT网关，检查到空闲TCP连接后便无情Kill。一暂停操作SSH连接就断实在不能忍！

SSH有个配置选项，每隔一段时间就发一个空包使TCP连接保持活跃，这样连接就不会无故断掉了。这个选项就是：`ServerAliveInterval`，配置值是一个整数，表示发送空包的周期，单位是秒。

<!--more-->

这个选项可以作为全局配置写在`/etc/ssh/ssh_config`，影响每个用户的每一个SSH连接：

``` text
ServerAliveInterval 60
```

也可以作为用户配置写在当前用户的配置文件`~/.ssh/config`里（如果这个配置文件不存在，可以手动创建）：
``` text
Host *
    ServerAliveInterval 60
```

`Host`可以指定对哪些服务器应用该配置，`*`通配符可以匹配所有服务器，`*.hostname.com`则可以匹配所有名字以`.hostname.com`结尾的服务器。

## 代理传递

假设用ssh登陆管理两台机器A和B：把公钥部署上去，然后本地就可以不通过密码验证登陆上去。如果要把A机器上一个文件传到B机器上，那么，比较显然易见的方式是：先用scp把文件从A拉到本地，然后再从本地传到B。这显然比较繁琐，如果A能够ssh到B，那就方便很多。但不论是把本地私钥部署到A机器还是在A机器生产私钥都是不妥的，万一A被攻破，那么B也就会遭殃。

这时候ssh代理传递（ssh agent forwarding）就派上用场了。通过代理传递，从本地ssh到机器A，那么在A就可以ssh到任何本地可以ssh的机器。假设这时ssh到B，A借助本地的私钥信息完成验证，而A上无须部署任何私钥。这个过程就像把本地的ssh私钥传递到机器A。

### 设置方法

只需要本地ssh配置文件~/.ssh/config加上如下配置：

``` text
Host server1.example.com
    ForwardAgent yes
```

这个配置的含义是，本地ssh可以传递到`server1.example.com`上，也就是ssh到`server1.example.com`后，你可以在服务器上ssh到其他机器。这里需要注意的是：`server1.example.com`必须是跟你的ssh命令保持一致`ssh server1.example.com`！如果ssh使用的是IP，那么配置里面也必须填IP；如果把`example.com`设置成域，ssh使用机器名`server1`，那么配置文件也必须使用机器名，尽管指的都是同一台机器。

## 反向代理

经常需要从家里连到公司内网的电脑，在没有VPN的情况下该怎么办呢？NAT技术解决了内网机器通过网关访问外网，但是在网关没有映射相关端口的情况下，从外网访问内网服务将是不可能的。

本质上，SSH反向代理就是把一台机器的某个服务端口映射到另一台机器上。如果采用SSH反向代理，把内网机器`A`的SSH端口映射到外网机器`B`的某个端口，不就解决问题了么？那么，怎么配置SSH反向代理呢？


在`A`机器上执行`ssh -fNR addrB:portB:addrA:portA user@addrB`，将在`B`机器地址`addrB`上开启`portB`端口，并将请求映射到本地（`A`机器）地址`addrA`上的`portA`端口。另外，`user`为`B`机器上的用户，`user@addrB`用来登录`B`机器。完成SSH反向代理后，访问`addrB`的`portB`端口即是访问`addrA`的`portA`端口。具体用法请参考`man ssh`文档。
