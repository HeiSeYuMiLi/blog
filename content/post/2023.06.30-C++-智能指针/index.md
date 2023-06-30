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


c++ 智能指针是一种对象，它可以像指针一样使用，但是它可以自动管理指向的资源的生命周期。c++ 智能指针可以避免手动分配和释放内存，防止内存泄漏和悬空指针。

c++11 引入了三种智能指针类型，分别是：

- **std::unique_ptr<T>**：独占资源所有权的指针，不能被复制或赋值，只能被移动。当它离开作用域时，它会自动删除指向的资源。
- **std::shared_ptr<T>**：共享资源所有权的指针，可以被复制或赋值，使用引用计数来跟踪资源的使用者数量。当引用计数变为零时，它会自动删除指向的资源。
- **std::weak_ptr<T>**：共享资源的观察者，需要和 std::shared_ptr 一起使用，不影响资源的生命周期。它可以从一个 std::shared_ptr 获得一个临时的 std::shared_ptr，来访问或使用资源。它主要用于解决循环引用的问题。

c++11 也废弃了旧的智能指针类型 **std::auto_ptr<T>**，因为它有一些缺陷和限制，比如不能指向数组，不能在容器中使用，复制或赋值时会改变原来的指针等。

### unique_ptr 和 shared_ptr 的区别

unique_ptr 和 shared_ptr 是两种常用的 c++ 智能指针，它们的主要区别如下：

- **所有权**：unique_ptr 独占所指向的对象，不能被复制或赋值，只能被移动。shared_ptr 共享所指向的对象，可以被复制或赋值，使用引用计数来管理对象的生命周期。
- **删除器**：unique_ptr 可以自定义删除器（deleter），用于指定如何释放资源。shared_ptr 的删除器是保存在控制块中，不影响 shared_ptr 对象的大小。
- **构造函数**：unique_ptr 没有类似 make_shared 的标准库函数返回一个 unique_ptr，必须使用 new 表达式或直接初始化来创建一个 unique_ptr。shared_ptr 可以使用 make_shared 函数来高效地创建一个 shared_ptr。
- **析构行为**：unique_ptr 在析构时会调用所指对象的基类的析构函数，可能导致对象不完全销毁。shared_ptr 在析构时会调用所指对象的实际类型的析构函数，能够正确销毁对象。

### 循环引用问题

循环引用问题是指当两个或多个对象互相持有对方的引用（通常是通过智能指针），导致它们的引用计数永远不会降为零，从而导致内存泄漏的情况。这种问题在使用 shared_ptr 时尤为突出，因为 shared_ptr 会自动管理对象的生命周期并维护引用计数。

解决循环引用问题的一种方法是使用 weak_ptr 来替换 shared_ptr 中的某些引用，从而打破循环。weak_ptr 是一种弱引用，它不会增加所指向对象的引用计数，也不会影响对象的生命周期。weak_ptr 可以从一个 shared_ptr 获得一个临时的 shared_ptr，来访问或使用对象。weak_ptr 主要用于解决循环引用的问题。

例如，假设我们有两个类 A 和 B，它们分别持有对方的引用，如下所示：

```cpp
class A {
public:
    std::shared_ptr<B> b_ptr;
};

class B {
public:
    std::shared_ptr<A> a_ptr;
};
```

当我们创建 A 和 B 对象，并使它们互相引用时：

```cpp
std::shared_ptr<A> a = std::make_shared<A>();
std::shared_ptr<B> b = std::make_shared<B>();
a->b_ptr = b;
b->a_ptr = a;
```

在这个例子中，A 对象持有 B 对象的 shared_ptr，B 对象持有 A 对象的 shared_ptr。这会导致循环引用，因为 A 和 B 的引用计数都为 2，它们都不会被自动销毁。这种情况下，即使 shared_ptr 超出其作用域，相关对象也不会被释放，从而导致内存泄漏。

为了解决这个问题，我们可以将 B 类中的 shared_ptr<A> 替换为 weak_ptr<A>，这样就可以打破循环：

```cpp
class B {
public:
    std::weak_ptr<A> a_ptr;
};
```

现在，当 B 对象使用 weak_ptr 指向 A 对象时，A 对象的引用计数不会增加。因此，在 a 和 b 超出作用域时，它们的析构函数会被调用，导致 A 对象和 B 对象的引用计数各自减 1。由于 A 对象的引用计数变为 0，它将被自动销毁。同时，B 对象的成员变量 a_ptr 不再指向任何对象，因此 B 对象的引用计数也降为 0，它也将被自动销毁。通过使用 weak_ptr ，我们成功地打破了循环引用，避免了内存泄漏。

需要注意的是，weak_ptr 无法直接访问其指向的对象。要访问对象，必须先将 weak_ptr 转换为 shared_ptr ，这可以通过 lock() 成员函数实现。同时，在访问之前，可以使用 expired() 成员函数检查 weak_ptr 是否悬空，以确保安全访问。

