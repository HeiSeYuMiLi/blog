+++
author = "baoguli"
title = "初识网络编程"
date = "2023-5-18"
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

基于 **TCP** 的连接，服务端和客户端的网络通信需要做三件事：

- 服务端与客户端进行连接

- 服务端与客户端之间传输数据

- 服务端与客户端之间断开连接


首先需要解决的问题是如何建立连接。

在 socket 编程中，服务端和客户端是通过 **socket** 进行连接的。

每一条 TCP 连接有两个端点，这个连接的端点就叫做**套接字(socket)**或**插口**。根据 RFC 793 的定义：端口号拼接到 IP 地址即构成套接字。所以：

>    套接字 socket = ( IP 地址 ：端口号 )

每一条 TCP 连接唯一地被通信两端的两个端点所确定。即：

>    TCP 连接 ::= { socket1, socket2 } = {( IP1: port1 ), (IP2:port2)}


想要建立连接，服务端需要做：

- 创建套接字 socket()

- 将  socket 与指定的 IP 和端口进行绑定

- 在绑定的端口处监听请求 accept()

而客户端会简单一些：

- 创建套接字 socket()

- 在指定的 IP 地址和端口处使用 socket 进行连接 connect()

连接建立后，两端便可进行通信，也就是收发数据。

- 接收数据：使用套接字接受数据并存在 buff 中

- 发送数据：使用套接字将 buff 中的数据发出去

接发数据结束后，通过断开连接来结束通信。
