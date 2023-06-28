+++
author = "baoguli"
title = "管理缓冲区的接口和类"
date = "2023-6-27"
description = "简单聊聊Boost.Asio的缓冲区管理"
tags = [
    "ASIO",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++ 



boost.asio 提供了一些缓冲区类和函数来管理数据的读写。

缓冲区是一块内存区域，用于存储或传输数据。boost.asio 支持两种类型的缓冲区：可变缓冲区和常量缓冲区。可变缓冲区可以被修改，常量缓冲区则不能。

boost.asio 提供了一些预定义的缓冲区类，如 boost::asio::streambuf, boost::asio::mutable_buffer, boost::asio::const_buffer 等。boost.asio 还提供了一些缓冲区函数，如 boost::asio::buffer, boost::asio::buffer_copy, boost::asio::buffer_size 等。这些函数可以用于创建、复制、查询或操作缓冲区。

### 缓冲区类

- **boost::asio::streambuf**：这是一个继承自 std::basic_streambuf 的类，用于将输入队列和输出队列与一个或多个字符数组类型对象关联起来，这些数组类型对象的元素可以存储任意值。这些字符数组对象是 streambuf 对象的内部对象，但是直接访问数组元素的方式也是被提供的，为了允许他们被用在 I/O 操作上，例如在socket 上的 send 或 receive 函数。

- **boost::asio::mutable_buffer**：这是一个表示可修改内存区域的类，它包含一个指向内存起始位置的 void* 指针和一个表示内存大小的 std::size_t 值。它可以从一个指针和大小构造，也可以从一个标准库容器或数组构造，只要它们满足一定的要求。

- **boost::asio::const_buffer**：这是一个表示不可修改内存区域的类，它包含一个指向内存起始位置的 const void* 指针和一个表示内存大小的 std::size_t 值。它也可以从一个指针和大小构造，或从一个标准库容器或数组构造。它还可以从一个可变缓冲区构造，但反过来不行。

- **boost::asio::buffered_stream**：这是一个封装了另一个流类型（如socket或iostream）的类，用于提供缓冲区功能。它可以在构造时指定缓冲区大小，并且可以使用 flush() 函数来强制发送或接收缓冲区中的数据。

- **boost::asio::buffered_read_stream**：这是一个封装了另一个流类型（如socket或iostream）的类，用于提供读取缓冲区功能。它可以在构造时指定缓冲区大小，并且可以使用 fill() 函数来填充缓冲区中的数据。

- **boost::asio::buffered_write_stream**：这是一个封装了另一个流类型（如socket或iostream）的类，用于提供写入缓冲区功能。它可以在构造时指定缓冲区大小，并且可以使用 flush() 函数来强制发送缓冲区中的数据。

其中，关于 streambuf 这个类的详细介绍：

它可以用来创建和管理一个动态的缓冲区，用于存储从流中读取的数据或者写入到流中的数据。

- 它有一个输入序列和一个输出序列，分别表示缓冲区中可以读取和写入的数据。

- 它可以使用 prepare()函数来分配一定大小的空间给输出序列，用于写入数据。

- 它可以使用 commit()函数来将输出序列中的数据移动到输入序列中，用于读取数据。

- 它可以使用 consume()函数来从输入序列中移除已经读取的数据，释放空间。

- 它可以使用 data()函数来获取输入序列中的数据，返回一个可变缓冲区序列（mutable_buffers_type）。

- 它可以使用 size()函数来获取输入序列中的数据大小。

- 它可以使用 max_size()函数来获取缓冲区的最大容量。

- 它可以使用 capacity()函数来获取缓冲区的当前容量。

下面是一个使用 streambuf 的例子：

```c++
boost::asio::streambuf b;
std::ostream os(&b);
os << "Hello, World!\n";
std::cout << &b;
```

这段代码创建了一个 streambuf 对象 b，并使用 std::ostream 对象 os 来向输出序列中写入 "Hello, World!\n"。然后，使用 std::cout 对象来从输入序列中读取并输出数据。注意，当 os 对象被销毁时，它会自动调用 commit() 函数，将输出序列中的数据移动到输入序列中。

### 缓冲区函数

- **boost::asio::buffer**：可以接受一个指针和大小、一个标准库容器或数组、一个预定义的缓冲区类，或者一个缓冲区序列。它会根据参数的类型返回相应的可变缓冲区或常量缓冲区。例如：

```c++
char data[128];
std::vector<char> vec(256);
boost::asio::streambuf buf;
boost::asio::mutable_buffer mb = boost::asio::buffer(data, sizeof(data)); // 返回可变缓冲区
boost::asio::const_buffer cb = boost::asio::buffer(vec); // 返回常量缓冲区
boost::asio::mutable_buffers_1 mbs = boost::asio::buffer(buf); // 返回可变缓冲区序列
```

- **boost::asio::buffer_copy**：可以接受两个缓冲区序列作为参数，分别表示目标和源。它会返回复制的字节数，如果目标空间不足，则只复制部分内容。例如：

```c++
char data[128];
std::vector<char> vec(256);
boost::asio::streambuf buf;
std::size_t n1 = boost::asio::buffer_copy(boost::asio::buffer(data), boost::asio::buffer(vec)); // 复制vec到data，返回复制的字节数
std::size_t n2 = boost::asio::buffer_copy(boost::asio::buffer(buf), boost::asio::buffer(data)); // 复制data到buf，返回复制的字节数
```

- **boost::asio::buffer_size**：可以接受一个缓冲区或一个缓冲区序列作为参数，返回其所占的字节数。例如：

```c++
char data[128];
std::vector<char> vec(256);
boost::asio::streambuf buf;
std::size_t s1 = boost::asio::buffer_size(boost::asio::buffer(data)); // 返回128
std::size_t s2 = boost::asio::buffer_size(boost::asio::buffer(vec)); // 返回256
std::size_t s3 = boost::asio::buffer_size(boost::asio::buffer(buf)); // 返回buf中数据的字节数
```

- **boost::asio::dynamic_buffer**：这是一个用于创建动态缓冲区对象的函数，它可以接受一个标准库容器或一个预定义的缓冲区类作为参数，返回一个支持增长和收缩的可变缓冲区序列。动态缓冲区对象提供了一些方法来操作数据，如size(), max_size(), capacity(), data(), consume(), grow(), shrink()等。例如：

```c++
std:vector<char> vec(256);
boost:asio:streambuf buf;
auto db1 = boost:asio:dynamic_buffer(vec); // 创建动态缓冲区对象
auto db2 = boost:asio:dynamic_buffer(buf); // 创建动态缓冲区对象
db1.grow(128); // 增加vec的大小为384字节
db2.consume(64); // 消费buf中前64字节的数据
```

buffer 和 streambuf 有如下几点区别：

- buffer 是一个函数，它可以根据给定的原始内存、数组、容器或者字符串来创建一个缓冲区对象，用于表示一块连续的内存区域。streambuf 是一个派生的类，它可以用来创建和管理一个动态的缓冲区，用于存储从流中读取的数据或者写入到流中的数据。

- buffer 是一个不可变的对象，它只能表示一个固定的内存范围，不能动态地分配或者释放内存。streambuf 是一个可变的对象，它可以根据需要动态地分配或者释放内存，也可以使用 prepare(), commit(), consume() 等函数来控制缓冲区中的数据。

- buffer 通常用于同步或者异步的读写操作，它可以接受一个可变缓冲区序列（如boost::asio::buffer）或者一个流缓冲区（如boost::asio::streambuf）作为读写数据的目标。streambuf 通常用于与流对象（如std::istream, std::ostream, boost::asio::ip::tcp::iostream等）配合使用，它可以作为流对象的内部缓冲区，也可以作为流对象的构造参数。

- buffer 通常是顺序访问的，它只能按照给定的顺序读写数据，不能随机地跳转到任意位置。streambuf 可以是随机访问的，它可以使用 seekg(), seekp(), tellg(), tellp() 等函数来改变或者获取输入输出位置。

