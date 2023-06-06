+++
author = "baoguli"
title = "ASIO-异步echo"
date = "2023-6-6"
description = "介绍如何使用BOOST.ASIO实现异步的echo服务端"
tags = [
    "ASIO",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++ 

本文将介绍如何以异步的方式实现echo服务器。

### Session 类

Session 类主要是处理客户端消息收发的会话类。

```c++
// Session.h
class Session:public std::enable_shared_from_this<Session>
{
public:
    Session(boost::asio::io_context& io_context, Server* server);
    tcp::socket& GetSocket();
    std::string& GetUuid();
    void Start();
    void Send(char* msg,  int max_length);
private:
    void HandleRead(const boost::system::error_code& error, size_t  bytes_transferred, shared_ptr<Session> _self_shared);
    void HandleWrite(const boost::system::error_code& error, shared_ptr<Session> _self_shared);
    tcp::socket _socket;
    std::string _uuid;
    char _data[MAX_LENGTH];
    Server* _server;
    std::queue<shared_ptr<MsgNode> > _send_que;
    std::mutex _send_lock;
};
```

为了延长对象的生命周期，可以采用伪闭包的方式。利用智能指针被复制或使用引用计数加一的原理保证内存不被回收，延长生命周期。

于是在 HandleRead 和 HandleWrite 这两个回调函数中参入智能指针为参数，在异步读写函数调用回调函数时，智能指针的引用计数加一。

但是，在实际的使用中会出现两个智能指针指向同一个地址，由于两个智能指针的引用计数不一致，会导致错误。所以要通过 shared_from_this() 函数返回智能指针，该智能指针和其他管理这块内存的智能指针共享引用计数。

shared_from_this() 函数并不是 Session 的成员函数，要使用这个函数需要继承 std::enable_shared_from_this<Session>

Session 具体实现如下：

```c++
// Session.cpp
Session::Session(boost::asio::io_context& io_context, Server* server):
    _socket(io_context), _server(server){
    boost::uuids::uuid  a_uuid = boost::uuids::random_generator()();
    _uuid = boost::uuids::to_string(a_uuid);
}
void Session::Start(){
    memset(_data, 0, MAX_LENGTH);
    _socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&Session::HandleRead, this, 
        std::placeholders::_1, std::placeholders::_2, shared_from_this()));
}
void Session::HandleRead(const boost::system::error_code& error, size_t  bytes_transferred, shared_ptr<Session> _self_shared){
    if (!error) {
        cout << "read data is " << _data << endl;
        //发送数据
        Send(_data, bytes_transferred);
        memset(_data, 0, MAX_LENGTH);
        _socket.async_read_some(boost::asio::buffer(_data, MAX_LENGTH), std::bind(&Session::HandleRead, this, 
            std::placeholders::_1, std::placeholders::_2, _self_shared));
    }
    else {
        std::cout << "handle read failed, error is " << error.what() << endl;
        _server->ClearSession(_uuid);
    }
}
void Session::HandleWrite(const boost::system::error_code& error, shared_ptr<Session> _self_shared) {
    if (!error) {
        std::lock_guard<std::mutex> lock(_send_lock);
        _send_que.pop();
        if (!_send_que.empty()) {
            auto &msgnode = _send_que.front();
            boost::asio::async_write(_socket, boost::asio::buffer(msgnode->_data, msgnode->_max_len),
                std::bind(&Session::HandleWrite, this, std::placeholders::_1, _self_shared));
        }
    }
    else {
        std::cout << "handle write failed, error is " << error.what() << endl;
        _server->ClearSession(_uuid);
    }
}
void Session::Send(char* msg, int max_length) {
	bool pending = false;
	std::lock_guard<std::mutex> lock(_send_lock);
	if (_send_que.size() > 0) {
		pending = true;
	}
	_send_que.push(make_shared<MsgNode>(msg, max_length));
	if (pending) {
		return;
	}
	auto& msgnode = _send_que.front();
	boost::asio::async_write(_socket, boost::asio::buffer(msgnode->_data, msgnode->_total_len),
		std::bind(&Session::HandleWrite, this, std::placeholders::_1, SharedSelf()));
}
```

### Server 类

Server 类主要的功能是监听客户端连接然后分发给 Session 类实例进行读写操作。

```c++
// Server.h
class Server
{
public:
	Server(boost::asio::io_context& io_context, short port);
	void ClearSession(std::string);
private:
	void HandleAccept(shared_ptr<Session>, const boost::system::error_code& error);
	void StartAccept();
	boost::asio::io_context& _io_context;
	short _port;
	tcp::acceptor _acceptor;
	std::map<std::string, shared_ptr<Session>> _sessions;
};
```

StartAccept 函数主要功能是将要接收连接的 acceptor 绑定到服务上，其内部就是将 accpeptor 对应的 socket 描述符绑定到 epoll 或 iocp 模型上，实现事件驱动。

HandleAccept 是一个回调函数，作为异步操作的完成处理函数。

Server 类的具体实现：

```c++
// Server.cpp
Server::Server(boost::asio::io_context& io_context, short port) :_io_context(io_context), _port(port),
_acceptor(io_context, tcp::endpoint(tcp::v4(), port))
{
	cout << "Server start success, listen on port : " << _port << endl;
	StartAccept();
}

void Server::HandleAccept(shared_ptr<Session> new_session, const boost::system::error_code& error) {
	if (!error) {
		new_session->Start();
		_sessions.insert(make_pair(new_session->GetUuid(), new_session));
	}
	else {
		cout << "session accept failed, error is " << error.what() << endl;
	}

	StartAccept();
}

void Server::StartAccept() {
	shared_ptr<Session> new_session = make_shared<Session>(_io_context, this);
	_acceptor.async_accept(new_session->GetSocket(), std::bind(&Server::HandleAccept, this, new_session, placeholders::_1));
}

void Server::ClearSession(std::string uuid) {
	_sessions.erase(uuid);
}

```

参考：[llfc](https://llfc.club/category?catid=225RaiVNI8pFDD5L4m807g7ZwmF#!aid/2ODYV1A2xbhTjWr0FJ1ZS22ijZO)