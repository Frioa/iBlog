---
title: Linux 消息队列
date: 2023-02-12 21:54:24
tags:
- Linux
- 学习
---

> 理解消息队列大致如何工作即可：

<!--more-->

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e9da15dd6c24eb3899474ae7051f556~tplv-k3u1fbpfcp-watermark.image?)

- 消息是存储在内核空间中的链表
- 每个消息都是由消息的标志符表示


## 总结：
1. 允许有多个进程同时读写消息，要求通信双方约定好消息类型与数据大小。
2. 通信过程：进程A发送一个消息给内核空间，进程 B 通过类型读取消息。