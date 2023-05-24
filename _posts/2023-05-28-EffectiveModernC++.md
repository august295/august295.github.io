---
layout: post
title: "EffectiveModernC++"
categories: Book
tags: Book C/C++
author: August
mathjax: true
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









