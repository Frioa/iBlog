---
title: Linux 消息队列
date: 22023-02-12 21:42:31
tags:
- Linux
- 学习
---

> 理解管道大致如何工作即可：

<!--more-->

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8f526cd54a846389baed2318e044c23~tplv-k3u1fbpfcp-watermark.image?)


1. Linux 内部构建了一个在内核空间管道概念
   分为两部分：
- 写段：进程 A 将数据写入管道，反之亦然
- 读读：进程 B 从管道中读数据，反之亦然


2. 整个过程中数据会经过两次复制
3. 使用时需要考虑：
   1. 管道大小
   2. 阻塞与非阻塞
   3. 缓冲区大小（为提高 IO 速度），如果数据超出缓冲区大小则可能阻塞
   4. 传输无格式字节流，需要事先定义好数据格式。
