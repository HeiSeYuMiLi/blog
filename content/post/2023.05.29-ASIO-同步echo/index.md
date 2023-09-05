+++
author = "baoguli"
title = "ASIO-同步echo"
date = "2023-5-29"
description = "介绍如何使用BOOST.ASIO实现同步的echo服务器"
tags = [
    "ASIO",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++ 


echo 服务器的主要功能是服务器接收客户端发来的数据，并且完完整整地发回去。依据之前说的同步的读写 API 就可以简单的实现同步 echo 服务器。

### 客户端

```C++
#include <iostream>
#include <boost/asio.hpp>
using namespace std;
using namespace boost::asio::ip;
const int MAX_LENGTH = 1024;

int main()
{
    try {
        //创建上下文服务
        boost::asio::io_context   ioc;
        //构造endpoint
        tcp::endpoint  remote_ep(address::from_string("127.0.0.1"), 10086);
        tcp::socket  sock(ioc);
        boost::system::error_code   error = boost::asio::error::host_not_found; ;
        sock.connect(remote_ep, error);
        if (error) {
            cout << "connect failed, code is " << error.value() << " error msg is " << error.message();
            return 0;
        }
        std::cout << "Enter message: ";
        char request[MAX_LENGTH];
        std::cin.getline(request, MAX_LENGTH);
        size_t request_length = strlen(request);
        boost::asio::write(sock, boost::asio::buffer(request, request_length));
        char reply[MAX_LENGTH];
        size_t reply_length = boost::asio::read(sock,
            boost::asio::buffer(reply, request_length));
        std::cout << "Reply is: ";
        std::cout.write(reply, reply_length);
        std::cout << "\n";
    }
    catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << endl;
    }
    return 0;
}
```

### 服务端

**服务端**的设计会复杂一些，因为可能同时多个客户端连接服务器，所以服务端需要对每一个用户单独开一个线程。

```C++
#include <iostream>
#include <boost/asio.hpp>
#include <memory>
#include <set>
using namespace boost::asio::ip;
const int MAX_LENGTH = 1024;
using socket_ptr = std::shared_ptr<tcp::socket>;
std::set<std::shared_ptr<std::thread>> thread_set;

//处理读和写 每有一个链接，开一个线程使用session处理
void session(socket_ptr sock) {
    try {
        for (;;) {
            char data[MAX_LENGTH];
            //将data清空为\0
            memset(data, '\0', MAX_LENGTH);
            boost::system::error_code error;
            size_t length = sock->read_some(boost::asio::buffer(data, MAX_LENGTH), error);
            if (error == boost::asio::error::eof) {
                //eof表示对端关闭，只需要关闭链接即可
                std::cout << "connection closed by peer" << std::endl;
                break;
            }
            else if(error){
                throw boost::system::system_error(error);
            }

            //remote_endpoint表示对端的端点
            std::cout << "receive from " << sock->remote_endpoint().address().to_string() << std::endl;
            std::cout << "receive message is " << data << std::endl;

            //回传数据
            boost::asio::write(*sock, boost::asio::buffer(data, length));
        }
    }
    catch (std::exception& e) {
        std::cerr << "Exception in thread:" << e.what() << std::endl;
    }
}

void server(boost::asio::io_context& ioc, unsigned short port) {
    tcp::acceptor acceptor(ioc, tcp::endpoint(tcp::v4(), port));
    for (;;) {
        socket_ptr sock(new tcp::socket(ioc));
        acceptor.accept(*sock);
        auto t = std::make_shared<std::thread>(session, sock);
        thread_set.insert(t);
    }
}

int main()
{
    try {
        boost::asio::io_context ioc;
        server(ioc, 10086);
        for (auto& t : thread_set) {
            t->join();
        }
    }
    catch (std::exception& e) {
        std::cerr << "Exception in thread:" << e.what() << std::endl;
    }
    return 0;
}
```