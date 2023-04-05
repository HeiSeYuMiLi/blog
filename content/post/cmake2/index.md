+++
author = ""
title = "创建库"
date = "2022-10-28"
description = "使用gcc/g++和cmake创建静态库和动态库"
tags = [
    "markdown",
    "cmake",
    "gcc/g++",
    "静态库",
    "动态库",
]
categories = [
    "themes",
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  


下面介绍一些cmake的常用方法  

### set方法
set方法用于给下面的变量赋值  

- 一般变量
- 缓存变量
- 环境变量

#### 设置一般变量 
**命令格式**：set(< variable > < value >... [PARENT_SCOPE])  
**命令解释**：`variable` 表示变量名，`value` 是要赋给 `variable` 的值，可以是0个或多个。`PARENT_SCOPE` 表示父作用域，没有设置的情况下，变量 `variable` 的作用域是调用 `set` 方法的函数或者是当前目录，而设置之后，变量的作用域会`传递`到上一层（若原本作用域是某函数则传递给该函数的调用者，若是某目录则传递给上一层目录）。
>作用域：每一个目录或者函数都会创建一个作用域，若没有设置 `PARENT_SCOPE` ，那么变量只能有外向内的传递。  

下面是一些实例：  
(1)首先，设置一个变量，并给定一个值  
```
set(normal_var hello)
message("value = ${normal_var}")
```
输出：
```
value = hello
```  

(2)设置一个变量，给定多个值  
```
set(normal_var hello world)
message("value = ${normal_var}")
```
输出：
```
value = hello;world
```
给一个变量设定多个值后，它输出的结果是用分号 `;` 隔开的。  

(3)设置一个变量，使用 `PARENT_SCOPE` 选项
```
set(normal_var hello world PARENT_SCOPE)
message("value = ${normal_var}")
```
输出：
```
value =
```
可以看到，使用了 `PARENT_SCOPE` 选项后，变量 `normal_var` 的作用域被`传递`到上一层。注意，这里说的是传递，而不是拓宽，所以在当前目录下，输出变量失败。  

(4)在一个函数中设置一个变量，使用 `PARENT_SCOPE` 选项
```
function(test_func1 arg)
	set(normal_var ${arg} PARENT_SCOPE)
	message("value = ${normal_var}")
endfunction(test_func1)

test_func1(hello)
```
输出：
```
value =
```
在另一个函数中调用该函数
```
function(test_func1 arg)
	set(normal_var ${arg} PARENT_SCOPE)
	message("value1 = ${normal_var}")
endfunction(test_func1)

function(test_func2 arg)
	test_func1(${arg})
	message("value2 = ${normal_var}")
endfunction(test_func2)

test_func2(hello) 
```
输出：
```
value1 =
value2 = hello
```
可以看到，test_func1 函数的调用者 test_func2 可以访问变量 normal_var。  

(5)变量作用域在目录中传递  
```
# /sub/CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(sub)

set(normal_var hello PARENT_SCOPE)
message("value1 = ${normal_var}")
```
```
# /CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(test)

add_subdirectory(sub)
message("value2 = ${normal_var}")
```
输出：
```
value1 =
value2 = hello
```  


#### 设置缓存变量
**命令格式**：set(< variable > < value >... CACHE < type > < docstring > [FORCE])  
**命令解释**：首先说明缓存变量实际上就是跨层访问的变量，就好似全局变量。缓存变量的 `<type>` 有一下几类：  

- `BOOL`：布尔值 `ON/OFF` 
- `FILEPATH`：文件路径
- `PATH`：目录路径
- `STRING/STRINGS`：文本行
- `INTERNAL`：文本行，只用于内部，不对外部呈现

`<docstring>`：字符串，作为变量的概要说明  
`FORCE`：强制修改变量的值  
在使用cmake构建的时候，可以使用 `-D` 修改变量的值

下面是几个实例：  
(1)缓存变量是可以跨层访问的变量
```
# /CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(test)

add_subdirectory(sub)
set(cache_var ON CACHE BOOL "BOOL")
message("value = ${cache_var}")
```
```
# /sub/CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(sub)

message("value2 = ${cache_var}")
```
输出：
```
value2 = ON
value = ON
```
可以发现，我们在顶层CMakeLists.txt中设置了变量 `cache_var` ，在下一层CMakeLists.txt中也可以使用它。  

(2)使用 `FORCE` 强制修改变量的值  
我们把上一个例子的sub标签下的CMakeLists.txt修改一下
```
cmake_minimum_required(VERSION 3.10)
project(sub)

set(cache_var OFF CACHE BOOL "BOOL" FORCE)
message("value2 = ${cache_var}")
```
输出：
```
value2 = OFF
value = OFF
```
(3)使用 `-D` 选项修改变量的值
```
cmake_minimum_required(VERSION 3.10)
project(test)

set(cache_var ON CACHE BOOL "BOOL")
message("value = ${cache_var}")
```
在使用cmake创建的时候输入命令：
```
cmake . -Dcache_var=OFF
```
输出：
```
value = OFF
```
>注意：变量名是紧跟在 `-D` 的后面的  


#### 设置环境变量
**命令格式**：set(ENV{< variable >} [< value >])  
**命令含义**：将环境变量设置为值 `<value>`（注意没有`...`），接着使用`$ENV{<variable>}`会得到新的值。  
环境变量设置的几个注意事项：  
(1)该命令设置的环境变量只在当前的cmake进程生效，既不会影响调用者的环境变量，也不会影响系统环境变量。  
(2)如果 `<value>` 值为空或者 `ENV{<variable>}` 后没有参数，则该命令会清除掉当前环境变量的值。  
(3)`<value>` 后的参数会被忽略。  

```
cmake_minimum_required (VERSION 3.10.2)
project (set_test)

message ("value = $ENV{CMAKE_PREFIX_PATH}")
set (ENV{CMAKE_PREFIX_PATH} "/test/sub")
message ("value = $ENV{CMAKE_PREFIX_PATH}")
set (ENV{CMAKE_PREFIX_PATH})
message ("value = $ENV{CMAKE_PREFIX_PATH}")
set (ENV{CMAKE_PREFIX_PATH} "/test/top/") 
message ("value = $ENV{CMAKE_PREFIX_PATH}")
set (ENV{CMAKE_PREFIX_PATH} "") 
message ("value = $ENV{CMAKE_PREFIX_PATH}")
```
输出：
```
>>> value =
>>> value = /test/sub
>>> value =
>>> value = /test/top
>>> value =
```