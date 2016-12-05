---
layout: post
title: "用黑客的方式写博客"
date: 2015-01-18 15:34:34 +0800
comments: true
categories: 
---

{% img /images/gitpress.png %}

<!--more-->

## 开始写博客

``` sh
# 新建post，标题为title
rake new_post["title"]
```

## 常见问题

自定义域名配置`CNAME`不生效，查了很久发现是`Jekyll`失效引起的。
