---
layout: post
title: "EffectiveModernC++笔记"
categories: Book
tags: Book C/C++
author: August
typora-root-url: ..
---

* content
{:toc}


该文记录《Effective Modern C++》读后记录。



# Effective Modern C++

![](/media/image/2023-05-28-EffectiveModernC++/EffectiveModernC++.svg)



## 1. 第1章 类型推导

`C++98` 有一套类型推导的规则：用于函数模板的规则。`C++11` 修改了其中的一些规则并增加了两套规则，一套用于 `auto` ，一套用于 `decltype`。`C++14` 扩展了 `auto` 和 `decltype` 可能使用的范围。

### 1.1. 条款一：理解模板类型推导

- 在模板类型推导时，有引用的实参会被视为无引用，他们的引用会被忽略
- 对于通用引用的推导，左值实参会被特殊对待
- 对于传值类型推导，`const` 和/或 `volatile` 实参会被认为是 `nonconst` 的和 `non-volatile` 的
- 在模板类型推导时，数组名或者函数名实参会退化为指针，除非它们被用于初始化引用

### 1.2. 条款二：理解auto类型推导

- `auto` 类型推导通常和模板类型推导相同，但是 `auto` 类型推导假定花括号初始化代表 `std::initializer_list`，而模板类型推导不这样做
- 在 `C++14` 中 `auto` 允许出现在函数返回值或者 `lambda` 函数形参中，但是它的工作机制是模板类型推导那一套方案，而不是 `auto` 类型推导

### 1.3. 条款三：理解decltype

```cpp
// C++11
template<typename Container, typename Index> //最终的C++11版本
auto authAndAccess(Container&& c, Index i)->decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}

// C++14
template<typename Container, typename Index> //最终的C++14版本
decltype(auto) authAndAccess(Container&& c, Index i)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

- `decltype` 总是不加修改的产生变量或者表达式的类型。
- 对于 `T` 类型的不是单纯的变量名的左值表达式，`decltype` 总是产出 `T` 的引用即 `T&`。
- `C++14` 支持 `decltype(auto)`，就像 `auto` 一样，推导出类型，但是它使用 `decltype` 的规则进行推导。

### 1.4. 条款四：学会查看类型推导结果

- 类型推断可以从 `IDE` 看出，从编译器报错看出，从 `Boost TypeIndex` 库的使用看出
- 这些工具可能既不准确也无帮助，所以理解 `C++` 类型推导规则才是最重要的



## 2. 第2章 auto

### 2.1. 条款五：优先考虑auto而非显式类型声明

```cpp
// 显示类型
std::function<bool(const std::unique_ptr<Widget> &, const std::unique_ptr<Widget> &)>
derefUPLess = [](const std::unique_ptr<Widget> &p1, const std::unique_ptr<Widget> &p2)
 { return *p1 < *p2; };

// auto 11
auto derefUPLess = [](const std::unique_ptr<Widget> &p1, const std::unique_ptr<Widget> &p2) { return *p1 < *p2; };

// 14
auto derefLess = [](const auto& p1, const auto& p2) { return *p1 < *p2; };
```

- `auto` 变量必须初始化，通常它可以避免一些移植性和效率性的问题，也使得重构更方便，还能让你少打几个字。
- `auto` 类型的变量可能会踩到一些陷阱。

### 2.2. 条款六：auto推导若非己愿，使用显式类型初始化惯用法

```cpp
// 计算矩阵，不可见的代理类通常不适用于auto（即类似于Sum<Sum<Sum<Matrix, Matrix>, Matrix>, Matrix>的东西）
Matrix sum = m1 + m2 + m3 + m4;
```

- 不可见的代理类可能会使 `auto` 从表达式中推导出“错误的”类型
- 显式类型初始器惯用法强制 `auto` 推导出你想要的结果



## 3. 第3章 移步现代C++

### 3.1. 条款七：区别使用()和{}创建对象

```cpp
//最令人头疼的解析！声明一个函数w2，返回Widget，实际想调用构造函数
Widget w2();
```

```cpp
//使用非std::initializer_list构造函数创建一个包含10个元素的std::vector，所有的元素的值都是20
std::vector<int> v1(10, 20); 

//使用std::initializer_list构造函数创建包含两个元素的std::vector，元素的值为10和20
std::vector<int> v2{10, 20};
```

- 括号初始化是最广泛使用的初始化语法，它防止变窄转换，并且对于 `C++` 最令人头疼的解析有天生的免疫性
- 在构造函数重载决议中，括号初始化尽最大可能与 `std::initializer_list` 参数匹配，即便其他构造函数看起来是更好的选择
- 对于数值类型的 `std::vector` 来说使用花括号初始化和小括号初始化会造成巨大的不同
- 在模板类选择使用小括号初始化或使用花括号初始化创建对象是一个挑战

### 3.2. 条款八：优先考虑nullptr而非0和NULL

### 3.3. 条款九：优先考虑别名声明而非typedef

```cpp
// using
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>; 
MyAllocList<Widget> lw; //用户代码

// typedef
template<typename T>
struct MyAllocList {
	typedef std::list<T, MyAlloc<T>> type;
};
MyAllocList<Widget>::type lw; //用户代码
```

- `typedef` 不支持模板化，但是别名声明支持。
- 别名模板避免了使用 `::type` 后缀，而且在模板中使用 `typedef` 还需要在前面加上 `typename`
- `C++14` 提供了 `C++11` 所有 `type traits` 转换的别名声明版本

### 3.4. 条款十：优先考虑限域enum而非未限域enum

```cpp
// 要为非限域enum指定底层类型，结果就可以前向声明：
// 非限域enum前向声明底层类型为std::uint8_t
enum Color: std::uint8_t;
```

- C++98 的 `enum` 即非限域 `enum`。
- 使用限域 `enum` 来减少命名空间污染。
- 限域 `enum` 的枚举名仅在 `enum` 内可见。要转换为其它类型只能使用 `cast`。
- 非限域/限域 `enum` 都支持底层类型说明语法，限域 `enum` 底层类型默认是 `int`。非限域 `enum` 没有默认底层类型。
- 限域 `enum` 总是可以前置声明。非限域 `enum` 仅当指定它们的底层类型时才能前置。

### 3.5. 条款十一：优先考虑使用deleted函数而非使用未定义的私有声明

```cpp
// deleted函数被声明为public而不是private。
template <class charT, class traits = char_traits<charT> >
class basic_ios : public ios_base {
public:
    basic_ios(const basic_ios& ) = delete;
    basic_ios& operator=(const basic_ios&) = delete;
};

// deleted函数还有一个重要的优势是任何函数都可以标记为deleted
bool isLucky(int number); 		//原始版本
bool isLucky(char) = delete; 	//拒绝char
bool isLucky(bool) = delete; 	//拒绝bool
bool isLucky(double) = delete; 	//拒绝float和double
```

- 比起声明函数为 `private` 但不定义，使用 `deleted` 函数更好
- 任何函数都能被删除（be deleted），包括非成员函数和模板实例

### 3.6. 条款十二：使用override声明重写函数

`C++11` 提供一个方法让你可以显式地指定一个派生类函数是基类版本的重写：将它声明为 `override`。`C++11` 引入了两个上下文关键字（contextualkeywords），`override` 和 `final`（向虚函数添加final可以防止派生类重写。final也能用于类，这时这个类不能用作基类）。

我们需要的是指明当 `data` 被右值 `Widget` 对象调用的时候结果也应该是一个右值。现在就可以使用引用限定，为左值 `Widget` 和右值 `Widget` 写一个 `data` 的重载函数来达成这一目的：

```cpp
class Widget {
public:
    using DataType = std::vector<double>;
    //对于左值Widgets,返回左值
    DataType& data() & 
    { return values; }

    //对于右值Widgets,返回右值
    DataType data() && 
    { return std::move(values); } 
private:
    DataType values;
};
```

- 为重写函数加上 `override`
- 成员函数引用限定让我们可以区别对待左值对象和右值对象（即*this）

### 3.7. 条款十三：优先考虑const_iterator而非iterator

- 优先考虑 `const_iterator` 而非 `iterator`
- 在最大程度通用的代码中，优先考虑非成员函数版本的 `begin`，`end`，`rbegin` 等，而非同名成员函数

### 3.8. 条款十四：如果函数不抛出异常请使用noexcept

```cpp
RetType function(params) noexcept; //极尽所能优化,c++11
RetType function(params) throw(); //较少优化,C++98
RetType function(params); //较少优化
```

- `noexcept` 是函数接口的一部分，这意味着调用者可能会依赖它
- `noexcept` 函数较之于 `non-noexcept` 函数更容易优化
- `noexcept` 对于移动语义，`swap`，内存释放函数和析构函数非常有用
- 大多数函数是异常中立的而不是 `noexcept`

### 3.9. 条款十五：尽可能的使用constexpr

`constexpr` 表明一个值不仅仅是常量，还是编译期可知的。

当 `constexpr` 被用于函数的时候不能确定。

```cpp
// C++11中，constexpr函数的代码不超过一行语句：一个return。听起
// 来很受限，但实际上有两个技巧可以扩展constexpr函数的表达能力。
// 第一，使用三元运算符“?:”来代替if-else语句，第二，使用递归代替循环。
constexpr int pow(int base, int exp) noexcept
{
 	return (exp == 0 ? 1 : base * pow(base, exp - 1));
}

// C++14 无限制
```

- `constexpr` 对象是 `const`，它被在编译期可知的值初始化
- 当传递编译期可知的值时，`constexpr` 函数可以产出编译期可知的结果
- `constexpr` 对象和函数可以使用的范围比 `non-constexpr` 对象和函数要大
- `constexpr` 是对象和函数接口的一部分

### 3.10. 条款十六：让const成员函数线程安全

- 确保 `const` 成员函数线程安全，除非你确定它们永远不会在并发上下文（concurrent context）中使用。
- 使用 `std::atomic` 变量可能比互斥量提供更好的性能，但是它只适合操作单个变量或内存位置。

### 3.11. 条款十七：理解特殊成员函数的生成

```cpp
class Base {
public:
    Base() = default;
    virtual ~Base() = default; //使析构函数virtual

    Base(Base&&) = default; //支持移动
    Base& operator=(Base&&) = default;

    Base(const Base&) = default; //支持拷贝
    Base& operator=(const Base&) = default;
};
```

`C++11` 对于特殊成员函数处理的规则如下：

- **默认构造函数**：和 `C++98` 规则相同。仅当类不存在用户声明的构造函数时才自动生成。
- **析构函数**：基本上和 `C++98` 相同；稍微不同的是现在析构默认 `noexcept`。和 `C++98` 一样，仅当基类析构为虚函数时该类析构才为虚函数。
- **拷贝构造函数**：和 `C++98` 运行时行为一样：逐成员拷贝 `non-static` 数据。仅当类没有用户定义的拷贝构造时才生成。如果类声明了移动操作它就是 `delete` 的。当用户声明了拷贝赋值或者析构，该函数自动生成已被废弃。
- **拷贝赋值运算符**：和 `C++98` 运行时行为一样：逐成员拷贝赋值 `non-static` 数据。仅当类没有用户定义的拷贝赋值时才生成。如果类声明了移动操作它就是 `delete` 的。当用户声明了拷贝构造或者析构，该函数自动生成已被废弃。
- **移动构造函数**和**移动赋值运算符**：都对非 `static` 数据执行逐成员移动。仅当类没有用户定义的拷贝操作，移动操作或析构时才自动生成。

请记住：

- 特殊成员函数是编译器可能自动生成的函数：默认构造函数，析构函数，拷贝操作，移动操作。
- 移动操作仅当类没有显式声明移动操作，拷贝操作，析构函数时才自动生成。
- 拷贝构造函数仅当类没有显式声明拷贝构造函数时才自动生成，并且如果用户声明了移动操作，拷贝构造就是 `delete`。拷贝赋值运算符仅当类没有显式声明拷贝赋值运算符时才自动生成，并且如果用户声明了移动操作，拷贝赋值运算符就是 `delete`。当用户声明了析构函数，拷贝操作的自动生成已被废弃。
- 成员函数模板不抑制特殊成员函数的生成。



## 4. 第4章 智能指针

在 `C++11` 中存在四种智能指针： `std::auto_ptr` ， `std::unique_ptr` ，`std::shared_ptr`，`std::weak_ptr`。都是被设计用来帮助管理动态对象的生命周期，在适当的时间通过适当的方式来销毁对象，以避免出现资源泄露或者异常行为。

### 4.1. 条款十八：对于独占资源使用std::unique_ptr

- `std::unique_ptr` 是轻量级、快速的、只可移动（move-only）的管理专有所有权语义资源的智能指针
- 默认情况，资源销毁通过 `delete` 实现，但是支持自定义删除器。有状态的删除器和函数指针会增加 `std::unique_ptr` 对象的大小
- 将 `std::unique_ptr` 转化为 `std::shared_ptr` 非常简单

### 4.2. 条款十九：对于共享资源使用std::shared_ptr

`std::shared_ptr` 通过引用计数（reference count）来确保它是否是最后一个指向某种资源的指针，引用计数关联资源并跟踪有多少 `std::shared_ptr` 指向该资源。

- `std::shared_ptr` 大小是原始指针的两倍，因为它内部包含一个指向资源的原始指针，还包含一个指向资源的引用计数值的原始指针。
- 引用计数的内存必须动态分配。
- 递增递减引用计数必须是原子性的，因为多个`reader`、`writer`可能在不同的线程。

![image-20230530223651983](/media/image/2023-05-28-EffectiveModernC++/shared_ptr.png)

- ``std::shared_ptr`` 为有共享所有权的任意资源提供一种自动垃圾回收的便捷方式。
- 较之于 `std::unique_ptr`，`std::shared_ptr` 对象通常大两倍，控制块会产生开销，需要原子性的引用计数修改操作。
- 默认资源销毁是通过 `delete`，但是也支持自定义删除器。删除器的类型是什么对于 `std::shared_ptr` 的类型没有影响。
- 避免从原始指针变量上创建 `std::shared_ptr`。

### 4.3. 条款二十：当std::shard_ptr可能悬空时使用std::weak_ptr

```cpp
//spw创建之后，指向的Widget的
auto spw = std::make_shared<Widget>(); //引用计数（ref count，RC）为1。
//std::make_shared的信息参见条款21
std::weak_ptr<Widget> wpw(spw); //wpw指向与spw所指相同的Widget。RC仍为1
spw = nullptr; //RC变为0，Widget被销毁。
//wpw现在悬空悬空的std::weak_ptr被称作已经expired（过期）。你可以用它直接做测试：
if (wpw.expired()) //如果wpw没有指向对象…
```

![image-20230530224014125](/media/image/2023-05-28-EffectiveModernC++/pointer_dangle.png)

- 原始指针。使用这种方法，如果A被销毁，但是C继续指向B，B就会有一个指向A的悬空指针。而且B不知道指针已经悬空，所以B可能会继续访问，就会导致未定义行为。
- `std::shared_ptr`。这种设计，A和B都互相持有对方的 `std::shared_ptr`，导致的 `std::shared_ptr` 环状结构（A指向B，B指向A）阻止A和B的销毁。甚至A和B无法从其他数据结构访问了（比如，C不再指向B），每个的引用计数都还是1。如果发生了这种情况，A和B都被泄漏：程序无法访问它们，但是资源并没有被回收。

- 用 `std::weak_ptr` 替代可能会悬空的 `std::shared_ptr`。
- `std::weak_ptr` 的潜在使用场景包括：缓存、观察者列表、打破 `std::shared_ptr` 环状结构。

### 4.4. 条款二十一：优先考虑使用std::make_unique和std::make_shared而非new

- 和直接使用 `new` 相比，`make` 函数消除了代码重复，提高了异常安全性。对于 `std::make_shared` 和`std::allocate_shared`，生成的代码更小更快。 
- 不适合使用 `make` 函数的情况包括需要指定自定义删除器和希望用花括号初始化。 
- 对于 `std::shared_ptrs`，其他不建议使用 `make` 函数的情况包括(1)有自定义内存管理的类；(2)特别关注内存的系统，非常大的对象，以及 `std::weak_ptrs` 比对应的 `std::shared_ptrs` 活得更久。

### 4.5. 条款二十二：当使用Pimpl惯用法，请在实现文件中定义特殊成员函数

一个已经被声明，却还未被实现的类型，被称为未完成类型（incompletetype）。 `Widget::Impl` 就是这种类型。 你能对一个未完成类型做的事很少，但是声明一个指向它的指针是可以的。`Pimpl` 惯用法利用了这一点。

根据编译器自动生成的特殊成员函数的规则（见 Item17），编译器会自动为我们生成一个析构函数。 在这个析构函数里，编译器会插入一些代码来调用类 `Widget` 的数据成员 `pImpl` 的析构函数。 `pImpl` 是一个`std::unique_ptr<Widget::Impl>` ，也就是说，一个使用默认删除器的 `std::unique_ptr`。 默认删除器是一个函数，它使用 `delete` 来销毁内置于 `std::unique_ptr` 的原始指针。然而，在使用 `delete` 之前，通常会使默认删除器使用 `C++11` 的特性 `static_assert` 来确保原始指针指向的类型不是一个未完成类型。当编译器为 `Widget w` 的析构生成代码时，它会遇到 `static_assert` 检查并且失败，这通常是错误信息的来源。 

```cpp
// widget.h
class Widget {
public:
    Widget(const Widget& rhs); //只有声明
    Widget& operator=(const Widget& rhs);
private: //跟之前一样
 struct Impl;
    std::unique_ptr<Impl> pImpl;
};


// widget.cpp
struct Widget::Impl { … }; //跟之前一样

Widget::~Widget() = default; //其他函数，跟之前一样

Widget::Widget(const Widget& rhs) //拷贝构造函数
    : pImpl(std::make_unique<Impl>(*rhs.pImpl))
{}

Widget::~Widget() = default; //跟之前一样
Widget::Widget(Widget&& rhs) = default; //这里定义
Widget& Widget::operator=(Widget&& rhs) = default;
```

- `Pimpl` 惯用法通过减少在类实现和类使用者之间的编译依赖来减少编译时间。
- 对于 `std::unique_ptr` 类型的 `pImpl` 指针，需要在头文件的类里声明特殊的成员函数，但是在实现文件里面来实现他们。即使是编译器自动生成的代码可以工作，也要这么做。
- 以上的建议只适用于 `std::unique_ptr`，不适用于 `std::shared_ptr`。



## 5. 第5章 右值引用，移动语义，完美转发

### 5.1. 条款二十三：理解std::move和std::forward

```cpp
template<typename T>
decltype(auto) move(T&& param) //C++14，仍然在std命名空间
{
    using ReturnType = remove_referece_t<T>&&;
    return static_cast<ReturnType>(param);
}
```

- `std::move` 执行到右值的无条件的转换，但就自身而言，它不移动任何东西。
- `std::forward` 只有当它的参数被绑定到一个右值时，才将参数转换为右值。
- `std::move` 和 `std::forward` 在运行期什么也不做。

### 5.2. 条款二十四：区分通用引用与右值引用
