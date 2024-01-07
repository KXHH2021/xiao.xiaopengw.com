---
title: C++核心指南之资源管理（上）概述
date: 2023-11-10 20:00:00
categories:
  - C++ Core Guidelines
tags:
  - C/C++
  - 标签2
  - 裸指针
  - 现代C++
  - 指针
  - RAII
  - 资源管理
description: C++ 核心指南（C++ Core Guidelines）是由 Bjarne Stroustrup、Herb Sutter 等顶尖 C++ 专家创建的一份 C++ 指南、规则及最佳实践。旨在帮助大家正确、高效地使用“现代 C++”。
cover: https://s2.loli.net/2024/01/07/Ra2zTbmiuV7fkJe.png
---
![image.png](https://s2.loli.net/2024/01/07/Ra2zTbmiuV7fkJe.png)

C++ 核心指南（C++ Core Guidelines）是由 Bjarne Stroustrup、Herb Sutter 等顶尖 C++ 专家创建的一份 C++ 指南、规则及最佳实践。旨在帮助大家正确、高效地使用“现代 C++”。

这份指南侧重于接口、资源管理、内存管理、并发等 High-level 主题。遵循这些规则可以最大程度地保证静态类型安全，避免资源泄露及常见的错误，使得程序运行得更快、更好。

## R. Resource Management
本章的基本目标是：不产生资源泄漏，不持有不再需要的资源。资源是指需要申请和（显式或隐式）释放的任何东西，如内存、文件句柄、套接字、锁等。如果一个实体“拥有”资源，则意味着该实体负责释放资源。在个别场景下，资源泄露是可以接受的。例如有足够的内存处理最大的输入，或者出于性能优化考虑，只申请不释放，本章的规则不适用于这类特殊情况。

资源管理规则总结
- R.1: 通过资源句柄和 RAII 自动管理资源
- R.2: 在接口中，裸指针只用来表示单个对象
- R.3: 裸指针（T*）不拥有资源
- R.4: 裸引用（T&）不拥有资源
- R.5: 优先使用对象，非必要不在堆上分配
- R.6: 避免使用非 const 的全局变量
  
## R.1: 通过资源句柄和 RAII 自动管理资源
  
避免泄露以及手动管理资源的复杂性。C++ 的构造/析构反映了资源获取/释放的固有对称性，如 fopen/fclose，lock/unlock，new/delete。在处理资源时，如果需要通过一对函数获取/释放资源，则可以把该资源封装到一个类中：在构造中获取资源，析构中释放资源。

**反面例子**
```undefined
void send(X* x, string_view destination)
{
    auto port = open_port(destination);
    my_mutex.lock();
    // ...
    send(port, x);
    // ...
    my_mutex.unlock();
    close_port(port);
    delete x;
}
```
在这个例子中，必须记得在所有路径上调用 unlock、close_port、delete，并且要保证刚好调用一次，并且一旦 ... 部分抛出异常，就会导致资源 x 泄露，my_mutex 也一直是 locked 状态。


**正面例子**
```undefined
void send(unique_ptr<X> x, string_view destination)
{
    Port port{destination};            // port 拥有 PortHandle
    lock_guard<mutex> guard{my_mutex}; // guard 拥有锁
    // ...
    send(port, x);
    // ...
} // 自动 unlock my_mutex 并且 delete x
```
现在所有资源都是自动释放的，并且保证在所有路径上，无论是否有异常抛出，都只释放一次。不仅如此，这个函数也明确宣布了它对指针 x 的所有权。

Port 只是一个简单的 wrapper，封装了资源：
```undefined
class Port {
    PortHandle port;
public:
    Port(string_view destination) : port{open_port(destination)} { }
    ~Port() { close_port(port); }
    operator PortHandle() { return port; }

    // port 句柄通常不能复制，如果有必要，禁用拷贝构造和拷贝赋值运算符
    Port(const Port&) = delete;
    Port& operator=(const Port&) = delete;
};
```
**注:**

如果资源不是以“具有析构函数的类”的方式呈现，就用类封装一下，或者使用 GSL 库中的 finally ，以便资源能够自动释放

相关惯用法

- RAII
  
## R.2: 在接口中，裸指针只用来表示单个对象

换句话说，不要用裸指针表示一个数组。数组最好用容器类型（如：拥有资源的用 vector、不拥有资源的用 span），容器、视图包含大小信息，可以进行边界检查。

**反面例子**
```undefined
void f(int* p, int n)   // n 是 p[] 元素的数量
{
    // ...
    p[2] = 7;   // bad: 裸指针下标
    // ...
}
```
编译器又不看注释，如果不看其他代码，你无法确定 p 是否真的指向 n 个元素。这种情况最好用 span。

**例子**

```undefined
void g(int* p, int fmt)   // 用 #fmt 格式打印 *p
{
    // ... 仅使用 *p 和 p[0] ...
}
```
**例外**
C 风格字符串通过单个指针（指向以 \0 结尾的字符序列）传递，如果你依赖这种约定，最好使用 zstring，而不是 char*

**注**

很多指向单一元素的指针都可以用引用替代，除非指针可能是 nullptr

**代码检查建议**

如果一个指针不是容器、视图或迭代器，且对该指针进行了算术运算（包括 ++），则标记该指针上的算术运算。如果将该规则应用在旧的代码上，可能产生大量误报

标记作为简单指针传递的数组

## R.3: 裸指针（T*）不拥有资源

在绝大多数的代码中，裸指针都是“非拥有”（non-owning）的，即指针不拥有资源，使用该指针的代码不负责释放指针指向的资源。

我们希望能够标识出“拥有指针”（owning pointer），这样就能安全地释放“拥有指针”指向的资源。

**例子**

```undefined
void f()
{
    int* p1 = new int{7};           // bad: 裸“拥有指针”，f() 要负责释放 p1 指向的资源
    auto p2 = make_unique<int>(7);  // OK: unique_ptr 拥有 int
    // ...
}
unique_ptr 可以保证及时释放资源，即使发生异常也不会产生泄漏，但是 T* 无法保证。
```
**例子**
```undefined
template<typename T>
class X {
public:
    T* p;   // bad: 不清楚 p 是否是“拥有指针”
    T* q;   // bad: 不清楚 q 是否是“拥有指针”
    // ...
};
```

上面的代码无法知道 p、q 是否是“拥有指针”，可以通过下面的方式明确所有权：

```undefined
template<typename T>
class X2 {
public:
    owner<T*> p;  // OK: p 是“拥有指针”
    T* q;         // OK: q 不是“拥有指针”
    // ...
};
```

**注**

owner<T*> 本质上也是 T*，用 owner<T*> 替代 T* 不会影响 ABI 及原来的代码，它只是用来提示程序员和代码分析工具。如果 owner<T*> 是类的成员，通常应该在类的析构中释放指向的资源。

**例外**

主要是历史遗留代码，尤其是那些要需要和 C 或者和 C 风格 C++ ABI 兼容。我们不能把所有的“拥有指针”转换成 unique_ptr 或 shared_ptr 来解决这个问题，部分原因是我们在基本的资源管理代码中需要/使用了裸“拥有指针”。例如常见的 vector 实现中，有一个“拥有指针”和两个“非拥有指针”。许多 ABI（以及所有 C 代码接口）都还使用 T*，其中有些是“拥有指针”。有些接口不能简单地用 owner<T*> 标注，因为要和 C 兼容（这个问题可以用宏来解决，只在 C++ 时展开宏，这是宏少有的正确使用场景）。

**反面例子**

返回裸指针使得调用者无法确切知道该如何进行生命周期管理：调用者是否需要释放指针指向的对象？

```undefined
Gadget* make_gadget(int n)
{
    auto p = new Gadget{n};
    // ...
    return p;
}

void caller(int n)
{
    auto p = make_gadget(n);   // 调用者要记得删除 p
    // ...
    delete p;
}
```
上述代码除了泄漏问题之外，还可能增加不必要的分配/释放操作。如果 Gadget 很小或者移动成本很低，可以直接按值返回（详见 R.5）：
```undefined
Gadget make_gadget(int n)
{
    Gadget g{n};
    // ...
    return g;
}
```
**注**

本规则适用于工厂方法

**注**

在必须使用指针的情况下（比如涉及到多态，返回基类指针），返回智能指针

代码检查建议
- 如果 delete 了一个不是 owner<T> 的裸指针，给出警告
- 如果没有在所有路径上 reset 或 delete 某个 owner<T>，给出警告
- 如果 new 返回的结果赋给一个裸指针，给出警告
- 如果函数返回了一个在函数内分配的对象，且该对象有移动构造，给出警告，建议考虑按值返回
  
## R.4: 裸引用（T&）不拥有资源
和 R.3 一样，大多数的裸引用也不拥有资源。

**例子**
```undefined
void f()
{
    int& r = *new int{7};  // bad: 裸“拥有引用”
    // ...
    delete &r;             // bad: 删除裸指针，违反 R.3 规则
}
```
## R.5: 优先使用作用域对象，非必要不在堆上分配
作用域对象（scoped object）可以是局部对象、全局对象或者类成员。除了作用域对象本身，没有额外的分配/释放的开销。作用域对象内部的成员的生命周期由作用域对象的构造/析构管理。

**例子**
下面代码存在的问题：
- 1. 不必要的分配/释放；
- 2. 如果 ... 部分抛异常，会导致内存泄漏；
- 3.  代码冗长
  
```undefined
void f(int n)
{
    auto p = new Gadget{n};
    // ...
    delete p;
}
```
改用局部对象则可解决上述 3 个问题：

```undefined
void f(int n)
{
    Gadget g{n};
    // ...
}
```
**代码检查建议**
- 如果一个函数内，某个对象在所有路径上都是先分配，再释放，给出警告。建议用局部栈对象。
- 如果一个局部非 const 的 unique_ptr 或者 shared_ptr 在其生命周期结束之前没有被 move、拷贝、重新赋值或者 reset，给出警告
- 例外：如果是一个指向动态数组的局部 unique_ptr，不给出警告，见下面例子：
**例外**
可以创建一个堆上分配缓冲区的 const unique_ptr<T[]>，这是表示动态数组的合法方式

例子
```undefined
int get_median_value(const std::list<int>& integers)
{
  const auto size = integers.size();

  // OK: 声明一个局部 unique_ptr<T[]>.
  const auto local_buffer = std::make_unique_for_overwrite<int[]>(size);

  std::copy_n(begin(integers), size, local_buffer.get());
  std::nth_element(local_buffer.get(), local_buffer.get() + size/2, local_buffer.get() + size);

  return local_buffer[size/2];
}
```

## R.6: 避免使用非 const 的全局变量

（同规则 I.2）非 const 的全局变量隐藏了依赖，容易受到无法预测的改变。

**例子**

```undefined
struct Data {
    // ... lots of stuff ...
} data;            // non-const data

void compute()     // don't
{
    // ... use data ...
}

void output()     // don't
{
    // ... use data ...
}
```
你很难知道是否还会有其他地方改变 **data**

**警告**

全局对象的初始化顺序不能完全确定，所以只能用常量去初始化全局对象。另外，即使是 const 对象，其初始化顺序也可能是未定义的。

**例外**

即便是全局对象，一般来说也比单例好

**注**

全局常量是有用的

本规则除了适用于“全局变量”外，对命名空间里的变量也适用

**替代方案**

如果是出于避免拷贝的目的而使用全局变量，可以考虑用 reference to const 来传递数据。或者把该数据作为对象的状态（即类的成员），然后通过成员函数是来操作该成员。

**警告**

小心数据争用（data race）：如果一个线程可以访问 non-local 数据（或者引用传递的数据），同时另一个线程执行 callee，就可能产生 data race。每个指向可变数据的指针或引用都可能产生数据争用

**注**

不可变数据（immutable data）不会产生静态条件（race condition）

**注**

这条规则是“尽量避免”，不是“禁止使用”。会有一些（极少数）例外，比如 cin、cout 和 cerr

**代码检查建议**

报告所有在命名空间/全局中声明的非 const 变量及指向非 const 对象的指针或引用

原文链接：**https://www.cnblogs.com/tengzijian/p/17500834.html**


