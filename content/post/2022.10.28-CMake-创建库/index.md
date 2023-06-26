+++
author = "baoguli"
title = "CMake创建库"
date = "2022-10-28"
description = "使用cmake创建静态库和动态库"
tags = [
    "cmake",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  

## 1.构建一个最简单的项目

使用cmake构建一个最简单的项目莫过于将一个源文件构建成一个可执行文件。有源文件如下：

```C++
//hello_world.cpp
#include<iostream>
int main(){
    std::cout<<"Hello world!"<<std::endl;
}
```

在同一目录下，有CMakeLists.txt如下：

```C++
#CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(Hello_world)
add_executable(Hello_world hello_world.cpp)
```

为了保证源码路径的干净整洁，在当前目录下创建一个build目录，存放构建项目产生的编译产物，这种方式叫作外部构建。反之，把编译产物存放在当前目录中叫作内部构建。  

现在准备工作完成，开始构建。在build目录下输入以下语句，以生成构建系统：

```bash
cmake ..
```

 `..` 表示CMakeLists.txt文件在上一级目录。  

 此时，继续输入：

 ```bash
 cmake --build .
 ```

 执行此命令后，将生成可执行文件，其中 `--build` 指明可执行文件的存放目录， `.` 表示当前目录。  

 输入命令执行：

 ```bash
 ./Hello_world
 ```

 便可得到输出 Hello world!  


## 2.创建库

Linux下静态库以.a为后缀结尾，动态库以.so为后缀结尾，二者最本质的区别是`打进可执行文件的时机不同`。静态库是在链接阶段打进可执行文件，而动态库是在可执行文件运行的时候才加载进来的。静态库的优点是文件执行快，但是缺点也很明显就是多消耗了内存空间，动态库刚好相反，节省了内存空间，但是需要花费时间在运行的时候加载库。

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
cmake_minimum_required(VERSION 3.10)  
project(sub)  
add_library(print STATIC print.cpp)  
```

顶层CMakeLists.txt 

```makefile
#test1/CMakeLists.txt  
cmake_minimum_required(VERSION 3.10)  
project(test)  
include_directories("${PROJECT_SOURCE_DIR}/sub")  
add_subdirectory(sub)  
add_executable(main main.cpp) 
target_link_libraries(main print)  
```

其中，add_library(print STATIC print.cpp)的含义是将指定的源文件print.cpp生成链接文件，然后添加到工程中。  

命令格式  

```bash
add_library(<name> [STATIC | SHARED | MODULE]  
            [EXCLUDE_FROM_ALL]  
            [source1] [source2] [...])    
```

命令解析  

[]内的内容是可选项。  

<name>  

构建成的库名  

STATIC|SHARED|MOUDLE

STATIC(静态库) SHARED(动态库) MODULE(模块库)用来指定库的类型。  

使用STATIC构建生成静态库(name.a)，使用SHARED构建生成动态库(name.so)。  

EXCLUDE_FROM_ALL  

添加EXCLUDE_FROM_ALL属性的库在默认编译的时候，不会被编译，如果要编译它们，需要手动编译。  

source1 source1 ...  

构建库的文件  

include_directories("${PROJECT_SOURCE_DIR}/sub")表示将指定目录添加到编译器的头文件搜索路径之下  

命令格式  

```makefile
include_directories ([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])  
```

命令解析  

AFTER|BEFORE

AFTER或BEFORE选项说明将指定目录添加到列表的前面或者后面。(默认情况下，include_directories命令会将目录添加到列表最后，也可以通过命令设置CMAKE_INCLUDE_DIRECTORIES_BEFORE变量为ON来改变它默认行为，将目录添加到列表前面。)  

SYSTEM  

把指定目录当成系统的搜索目录。  

```
在上面的例子中，如果不使用include_directories包含子目录sub,直接在main.cpp里面包含"print.h"，执行cmake --build .，会提示找不到头文件的错误。  
```


add_subdirectory(sub)表示添加一个子目录sub并构建该子目录。  

命令格式  

```makefile
add_subdirectory (source_dir [binary_dir] [EXCLUDE_FROM_ALL])   
```

命令解析  

source_dir  

必选参数。该参数指定一个子目录，子目录下应该包含CMakeLists.txt文件和代码文件。  

binary_dir  

可选参数。该参数指定一个目录，用于存放输出文件。如果该参数没有指定，则默认的输出目录使用source_dir。  

EXCLUDE_FROM_ALL  

可选参数。当指定了该参数，则子目录下的目标不会被父目录下的目标文件包含进去，父目录的CMakeLists.txt不会构建子目录的目标文件，必须在子目录下显式去构建。例外情况：当父目录的目标依赖于子目录的目标，则子目录的目标仍然会被构建出来以满足依赖关系（例如使用了target_link_libraries）。  

target_link_libraries(main print)表示将目标文件main与库文件print进行链接。  

命令格式  

```makefile
target_link_libraries(<target> [item1] [item2] [...])   
```

命令解析  

target_link_libraries 要写在 add_executable 之后。   

<target>是指通过add_executable()或add_library()指令生成已经创建的目标文件。[item]表示库文件。  
target_link_libraries里库文件的顺序符合gcc链接顺序的规则，即被依赖的库放在依赖它的库的后面，比如  

```makefile
target_link_libraries(hello A.so B.a C.so)  
```

在上面的命令中，libA.so可能依赖于libB.a和libC.so，如果顺序有错，链接时会报错。


完成上面的操作后，在test1目录下新建build文件夹，然后进入  

使用cmake .. 指令构建，并执行可执行文件  

```bash
cmake ..
cmake --build .
./main  
```

便可以得到输出  

```bash
hello world
```
