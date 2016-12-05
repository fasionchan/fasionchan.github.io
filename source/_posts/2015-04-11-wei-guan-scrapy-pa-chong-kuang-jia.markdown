---
layout: post
title: "围观Scrapy爬虫框架"
date: 2015-04-11 11:22:24 +0800
comments: true
categories: 
---

对于Scrapy这个爬虫框架，先前也是知道有这么个东西存在，仅此而已。最近在面试时，发现很多人写过Python爬虫采集数据，所用的框架几乎都是Scrapy。今天刚好闲着没事做，就来玩玩Scrapy呗。

## 简介

## 框架

{% img /images/scrapy.png %}

<!--more-->

## 安装

需要先安装几个依赖库：`apt-get install libxml2-dev libxslt1-dev python-dev libffi-dev`。采用PIP方式安装Scrapy，被依赖的Twisted也将自动安装：`pip install scrapy`。安装时，可能会报libxml头文件找不到：`fatal error: libxml/xmlversion.h: No such file or directory`。在我的机器上，运行`dpkg -L libxml2-dev`看看先前安装的依赖包`libxml2-dev`所有文件安装点，发现其头文件位于`/usr/include/libxml2/libxml`，将其软链到`/usr/include/`便解决问题。

## 组件

## 示例
