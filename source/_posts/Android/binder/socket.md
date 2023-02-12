---
title: Linux Socket
date: 2023-02-12 22:06:45
tags:
- Linux
- 学习
---

> 理解 Socket 大致如何工作即可：

<!--more-->



![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2082de3a6414ed48ea58330a91fa838~tplv-k3u1fbpfcp-watermark.image?)

> 内核空间：拥有**两个缓冲区**，一个负责读，另一个负责写

## 工作流程：
发送与接收消息，都需要从一个 `Socket` 缓冲区拷贝到另一个 `Socket` 缓冲区中，最后由接收进程读取数据。

## 特点
- C/S 架构模式
- 数据经过两次拷贝
- 安全性较低，如：进程A，B同时使用一个 Socket