+++
author = "baoguli"
title = "简谈STL六大部件"
date = "2023-6-23"
description = "简单聊聊 C++ STL 的六大部件"
tags = [
    "c++",
]
categories = [
    "syntax",
]
aliases = ["create-library"]
image = ""
+++  


**STL** 是 C++ 标准模板库 (Standard Template Library) 的缩写，是 C++ 标准库的一部分，不需要单独安装。STL 利用**模板 (Template)** 实现了常用的数据结构和算法，并且做到了数据结构和算法的分离。

STL 主要包括六大部件：容器 (Containers)、分配器 (Allocators)、算法 (Algorithms)、迭代器 (Iterators)、适配器 (Adapters) 和仿函数 (Functors)。STL 可以提高 C++ 编程的效率和质量，是 C++ 开发者必备的知识。

### 容器

STL 容器按照类型可以分为**顺序容器**和**关联容器**。

#### 顺序容器

顺序容器主要有：`vector`、`deque`、`list`、`forward_list` 和 `array`。

主要说说 vector，它实现了一个动态数组，可以进行元素的插入和删除，在此过程中，vector会动态调整所占用的内存空间。

vector 的特点是：

- 它支持随机访问，即可以通过下标或者迭代器来访问或修改容器中的元素，时间复杂度为 O(1)。

- 它在尾部插入或删除元素的效率很高，时间复杂度为 O(1)。

- 它在头部或中部插入或删除元素的效率较低，因为需要移动其它元素，时间复杂度为 O(n)。

- 它没有空间预留习惯，所以每分配一个元素都会从内存中分配，每删除一个元素都会释放它占用的内存。

vector 是最常用的容器。

顺序容器的使用场合：

一般大多数的情况都可以使用 vector 容器，除非有特定需求使用其他容器更加合理方便；

如果需要在一串数字的头尾进行操作，偏向 deque；

对于中间的元素插入或删除，可采用forward_list (单向链表)或list (双向链表)，不需要移动元素，只需改变相关结点的指针域即可。

#### 关联容器

关联容器主要有：`set`、`map`、`multiset`、`multimap`、`unordered_set`、`unordered_map`、`unordered_multiset`、`unordered_multiamp`。

关联式容器每一个元素都有一个键值(key)，对于二元关联容器，还拥有实值(value)容器中的元素顺序不能由程序员来决定，有 set(集合)和 map(映射)这两大类，它们均是以 RB-Tree(red-black tree，红黑树)为底层架构。

关联容器的使用场合：

如果只负责查找内容，可以使用 set 或 mutliset；

如果需要同时放入容器的数据不止一个，并且是不同类型，比如一个为整型 int，一个为 string 字符串型，就可以考虑使用 map 或 mutlimap；

### 分配器

分配器主要用来处理所有给定容器(vector，list，map 等)内存的分配和释放。

分配器可以帮助我们将内存分配和对象构造分离开来，提供一种类型感知的内存分配方法，它分配的内存是原始的、未构造的。

分配器是一个类，有着叫 allocate() 和 deallocate() 成员函数(相当于 malloc 和 free)，还有用于维护所分配的内存的辅助函数和指示如何使用这些内存的 typedef (指针或引用类型的名字)。

STL 中的分配器分为两级，第一级分配器较为简单，涉及到的语法包含函数指针，主要使用了C语言中的 malloc 和 free 函数，还有一个在超出内存情况下的处理函数(oom：out of memory)。第二级分配器是 STL 默认使用的分配器，它采用了内存池(memory pool)的技术，以空间换时间，提高了内存分配和释放的效率。

第二级分配器会维护16个自由链表(free list)，每个自由链表管理一定大小范围的内存块，从8字节到128字节，每次增加8字节。它适用于小于等于128字节的内存需求。当需要分配内存时，第二级分配器会根据请求大小找到合适的自由链表，并从中取出一个内存块返回。当需要释放内存时，第二级分配器会根据释放块的大小找到对应的自由链表，并将其归还。

当STL分配器收到一个内存分配的请求时，它会先判断请求的大小是否大于128字节，如果是，就直接调用第一级分配器来分配内存；如果不是，就通过一个哈希函数计算出对应的自由链表的索引，然后从该链表中取出一个空闲的内存块返回给用户。

如果对应的自由链表中没有空闲的内存块，那么第二级分配器就会从内存池中申请一块新的内存，并将其切割成若干个相同大小的内存块，其中一个返回给用户，其余的加入到自由链表中。

如果内存池中也没有足够的空间，那么第二级分配器就会尝试从堆中申请更多的空间，并将其加入到内存池中。如果堆中也没有足够的空间，那么第二级分配器就会调用第一级分配器的处理函数，尝试释放一些未使用的内存或者抛出异常。

一个简单分配器的有如下实现：

```c++
template <typename T>
class MyAllocator {
public:
    typedef T value_type;
    MyAllocator() noexcept {}
    template <typename U>
    MyAllocator(const MyAllocator<U>&) noexcept {}
    T* allocate(std::size_t n) {
        if (n > std::size_t(-1) / sizeof(T)) throw std::bad_alloc();
        if (auto p = static_cast<T*>(std::malloc(n * sizeof(T)))) return p;
        throw std::bad_alloc();
    }
    void deallocate(T* p, std::size_t) noexcept {
        std::free(p);
    }
};
```

### 算法

STL 算法是一些通用的函数模板，用来处理各种容器中的元素，实现了一些常见的操作，如查找、排序、复制、修改等。

STL算法可以分为四类：

- 非可变序列算法：指不直接修改其所操作的容器内容的算法，如 count、find、equal、for_each 等。

- 可变序列算法：指可以修改它们所操作的容器内容的算法，如 copy、fill、replace、remove、reverse 等。

- 排序算法：包括对序列进行排序和合并的算法、搜索算法以及有序序列上的集合操作，如 sort、merge、binary_search、set_union 等。

- 数值算法：指对容器中的数值元素进行一些计算的算法，如 accumulate、inner_product、partial_sum 等。

STL算法的特点是：

- 算法和容器分离：STL 算法不依赖于具体的容器类型，而是通过迭代器来访问容器中的元素，这样就可以实现算法和容器之间的解耦，提高了代码的复用性和可扩展性。

- 算法和函数对象配合：STL 算法通常有两个版本，一个是默认版本，另一个是接受一个函数对象作为参数的版本，这样就可以根据不同的需求来定制算法的行为，提高了代码的灵活性和可定制性。

- 算法和迭代器配合：STL 算法根据所需的迭代器类型来确定其最低要求，比如 find 只需要一个输入迭代器，而 sort 需要一个随机访问迭代器，这样就可以保证算法在满足最低要求的情况下正常工作，同时也可以接受更高类型的迭代器。

### 迭代器

STL迭代器是一种类似于指针的对象，用来访问容器中的元素，实现了容器和算法之间的解耦。

STL 迭代器有以下的特点：

- 迭代器可以指向容器中的某个元素，通过解引用操作符(*)或者箭头操作符(->)来读写它指向的元素。

- 迭代器可以通过自增或自减操作符(++或--)来移动到容器中的下一个或上一个元素。

- 迭代器可以通过比较操作符(==或!=)来判断是否指向同一个元素。

- 迭代器可以通过容器的成员函数(如 begin()或 end())来获取指向容器中第一个或最后一个元素的迭代器。

STL 迭代器根据其功能强弱可以分为五种类型：

- 输入迭代器(InputIterator)：支持读取元素，单向移动，相等比较。

- 输出迭代器(OutputIterator)：支持写入元素，单向移动。

- 前向迭代器(ForwardIterator)：支持读写元素，单向移动，相等比较，继承自输入迭代器和输出迭代器。

- 双向迭代器(BidirectionalIterator)：支持读写元素，双向移动，相等比较，继承自前向迭代器。

- 随机访问迭代器(RandomAccessIterator)：支持读写元素，随机访问，相等比较，大小比较，算术运算，继承自双向迭代器。

不同类型的容器提供了不同类型的迭代器，如 vector 和 deque 提供了随机访问迭代器，list 提供了双向迭代器，set 和 map 提供了双向或者随机访问迭代器。不同类型的迭代器决定了容器能够支持哪些算法，如排序算法需要随机访问迭代器，因此不能用于 list 容器。

### 适配器

STL 适配器是一种设计模式，用来将一个类的接口转换为另一个类的接口，使得原本不兼容的类可以一起运作。

STL适配器可以分为三类：

- 容器适配器(container adapter)：改变容器的接口，提供一些特定的功能，如 stack(栈)、queue(队列)和 priority_queue(优先队列)。

- 迭代器适配器(iterator adapter)：改变迭代器的接口，提供一些特定的访问方式，如 front_insert_iterator(前插入迭代器)、back_insert_iterator(后插入迭代器)、insert_iterator(插入迭代器)、reverse_iterator(反向迭代器)和 stream_iterator(流迭代器)。

- 仿函数适配器(functor adapter)：改变仿函数的接口，提供一些特定的操作方式，如 bind(绑定)、negate(否定)、compose(组合)和 mem_fun(成员函数)。

STL适配器的特点是：

- 适配器本身是一个新的自定义类型，内部包含一个或多个被适配的类型的成员，并对其接口进行改造，向外部提供新的接口。
- 适配器可以对容器、迭代器和仿函数进行二次包装，增加或减少其功能，提高其灵活性和可定制性。
- 适配器可以通过一些辅助函数或模板来创建，如 front_inserter、back_inserter、inserter、rbegin、rend、bind、not1、not2等。

### 仿函数

STL 仿函数是一种通过重载()运算符模拟函数行为的类，也称为函数对象。

STL 仿函数的特点是：

- 仿函数可以作为算法的参数，提供一些特定的功能，如算术运算、关系运算、逻辑运算等。
- 仿函数可以定义自己的相应类型，如参数类型、返回值类型等，以便和STL其他组件(如配接器adapter)搭配，产生更灵活的变化。
- 仿函数可以保存状态信息，而普通函数或函数指针不能。

STL仿函数可以分为两种：

- 一元仿函数(unary function)：只接受一个参数的仿函数，继承自unary_function类模板。
- 二元仿函数(binary function)：接受两个参数的仿函数，继承自binary_function类模板。

STL仿函数还可以根据功能划分为三大类：

- 算术运算类(arithmetic)：提供加、减、乘、除、取模、取负等操作，如plus、minus、multiplies、divides、modulus、negate等。
- 关系运算类(relational)：提供等于、不等于、大于、小于、大于等于、小于等于等操作，如equal_to、not_equal_to、greater、less、greater_equal、less_equal等。
- 逻辑运算类(logical)：提供与、或、非等操作，如logical_and、logical_or、logical_not等。
