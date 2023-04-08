+++
author = "baoguli"
title = "C++引用"
date = "2023-4-5"
description = "简述C++的引用"
tags = [
    "c++",
    "引用",
    "左值、右值",
]
categories = [
    "themes",
    "syntax",
]
aliases = ["create-library"]
image = ""
+++ 

### 何为引用

**引用**即对象的别名，它并不是一个新的对象。定义引用后，程序将引用和它的初始值**绑定**在一起，对引用的所有操作都在绑定的对象上发生。

```
int ival=0;
int &refVal=ival;       //refVal指向ival
int &refVal2;           //错误：引用必须被初始化
int &refVal3=refVal;   //引用refVal引用的对象
refVal=1;               //如同ival=1
```

因为对引用的所有操作都是在绑定的对象上发生的，以引用为初始值定义引用，实际上就是以绑定的对象为初始值。

### 何为左值、右值

简单地说，左值是可以取地址、有名字的值，一般的，左值在等号的左边(没有const声明时)；右值是不可取地址、没有名字的值，所以临时变量都是右值。

```
std::string str=std::string{"hello"};
```

 str 可以用&取地址，是一个有名字的值，所以是一个左值。

 std::string{"hello"} 不可以用&取地址，是一个临时变量，所以是一个右值。

### 左值引用

上面所说的引用就是左值引用。左值引用就是指向左值的引用，在const修饰下可以指向右值。

```
const std::string &str2=std::string{"hello"};
```

### 右值引用

为了实现移动语义，c++11引入了右值引用的概念。定义方式为 `type &&d` 。右值引用就是指向右值的引用，不可以指向左值。

```
std::string &&str3=std::string{"hello"};
std::string &&str4=str;    //error
```

### 右值引用实现移动语义

```
#include<iostream>

class A {
public:
	char[] a_{new char[10]};
	A() {};
	A& operator=(const A& a) {
		std::cout << "拷贝赋值" << std::endl;
		std::memcpy(a_,a.a_,10);
		return *this;
	}
	A& operator=(A&& a) {
        std::cout << "移动赋值" << std::endl;
		a_=a.a_;
        a.a_=nullptr;
		return *this;
    }
    ~A() {
        delete[] a_;
    }
};

int main() {
	A a,b;
    a=b;     //拷贝赋值
    a=A{};   //移动赋值
}
```

比较两种不同的赋值方法，拷贝赋值接收一个左值，移动赋值接收一个右值，而这个右值 A{} 的生命周期只在本行内，所以执行移动赋值的时候，把所有的资源转移，而非复制，从而提高了性能。

### std::move

**std::move** 的意思应该是转换而非移动，它可以将实参的类型转换成右值引用。

```
a=std::move(b);  //移动赋值
```

通过 move 的类型转换，上面的赋值便匹配上了右值版本的移动赋值。

### 万能引用

在声明函数时，形参的类型如果是 type& ，则可以接收一个左值。如果还想要接收右值的话，可以将形参类型声明为 const type& ，但是因为是const的，所以函数不能对参数有修改。那如果想要修改参数呢？

这时候就有了万能引用： T&& 。

```
#include<iostream>

template<typename T>
void func(T&& param){
    std::cout<<param<<std::endl;
}

int main(){
    int a=100;
    func(a);      //ok
    func(100);    //ok
}
```

无论传入左值还是右值，万能引用都能接收，并且没有不可修改的限制。

### auto&&

与万能引用类似，它并不表示右值引用，而是根据初始值的类型进行推导，若初始值是一个左值，则表示左值引用；若初始值是一个右值，则表示右值引用。

```
int a=100;
auto&& b=a;     //b为左值引用
auto&& c{100};  //c为右值引用

const int& m=0;
auto&& n=m;     //n为const int&
```

