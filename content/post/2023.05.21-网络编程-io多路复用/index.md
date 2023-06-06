+++
author = "baoguli"
title = "io多路复用"
date = "2023-5-21"
description = "Custom Test Page"
tags = [
    "网络编程",
]
categories = [
    "syntax",
]
aliases = ["migrate-from-jekyl"]
image = ""
+++

### 什么是 io 多路复用

单线程或单进程同时监测多个文件描述符是否进行io操作的技术就是io多路复用。

在了解io多路复用之前我们要先知道 BIO 和 NIO 。

### BIO

**BIO** ：同步阻塞 io ( block io )。同步阻塞 io 是我们在发出 io 操作后，一直等待返回结果，在此期间程序不做任何事情。

缺点：如果是在单线程环境下，由于是阻塞地获取结果，只能有一个客户端连接。而如果是在多线程环境下，需要不断地新建线程来接收客户端，这样会浪费大量的空间。

### NIO

**NIO** ：同步非阻塞 io (non-block io)。非阻塞 io 是发出 io 操作后，不等待返回结果，继续进行下一步操作。

大致步骤：

创建 socket ，绑定端点，进行监听；

将文件描述符标识为非阻塞；

进行循环，监听到客户端连接，返回一个新的 socket ，并将它存储在 list 列表中；

遍历 list ，检查是否有读写事件；

缺点：需要遍历 list 中的每个元素查看有无监听的事件发生，时间复杂度为 O ( n ) ，浪费 CPU 资源。

### io 多路复用

io 多路复用由 select、poll、epoll 实现。

### select poll

select 和 poll 函数模型大致实现：


多路复用器 ( 内核 ) 通过进程得知所有的 socket 号，从而获取每一个 socket 的状态；

当程序获取到某个 socket 号有事件发生了，则在该 socket 号上进行处理对应的事件；


select 和 poll 函数模型的主要区别就是前者底层是数组，有最大连接数的限制，后者是链表，无最大连接数的限制。


缺点：

同样与 NIO 相同，需要遍历所有 socket ，复杂度 O ( N ) ；

每次都要根据进程不断重复从用户态向内核态传递所有的 socket 号去遍历每一个 socket ，获取它们的状态。浪费资源与效率；

### epoll

epoll 函数模型：

epoll 函数模型主要是调用了三个函数：epoll_create() , epoll_ctl() , epoll_wait()；


大致流程：

通过 epoll_create() 函数创建一个文件，返回一个文件描述符 fd ；

创建 socket 号，绑定端点，监听事件，标记为非阻塞；

通过 epoll_ctl() 函数将该 socket 号以及需要监听的事件写入 fd 中；

循环调用 epoll_wait() 函数进行监听，返回已经就绪事件序列的长度 ( 返回0则说明无状态，大于0则说明有n个事件已就绪 ) ；


优点：不需要再遍历所有的 socket 号来获取每一个 socket 的状态，只需要管理活跃的连接。即监听在通过 epoll_create() 创建的文件中注册的 socket 号以及对应的事件。只有产生就绪事件，才会处理，所以操作都是有效的，为 O ( 1 ) 。

