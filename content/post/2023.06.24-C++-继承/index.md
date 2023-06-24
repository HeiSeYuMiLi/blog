+++
author = "baoguli"
title = "c++继承"
date = "2023-6-24"
description = "简单聊聊 C++ 继承"
tags = [
    "c++",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  


c++ 继承是一种面向对象的特性，它允许我们根据一个已有的类来定义一个新的类，从而实现代码的重用和扩展。

继承有以下几个概念：

- 基类(Base Class)：也叫父类或超类，是被继承的类。
- 派生类(Derived Class)：也叫子类或继承类，是继承自基类的类。
- 继承方式(Inheritance Type)：指定派生类对基类成员的访问权限，有 public、protected 和 private 三种。
- 单一继承(Single Inheritance)：指一个派生类只有一个基类。
- 多重继承(Multiple Inheritance)：指一个派生类有多个基类。
- 虚函数(Virtual Function)：在基类中声明为 virtual 的成员函数，可以在派生类中被重写，实现多态性。
- 显式重写(Explicit Override)：在派生类中使用 override 关键字来重写基类的虚函数，增加代码的可读性和安全性。
- 抽象类(Abstract Class)：包含纯虚函数(pure virtual function)的类，不能被实例化，只能作为基类。
- 范围规则(Scope Rules)：在派生类中使用基类的成员时，需要遵循一定的规则，如使用 :: 运算符或 using 声明。

### 虚函数

虚函数是一种在基类中使用关键字 virtual 声明的成员函数，它可以在派生类中被重写，从而实现动态多态。虚函数的作用是让基类的指针或引用能够根据实际指向或引用的对象类型来调用正确的函数版本，而不是根据自身的类型。

虚函数的底层实现原理是通过虚函数表和虚函数表指针来实现的。

虚函数表是一个存储了类中所有虚函数地址的静态数组，每个包含虚函数的类都有一个自己的虚函数表。

虚函数表指针是一个隐藏的成员指针，它在类对象创建时自动设置为指向该类的虚函数表。当基类指针或引用调用虚函数时，会通过虚函数表指针找到对应的虚函数表，然后在虚函数表中根据偏移量找到正确的虚函数地址，最后调用该地址处的函数。

下面是一个简单的例子，演示了虚函数表和虚函数表指针的工作过程：

```c++
// 基类
class Animal {
public:
    // 虚函数
    virtual void Sound() {
        std::cout << "Animal makes a sound." << std::endl;
    }
};

// 派生类
class Dog : public Animal {
public:
    // 重写基类的虚函数
    void Sound() override {
        std::cout << "Dog barks." << std::endl;
    }
};

int main() {
    Animal a; // 创建一个Animal对象
    Dog d; // 创建一个Dog对象

    Animal* p = &a; // 声明一个Animal指针，指向Animal对象
    p->Sound(); // 调用Animal对象的Sound函数

    p = &d; // 指针指向Dog对象
    p->Sound(); // 调用Dog对象的Sound函数

    return 0;
}
```

在这个例子中，编译器会为 Animal 类和 Dog 类各生成一个虚函数表，如下所示：

|   Animal::vtable   |   Dog::vtable   |
|   --------------   |   -----------   |
|   &Animal::Sound   |   &Dog::Sound   |

同时，在 Animal 类和 Dog 类中都会有一个隐藏的成员指针 __vptr ，它在对象创建时会被设置为指向对应类的虚函数表。例如：

```c++
Animal a; // a.__vptr = &Animal::vtable
Dog d; // d.__vptr = &Dog::vtable
```

当 Animal 指针 p 调用 Sound 函数时，会先通过 p->__vptr 找到对应的虚函数表，然后在虚函数表中找到 Sound 函数的地址，最后调用该地址处的函数。例如：

```c++
p = &a; // p->__vptr = &Animal::vtable
p->Sound(); // 调用 (*p->__vptr)[0] 即 Animal::Sound

p = &d; // p->__vptr = &Dog::vtable
p->Sound(); // 调用 (*p->__vptr)[0] 即 Dog::Sound
```

由于派生类重写了基类的虚函数，所以 Dog 的虚函数表中，存储的是重写的虚函数，因此调用的也是重写的虚函数。

### 多态

多态是指不同的对象对同一个消息(函数调用)做出不同的响应。

在c++中，多态主要有两种形式：

- 静态多态(Static Polymorphism)：也叫编译时多态或早绑定，是指在编译期间就确定函数调用的具体版本，如函数重载和模板。
- 动态多态(Dynamic Polymorphism)：也叫运行时多态或晚绑定，是指在运行期间根据对象的实际类型来确定函数调用的具体版本，如虚函数。

上述的虚函数的例子就是动态多态。

静态多态举例说明一下：

```c++
// 静态多态的例子：函数重载
#include <iostream>

// 重载了三个不同参数类型的max函数
int max(int a, int b) {
    return a > b ? a : b;
}

double max(double a, double b) {
    return a > b ? a : b;
}

char max(char a, char b) {
    return a > b ? a : b;
}

int main() {
    // 根据传入参数的类型，在编译期间就确定了调用哪个版本的max函数
    std::cout << max(10, 20) << std::endl; // 调用int版本
    std::cout << max(3.14, 2.71) << std::endl; // 调用double版本
    std::cout << max('a', 'b') << std::endl; // 调用char版本
    return 0;
}
```

### 继承问题：菱形继承

在多重继承的时候可能会出现以下情况：

```c++
class A{
public:
    int value = 3;
    virtual void func(){
        std::cout << "A.func" << std::endl;
    }
};

class B : public A{
public:
    void func() override{
        std::cout << "B.func" << std::endl;
    }
};

class C : public A{
public:
    void func() override{
        std::cout << "C.func" << std::endl;
    }
};

class D : public B, public C{
public:
    void func() override{
        std::cout << "D.func" << std::endl;
    }
};

int main(){
    D d;
    A* p = &d; // error 基类 "A" 不明确
    std::cout << d.value << std::endl; // error "D::value" 不明确
    p->func();
}
```

使用虚继承便可解决这个问题：

```c++
class A{
public:
    int value = 3;
    virtual void func(){
        std::cout << "A.func" << std::endl;
    }
};

class B : virtual public A{
public:
    void func() override{
        std::cout << "B.func" << std::endl;
    }
};

class C : virtual public A{
public:
    void func() override{
        std::cout << "C.func" << std::endl;
    }
};

class D : public B, public C{
public:
    void func() override{
        std::cout << "D.func" << std::endl;
    }
};

int main(){
    D d;
    A* p = &d; // error 基类 "A" 不明确
    std::cout << d.value << std::endl; // error "D::value" 不明确
    p->func();
}
```