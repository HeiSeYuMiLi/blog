+++
author = "baoguli"
title = "Boost.Asio的读接口"
date = "2023-6-26"
description = "简单聊聊Boost.Asio的读接口"
tags = [
    "ASIO",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  

boost.asio 是一个支持异步输入输出的 C++ 库，它可以用来处理网络通信、串口通信和其他类型的数据传输。

boost.asio 提供了一些类和函数来实现不同类型的读操作：

- read, read_at, read_until: 这些函数可以同步地从一个流或者一个缓冲区中读取一定数量或者条件的数据。

- async_read, async_read_at, async_read_until: 这些函数可以异步地从一个流或者一个缓冲区中读取一定数量或者条件的数据，并在完成时调用一个回调函数。

- read_some, async_read_some: 这些函数可以同步或者异步地从一个流或者一个缓冲区中读取一些可用的数据，但不保证读取指定的数量。

- streambuf, streambuf::prepare, streambuf::commit, streambuf::consume: 这些类和函数可以用来创建和管理一个动态的缓冲区，用于存储从流中读取的数据。

### 异步的读

异步读接口是指那些以async_开头的函数，它们可以让你在不阻塞当前线程的情况下，从一个流或者一个缓冲区中读取数据。

异步读接口的特点是：

- 它们需要一个 io_context 对象来调度和执行异步操作，你必须在代码中显式地调用 io_context::run() 函数来启动事件循环。

- 它们需要一个回调函数来处理异步操作的结果，你可以使用函数指针、函数对象、lambda 表达式或者 std::bind 等方式来指定回调函数。

- 它们会立即返回，不会等待异步操作完成，你可以在当前线程中继续执行其他任务。

- 它们会把错误码作为回调函数的第一个参数传递，而不是抛出异常，你需要在回调函数中检查错误码是否为0，如果不为0，表示发生了错误。

下面是一个使用async_read函数的例子：

```c++
auto self(shared_from_this());
	socket.async_read_some(boost::asio::buffer(buffer),
		[this, self](boost::system::error_code ec, std::size_t bytes_transferred)
		{
			if (!ec)
			{
				do_write();
			}
			else
			{
				std::cout << "handle read error" << std::endl;
			}
		});
```

这段代码中，函数 async_read_some 以一个 lambda 表达式为回调函数。

这个回调函数又称作完成处理函数，它的函数签名是有明确要求的，完成处理函数的类型必须是 void(const boost::system::error_code&, std::size_t)，或者是可以隐式转换为这个类型的类型。

完成处理函数的第一个参数是一个 boost::system::error_code 类型的对象，表示异步操作的结果。如果操作成功，该对象的值为 boost::system::error_code()，否则为一个具体的错误码。

完成处理函数的第二个参数是一个 std::size_t 类型的对象，表示传输的字节数。如果操作失败，该对象的值为 0。

### read_at、async_read_at 函数

它们可以同步或者异步地从一个随机访问设备（如文件或者内存映射）的指定偏移量处读取一定数量或者条件的数据，并返回读取的字节数。

以 async_read_at 函数为例：

```c++
void read_handler(const boost::system::error_code& ec, std::size_t bytes_transferred)
{
    if (!ec)
    {
        std::cout << "Read " << bytes_transferred << " bytes\n";
    }
}

int main()
{
    boost::asio::io_context context;
    // 使用CreateFile函数创建文件句柄
    HANDLE handle = ::CreateFile(L"test.txt", GENERIC_READ, 0, NULL, OPEN_EXISTING, FILE_FLAG_OVERLAPPED, NULL);
    // 检查文件句柄是否有效
    if (handle == INVALID_HANDLE_VALUE)
    {
        std::cerr << "CreateFile failed: " << ::GetLastError() << "\n";
        return 1;
    }
    // 使用文件句柄创建windows::random_access_handle对象
    boost::asio::windows::random_access_handle f(context, handle);
    char buf[512];
    // 从文件的第100个字节开始异步读取512个字节到buf中，并指定完成处理函数
    async_read_at(f, 100, boost::asio::buffer(buf), read_handler);
    context.run();
}
```

async_read_at 函数接受四个参数：一个 AsyncRandomAccessReadDevice 类型的对象，表示要读取的设备；一个 uint64_t 类型的对象，表示要读取的位置；一个 MutableBufferSequence 类型的对象，表示要存放数据的缓冲区；一个 ReadHandler 类型的对象，表示要执行的完成处理函数。

### read_until、async_read_until 函数

以异步的 async_read_until 函数为例：

async_read_until 可以从一个流或者一个缓冲区中读取数据，直到它包含一个分隔符，匹配一个正则表达式，或者一个函数对象指示一个匹配，并在完成时调用一个回调函数。

它可以接受一个可变缓冲区序列（如boost::asio::buffer）或者一个流缓冲区（如boost::asio::streambuf）作为读取数据的目标。并且它可以接受一个完成条件（如boost::asio::transfer_all）来指定读取数据的终止条件，如果不指定，默认为读取缓冲区序列或者流缓冲区的大小。

下面是一个使用async_read_until函数的例子：

```cpp
void read_handler(const boost::system::error_code &ec, std::size_t bytes_transferred)
{
    if (!ec)
    {
        std::cout << "Read " << bytes_transferred << " bytes\n";
    }
}

int main()
{
    boost::asio::io_context context;
    boost::asio::ip::tcp::socket sock(context);
    sock.connect(boost::asio::ip::tcp::endpoint(boost::asio::ip::address::from_string("127.0.0.1"), 8001));
    boost::asio::streambuf b;
    boost::asio::async_read_until(sock, b, "\n", read_handler);
    context.run();
}
```

async_read_until函数会一直读取数据，直到遇到换行符"\n"或者发生错误为止。