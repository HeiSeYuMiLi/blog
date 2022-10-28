+++
author = "baoguli"
title = "函数指针和std::function"
date = "2022-10-07"
description = "Custom Test Page"
tags = [
    "c++",
    "std::function",
]
categories = [
    "syntax",
]
aliases = ["migrate-from-jekyl"]
+++

This article offers a sample of basic Markdown syntax that can be used in Hugo content files, also it shows whether basic HTML elements are decorated with CSS in a Hugo theme.
<!--more-->

### 函数指针   
使用指针指向内存中的一段代码，就是**函数指针**：    
>#include<iostream>
>  
>bool compare(int a, int b) {   
>	return a > b;   
>}  
>  
>void print(int a, int b,bool (*p)(int,int)) {   
>	std::cout << p(a,b) << std::endl;   
>}    
>   
>int main() {   
>	bool (*p)(int,int) = compare;  
>	print(1, 2, p);   
>	return 0;   
>}   
上面的实例中，定义了一个函数指针p，它指向函数compare，函数print通过函数指针p回调了compare。  

### std::function    
[std::function](https://en.cppreference.com/w/cpp/utility/functional/function "std::function - cppreference.com")是一个函数包装模板，用于表示各种可调用对象。std::function对象可被拷贝和转移，并且可以使用指定的调用特征来直接调用目标元素。当std::function对象未包裹任何实际的可调用元素，调用该std::function对象将抛出std::bad_function_call异常。  

用std::function改写上面的例子：   
>#include<iostream>  
>#include<functional>  
>  
>bool compare(int a, int b) {  
>	return a > b;  
>}  
>  
>void print(int a, int b,std::function<bool(int,int)> p) {  
>	std::cout << p(a,b) << std::endl;  
>}  
>int main() {  
>	std::function<bool(int, int)> p = compare;  
>	print(1, 2, p);  
>	return 0;  
>}  

### 区别  
那么，既然有函数指针，为什么还要搞一个std::function呢  
函数指针的作用就是把一段代码当作变量传递，即**回调函数**。
