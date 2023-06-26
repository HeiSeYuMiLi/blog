+++
author = "baoguli"
title = "c++ cast转换"
date = "2023-6-26"
description = "简单聊聊 C++ cast转换"
tags = [
    "c++",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  

C++ 的显示转换 (cast) 是一种强制类型转换的方法，可以在不同的类型之间进行转换。

C++ 提供了四种显示转换运算符：static_cast, dynamic_cast, const_cast, reinterpret_cast。

它们的作用和区别如下：

- static_cast: 可以用来进行隐式转换和用户定义转换的组合，例如整数和浮点数之间的转换，或者基类和派生类之间的指针或引用的转换。但是不能移除 const 或 volatile 限定，也不能进行不安全的向下转换。

- dynamic_cast: 可以用来进行多态类型之间的指针或引用的安全转换，例如基类和派生类之间的向上、向下或横向转换。它需要运行时类型信息 (RTTI) 来检查转换的有效性，如果转换失败，会返回空指针或抛出异常。它比 static_cast 更安全，但也更耗时。

- const_cast: 可以用来移除或增加表达式的 const 或 volatile 限定，例如将一个 const 指针传入一个需要非 const 指针的函数中。但是不能改变表达式本身的类型，也不能修改原始对象的 const 限定。

- reinterpret_cast: 可以用来进行任意指针类型之间的转换，即使它们没有任何关联。它只是简单地重新解释表达式的二进制表示，不会进行任何检查或调整。它也可以用来进行指针和整数之间的转换，但是结果取决于平台。它是最危险的一种显示转换，可能导致未定义行为或不可移植性。

### static_cast

static_cast 可以用来进行基本类型和多态类型之间的转换，也可以用来进行用户定义的转换。例如：

```c++
int i = 65; // 定义一个整数变量i
char ch = static_cast<char> (i); // 将i转换为字符类型，并赋值给ch

float f = 2.5; // 定义一个浮点数变量f
double dbl = static_cast<double> (f); // 将f转换为双精度浮点数类型，并赋值给dbl

enum Color {red, green, blue}; // 定义一个枚举类型Color
Color c = red; // 定义一个Color变量c，并赋值为red
int n = static_cast<int> (c); // 将c转换为整数类型，并赋值给n
```

这些例子中，static_cast 都是用来进行隐式转换的逆向操作，即将一种类型转换为另一种类型，而不会改变表达式的值或含义。这些转换都是安全的，因为它们不会导致数据丢失或错误。

static_cast 也可以用来进行基类和派生类之间的指针或引用的转换，例如：

```c++
class B {
    public: virtual void Test() { 
        cout << "B::Test" << endl; 
    } 
}; // 定义一个基类B，有一个虚函数Test
class D : public B { 
    public: void Test() override { 
        cout << "D::Test" << endl; 
    } 
}; // 定义一个派生类D，继承自B，并重写Test函数

B* pb = new B(); // 定义一个指向B对象的指针pb
D* pd = new D(); // 定义一个指向D对象的指针pd
B* pb2 = static_cast<B*> (pd); // 将pd转换为指向B对象的指针，并赋值给pb2
D* pd2 = static_cast<D*> (pb); // 将pb转换为指向D对象的指针，并赋值给pd2
```

这些例子中，static_cast 都是用来进行多态类型之间的向上转换或向下转换，即将一种类型转换为它的基类或派生类。这些转换不一定都是安全的，因为它们可能会导致运行时错误。例如：

```c++
pb2->Test(); // 调用B类的Test方法，没有问题
pd2->Test(); // 调用D类的Test方法，可能会出错，因为pb实际上指向的是B对象，而不是D对象
```

为了避免这种错误，可以使用 dynamic_cast 来进行多态类型之间的安全转换，它会在运行时检查转换的有效性。如果转换失败，它会返回空指针或抛出异常。例如：

```c++
D* pd3 = dynamic_cast<D*> (pb); // 尝试将pb转换为指向D对象的指针，并赋值给pd3
if (pd3) {
    pd3->Test(); 
} else {
    std::cout << "Invalid cast" << std::endl; 
} // 检查pd3是否为空指针，如果不为空，则调用Test方法；如果为空，则输出错误信息
```

这样就可以避免调用不存在或不正确的方法，保证程序的正确性和安全性。

### dynamic_cast

dynamic_cast 可以用来进行多态类型之间的指针或引用的安全转换，即将一种类型转换为它的基类或派生类。它需要运行时类型信息 (RTTI) 来检查转换的有效性，如果转换失败，会返回空指针或抛出异常。例如：

```c++
B* pb = new B(); // 定义一个指向B对象的指针pb
D* pd = new D(); // 定义一个指向D对象的指针pd
B* pb2 = dynamic_cast<B*> (pd); // 将pd转换为指向B对象的指针，并赋值给pb2
D* pd2 = dynamic_cast<D*> (pb); // 尝试将pb转换为指向D对象的指针，并赋值给pd2
```

这些例子中，dynamic_cast 都是用来进行多态类型之间的向上转换或向下转换，即将一种类型转换为它的基类或派生类。这些转换都是安全的，因为它们会在运行时检查转换的有效性。例如：

```c++
pb2->Test(); // 调用D类的Test方法，没有问题，因为pb2实际上指向的是D对象
if (pd2) { 
    pd2->Test(); 
} else { 
    std::cout << "Invalid cast" << std::endl; 
} // 检查pd2是否为空指针，如果不为空，则调用Test方法；如果为空，则输出错误信息，因为pb实际上指向的是B对象，而不是D对象
```

这样就可以避免调用不存在或不正确的方法，保证程序的正确性和安全性。

dynamic_cast 也可以用来进行多态类型之间的横向转换，即将一种类型转换为它的同级类。例如：

```c++
class A { 
    public: virtual void Test() { 
        std::cout << "A::Test" << std::endl; 
    } 
}; // 定义一个基类A，有一个虚函数Test
class B : public A {}; // 定义一个派生类B，继承自A
class C : public A {}; // 定义一个派生类C，继承自A

A* pa = new C(); // 定义一个指向C对象的指针pa
B* pb = dynamic_cast<B*> (pa); // 尝试将pa转换为指向B对象的指针，并赋值给pb
```

这个例子中，dynamic_cast 是用来进行多态类型之间的横向转换，即将一种类型转换为它的同级类。这个转换也是安全的，因为它也会在运行时检查转换的有效性。例如：

```c++
if (pb) { 
    pb->Test(); 
} else { 
    std::cout << "Invalid cast" << std::endl; 
} 
```

#### 为什么dynamic_cast比static_cast更安全

如上所述，我们可以知道 dynamic_cast 比 static_cast 更安全，因为它可以在运行时检查转换的有效性，而 static_cast 只在编译时进行转换，不会进行任何检查。

上面例子中，static_cast 能成功，但是可能会导致运行时错误，这些错误是因为static_cast不会进行任何检查，只是简单地将一种类型转换为另一种类型，即使它们没有任何关联。这样可能会导致访问不存在或不正确的方法或数据。

而 dynamic_cast 则会在运行时检查转换的有效性，如果转换失败，会返回空指针或抛出异常。这样就可以避免调用不存在或不正确的方法或数据，保证程序的正确性和安全性。

因此，dynamic_cast 比 static_cast 更安全，但也更耗时。一般来说，在进行多态类型之间的转换时，应该优先使用 dynamic_cast。

### const_cast

const_cast 可以用来移除或增加表达式的 const 或 volatile 限定，例如：

```c++
void print(char* str) { 
    std::cout << str << std::endl; 
}
const char* msg = "Hello"; // 定义一个指向常量字符串的const指针

// print(msg); // 编译错误，不能将const指针转换为非const指针
print(const_cast<char*> (msg)); // 使用const_cast移除msg的const限定，并传入print函数，没有编译错误
```

这个例子中，const_cast 是用来移除 msg 的 const 限定，使得它可以作为 print 函数的参数。

但是这并不意味着我们可以通过这个指针修改字符串的内容，因为字符串本身是常量，如果我们尝试修改它，会导致未定义行为。例如：

```c++
const_cast<char*> (msg) [0] = 'h'; // 运行时错误，试图修改常量字符串
```

因此，使用 const_cas t时要注意不要修改原始对象的 const 限定。

另一个例子是使用 const_cast 来实现一个逻辑上是常量的成员函数，但是需要修改一些辅助数据成员。例如：

```c++
class Counter { 
public: 
    Counter() : value(0), times(0) {} 
    int getValue() const { 
        times++;
        return value; 
    } 
    void increase() { 
        value++; 
    } 
private: 
    int value; 
    mutable int times; // 使用mutable关键字使得times可以在常量成员函数中被修改 
};
```

这个例子中，Counter 类有一个数据成员 times，用来记录 getValue 函数被调用的次数。但是 getValue 函数是一个常量成员函数，它不能修改数据成员。为了解决这个问题，我们可以使用 mutable 关键字来声明 times，使得它可以在常量成员函数中被修改。

但是如果我们不想使用 mutable 关键字，或者 times 不是我们自己定义的类的数据成员，我们可以使用 const_cast 来实现同样的功能。例如：

```c++
class Counter { 
public: 
    Counter() : value(0), times(0) {} 
    int getValue() const { 
        const_cast<Counter*> (this)->times++; // 使用const_cast移除this指针的const限定，并修改times 
        return value; 
    } 
    void increase() { 
        value++; 
    } 
private: 
    int value; 
    int times; 
};
```

这个例子中，我们使用 const_cast 移除 this 指针的 const 限定，并通过它来修改 times。这样就可以实现一个逻辑上是常量的成员函数。

### reinterpret_cast

reinterpret_cast 可以用来进行任意指针类型之间的转换，即使它们没有任何关联。它只是简单地重新解释表达式的二进制表示，不会进行任何检查或调整。它也可以用来进行指针和整数之间的转换，但是结果取决于平台。它是最危险的一种显示转换，可能导致未定义行为或不可移植性。例如：

```c++
int i = 100;
char* p = reinterpret_cast<char*> (i); // 将i转换为指向字符的指针，并赋值给p
std::cout << p << std::endl; // 输出p指向的内容，可能会出错或输出乱码
```

这个例子中，reinterpret_cast 是用来将一个整数转换为一个指针，但是这个转换是没有意义的，因为 i 并不是一个有效的地址，p 指向的内容也是未知的。这样可能会导致程序崩溃或输出乱码。

```c++
B* pb = new B();
D* pd = reinterpret_cast<D*> (pb); // 将pb转换为指向D对象的指针，并赋值给pd
pd->Test(); // 调用D类的Test方法，可能会出错或执行错误的操作
```

这个例子中，reinterpret_cast 是用来将一个基类指针转换为一个派生类指针，但是这个转换是不安全的，因为 pb 实际上指向的是 B 对象，而不是 D 对象。这样可能会导致访问不存在或不正确的方法或数据。

```c++
int i = 0x12345678;
char* p = reinterpret_cast<char*> (&i); // 将i的地址转换为指向字符的指针，并赋值给p
std::cout << std::hex << static_cast<int> (*p) << std::endl; // 输出p指向的内容（即i的第一个字节），结果取决于平台
```

这个例子中，reinterpret_cast 是用来将一个整数的地址转换为一个指针，然后通过这个指针访问整数的第一个字节。这个结果取决于平台是大端序还是小端序。

### 四种 cast 之间的优劣

- static_cast: 优点是可以进行基本类型和多态类型之间的转换，可以实现隐式转换的逆向操作，也可以进行用户定义的转换。缺点是不能移除 const 或 volatile 限定，也不能进行不安全的向下转换，可能导致运行时错误。

- dynamic_cast: 优点是可以进行多态类型之间的安全转换，可以检查转换的有效性，可以进行向下或横向转换。缺点是需要运行时类型信息 (RTTI) 来支持，比较耗时，也不能移除 const 或 volatile 限定。

- const_cast: 优点是可以移除或增加表达式的 const 或 volatile 限定，可以实现一些特殊的功能，例如修改常量指针指向的内容。缺点是不能改变表达式本身的类型，也不能修改原始对象的 const 限定，可能导致未定义行为。

- reinterpret_cast: 优点是可以进行任意指针类型之间的转换，也可以进行指针和整数之间的转换，可以实现一些低级操作，例如内存操作。缺点是最危险的一种显示转换，可能导致未定义行为或不可移植性，不会进行任何检查或调整。

一般来说，C++ 显示转换 (cast) 的使用原则是：

- 尽量避免使用显示转换，除非必要。
- 如果需要进行基本类型之间的转换，或者多态类型之间的向上转换，使用 static_cast。
- 如果需要进行多态类型之间的向下或横向转换，使用 dynamic_cast。
- 如果需要移除或增加表达式的 const 或 volatile 限定，使用 const_cast。
- 如果需要进行任意指针类型之间的转换，或者指针和整数之间的转换，使用 reinterpret_cast。
