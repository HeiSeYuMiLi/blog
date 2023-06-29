+++
author = "baoguli"
title = "字节序"
date = "2023-6-29"
description = "简单聊聊网络字节序"
tags = [
    "网络编程",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  

在计算机网络中，由于不同的计算机使用的 CPU 架构和字节顺序可能不同，从对端接收到的数据可能字节顺序与本机不一致，因此在传输数据时需要对数据的字节序进行统一，以保证数据能够正常传输和解析。

网络字节序是 TCP/IP 协议规定的一种标准的数据表示格式，它与具体的 CPU 类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释。

计算机内部存储数据的方式有两种：大端序（Big-Endian）和小端序（Little-Endian）。

- **大端模式**：是指数据的**高位字节**存放在**低位地址**，而**低位字节**存放在**高位地址**。这种方式类似于人类阅读数字的顺序，从高位到低位。

- **小端模式**：是指数据的**低位字节**存放在**低位地址**，而**高位字节**存放在**高位地址**。这种方式利用了地址递增的特性，方便数据的处理和拼接。

在网络通信过程中，通常使用的是大端序。因此，在进行网络编程时，需要将主机字节序转换为网络字节序，也就是将数据从本地字节序转换为大端序。可以使用诸如 htonl、htons、ntohl 和 ntohs 等函数来实现字节序转换操作。


### 区分大小端

以一个32位的整数12345678为例，它的二进制表示为00000000011101011011100001001110，它的十六进制表示为00 75 B8 4E。如果按照一个字节为一个存储单元，那么它在内存中的存储方式如下：

- 大端模式：

地址  |  0x100  |  0x101  |  0x102  |  0x103  |
内容  |    00   |    75   |    B8   |    4E   |

- 小端模式：

地址  |  0x100  |  0x101  |  0x102  |  0x103  |
内容  |    4E   |    B8   |    75   |    00   |

一般来说，网络传输使用的是大端模式，也称为网络字节序。

如果想判断本机采用了哪种字节序，可以使用以下方法：

- 利用强制类型转换，将一个整数变量的地址转换为一个字符指针，然后通过指针访问该地址处的内容，如果是1，则为小端模式，如果是0，则为大端模式。例如：

```c++
#include <iostream>
int main()
{
    int i = 1;
    char *p = (char *)&i;
    if (*p == 1)
        std::cout<<"小端模式\n";
    else
        std::cout<<"大端模式\n";
    return 0;
}
```

- 利用联合体（union），将一个整数变量和一个字符变量共享同一块内存空间，然后通过字符变量访问该空间的内容，如果是1，则为小端模式，如果是0，则为大端模式。例如：

```c++
#include <iostream>
int check_endian()
{
    union {
        int i;
        char c;
    } u;
    u.i = 1;
    return u.c; //如果是小端返回1，如果是大端返回0
}
int main()
{
    if (check_endian())
        std::cout<<"小端模式\n";
    else
        std::cout<<"大端模式\n";
    return 0;
}
```

### 转换字节序的函数

网络编程中常用的转换字节序的函数有 htonl、htons、ntohl 和 ntohs 等函数。boost.asio 也提供了一些函数提供字节序转换的功能：

- **boost::asio::detail::socket_ops::host_to_network_short**：用于将主机字节序的 **unsigned short** 型数据转换成网络字节序，相当于 **htons** 函数。
- **boost::asio::detail::socket_ops::host_to_network_long**：用于将主机字节序的 **unsigned int** 型数据转换成网络字节序，相当于 **htonl** 函数。
- **boost::asio::detail::socket_ops::network_to_host_short**：用于将网络字节序的 **unsigned short** 型数据转换成主机字节序，相当于 **ntohs** 函数。
- **boost::asio::detail::socket_ops::network_to_host_long**：用于将网络字节序的 **unsigned int** 型数据转换成主机字节序，相当于 **ntohl** 函数。

- **boost::asio::detail::byte_order::host_to_big_endian**：用于将主机字节序的任意长度的数据转换成大端字节序。
- **boost::asio::detail::byte_order::big_endian_to_host**：用于将大端字节序的任意长度的数据转换成主机字节序。
- **boost::asio::detail::byte_order::host_to_little_endian**：用于将主机字节序的任意长度的数据转换成小端字节序。
- **boost::asio::detail::byte_order::little_endian_to_host**：用于将小端字节序的任意长度的数据转换成主机字节序。

```c++
uint32_t host_long_value = 0x12345678;
uint16_t host_short_value = 0x5678;
uint32_t network_long_value = boost::asio::detail::socket_ops::host_to_network_long(host_long_value);
uint16_t network_short_value = boost::asio::detail::socket_ops::host_to_network_short(host_short_value);
std::cout << "Host long value: 0x" << std::hex << host_long_value << std::endl;
std::cout << "Network long value: 0x" << std::hex << network_long_value << std::endl;
std::cout << "Host short value: 0x" << std::hex << host_short_value << std::endl;
std::cout << "Network short value: 0x" << std::hex << network_short_value << std::endl;
```