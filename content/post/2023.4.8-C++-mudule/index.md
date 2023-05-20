+++
author = "baoguli"
title = "模块module"
date = "2023-4-8"
description = "Custom Test Page"
tags = [
    "c++",
]
categories = [
    "syntax",
]
aliases = ["migrate-from-jekyl"]
image = ""
+++

C++20 引入了[模块](https://en.cppreference.com/w/cpp/language/modules "Module")，这是一种新式解决方案，可将 C++ 库和程序转换为组件。  


模块是一组源代码文件，这些文件独立于导入它们的翻译单元进行编译。模块消除或减少与使用头文件相关的许多问题。它们通常会**减少编译时间**。 模块中声明的宏、预处理器指令和非导出名称在模块外部不可见。它们不会影响导入模块的翻译单元的编译。编译器可以比头文件更快地处理该文件，而且，编译器可以在项目中导入模块的每个位置重复使用它。  

### 模块module解决的痛点


能解决的最大的痛点就是由于**头文件的重复替换**，导致的编译慢。


在多个翻译单元中，可能会重复的调用同一个头文件，每次调用都需要处理一遍，预处理后的源文件就会立即膨胀，导致了编译慢的情况。  


而module就可以很好地解决这个问题，已经编译好的module会变成编译器的中间表示进行保存，需要什么拿什么，大大的提升了编译速度。


### module为什么会更快？


模块module更快的原因：


Module编译后，会得到编译器缓存（源代码信息）。编译器处理好了源代码，并且将内部的缓存保留了下来，并且有**懒加载**（lazy loading）的支持，调用的时候只需要取出需要的部分，因此更快。


### 如何使用module


三个关键字 **module** **import** **export**  

module关键字用于命名模块  

>例： module Module1;

export关键字用于  

1.	对模块的主接口文件使用 export module 声明  

>例： export module Module2;  

2.	在接口文件中，对要作为公共接口的部分的名称使用 export 修饰符  


例： 
```
// Module2.ixx  
  
export module Module2;  
  
namespace Module2_NS  
{  
   export int f();//外部可见接口  
   export double d();  
   double internal_f(); // not exported 外部不可见  
}  
  
//MyProgram.cpp  
  
import Module2;  
  
int main() {  
    Module2_NS::f(); // OK  
    Module2_NS::d(); // OK  
    Module2_NS::internal_f(); // Module2_NS::internal_f() is not exported  
}  
```

import关键字用于导入声明  


例：

```
//main.cpp  
#include<iostream>  
import std.core;  
import Module1;  
  
int main(){…}  

```

使用 import 声明使模块名称在程序中可见。 import 声明必须出现在 module 声明之后以及任何 #include 指令之后，但必须出现在文件中的任何声明之前。
