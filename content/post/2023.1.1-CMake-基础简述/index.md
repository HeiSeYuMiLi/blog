+++
author = "baoguli"
title = "CMake基础简述"
date = "2023-1-1"
description = "简述CMake的基本使用方法"
tags = [
    "cmake",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++ 

## 简介

CMake是一个跨平台的编译(Build)工具,可以用简单的语句来描述所有平台的编译过程。

打个比方：现有一个工程列表，由若干个互相调用的工程组成，它们之间的调用关系复杂而严格，如果我想在这样复杂的框架下进行二次开发，显然只拥有它的源码是远远不够的，还需要清楚的明白这若干个项目之间的复杂关系，在没有原作者的帮助下进行这项工作几乎是不可能的。

这个时候，就由CMake大显神威了，原作者只要写一份 `CMakeLists.txt文档` ，框架的使用者们只需要在下载源码的同时下载作者提供的CMakeLists.txt，就可以利用CMake，在进行 `工程的搭建` 。

### 什么是makefile

**makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作。**makefile带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。make是一个命令工具，是一个解释makefile中指令的命令工具，一般来说，大多数的IDE都有这个命令工具，比如：Visual C++的nmake，Linux下GNU的make。可见，makefile都成为了一种在工程方面的编译方法。

CMake就是用来makefile的一个工具：读入所有源文件之后，自动生成makefile。


下面，是使用CMake的基本方法，参考[CMake 教程](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)。

## 第一步 基本起点

### 构建最基本的项目

使用CMake构建一个最简单的项目莫过于将一个源文件构建成一个可执行文件。有源文件如下：

```C++
//main.cpp
#include<iostream>
int main(){
    std::cout<<"Hello world!"<<std::endl;
}
```

在同一目录下，有CMakeLists.txt如下：

```makefile
#CMakeLists.txt
cmake_minimum_required(VERSION 3.10)

# 设置工程名
project(Tutorial)

# 添加文件
add_executable(Tutorial main.cpp)
```

为了保证源码路径的干净整洁，在当前目录下创建一个 build 目录，存放构建项目产生的编译产物，这种方式叫作 `外部构建` 。反之，把编译产物存放在当前目录中叫作 `内部构建` 。  

接下来，导航到该构建目录并运行 cmake 以配置项目并生成本机构建系统：

```
cd build
cmake ..
```

`..` 表示CMakeLists.txt文件在上一级目录。  

然后调用该构建系统来实际编译/链接项目：

```
cmake --build .
```

执行此命令后，将生成可执行文件，其中 `--build` 指明可执行文件的存放目录， `.` 表示当前目录。  

输入命令执行：

```
./Tutorial
```

便可得到输出 Hello world!

### 指定c++版本

例如指定c++11，只需要在CMakeLists.txt里添加：

```makefile
set(CMAKE_CXX_STANDARD 11)
```

### 添加版本号和配置的头文件

```makefile
cmake_minimum_required(VERSION 3.10)

# 设置工程名
project(Tutorial VERSION 1.0)

# 设置最大版本号和最小版本号
set(Tutorial_version_major 1)
set(Tutorial_version_minor 0)

#将版本号传递给源文件，.h.in是新建文档，.h文件是自动生成文件
configure_file(
	"${Tutorial_SOURCE_DIR}/TutorialConfig.h.in"
	"${Tutorial_BINARY_DIR}/TutorialConfig.h"
)

# 添加文件
add_executable(Tutorial main.cpp)

#包含头文件
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

头文件TotorialConfig.h.in的内容：

```C++
// 与tutorial相关的配置好的选项与设置；
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

使用 `@变量名@` 表示CMakeLists.txt里面对应变量名的值。

在工程中包含头文件 `TutorialConfig.h` 即可获得版本号。生成的.h文件，其实也就是定义了两个宏，而宏的数据是由.h.in文件传过去的。


## 第二步 添加库

### 创建库

Linux下静态库以.a为后缀结尾，动态库以.so为后缀结尾，二者最本质的区别是 `打进可执行文件的时机不同` 。静态库是在链接阶段打进可执行文件，而动态库是在可执行文件运行的时候才加载进来的。静态库的优点是文件执行快，但是缺点也很明显就是多消耗了内存空间，动态库刚好相反，节省了内存空间，但是需要花费时间在运行的时候加载库。

现有test1目录结构如下  
├── CMakeLists.txt　　　　　　#父目录的CMakeList.txt  
├── main.cpp　　　　　　　　#源文件，包含main函数  
├── sub　　　　　　　　　　#子目录    
 └── CMakeLists.txt　　　　#子目录的CMakeLists.txt  
 └── print.h　　　　　　　#子目录头文件  
 └── print.cpp　　　　　　#子目录源文件  

print.h中的内容如下  

```C++
// sub/print.h  
#include<iostream>  
void print(std::string str);  
```

print.cpp中的内容如下  

```C++
// sub/print.cpp  
#include "print.h"  
void print(std::string str){  
    std::cout<<str<<std::endl;  
}  
```

main.cpp中的内容如下   

```C++
// main.cpp  
#include "print.h"  
int main(){  
    print("hello world");  
}  
```

sub目录下，CMakeLists.txt  

```makefile
#sub/CMakeLists.txt

# 将指定的源文件print.cpp生成静态库  然后添加到工程中 STATIC(静态库) SHARED(动态库) MODULE(模块库)
add_library(print STATIC print.cpp)  
```

顶层CMakeLists.txt 

```makefile
#test1/CMakeLists.txt  
cmake_minimum_required(VERSION 3.10)  
project(test)  

# 添加目录  将指定目录添加到编译器的头文件搜索路径之下 
include_directories("${PROJECT_SOURCE_DIR}/sub")  

# 添加一个子目录sub并构建该目录  即会执行子目录中的CMakeLists.txt文件
add_subdirectory(sub)  

add_executable(main main.cpp) 

# 将目标文件main与库文件print进行链接
target_link_libraries(main print)  
```

**注意：target_link_libraries 要写在 add_executable 之后**，因为 target_link_libraries 命令格式为 `target_link_libraries(<target> [item1] [item2] [...])` ，其中<target>是指通过 add_executable() 或 add_library() 指令生成的已经创建的目标文件，[item]表示库文件。 


### 设置为可选库

在大工程中，对于更大型的库或者依赖于第三方代码的库，你可能需要将库变为可选的。修改顶层 CMakeLists.txt：

```makefile
# 设置option 相当于宏
# 定义选项默认状态，一般是OFF或者ON，除去ON之外，其他所有值(不赋值)都认为是OFF
option(USE_LIBRARY "Use Library" ON) 

if(USE_LIBRARY)
  include_directories("${PROJECT_SOURCE_DIR}/sub")
  add_subdirectory(sub)
  set(EXTRA_LIBS print)
endif(USE_LIBRARY)

# 添加可执行文件
add_executable(main main.cpp)

target_link_libraries(main  ${EXTRA_LIBS})
```

同时，main.cpp源码中也要做出改变，具体如下：

```C++
// main.cpp 
#ifdef USE_LIBRARY 
#include "print.h"  
#endif

int main(){  
    print("hello world");  

#ifdef USE_LIBRARY
    cout<<"use library"<<endl;
#else
    cout<<"do not use library"<<endl;
#endif 
}
```

为了让 `main.cpp` 可以使用 `USE_LIBRARY` ，我们还要在 **TutorialConfig.h.in** 配置文件中添加一句话：

```C++
#cmakedefine USE_LIBRARY
```

## 添加一个生成文件以及生成器

这一节我们将展示如何将生成的源文件添加到在应用程序的构建过程中。

在sub子目录下，添加一个add.c文件如下：

```C++
#include <stdio.h>

int main(int argc, char *argv[]){
    int a=1;
    int b=2;
	printf("%d",a+b);
}
```

接下来需要在 sub 的 CMakeLists.txt 文件中添加合适的命令来生成可执行文件 add ，然后作为构建过程的一部分来运行它，需要一些命令来完成此任务，代码如下：

```makefile
add_executable(add add.c)

# 添加自定义命令
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/result.h
    COMMAND add ${CMAKE_CURRENT_BINARY_DIR}/result.h
    DEPENDS add
)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_library(print STATIC print.cpp ${CMAKE_CURRENT_BINARY_DIR}/result.h)
```


首先，像添加其他可执行文件一样添加 add 可执行文件，即第一条命令。

add_custom_command():添加自定义命令，第一个签名 OUTPUT 指定生成的文件，第二个签名 COMMAND 为生成文件所需执行的命令，第三个签名 DEPENDS 为依赖项。所以代码中这条命令指定了如何运行 add 来生成 result.h。

接下来我们需要让 CMake 知道 print.cpp 依赖我们生成的 result.h ，这通过将生成的 result.h 加入到 print 库的源列表来实现。并且我们还需要调用 include_directories() 命令将当前二进制目录加入到 include 目录下，这样 result.h 才能被找到并且被 print.cpp 调用。

当项目被构建时，首先将构建 add 的可执行文件，接着将运行 add 可执行文件生成 result.h ，最后才编译包含 result.h 的 print.cpp 来生成 print 库。

`PROJECT_BINARY_DIR` 、 `PROJECT_SOURCE_DIR` 和 `CMAKE_CURRENT_BINARY_DIR` 区别：

`PROJECT_BINARY_DIR` 指向工程构建目录的全路径，比如本文的工程都是在build目录下构建的，所以就表示build所在的路径。

`PROJECT_SOURCE_DIR` 当前工程的源码路径，即当前工程最上层的目录(顶层CMakeLists.txt所在的目录)。

`CMAKE_CURRENT_BINARY_DIR` 当前正在被处理的二进制目录的路径。

## 安装和测试

下一步我们会为我们的工程引入安装规则以及测试支持。安装规则相当直白，对于print库，我们通过向sub的CMakeLists文件添加如下两条语句来设置要安装的库以及头文件：

```makefile
install (TARGETS print DESTINATION bin)
install (FILES print.h DESTINATION include)
```

对于应用程序，在顶层CMakeLists文件中添加下面几行，它们用来安装可执行文件以及配置头文件：

### 添加安装目标

```makefile
install (TARGETS main DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/mainConfig.h" DESTINATION include)
```

这就是要做的全部；现在你应该可以构建 mian 工程了。然后，敲入命令 make install 然后它就会安装需要的头文件，库以及可执行文件CMake的变量CMAKE_INSTALL_PREFIX用来确定这些文件被安装的根目录。添加测试同样也只需要相当浅显的过程。在顶层CMakeLists文件的的尾部补充许多基本的测试代码来确认应用程序可以正确工作。

### 测试

add_test (run main)

测试用例仅仅用来验证程序可以运行，没有出现段错误或其他的崩溃，并且返回值必须是0。