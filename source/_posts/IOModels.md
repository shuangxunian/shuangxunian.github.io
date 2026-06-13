---
title: I/O 模型：阻塞、非阻塞与多路复用（select/poll/epoll）
excerpt: 梳理五种 Unix I/O 模型的核心差异，重点对比 select/poll/epoll 的实现原理与性能瓶颈，以及 epoll 的 LT/ET 两种触发模式。
date: 2026-05-08
categories:
- 开发技术
tags:
  - IO模型
  - epoll
  - 网络编程
  - 操作系统
  - 面试
---

以网络读取为例，一次 I/O 分两个阶段：
1. **等待数据就绪**（数据从网卡到内核缓冲区）
2. **数据从内核拷贝到用户空间**

不同 I/O 模型的区别，主要在第一阶段怎么处理。

## 阻塞 I/O（Blocking I/O）

```
用户进程        内核
  |--- read() --->|
  |    (阻塞等待) |  等待数据就绪
  |               |  数据拷贝到用户空间
  |<-- 返回数据 --|
```

- 调用 `read()` 后进程挂起，直到数据准备好并拷贝完才返回
- 简单直观，但一个线程同一时间只能处理一个连接

## 非阻塞 I/O（Non-blocking I/O）

```
用户进程        内核
  |--- read() --->|
  |<-- EAGAIN ----|  数据未就绪，立即返回
  |--- read() --->|
  |<-- EAGAIN ----|  继续轮询...
  |--- read() --->|
  |<-- 数据 ------|  就绪了，拷贝并返回
```

- `read()` 立即返回，数据没好就返回 `EAGAIN`
- 需要应用层不断轮询（busy-wait），**CPU 空转严重**
- 单独使用很少见，通常配合 I/O 多路复用

## I/O 多路复用（I/O Multiplexing）

核心思想：**用一个线程同时监听多个 fd，哪个就绪就处理哪个**。

### select

```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- 把所有 fd 放进 `fd_set`（本质是 bitmap），调用 `select` 阻塞等待
- 有 fd 就绪后返回，应用层**遍历所有 fd** 找出哪个就绪
- 缺点：
  - fd 数量上限 **1024**（`FD_SETSIZE`）
  - 每次调用都要把 fd_set 从用户空间**拷贝到内核**
  - 返回后需要 **O(n) 遍历**找就绪 fd

### poll

- 用链表替代 bitmap，**去掉了 1024 限制**
- 其他缺点和 select 一样：每次调用仍需拷贝所有 fd，仍需 O(n) 遍历

### epoll（Linux 特有，现代首选）

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);  // 注册/修改/删除
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

- `epoll_ctl` 把 fd 注册到内核的红黑树，**只注册一次**，不用每次调用都拷贝
- 内核维护一个**就绪链表**，fd 就绪时直接加入
- `epoll_wait` 只返回**就绪的 fd**，不需要遍历全部

| 对比 | select | poll | epoll |
|------|--------|------|-------|
| fd 上限 | 1024 | 无限制 | 无限制 |
| fd 拷贝 | 每次调用全量拷贝 | 每次调用全量拷贝 | 只注册一次 |
| 就绪查找 | O(n) 遍历 | O(n) 遍历 | O(1) 直接返回就绪列表 |
| 适用场景 | 连接数少 | 连接数少 | **高并发首选** |

**epoll 两种触发模式：**

- **LT（水平触发，默认）**：fd 就绪后，只要缓冲区还有数据，每次 `epoll_wait` 都会通知。没读完没关系，下次还会提醒。
- **ET（边缘触发）**：只在状态变化时通知一次，必须一次性把数据读完（循环读到 `EAGAIN`）。效率更高，但编程复杂，Nginx 用的就是 ET 模式。

## 总结

> 阻塞 I/O 简单但一线程一连接；非阻塞需要轮询浪费 CPU；多路复用用一个线程监听多个连接——select/poll 有 O(n) 遍历问题，epoll 用红黑树+就绪链表做到 O(1)，是高并发服务器的标准选择。
