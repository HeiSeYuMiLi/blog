+++
author = ""
title = "CMake与gcc/g++"
date = "2022-10-28"
description = "使用gcc/g++和cmake创建静态库和动态库"
tags = [
    "cmake",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  


编译一个文件，一共有四个步骤：预处理，编译，汇编，链接。  

预处理：将文件包含的头文件和宏定义等展开或替换。  

编译：将预处理后的文件翻译成更底层的`汇编语言`。  

汇编：将编译后的汇编程序翻译成二进制文件(即`目标文件`)。  

链接：将目标文件链接库，生成`可执行文件`。  

## 使用gcc/g++编译  

使用gcc/g++编译器编译一个文件，只需在命令行中输入(以g++为例)

```bash
g++ main.cpp -o test
```

便可以得到一个名为test的可执行文件，然后执行：

```bash
./test
```

## gcc/g++ 创建库  

有目录test1内结构如下  
├── main.cpp　　　　　　　#源文件，包含main函数  
├── sub　　　　　　　　　#子目录    
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

### 参数-I 头文件搜索路径
使用`g++ main.cpp -o test`命令对上面的文件main.cpp进行编译会得到一个错误 —— 头文件print未声明。这是因为编译器按照默认的头文件搜索路径找不到头文件print，解决的办法是使用参数`-I`告诉编译器，按照指定的路径查找头文件，具体命令如下：  
```bash
g++ main.cpp sub/print.cpp -Isub -o test
```
自定义的搜索路径紧跟在`-I`的后面，这样便可以得到可执行文件。  



### 静态库 动态库
首先说明一下静态库和动态库的区别：Linux下静态库以.a为后缀结尾，动态库以.so为后缀结尾，而二者最本质的区别是`打进可执行文件的时机不同`。静态库是在链接阶段打进可执行文件，而动态库是在可执行文件运行的时候才加载进来的。静态库的优点是文件执行快，但是缺点也很明显就是多消耗了内存空间，动态库刚好相反，节省了内存空间，但是需要花费时间在运行的时候加载库。  


下面说明如何创建库，先是静态库。  
静态库实则是一个二进制目标文件，链接时直接打进可执行文件。知道了原理就好操作了，首先生成一个.o的二进制文件：
```bash
g++ sub/print.cpp -Isub -c -o print.o
```
>`-c`参数表示只进行预处理、编译、汇编操作，从而得到一个.o的文件  

然后将这个二进制目标文件归档为.a的静态库。库的命名格式为`lib+库名+后缀`。下面使用`ar`命令创建一个名为print的静态库：
```bash
ar rs libprint.a print.o
```
至此，静态库创建完毕，下面将库链接到main.cpp中：
```bash
g++ main.cpp -Isub -L. -lprint -o test
```
得到了可执行文件test。其中，命令需要使用-I指明头文件搜索路径sub，使用-L指明库的搜索路径(当前目录下用 . 表示)，使用-l指明库的名称。   


下面是动态库(又称共享库)  
执行命令：
```bash
g++ sub/print.cpp -Isub -fPIC -shared -o libprint.so
```
`-fPIC`编译选项，表示生成位置无关的代码，就是说代码可以在任意位置执行，这是动态库需要的特性。`-shared`链接选项，告诉编译器生成动态库而不是可执行文件。所以上面的命令也可以分两步完成：
```bash
g++ sub/print.cpp -Isub -c -fPIC -o print.o
g++ print.o -shared -o libprint.so
```
然后和静态库一样，链接到main.cpp中：
```bash
g++ main.cpp -Isub -L. -lprint -o test
```


## cmake 创建库
接下来使用cmake建立一个动态库和静态库，完成gcc/g++对应的操作。

#### 建立静态库
在test1目录下，新建子目录sub  
进入sub目录，新建CMakeLists.txt  
```makefile
#sub/CMakeLists.txt
cmake_minimum_required(VERSION 3.10)  
project(sub)  
add_library(print STATIC print.cpp)  
```

回到test1，新建CMakeLists.txt 
```makefile
#test1/CMakeLists.txt  
cmake_minimum_required(VERSION 3.10)  
project(test)  
include_directories("${PROJECT_SOURCE_DIR}/sub")  
add_subdirectory(sub)  
add_executable(main main.cpp) 
target_link_libraries(main print)  
```

当前目录结构如下  
├── CMakeLists.txt　　　　　　#父目录的CMakeList.txt  
├── main.cpp　　　　　　　　#源文件，包含main函数  
├── sub　　　　　　　　　　#子目录    
 └── CMakeLists.txt　　　　#子目录的CMakeLists.txt  
 └── print.h　　　　　　　#子目录头文件  
 └── print.cpp　　　　　　#子目录源文件  

其中，add_library(print STATIC print.cpp)的含义是将指定的源文件print.cpp生成链接文件，然后添加到工程中。  
命令格式  
```makefile
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

### 创建动态库
创建动态库只需要在sub/CmakeList.txt中改写一行即可：
```makefile
add_library(print SHARED print.cpp)  
```

## 结语
使用gcc/g++编译器或者cmake都可以对一个文件进行编译，其中g++是一步一步的进行编译，更利于对编译过程的理解，而cmake编译一个文件可以说一步到位，十分方便简单。