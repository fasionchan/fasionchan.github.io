---
layout: post
title: "Linux通用IO模型"
date: 2016-09-17 21:54:29 +0800
comments: true
categories: Linux IO
keywords: linux,syscall,io,系统调用
---

学习`Linux`系统编程，文件I/O是一个不错的切入点。首先，日常操作中或多或少都使用过文件，有一定的概念；其次，文件I/O可以由几个最最基础的系统调用完成，降低入门理解难度。

# 基础系统调用

`Linux`下`I/O`操作是通用化的，不仅仅可以用来操作文件输入输出，还可以用来操作管道、`FIFO`、`socket`、终端设备等。将设备抽象成一个文件，用`I/O`操作控制设备是类`Unix`系统一大特色。

<!--more-->

最最基础的`I/O`操作系统调用包括：

- **fd = open(pathname, flags, mode)**
- **rlen = read(fd, buf, count)**
- **wlen = write(fd, buf, count)**
- **status = close(fd)**

注意到，相关系统调用都围绕文件描述符`fd`展开。文件描述符在下节详细介绍。

另外，需要特别提示一下，系统调用函数接口（其实是库函数封装）可以通过`man`命令查看详细的参数、返回值及用法说明。比如，终端下运行`man read`，就可以看到`read`这个系统调用的相关文档说明：

![man open](http://upload-images.jianshu.io/upload_images/2740477-bb2610e81244463b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 文件描述符

`I/O`操作系统调用都以文件描述符(一个非负整数)，指代打开的文件。每个进程都有一个打开文件表，可以理解成一个数组，文件描述符可以理解成数组的下标。相关`I/O`操作系统调用以文件描述符为参数，便可以通过数组访问定位到指定的文件对象，进而进行`I/O`操作。

一般情况下，进程的标准输入输出由3个特定的文件描述符指定，列举如下：

|文件描述符|用途|POSIX名称|stdio流|
|:----|:----|:----|:----|
|0|标准输入|STDIN_FILENO|stdin|
|1|标准输出|STDOUT_FILENO|stdout|
|2|标准错误|STDERR_FILENO|stderr|

正常情况下，程序在开始运行之前，由`shell`准备好这3个文件描述符。更准确的说法是，程序继承了`shell`文件描述符的副本，一般是指向`shell`所在的终端。当然了，可以通过在`shell`中对输入/输出进行重定向或者在程序启动后关闭并重新打开文件描述符，修改文件描述符指向。

# 实战案例

## 读文件

首先，通过一个例子，探索一下如何通过`I/O`操作系统调用读取文件内容并输出：

```
#include <stdio.h>
#include <fcntl.h>

int
main(int argc, char *argv[])
{
    // 打开文件
    int fd = open("a.txt", O_RDONLY);
    if (-1 == fd)
    {
        perror("fail to open a.txt");
        return -1;
    }

    // 准备缓冲区并读文件
    char buf[1024];
    int rlen = read(fd, buf, sizeof(buf)-1);
    if (-1 == rlen)
    {
        perror("fail to read a.txt");
        return -2;
    }

    // 输出内容
    buf[rlen] = '\0';
    printf("read %d bytes, which is:\n", rlen);
    printf("%s\n", buf);

    // 关闭文件
    int status = close(fd);
    if (-1 == status)
    {
        perror("fail to close a.txt");
        return -3;
    }

    return 0;
}
```

## 写文件

```
#include <stdio.h>
#include <string.h>
#include <fcntl.h>

int
main(int argc, char *argv[])
{
    // 打开文件
    int fd = open("a.txt", O_WRONLY);
    if (-1 == fd)
    {
        perror("fail to open a.txt");
        return -1;
    }

    // 准备数据并写到文件
    char data[] = "hello, this is line written by write syscall\n";
    int wlen = write(fd, data, strlen(data));
    if (-1 == wlen)
    {
        perror("fail to read a.txt");
        return -2;
    }

    // 输出成功写入字节数
    printf("write %d bytes of %d total\n", wlen, strlen(data));

    // 关闭文件
    int status = close(fd);
    if (-1 == status)
    {
        perror("fail to close a.txt");
        return -3;
    }

    return 0;
}
```
