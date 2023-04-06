---
layout: post
title: "EffectiveSTL"
categories: Book
tags: Book C/C++
author: August
mathjax: true
typora-root-url: ..
---

* content
{:toc}


该文记录《Effective STL》读后记录。



# Effective STL

50 条有效使用 `STL` 的经验



## 1. 容器

### 1.1. 第1条：慎重选择容器类型

- **标准 `STL` 序列容器**：`vector`、`string`、`deque` 和 `list`。
- **标准 `STL` 关联容器**：`set`、`multiset`、`map` 和 `multimap`。
- **非标准序列容器**：`slist` 和 `rope`。`slist` 是一个单向链表，`rope` 本质上是一“重型” `string`。
- **非标准关联容器**：`hash_set`、`hash_multiset`、`hash_map` 和 `hash_multimap`。
- **`vector<char>` 作为 `string` 的替代** 。
- **`vector` 作为标准关联容器的替代** 。
- **几种标准的非 `STL` 容器**：包括数组、`bitset`、`valarray`、`stack`、`queue` 和 `priority_queue`。

使用条件

- **你是否需要在容器的任意位置插入新元素 ？**如果需要，就选择序列容器；关联容器是不行的。
- **你是否关心容器中的元素是排序的 ？**如果不关心，则散列容器是一个可行的选择方案；否则，你要避免散列容器。
- **你选择的容器必须是标准C++的一部分吗 ？**如果必须是，就排除了散列容器、`slist`和`rope`。
- **你需要哪种类型的迭代器 ？**如果它们必须是随机访问迭代器，则对容器的选择就被限定为`vector`、`deque`和`string`。或许你也可以考虑`rope`（有关rope的资料，见第50条）。如果要求使用双向迭代器，那么你必须避免`slist`（见第50条）以及散列容器的一个常见实现（见第25条）。
- **当发生元素的插入或删除操作时，避免移动容器中原来的元素是否很重要 ？**如果是，就要避免连续内存的容器（见第5条）。
- **容器中数据的布局是否需要和C兼容 ？**如果需要兼容，就只能选择`vector`（见第16条）。
- **元素的查找速度是否是关键的考虑因素 ？**如果是，就要考虑散列容器（见第25条）、排序的`vector`（见第23条）和标准关联容器——或许这就是优先顺序。
- **如果容器内部使用了引用计数技术（reference counting） ，你是否介意？**如果是，就要避免使用`string`，因为许多`string`的实现都使用了引用计数。`rope`也需要避免，因为权威的`rope`实现是基于引用计数的（见第50条）。当然，你需要某种表示字符串的方法，这时你可以考虑`vector<char>`。
- **对插入和删除操作，你需要事务语义（transactional semantics）吗 ？**也就是说，在插入和删除操作失败时，你需要回滚的能力吗？如果需要，你就要使用基于节点的容器。如果对多个元素的插入操作（即针对一个区间的形式——见第5条）需要事务语义，则你需要选择`list`，因为在标准容器中，只有`list`对多个元素的插入操作提供了事务语义。对那些希望编写异常安全（exception-safe）代码的程序员，事务语义显得尤为重要。（使用连续内存的容器也可以获得事务语义，但是要付出性能上的代价，而且代码也显得不那么直截了当。
- **你需要使迭代器、指针和引用变为无效的次数最少吗 ？**如果是这样，就要使用基于节点的容器，因为对这类容器的插入和删除操作从来不会使迭代器、指针和引用变为无效（除非它们指向了一个你正在删除的元素）。而针对连续内存容器的插入和删除操作一般会使指向该容器的迭代器、指针和引用变为无效。
- **如果序列容器的迭代器是随机访问类型，而且只要没有删除操作发生，且插入操作只发生在容器的末尾，则指向数据的指针和引用就不会变为无效，这样的容器是否对你有帮助 ？**这是非常特殊的情形，但如果你面对的情形正是如此，则`deque`是你所希望的容器。（有意思的是，当插入操作仅在容器末尾发生时，`deque`的迭代器有可能会变为无效。`deque`是唯一的、迭代器可能会变为无效而指针和引用不会变为无效的STL标准容器。）

### 1.2. 第2条：不要试图编写独立于容器类型的代码

`STL` 是以泛化（generalization）原则为基础的：数组被泛化为“以其包含的对象的类型为参数”的容器；函数被泛化为“以其使用的迭代器的类型为参数”的算法；指针被泛化为“以其指向的对象的类型为参数”的迭代器。

- 试图编写对序列容器和关联容器都适用的代码几乎是毫无意义的。很多成员函数仅当其容器为某一种类型时才存在，例如，只有序列容器才支持 `push_front` 或 `push_back`，只有关联容器才支持 `count` 和 `lower_bound`，等等。即使是 `insert` 和 `erase` 这样的基本操作，也会随容器类型的不同而表现出不同的原型和语义。
- 假设你想编写对于大多数通用的序列容器（即vector、deque和list）都适用的代码，你的程序只能使用它们的功能的交集，这意味着你不能使用 `reserve` 或 `capacity`（见第14条），因为 `deque` 和 `list` 中没有这样的操作。
- 你不能把容器中的数据传递到 `C` 接口中，因为只有 `vector` 支持这一点。

考虑到有时候不可避免地要从一种容器类型转到另一种，你可以使用常规的方式来实现这种转变：使用封装（encapsulation）技术。最简单的方式是通过对容器类型和其迭代器类型使用类型定义（typedef）。

```cpp
class CustomerList {
private:
    typedef list<Customer> CustomerContainer;
    typedef CustomerContainer::iterator CCIterator;
    
    CustomerContainer customers;
}
```

### 1.3. 第3条：确保容器中的对象副本正确而高效

当向容器中加入对象时，存入容器的是你所指定的对象的副本。当从容器中取出一个对象时，你所得到的是容器中所保存的对象的副本。进去的是副本，出来的也是副本（copy in，copy out）。这就是STL的工作方式。

利用一个对象的复制成员函数就可以很方便地复制该对象，特别是对象的复制构造函数（copy constructor）和复制赋值操作符（copy assignment operator）。如果你并没有声明这两个函数，则编译器会为你声明它们。内置类型（built-in type）（如整型、指针类型等）的实现总是简单地按位复制。

- 在存在继承关系的情况下，复制动作会导致剥离（slicing）。也就是说，如果你创建了一个存放基类对象的容器，却向其中插入派生类的对象，那么在派生类对象（通过基类的复制构造函数）被复制进容器时，它所特有的部分（即派生类中的信息）将会丢失：

```cpp
vector<Widget> vw;
class SpecialWidget : public Widget {}

SpecialWidget sw;
vw.push_back(sw) // sw 作为基类对象被复制进 vw 中，派生类特有部分在复制时丢失
```

使复制动作高效、正确，并防止剥离问题发生的一个简单办法是使容器包含指针而不是对象。

### 1.4. 第4条：调用empty而不是检查size()是否为0

你应该使用 `empty` 形式，理由很简单：`empty` 对所有的标准容器都是常数时间操作，而对一些 `list` 实现， `size` 耗费线性时间。

### 1.5. 第5条：区间成员函数优先于与之对应的单元素成员函数

给定 `v1` 和 `v2` 两个矢量（vector），使 `v1` 的内容和 `v2` 的后半部分相同的最简单操作是什么？

```cpp
v1.assign(v2.begin() + v2.size() / 2, v2.end())
```

![image-20230406001648702](/media/image/2023-04-09-EffectiveSTL/list_insert.png)

- 当通过调用 `list` 的单元素 `insert` 把一系列节点逐个加入进来时，除了最后一个新节点，其余所有的节点都要把其 `next` 指针赋值两次：一次指向 `A` ，另一次指向在它之后插入的节点。
- 区间插入只有需要插入链表头部节点和尾部节点更改指向。

了解哪些成员函数支持区间，这对于知道在何种情况下使用区间操作大有好处。在下面的函数原型中，参数类型 `iterator` 按其字面意义理解为容器的迭代器类型，即 `container::iterator`。另一方面，参数类型 `InputIterator` 表示任何类型的输入迭代器都是可接受的。

- 区间创建 。
- 区间插入 。
- 区间删除 。
- 区间赋值 。

```cpp
container::container(InputIterator begin, InputIterator end);

void container::insert(iterator position, InputIterator begin, InputIterator end);

iterator container::erase(InputIterator begin, InputIterator end);
void container::erase(InputIterator begin, InputIterator end);

void container::assign(InputIterator begin, InputIterator end);
```

使用区间成员函数而不是其相应的单元素成员函数的原因：

- 通过使用区间成员函数，通常可以少写一些代码。
- 使用区间成员函数通常会得到意图清晰和更加直接的代码。
- 更高的效率。

### 1.6. 第6条：当心C++编译器最烦人的分析机制

`C++` 中有一条普遍规律，即尽可能地解释为函数声明。

```cpp
// 下面这行代码声明了一个带double参数并返回int的函数
int f(double d);
// 下面这行也一样。参数d两边的括号是多余的，会被忽略
int f(double(d));
// 下面这行声明了同样的函数，只是它省略了参数名称
int f(double);

// 第一个声明了一个函数g，它的参数是一个指向不带任何参数的函数的指针，该函数返回double值
int g(double (*pf)());
// 有另外一种方式可表明同样的意思。唯一的区别是，pf用非指针的形式来声明（这种形式在C和C++ 中都有效）
int g(double pf());
// 下面是g的第三种声明，其中参数名pf被省略了
int g(double());

// 歧义版
list<int> data(istream_iterator<int>(dataFile), istream_iterator<int>());
```

上述声明了一个函数 `data`，其返回值是 `list<int>`。这个 `data` 函数有两个参数：

- 第一个参数的名称是 `dataFile`。它的类型是 `istream_iterator<int>`。d`ataFil`e 两边的括号是多余的，会被忽略。
- 第二个参数没有名称。它的类型是指向不带参数的函数的指针，该函数返回一个 `istream_iterator<int>`。

使用命名的迭代器对象与通常的 `STL` 程序风格相违背，但你或许觉得为了使代码对所有编译器都没有二义性，并且使维护代码的人理解起来更容易，这一代价是值得的。

```cpp
// 显示声明版
ifstream dataFile("ints.dat");
istream_iterator<int> dataBegin(dataFile);
istream_iterator<int> dataEnd;
list<int> data(dataBegin, dataEnd);
```

### 1.7. 第7条：如果容器中包含了通过new操作创建的指针，切记在容器对象析构前将指针delete掉

`STL` 容器很智能，但没有智能到知道是否该删除自己所包含的指针的程度。当你使用指针的容器，而其中的指针应该被删除时，为了避免资源泄漏，你必须或者用引用计数形式的智能指针对象（比如Boost的shared_ptr）代替指针，或者当容器被析构时手工删除其中的每个指针。

### 1.8. 第8条：切勿创建包含auto_ptr的容器对象

`auto_ptr` 的容器（简称COAP）是被禁止的。千万别创建包含 `auto_ptr` 的容器，即使你的 `STL` 平台允许你这样做。

当你复制一个 `auto_ptr` 时，它所指向的对象的所有权被移交到拷入的 `auto_ptr` 上，而它自身被置为 `NULL`。

```cpp
class Widget {};
auto_ptr<Widget> pw1(new Widget); // pw1 指向一个 Widget
auto_ptr<Widget> pw2(pw1);        // pw2 指向 pw1 的 Widget: pw1 被置为 NULL
                                  // Widget 的所有权从 pw1 转移到 pw2 上
pw1 = pw2;                        // pw1 指向 Widget: pw2 被置为 NULL
```

### 1.9. 第9条：慎重选择删除元素的方法

- 要删除容器中有特定值的所有对象：如果容器是 `vector`、`string` 或 `deque`，则使用 `erase-remove` 习惯用法。
- 如果容器是 `list`，则使用 `list::remove`。
- 如果容器是一个标准关联容器，则使用它的 `erase` 成员函数。
- 要删除容器中满足特定判别式（条件）的所有对象：
  - 如果容器是 `vector`、`string` 或 `deque`，则使用` erase-remove_if` 习惯用法。
  - 如果容器是 `list`，则使用 `list::remove_if`。
  - 如果容器是一个标准关联容器，则使用 `remove_copy_if` 和 `swap`，或者写一个循环来遍历容器中的元素，记住当把迭代器传给 `erase` 时，要对它进行后缀递增。
- 要在循环内部做某些（除了删除对象之外的）操作：
  - 如果容器是一个标准序列容器，则写一个循环来遍历容器中的元素，记住每次调用 `erase` 时，要用它的返回值更新迭代器。
  - 如果容器是一个标准关联容器，则写一个循环来遍历容器中的元素，记住当把迭代器传给 `erase` 时，要对迭代器做后缀递增

```cpp
vector<int> vec;
// erase-remove 模式
vec.erase(remove(vec.begin(), vec.end(), 1963), vec.end);

// 序列容器遍历删除
for (auto iter = vec.begin(); iter != vec.end(); /*什么也不做*/)
{
    if (*iter % 2 == 0)
    {
        vec.erase(iter++); // 后缀++使迭代器已近移动，删除返回副本
    }
    else
    {
        ++iter;
    }
}

map<int, int> m;
// 关联容器遍历删除
for (auto iter = m.begin(); iter != m.end(); /*什么也不做*/)
{
    if ((*iter).first % 2 == 0)
    {
        iter = m.erase(iter); // 关联容器删除会返回随被删除元素的下一个元素的有效迭代器
    }
    else
    {
        ++iter;
    }
}
```

### 1.10. 了解分配子（allocator）的约定和限制

在 `C++` 标准中，一个类型为 `T` 的对象，它的默认分配子（称为`allocator<T>`）提供了两个类型定义，分别为 `allocator<T>::pointer` 和 `allocator<T>::reference`，用户定义的分配子也应该提供这些类型定义。

每个分配子模板 `A`（如std::allocator、SpecialAllocator等）都要有一个被称为 `rebind` 的嵌套结构模板。`rebind` 带有唯一的类型参数 `U`，并且只定义了一个类型定义 `other`。`other` 仅仅是 `A<U>` 的名字。结果，通过引用 `Allocator::rebind<ListNode>::other`, `list<T>` 就能从T对象的分配子（称为Allocator）得到相应的 `ListNode` 对象的分配子。

```cpp
template <typename _Tp1>
struct rebind
{
    typedef allocator<_Tp1> other;
};
```

- 你的分配子是一个模板，模板参数 `T` 代表你为它分配内存的对象的类型。
- 提供类型定义 `pointer` 和 r`eference`，但是始终让 `pointer` 为 `T*` ，`reference` 为 `T&`。
- 千万别让你的分配子拥有随对象而不同的状态（per-objectstate）。通常，分配子不应该有非静态的数据成员。
- 记住，传给分配子的 `allocate` 成员函数的是那些要求内存的对象的个数，而不是所需的字节数。同时要记住，这些函数返回 `T*` 指针（通过pointer类型定义），即使尚未有 `T` 对象被构造出来。
- 一定要提供嵌套的 `rebind` 模板，因为标准容器依赖该模板。

### 1.11. 第11条：理解自定义分配子的合理用法

### 1.12. 第12条：切勿对STL容器的线程安全性有不切实际的依赖

对一个STL实现你最多只能期望：

- **多个线程读是安全的** 。多个线程可以同时读同一个容器的内容，并且保证是正确的。自然地，在读的过程中，不能对容器有任何写入操作。
- **多个线程对不同的容器做写入操作是安全的** 。多个线程可以同时对不同的容器做写入操作

考虑当一个库试图实现完全的容器线程安全性时可能采取的方式：

- 对容器成员函数的每次调用，都锁住容器直到调用结束。
- 在容器所返回的每个迭代器的生存期结束前，都锁住容器（比如通过 begin 或 end 调用）。
- 对于作用于容器的每个算法，都锁住该容器，直到算法结束。（实际上这样做没有意义。因为，如同在第32条中解释的，算法无法知道它们所操作的容器。尽管如此，在这里我们仍要讨论这一选择。因为即便这是可能的，我们也会发现这种做法仍不能实现线程安全性，这对于我们的讨论是有益的。）

```cpp
vector<int> vec;
mutex       mut;
{
    lock_guard<mutex>     lock(mut); // 获取互斥体
    vector<int>::iterator first5(find(vec.begin(), vec.end(), 5));
    if (first5 != vec.end())
    {
        *first5 = 0;
    }
} // 代码块结束，自动释放
```



## 2. vector和string



