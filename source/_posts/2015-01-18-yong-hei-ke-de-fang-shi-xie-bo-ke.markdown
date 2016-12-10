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

## 常用语法

### 关键字

```
---
layout: post
title: "Linux文件描述符"
date: 2016-09-20 22:16:26 +0800
comments: true
categories: Linux
keywords: linux, file descriptor, 文件描述符
---
```

### 图片

```
{% img /images/bei-hai-gong-yuan-bai-ta.jpg %}
```

## 常见问题

自定义域名配置`CNAME`不生效，查了很久发现是`Jekyll`失效引起的。
