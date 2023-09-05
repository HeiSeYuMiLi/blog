+++
author = "baoguli"
title = "Drogon框架"
date = "2023-7-30"
description = "浅谈drogon框架"
tags = [
    "drogon",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  


Drogon 是一个基于C++14的高性能Web框架，它支持 HTTP1.0/1.1 和 WebSocket 协议，提供了一套完整的 MVC 架构，包括路由、控制器、视图、模板引擎、ORM、会话管理等功能。

### 优缺点

优点：
- 高性能：Drogon 框架使用了 trantor 库作为网络层的实现(trantor 库是一个基于 epoll/kqueue 的非阻塞 I/O 库，它封装了事件循环，定时器，异步 DNS 解析等功能)；使用了多线程模型，一个主线程负责监听端口和接受新连接，多个IO线程负责处理已建立的连接的读写事件；使用了高效的数据结构和算法，如 HttpViewData 类用于存储和传递视图数据，HttpParser 类用于解析 HTTP 报文，HttpControllersRouter 类用于路由 HTTP 请求等。

- 简洁和灵活：Drogon 框架提供了一套简洁和灵活的 API，可以方便地开发Web应用程序，支持RESTful风格的接口设计。

- 多数据库支持：Drogon 框架支持多种数据库，包括 MySQL, PostgreSQL, SQLite3, Redis 等，并提供了一个基于对象关系映射（ORM）的数据库客户端，可以使用 C++ 语言直接操作数据库。Drogon 框架还支持 JSON 和 Protobuf 等数据格式，可以方便地进行数据交换和序列化。

- 安全性：Drogon 框架支持 HTTPS 协议，可以实现安全的网络传输。Drogon 框架还提供了一些安全相关的功能，如防止跨站请求伪造（CSRF），防止 SQL 注入等。

缺点：
- Drogon 框架相对于其他流行的 Web 框架，如 Django, Flask, Express 等，还比较年轻和小众，社区资源和文档可能不够丰富和完善。

- Drogon 框架目前只支持 Linux 和 MacOS 平台，不支持 Windows 平台。

- Drogon 框架在处理分布式事务、服务治理、服务降级等方面可能不如其他微服务框架成熟和完善。

Drogon 框架的性能是非常高的，根据[Techempower Web Framework Benchmarks](^1^)的最新测试结果，Drogon框架在所有的 Web 框架中排名第一，每秒可以处理超过50万次请求。


### 源码结构与实现原理

Drogon 框架源码的主要部分是 lib、nosql_lib、orm_lib 这三个目录：

- lib：这个目录是 Drogon 框架的核心部分，包含了所有的源文件和头文件，主要的子目录是 inc、src：

    - inc：这个目录包含了 Drogon 框架的头文件，该目录下包含了一个 drogon 子目录：

        - drogon：这个子目录包含了 Drogon 框架对外暴露的头文件，还有两个子目录 plugins、utils：

            - plugins：这个子目录包含了一些插件相关的头文件，主要有以下头文件：

                - Plugin.h：这个文件定义了 Plugin 基类，这是插件的抽象基类，它提供了一个初始化插件的虚函数接口。

                - Session.h：这个文件定义了 Session 类，这是会话管理器的类，它负责创建和管理会话对象，并提供一些设置和获取会话属性和内容的接口。

                - SessionManager.h：这个文件定义了 SessionManager 接口，这是会话管理器的抽象接口，它提供了一个获取会话对象的虚函数接口。

                - RedisSessionManager.h：这个文件定义了 RedisSessionManager 类，这是基于 Redis 实现的会话管理器的类，它继承自 SessionManager接口，并实现了获取会话对象的函数。

            - utils：这个子目录包含了一些工具相关的头文件，主要有以下几类：

                - Utilities.h：这个文件定义了一些通用的工具函数，如编码转换，字符串处理，时间格式化等。
                - FunctionTraits.h：这个文件定义了一个 FunctionTraits 模板类，用于获取函数类型和参数类型等信息。
                - HttpConstraint.h：这个文件定义了一些 HTTP 相关的常量和枚举类型，如 HTTP 方法，HTTP 状态码等。
                - HttpUtils.h：这个文件定义了一些 HTTP 相关的工具函数，如解析URL参数，生成 Cookie 字符串等。

            - HttpAppFramework.h：这个文件定义了HttpAppFramework接口，这是应用框架的抽象接口，它提供了一些配置应用框架属性和功能的接口。
            - HttpController.h：这个文件定义了HttpController基类，这是控制器的抽象基类，它提供了一些注册控制器处理函数和路径参数的宏。
            - HttpSimpleController.h：这个文件定义了HttpSimpleController基类，这是简单控制器的抽象基类，它继承自HttpController基类，并提供了一个异步处理请求的虚函数接口。
            - HttpRequest.h：这个文件定义了HttpRequest接口，这是HTTP请求的抽象接口，它提供了一些获取请求属性和内容的接口。
            - HttpResponse.h：这个文件定义了HttpResponse接口，这是HTTP响应的抽象接口，它提供了一些设置响应属性和内容的接口。
            - HttpServer.h：这个文件定义了HttpServer接口，这是HTTP服务器的抽象接口，它提供了一些配置服务器属性和功能的接口。
            - HttpViewData.h：这个文件定义了HttpViewData接口，这是视图数据的抽象接口，它提供了一些设置数据属性和内容的接口。
            - WebSocketConnection.h：这个文件定义了WebSocketConnection接口，这是WebSocket连接的抽象接口，它提供了一些发送和接收消息的接口。
            - WebSocketController.h：这个文件定义了WebSocketController基类，这是WebSocket控制器的抽象基类，它提供了一些注册控制器处理函数和回调函数的宏。

    - src：这个目录包含了 Drogon 框架的实现文件，主要有以下几类：

        - HttpAppFrameworkImpl.cc：这个文件实现了 HttpAppFrameworkImpl 类，这是应用框架的核心类，它管理了所有的控制器，视图，过滤器，插件等对象，并提供了一些公共的接口和服务。

        - HttpControllersRouter.cc：这个文件实现了 HttpControllersRouter 类，这是路由器的核心类，它负责根据请求的路径和方法找到对应的控制器处理函数，并执行过滤器链。

        - HttpParser.cc：这个文件实现了 HttpParser 类，这是HTTP报文解析器的核心类，它负责解析请求报文，并封装成 HttpRequestImpl 对象。

        - HttpResponseImpl.cc：这个文件实现了 HttpResponseImpl 类，这是 HTTP 响应封装器的核心类，它负责封装响应报文，并提供一些设置响应属性和内容的接口。

        - HttpServer.cc：这个文件实现了 HttpServer 类，这是 HTTP 服务器的核心类，它负责监听端口并接受新的连接，并将连接分配给 IO 线程处理。

        - HttpViewData.cc：这个文件实现了 HttpViewData 类，这是视图数据封装器的核心类，它负责存储和传递视图数据，并提供一些设置数据属性和内容的接口。

        - WebSocketConnectionImpl.cc：这个文件实现了 WebSocketConnectionImpl 类，这是 WebSocket 连接封装器的核心类，它负责处理 WebSocket 协议，并提供一些发送和接收消息的接口。

        - WebSocketControllerImpl.cc：这个文件实现了 WebSocketControllerImpl 类，这是 WebSocket 控制器封装器的核心类，它负责调用 WebSocket 控制器处理函数，并提供一些设置回调函数和关闭连接的接口。

- nosql_lib：这个子目录包含了 NoSQL 库相关的实现文件，主要有以下几类：

    - redis：这个子目录包含了 Redis 客户端相关的实现文件，主要有以下几类：

        - RedisClient.cc：这个文件实现了 RedisClient 类，这是 Redis 客户端的核心类，它负责创建和管理 Redis 连接，并提供一些执行 Redis 命令和事务操作的接口。

        - RedisConnection.cc：这个文件实现了 RedisConnection 类，这是 Redis 连接的核心类，它负责与 Redis 服务器进行通信，并提供一些发送和接收数据的接口。

        - RedisParser.cc：这个文件实现了 RedisParser 类，这是 Redis 报文解析器的核心类，它负责解析 Redis 报文，并封装成 RedisResult 对象。

        - RedisTransactionImpl.cc：这个文件实现了 RedisTransactionImpl 类，这是 Redis 事务封装器的核心类，它负责管理 Redis 事务，并提供一些设置回调函数和提交事务的接口。

    
- orm_lib：这个子目录包含了ORM库相关的头文件，主要有以下几类：
    - DbClient.h：这个文件定义了DbClient接口，这是数据库客户端的抽象接口，它提供了一些执行SQL语句和事务操作的接口。
    - DbConnection.h：这个文件定义了DbConnection接口，这是数据库连接的抽象接口，它提供了一些发送和接收数据的接口。
    - SqlBinder.h：这个文件定义了SqlBinder类，这是SQL语句绑定器的类，它负责将SQL语句和参数绑定在一起，并提供一些设置参数类型和值的接口。
    - SqlBuilder.h：这个文件定义了SqlBuilder类，这是SQL语句构建器的类，它负责根据模型对象和条件生成SQL语句，并提供一些设置查询选项和条件的接口。
    - Mapper.h：这个文件定义了Mapper类，这是模型对象映射器的类，它负责将模型对象和数据库表进行映射，并提供一些增删改查操作的接口。
    - ModelTraits.h：这个文件定义了一些模型对象特性的宏，用于指定模型对象的表名，主键，列名等信息。







