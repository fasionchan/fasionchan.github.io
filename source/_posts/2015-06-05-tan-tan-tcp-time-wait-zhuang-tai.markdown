---
layout: post
title: "谈谈TCP TIME_WAIT状态"
date: 2015-06-05 21:16:18 +0800
comments: true
categories: Linux TCP
keywords: linux, tcp, timewait
---

学过`TCP`协议的童鞋对以下状态变迁图应该不陌生，本文着重讨论`TIME_WAIT`状态。

{% img /images/2015/tcp-state-machine.gif %}

从图中可以清楚看到：主动关闭的一端最终会进入`TIME_WAIT`状态，并维持`2*MSL`时间，一般情况下为`240`秒。
那么，为什么需要`TIME_WAIT`状态呢？为什么需要持续`2*MSL`时长呢？其中有什么特别的考虑吗？

## 作用

从上图可以看到，进入`TIME_WAIT`状态前，收到对端`FIN`包并且回复`ACK`包。
网络是不可靠的，也就是说最后这个`ACK`包可能被中间链路丢掉了。
对端协议栈在还有收到`ACK`包的情况下，将重传`FIN`包，本端需要对`FIN`包进行相应。
如果此时连接资源已经回收，系统只能响应`RST`包，这时对端就会认为是异常退出。
因此，保留`TIME_WAIT`状态套接字并维持一段时间，就可以正确响应对端重传的`FIN`包，让`TCP`连接优雅关闭。

另一场景是，对端可能还有`FIN`包还在网络逗留(延迟)。
如果新连接复用旧连接的所有要素(包括地址-端口对、`TCP`序列号)，旧连接迟到的`FIN`将**终止新连接**！
当然了，这个场景出现的概率比较低，但还是存在。

## 问题

当一个系统主动关闭大量`TCP`连接时，由于`2MSL`存在，将产生大量的`TIME_WAIT`状态连接。
`TCP`连接需要消耗一定的系统资源，因此，极端情况下将导致活跃连接**响应速度下降**甚至**停止服务**。

发起主动关闭的一端一般是客户端，这时候意味着服务端可以高枕无忧呢？

肯定不是的。服务端也有很多场景会发起主动关闭，最常见的应该是类似`nginx`之类的反向代理了。

{% img /images/2015/nginx-forward.png %}

如上图，`Web`服务经常按照业务逻辑进行划分并独立部署，前端采用`Nginx`做统一接入以及负载均衡。
对`Web`服务器来说(黄色部分)，`Nginx`服务器为客户端。在`Nginx`转发请求完成后，主动关闭不可避免。
这样，当系统存在高并发短连接请求时，`Nginx`服务器上有大量`TIME WAIT`状态连接存在也就不奇怪了。

## 解决方案

### 开启`SYN Cookies`

当`SYN`队列溢出时，采用`Cookie`来处理，继续接受新连接。
`SYN Cookie`可以在一定程度防范`SYN Flood`攻击，详情请参看这里。
配置选项为`net.ipv4.tcp_syncookies`，可以采用`sysctl`或者`proc`伪文件系统两种方式设置。

**`sysctl`方式**。编辑文件`/etc/sysctl.conf`，调整或增加以下行：

```
net.ipv4.tcp_syncookies = 1
```

调整完成后，运行命令`sysctl -p`即可生效。

**`proc`伪文件系统方式**。

```
# 查看当前值
$ cat /proc/sys/net/ipv4/tcp_syncookies
0

# 开启
$ echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# 关闭
$ echo 0 > /proc/sys/net/ipv4/tcp_syncookies
```

### 开启TIME_WAIT重用

开启`net.ipv4.tcp_tw_reuse`选项，将允许将`TIME_WAIT`状态`socket`重新用于新的`TCP`连接。
设置方式也可用通过`sysctl`和`proc`伪文件系统方式，请参考上节，下文不在赘述。

### 开启TIME_WAIT快速回收

开启`net.ipv4.tcp_tw_recycle`选项，快速回收处于`TIME_WAIT`状态的`socket`。

### 修改FIN超时时间

修改`net.ipv4.tcp_fin_timeout`选项。

### 扩大外连端口范围

修改`net.ipv4.ip_local_port_range`选项。这个选项表示向外连接的端口范围。
当然了，扩大范围只能缓解问题，无法彻底解决，另外端口也存在最大值`65000`左右。

`net.ipv4.tcp_max_syn_backlog`，`net.ipv4.tcp_max_tw_buckets`，`net.ipv4.tcp_keepalive_time`。

### 设置SO_LINGER选项

