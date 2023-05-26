+++
author = "baoguli"
title = "ASIO-基础介绍"
date = "2023-5-23"
description = "介绍BOOST.ASIO基本的使用方法"
tags = [
    "ASIO",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  

## ASIO

Boost.Asio 是一个跨平台的、主要用于网络和其他一些底层输入/输出编程的 C++ 库。

Boost.Asio 在网络通信、COM 串行端口和文件上成功地抽象了输入输出的概念。你可以基于这些进行同步或者异步的输入输出编程。

作为一个跨平台的库，Boost.Asio 可以在大多数操作系统上使用。能同时支持数千个并发的连接。其网络部分的灵感来源于伯克利软件分发 ( BSD ) socket ，它提供了一套可以支持传输控制协议 ( TCP ) socket 、用户数据报协议 ( UDP ) socket 和 Internet 控制消息协议 ( IMCP ) socket 的 API ，而且如果有需要，你可以对其进行扩展以支持你自己的协议。

## 基本的使用

### 创建端点 endpoint

```C++
//客户端
int client_endpoint() {
	std::string raw_ip_address = "127.4.8.1";
	unsigned short port_num = 3333;
	boost::system::error_code ec;
	boost::asio::ip::address ip_address = boost::asio::ip::address::from_string(raw_ip_address, ec);
	if (ec.value() != 0) {
		std::cout << "Failed\n";
		return ec.value();
	}
	boost::asio::ip::tcp::endpoint ep(ip_address, port_num);
	return 0;
}
```

```C++
//服务端
int server_endpoint() {
	unsigned short port_num = 3333;
	//any表示可以接收任何v6的地址
	boost::asio::ip::address ip_address = boost::asio::ip::address_v6::any();
	boost::asio::ip::tcp::endpoint ep(ip_address, port_num);
	return 0;
}
```

服务端可以不指明地址，而是接收任何地址，如 tcp::v4() 、 tcp::v6() 等等。

### 创建 socket

```C++
boost::asio::io_context ioc;
boost::asio::ip::tcp::socket sock(ioc);
```

**io_context** ( 之前的版本是 io_service ) 对象是 asio 框架中的调度器，所有的异步事件都是通过它来分发处理的。

### 创建接收器 acceptor

```C++
int create_acceptor_socket() {
	boost::asio::io_context ioc;
	boost::asio::ip::tcp::acceptor acceptor(ioc,boost::asio::ip::tcp::endpoint(boost::asio::ip::tcp::v4(),3333));
	return 0;
}
```

服务端打开端口，使用接收器监听，属于被动连接。

### 客户端通过端点进行连接

```C++
int client_connect_to_end() {
	std::string raw_ip_address = "192.168.1.124";
	unsigned short port_num = 3333;
	try {
		boost::asio::ip::tcp::endpoint ep(boost::asio::ip::address::from_string(raw_ip_address), port_num);
		boost::asio::io_context ioc;
		boost::asio::ip::tcp::socket sock(ioc, ep.protocol());
		sock.connect(ep);
	}
	catch (boost::system::system_error& e) {
		std::cout << "Failed\n";
		return e.code().value();
	}
	return 0;
}
```

客户端主动连接指定端点的服务端。

### 客户端通过域名连接

```C++
int client_connect_to_end_by_dns() {
	//客户端不知道服务端地址，而是通过域名进行链接（少用）
	std::string host = "www.baidu.com";
	std::string port_num = "80";
	boost::asio::io_context ioc;
	//生成一个查询器
	boost::asio::ip::tcp::resolver::query query(host, port_num, boost::asio::ip::tcp::resolver::query::numeric_service);
	//生成解析器
	boost::asio::ip::tcp::resolver resolver(ioc);
	try {
		//解析 一个域名可能对应多个ip，所以解析器返回一个迭代器，指向解析到的所有ip
		boost::asio::ip::tcp::resolver::iterator it = resolver.resolve(query);
		boost::asio::ip::tcp::socket sock(ioc);
		//需要使用自由函数connect
		boost::asio::connect(sock, it);
	}
	catch (boost::system::system_error& e) {
		std::cout << "Failed\n";
		return e.code().value();
	}
}
```

### 服务端接收客户端连接

```C++
int acceptor_accept_connection() {
	//创建接收器后，接收器会监听客户端链接，未来得及处理的链接放在缓冲队列中
	//队列大小 由于tcp算法 实际数值大于30（或是30*2）
	const int BACKLOG_SIZE = 30;
	unsigned short port_num = 3333;
	boost::asio::ip::tcp::endpoint ep(boost::asio::ip::address_v4::any(), port_num);
	boost::asio::io_context ioc;
	try {
		boost::asio::ip::tcp::acceptor acceptor(ioc, ep.protocol());
		acceptor.bind(ep);
		acceptor.listen(BACKLOG_SIZE);
		boost::asio::ip::tcp::socket sock(ioc);
		//接收器将收到的链接交给sock处理
		acceptor.accept(sock);
	}
	catch (boost::system::system_error& e) {
		std::cout << "Failed\n";
		return e.code().value();
	}
}
```

### 处理发送的数据

ASIO发送的数据有特定的格式，使用 boost::asio::buffer 函数可以将任何格式的数据转换成特定的格式。

```C++
void use_const_buffer() {
	std::string buf{ "hello world" };
	//将字符串转化为constbuffer
	boost::asio::const_buffer asio_buf(buf.c_str(), buf.length());
	//将constbuffer存在vector里面
	std::vector<boost::asio::const_buffer> buffer_sequence;
	buffer_sequence.push_back(asio_buf);
	//这样在send read 函数就可以使用vector
	//send(buffer_sequence);
}

void use_buffer_str() {
	boost::asio::const_buffers_1 output_buf = boost::asio::buffer("hello world");
	//send(output_buf);
	//send(boost::asio::buffer("hello world")); 与上等价
}

void use_buffer_array() {
	const size_t BUF_SIZE_BYTES = 20;
	//使用智能指针管理动态内存
	std::unique_ptr<char[]> buf(new char[BUF_SIZE_BYTES]);
	auto output_buf = boost::asio::buffer(static_cast<void*>(buf.get()), BUF_SIZE_BYTES);
	//send(output_buf);
}
```

### 发送数据

```C++
void write_to_socket(boost::asio::ip::tcp::socket& sock) {
	std::string buf{ "hello world" };
	std::size_t total_bytes_written = 0;
	//循环发送
	while (total_bytes_written != buf.length()) {
		total_bytes_written += sock.write_some(boost::asio::buffer(
			buf.c_str() + total_bytes_written, buf.length() - total_bytes_written));
	}
}
```

### 接收数据

```C++
void read_from_socket(boost::asio::ip::tcp::socket& sock,std::size_t buf_len) {
	char data[1024];
    std::size_t total_bytes_written = 0;
	//循环读取
	while (total_bytes_written != buf_len) {
		total_bytes_written += sock.read_some(boost::asio::buffer(
			data + total_bytes_written, buf_len - total_bytes_written));
	}
}
```