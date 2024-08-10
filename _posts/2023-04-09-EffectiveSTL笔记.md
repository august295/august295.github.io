---
layout: post
title: "EffectiveSTL笔记"
categories: Book
tags: Book
author: August
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

```cpp
vector<int> vec;
// erase-remove 模式
vec.erase(remove(vec.begin(), vec.end(), 1963), vec.end);
```

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
vector<int> vec{1, 2, 3, 4, 5, 6};
// 序列容器遍历删除
for (auto iter = vec.begin(); iter != vec.end(); /*什么也不做*/)
{
    if (*iter % 2 == 0)
    {
        iter = vec.erase(iter); // 迭代器失效，记录序列容器删除会返回随被删除元素的下一个元素的有效迭代器
    }
    else
    {
        ++iter;
    }
}
// 错误版本
//for (auto iter = vec.begin(); iter != vec.end(); /*什么也不做*/)
//{
//    if (*iter % 2 == 0)
//    {
//        vec.erase(iter++); // 迭代器失效
//    }
//    else
//    {
//        ++iter;
//    }
//}

map<int, int> m{ {1, 1}, {2, 2}, {3, 3}, {4, 4} };
// 关联容器遍历删除
for (auto iter = m.begin(); iter != m.end(); /*什么也不做*/)
{
    if ((*iter).first % 2 == 0)
    {
        m.erase(iter++); // 后缀++使迭代器已近移动，删除返回副本
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

### 2.1. 第13条：vector和string优先于动态分配的数组

当你决定用 `new` 来动态分配内存时，这意味着你将承担以下责任：

- 你必须确保以后会有人用 `delete` 来删除所分配的内存。如果没有随后的 `delete`，那么你的new将会导致一个资源泄漏。
- 你必须确保使用了正确的 `delete` 形式。如果分配了单个对象，则必须使用 `delete`；如果分配了数组，则需要用 `delete[]`。如果使用了不正确的 `delete` 形式，那么结果将是不确定的。在有些平台上，程序会在运行时崩溃。在其他平台上，它会妨碍进一步运行，有时会泄漏资源和破坏内存。
- 你必须确保只 `delete` 了一次。如果一次分配被多次 `delete`，结果同样是不确定的。

`vector` 和 `string` 它们自己管理内存。当元素被加入到容器中时，它们的内存会增长；而当 `vector` 或 `string` 被析构时，它们的析构函数会自动析构容器中的元素并释放包含这些元素（只是对象不能自动释放指针）的内存。

`vector` 和 `string` 是功能完全的 `STL` 序列容器，所以，凡是适合于序列容器的 `STL` 算法，你都可以使用。

### 2.2. 第14条：使用reserve来避免不必要的重新分配

对于 `vector` 和 `string`，增长过程是这样来实现的：每当需要更多空间时，就调用与 `realloc` 类似的操作。这一类似于 `realloc` 的操作分为以下4部分：

- 分配一块大小为当前容量的某个倍数的新内存。在大多数实现中，`vector` 和 `string` 的容量每次以2的倍数增长，每当容器需要扩张时，它们的容量就加倍。
- 把容器的所有元素从旧的内存复制到新的内存中。
- 析构掉旧内存中的对象。
- 释放旧内存。

在标准容器中，只有 `vector` 和 `string` 提供了所有这4个函数。

- `size()` 告诉你该容器中有多少个元素。
- `capacity()` 告诉你该容器利用已经分配的内存可以容纳多少个元素。
- `resize(Container::size_type n)` 强迫容器改变到包含 `n` 个元素的状态。
- `reserve(Container::size_type n)` 强迫容器把它的容量变为至少是 `n`，前提是 `n` 不小于当前的大小。

若能确切知道或大致预计容器中最终会有多少元素，则此时可使用 `reserve`。

```cpp
vector<int> vec;
vec.reserve(1000);	// 提前分配，避免多次扩展
for (size_t i = 0; i < 1000; i++)
{
    vec.push_back(i);
}
```

### 2.3. 第15条：注意string实现的多样性

- `string` 的值可能会被引用计数，也可能不会。很多实现在默认情况下会使用引用计数，但它们通常提供了关闭默认选择的方法，往往是通过预处理宏来做到这一点。第13条给出了你想将其关闭的一种特殊情况，但其他的原因也可能会让你这样做。比如，只有当字符串被频繁复制时，引用计数才有用，而有些应用并不经常复制内存，这就不值得使用引用计数了。
- `string` 对象大小的范围可以是一个 `char*` 指针的大小的1倍到7倍。
- 创建一个新的字符串值可能需要零次、一次或两次动态分配内存。
- `string` 对象可能共享，也可能不共享其大小和容量信息。
- `string` 可能支持，也可能不支持针对单个对象的分配子。
- 不同的实现对字符内存的最小分配单位有不同的策略。

### 2.4. 第16条：了解如何把vector和string数据传给旧的API

如果你有一个 `vector v`，而你需要得到一个指向 `v` 中数据的指针，从而可把 `v` 中的数据作为数组来对待，那么只需使用 `&v[0]` 就可以了。

```cpp
void doSomething(const int* pInts, size_t numInts);

vector<int> vec;
if (!vec.empty())
{
    doSomething(&vec[0], vec.size());
}

// 如果使用 begin() 但是不推荐
if (!vec.empty())
{
    doSomething(&*vec.begin(), vec.size());
}
```

对于 `string s`，对应的形式是 `s.c_str()`。

- `string` 中的数据不一定存储在连续的内存中；
- `string` 的内部表示不一定是以空字符结尾的。

```cpp
void doSomething(const char* pString);

string str;
doSomething(str.c_str());
```

除了 `vector` 和 `string` 以外的其他 `STL` 容器也能把它们的数据传递给 `C API`。你只需把每个容器的元素复制到一个 `vector` 中，然后传给该 `API`。

```cpp
size_t fillArray(double* pArray, size_t arraySize);

vector<double> vd(100);
vd.resize(fillArray(&vd[0], vd.size()));
deque<double> d(vd.begin(), vd.end());
list<double>  l(vd.begin(), vd.end());
set<double>   s(vd.begin(), vd.end());
```

### 2.5. 第17条：使用“swap技巧”除去多余的容量

为了避免矢量仍占用不再需要的内存，你希望有一种方法能把它的容量从以前的最大值缩减到当前需要的数量。这种对容量的缩减通常被称为 `shrink to fit`（压缩至适当大小）。

```cpp
vector<int> v{1, 2};
v.clear();             // erase 和 clear 只会清除元素，不会减小容量
vector<int>().swap(v); // swap 技巧可以压缩至适当大小

string s{"hello world hello world"};
s.clear();
string().swap(s); // 清除 s 并把它的容量变为最小，string 不是 0
```

在做 `swap` 的时候，不仅两个容器的内容被交换，同时它们的迭代器、指针和引用也将被交换（string除外）。在 `swap` 发生后，原先指向某容器中元素的迭代器、指针和引用依然有效，并指向同样的元素——但是，这些元素已经在另一个容器中了。

### 2.6. 第18条：避免使用`vector<bool>`

作为一个 `STL` 容器，`vector<bool>` 只有两点不对。

- 它不是一个 `STL` 容器。
- 它并不存储 `bool`。

`vector<bool>` 是一个假的容器，它并不真的储存 `bool`，在一个典型的实现中，储存在 `vector` 中的每个 `bool` 仅占一个二进制位，一个8位的字节可容纳8个 `bool`。在内部，`vector<bool>` 使用了与位域（bitfield）一样的思想，来表示它所存储的那些 `bool`；实际上它只是假装存储了这些 `bool`。

标准库提供了两种选择。

- 第一种是 `deque<bool>`。`deque` 中元素的内存不是连续的，所以你不能把 `deque<bool>` 中的数据传递给一个期望 `bool` 数组的 `C API`。
- 第二种是 `bitset`。`bitset` 不是 `STL` 容器，但它是标准 `C++` 库的一部分。与 `STL` 容器不同的是，它的大小（即元素的个数）在编译时就确定了，所以它不支持插入和删除元素。



## 3. 关联容器

### 3.1. 第19条：理解相等（equality）和等价（equivalence）的区别

`find` 对“相同”的定义是相等，是以 `operator==` 为基础的。`set::insert` 对“相同”的定义是等价，是以 `operator<` 为基础的。

### 3.2. 第20条：为包含指针的关联容器指定比较类型

`ssp` 会按顺序保存它的内容，但因为它包含的是指针，所以会按照指针的值进行排序，而不是按字符串的值。

```cpp
#include <functional>
#include <iostream>
#include <set>
#include <string>

using namespace std;

// 自定义指针比较
struct DereferenceLess
{
    template <typename PtrType>
    bool operator()(PtrType ps1, PtrType ps2)
    {
        return *ps1 < *ps2;
    }
};

int main()
{
    // 1. 无序输出
    set<string*> ssp;
    ssp.insert(new string("Anteater"));
    ssp.insert(new string("Wombat"));
    ssp.insert(new string("Lemur"));
    ssp.insert(new string("Penguin"));
    for (auto i = ssp.begin(); i != ssp.end(); ++i)
    {
        cout << **i << endl;
    }

    // 2. 自定义指针比较
    struct StringPtrLess : public binary_function<const string*, const string*, bool>
    {
        bool operator()(const string* ps1, const string* ps2) { return *ps1 < *ps2; }
    };

    set<string*, StringPtrLess> ssp2;
    ssp2.insert(new string("Anteater"));
    ssp2.insert(new string("Wombat"));
    ssp2.insert(new string("Lemur"));
    ssp2.insert(new string("Penguin"));
    for (auto i = ssp2.begin(); i != ssp2.end(); ++i)
    {
        cout << **i << endl;
    }

    // 3. 自定义指针比较 模板
    set<string*, DereferenceLess> ssp3;
    ssp3.insert(new string("Anteater"));
    ssp3.insert(new string("Wombat"));
    ssp3.insert(new string("Lemur"));
    ssp3.insert(new string("Penguin"));
    for (auto i = ssp3.begin(); i != ssp3.end(); ++i)
    {
        cout << **i << endl;
    }

    return 0;
}
```

### 3.3. 第21条：总是让比较函数在等值情况下返回false

比较函数的返回值表明的是按照该函数定义的排列顺序，一个值是否在另一个之前。相等的值从来不会有前后顺序关系，对于相等的值，比较函数应当始终返回 `false`。

```cpp
set<int, less_equal<int>> s;
s.insert(10);
s.insert(10);	// less_equal 导致未定义行为，程序可能错误
```

### 3.4. 第22条：切勿直接修改set或multiset中的键

对于 `map` 和 `multimap` 尤其简单，因为如果有程序试图改变这些容器中的键，它将不能通过编译，对于一个 `map<K,V>` 或 `multimap<K,V>` 类型的对象，其中的元素类型是 `pair<const K,V>`。（如果利用 `const_cast`，你或许可以修改它，后面我们将会看到。不管你是否相信，有时你可能希望这样做。）

```cpp
map<int, string> m;
m.emplace(1, "1");
m.begin()->first = 10; // 错误，map 主键不能被修改

multimap<int, string> mm;
mm.emplace(1, "1");
mm.begin()->first = 20; // 错误，multimap 主键不能被修改
```

实际上，雇员的 `ID` 号是这个 `set` 中的元素的键（key），其他的雇员数据只不过跟这个键绑在一起而已。我们就没有理由不可以把个别雇员的头衔改为某个有特定含义的值，因为我们在这里所做的是修改雇员中与集合的排序方式无关的部分（雇员记录中不属于键的部分），所以这段代码不会破坏该集合。

为了能够修改其他部分，我们必须把 `*i` 的常量性质（constness）转换掉。

```cpp
#include <functional>
#include <iostream>
#include <set>
#include <string>

using namespace std;

class Employee
{
public:
    Employee(int number) : m_number(number) {}

    const string& name() const {};                                    // 获取雇员名字
    void          setName(const string& name){};                      // 设置雇员名字
    const string& title() const {};                                   // 获取雇员头衔
    void          setTitle(const string& title) { m_title = title; }; // 设置雇员头衔
    int           idNumber() const { return m_number; };              // 获取雇员ID

private:
    int    m_number;
    string m_title;
};

struct IDNumberLess : public binary_function<Employee, Employee, bool>
{
    bool operator()(const Employee& lhs, const Employee& rhs) const { return lhs.idNumber() < rhs.idNumber(); }
};

int main()
{
    typedef set<Employee, IDNumberLess> EmplSet;

    EmplSet  se; // se 是按照 ID 号进行排序
    Employee selectID(1);
    se.insert(selectID);
    auto i = se.find(selectID);
    if (i != se.end())
    {
        // 设置雇员新头衔
        // VS2017 中不合法，因为 *i 为 const
        const_cast<Employee&>(*i).setTitle("New Title 1");
        // 未能更改，因为这样强转 const 会产生临时对象
        static_cast<Employee>(*i).setTitle("New Title 2");
    }

    return 0;
}
```

如果你想以一种总是可行而且安全的方式来修改 `set`、`multiset`、`map` 和 `multimap` 中的元素，则可以分5个简单步骤来进行：

- 找到你想修改的容器的元素。如果你不能肯定最好的做法，第45条介绍了如何执行一次恰当的搜索来找到特定的元素。
- 为将要被修改的元素做一份副本。在 `map` 或 `multimap` 的情况下，不要把该副本的第一个部分声明为 `const`。
- 修改该副本，使它具有你期望它在容器中的值。
- 把该元素从容器中删除，通常是通过调用 `erase` 来进行的（见第9条）。
- 把新的值插入到容器中。如果按照容器的排列顺序，新元素的位置可能与被删除元素的位置相同或紧邻，则使用“提示”（hint）形式的 `insert`，以便把插入的效率从对数时间提高到常数时间。把你从第1步得来的迭代器作为提示信息。

```cpp
#include <functional>
#include <iostream>
#include <set>
#include <string>

using namespace std;

class Employee
{
public:
    Employee(int number) : m_number(number) {}

    const string& name() const {};                                    // 获取雇员名字
    void          setName(const string& name){};                      // 设置雇员名字
    const string& title() const {};                                   // 获取雇员头衔
    void          setTitle(const string& title) { m_title = title; }; // 设置雇员头衔
    int           idNumber() const { return m_number; };              // 获取雇员ID

private:
    int    m_number;
    string m_title;
};

struct IDNumberLess : public binary_function<Employee, Employee, bool>
{
    bool operator()(const Employee& lhs, const Employee& rhs) const { return lhs.idNumber() < rhs.idNumber(); }
};

int main()
{
    typedef set<Employee, IDNumberLess> EmplSet;

    EmplSet  se; // se 是按照 ID 号进行排序
    Employee selectID(1);
    Employee selectID2(2);
    se.insert(selectID);
    se.insert(selectID2);
    auto i = se.find(selectID); // 1. 找到待修改元素
    if (i != se.end())
    {
        Employee e(*i);            // 2. 复制该元素
        e.setTitle("New Title 1"); // 3. 修改副本
        se.erase(i++);             // 4. 删除该元素，后缀递增迭代器以保持其有效性
        se.insert(i, e);           // 5. 插入新元素和原来位置相同
    }

    return 0;
}
```

### 3.5. 第23条：考虑用排序的vector替代关联容器

很多应用程序使用其数据结构的方式并不这么混乱。它们使用其数据结构的过程可以明显地分为3个阶段，总结如下。

- 设置阶段。创建一个新的数据结构，并插入大量元素。在这个阶段，几乎所有的操作都是插入和删除操作。很少或几乎没有查找操作。
- 查找阶段。查询该数据结构以找到特定的信息。在这个阶段，几乎所有的操作都是查找操作，很少或几乎没有插入和删除操作。
- 重组阶段。改变该数据结构的内容，或许是删除所有的当前数据，再插入新的数据。在行为上，这个阶段与第（1）阶段类似。当这个阶段结束以后，应用程序又回到第（2）段。

只有对排序的容器才能够正确地使用查找算法 `binary_search` 、 `lower_bound` 和 `equal_range` 等。

为什么通过（排序的）`vector` 执行的二分搜索，比通过二叉查找树执行的二分搜索具有更好的性能呢？

- 大小的因素。如果选择了关联容器，则我们几乎肯定在使用平衡二叉树。这样的树是由树节点构成的，每个节点不仅包含了一个 `Widget`，而且还包含了几个指针：一个指针指向该节点的左儿子，一个指针指向该节点的右儿子，（通常）还有一个指针指向它的父节点。这意味着在一个关联容器中存储一个 `Widget` 所伴随的空间开销至少是 3 个指针。
- 引用的局域性。假定我们的数据结构足够大，它们被分割后将跨越多个内存页面，但 `vector` 将比关联容器需要更少的页面。如果你的 `STL` 实现没有采取措施来提高这些树节点的引用局域性，那么，这些节点将会散布在你的全部地址空间中。这将会导致更多的页面错误。

### 3.6. 第24条：当效率至关重要时，请在map::operator[]与map::insert之间谨慎做出选择。

`map::operator[]` 的设计目的是为了提供“添加和更新”（add orupdate）的功能。首先会检查键 `k` 是否已经在 `map` 中了。如果没有，它就被加入，并以v作为相应的值。如果 `k` 已经在映射表中了，则与之关联的值被更新为 `v`。但如果 `k` 还没有在映射表中，那就没有 `operator[]` 可以指向的值对象。在这种情况下，它使用值类型的默认构造函数创建一个新的对象，然后 `operator[]` 就能返回一个指向该新对象的引用了。

当效率至关重要时，你应该在 `map::operator[]` 和 `map::insert` 之间仔细做出选择。如果要更新一个已有的映射表元素，则应该优先选择 `operator[]`；但如果要添加一个新的元素，那么最好还是选择 `insert`。

### 3.7. 第25条：熟悉非标准的散列容器



## 4. 迭代器

`STL` 标准容器实际上提供了4种不同的迭代器类型：`iterator` 、 `const_iterator` 、 `reverse_iterator` 和 `const_reverse_iterator`

### 4.1. 第26条：iterator优先于const_iterator、reverse_iterator以及const_reverse_iterator

![image-20230413235923185](/media/image/2023-04-09-EffectiveSTL/iterator_relation.png)



我们有足够的信息来理解为什么应该尽可能使用 `iterator` ，而避免使用 `const` 或者 `reverse` 型的迭代器：

- 有些版本的 `insert` 和 `erase` 函数要求使用 `iterator`。如果你需要调用这些函数，那你就必须使用 `iterator`。`const` 和 `reverse` 型的迭代器不能满足这些函数的要求。
- 要想隐式地将一个 `const_iterator` 转换成 `iterator` 是不可能的，第27条中讨论的将 `const_iterator` 转换成 `iterator` 的技术并不普遍适用，而且效率也不能保证。
- 从 `reverse_iterator` 转换而来的 `iterator` 在使用之前可能需要相应的调整，第28条讨论了为什么需要调整以及何时进行调整。

### 4.2. 第27条：使用distance和advance将容器的const_iterator转换成iterator

要想让 `distance` 调用顺利地通过编译，你需要排除这里的二义性。最简单的办法是显式地指明 `distance` 所使用的类型参数，从而避免让编译器来推断该类型参数。

```cpp
vector<int> intVec{ 1,2,3 };
vector<int>::const_iterator ci = intVec.cend() - 1;
vector<int>::iterator i = intVec.begin();
advance(i, distance<vector<int>::const_iterator>(i, ci));
```

它的效率取决于你所使用的迭代器。对于随机访问的迭代器（如vector、string和deque产生的迭代器）而言，它是一个常数时间的操作；对于双向迭代器（所有其他标准容器的迭代器，以及某些散列容器实现（见第25条）的迭代器）而言，它是一个线性时间的操作。

### 4.3. 第28条：正确理解由reverse_iterator的base()成员函数所产生的iterator的用法

```cpp
vector<int> intVec{ 1,2,3,4,5 };
vector<int>::reverse_iterator ri = find(intVec.rbegin(), intVec.rend(), 3);
vector<int>::iterator i(ri.base());
```

![image-20230417232650024](/media/image/2023-04-09-EffectiveSTL/reverse_iterator.png)

在 `reverse_iterator` 与对应的由 `base()` 产生的 `iterator` 之间存在偏移，这段偏移也正好勾画出了`rbegin()` 和 `rend()` 与对应的 begin() 和 `end()` 之间的偏移。

- 如果要在一个 `reverse_iterator ri` 指定的位置上插入新元素，则只需在 `ri.base()` 位置处插入元素即可。对于插入操作而言，`ri` 和 `ri.base()` 是等价的，`ri.base()` 是真正与 `ri` 对应的 `iterator`。
- 如果要在一个 `reverse_iterator ri` 指定的位置上删除一个元素，则需要在 `ri.base()` 前面的位置上执行删除操作。对于删除操作而言，`ri` 和 `ri.base()` 是不等价的，`ri.base()` 不是与 `ri` 对应的 `iterator`。

```cpp
vector<int>                   intVec{1, 2, 3, 4, 5};
vector<int>::reverse_iterator ri = find(intVec.rbegin(), intVec.rend(), 3);
vector<int>::iterator         i(ri.base());
// 因为插入实在前方插入，所以转换后就是在 3 后面插入
intVec.insert(ri.base(), 99);
// 这样就能正确删除 ri 指向元素 3
intVec.erase((++ri).base());
```

### 4.4. 第29条：对于逐个字符的输入请考虑使用istreambuf_iterator

因为 `istream_iterator` 使用 `operator>>` 函数来完成实际的读操作，而默认情况下 `operator>>` 函数会跳过空白字符。假定你希望保留空白字符，那么所需要做的工作是改写这种默认行为，只要清除输入流的 `skipws` 标志即可：

```cpp
ifstream inputFile("../../test/read.txt");
string fileData((istream_iterator<char>(inputFile)), istream_iterator<char>());

ifstream inputFile2("../../test/read.txt");
// 禁止忽略空格
inputFile2.unsetf(ios::skipws);
string fileData2((istream_iterator<char>(inputFile2)), istream_iterator<char>());

// 同样能完全读取并效率更高
ifstream inputFile3("../../test/read.txt");
string   fileData3((istreambuf_iterator<char>(inputFile3)), istreambuf_iterator<char>());
```

如果你需要从一个输入流中逐个读取字符，那么就不必使用格式化输入；如果你关心的是读取流的时间开销，那么使用 `istreambuf_iterator` 取代 `istream_iterator` 只是多输入了 3 个字符，却可以获得明显的性能改善。对于非格式化的逐个字符输入过程，你总是应该考虑使用 `istreambuf_iterator`。

同样地，对于非格式化的逐个字符输出过程，你也应该考虑使用 `ostreambuf_iterator`。它可以避免因使用 `ostream_iterator` 而带来的额外负担（但同时也损失了格式化输出的灵活性），从而具有更为优越的性能。



## 5. 算法

### 5.1. 第30条：确保目标区间足够大

需要牢记的是：无论何时，如果所使用的算法需要指定一个目标区间，那么必须确保目标区间足够大，或者确保它会随着算法的运行而增大。要在算法执行过程中增大目标区间，请使用插入型迭代器，比如 `ostream_iterator`，或者由`back_inserter`、`front_inserter`和 `inserter` 返回的迭代器。

### 5.2. 第31条：了解各种与排序有关的选择

- 如果需要对 `vector`、`string`、`deque` 或者数组中的元素执行一次完全排序，那么可以使用 `sort` 或者 `stable_sort`。
- 如果有一个 `vector`、`string`、`deque` 或者数组，并且只需要对等价性最前面的 `n` 个元素进行排序，那么可以使用 `partial_sort`。
- 如果有一个 `vector`、`string`、`deque` 或者数组，并且需要找到第 `n` 个位置上的元素，需要找到等价性最前面的 `n` 个元素但又不必对这 `n` 个元素进行排序，`nth_element` 正是你所需要的函数。
- 如果需要将一个标准序列容器中的元素按照是否满足某个特定的条件区分开来，`partition` 和 `stable_partition` 可能正是你所需要的。
- 如果你的数据在一个 `list` 中，那么你仍然可以直接调用 `partition` 和 `stable_partition` 算法；你可以用 `list::sort` 来替代 `sort` 和 `stable_sort` 算法。 如果你需要获得 `partial_sort` 或 `nth_element` 算法的效果，正如前面我所提到的那样，你可以有一些间接的途径来完成这项任务。

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <iterator>
#include <vector>

using namespace std;

int main()
{
    vector<int> intVec{6, 7, 1, 2, 3, 4, 5, 8, 9};
    // 默认将最小元素顺序排列到前 3 个
    partial_sort(intVec.begin(), intVec.begin() + 3, intVec.end());

    vector<int> intVec2{6, 7, 1, 2, 3, 4, 5, 8, 9};
    // 默认将最大元素顺序排列到前 3 个
    partial_sort(intVec2.begin(), intVec2.begin() + 3, intVec2.end(), [](int a, int b) { return a > b; });

    vector<int> intVec3{6, 7, 1, 2, 3, 4, 5, 8, 9};
    // 默认将最小元素排列到前 3 个，不一定考虑顺序
    nth_element(intVec3.begin(), intVec3.begin() + 3, intVec3.end());

    vector<int> intVec4{6, 7, 1, 2, 3, 4, 5, 8, 9};
    // 所有满足条件元素移到前部，返回第一个不满住条件的迭代器
	vector<int>::iterator goodEnd = partition(intVec4.begin(), intVec4.end(), [](int a) { return a > 7; });

    return 0;
}
```

### 5.3. 第32条：如果确实需要删除元素，则需要在remove这一类算法之后调用erase

`remove` 不是真正意义上的删除，因为它做不到。（list::remove会真正删除元素）

在内部，`remove` 遍历整个区间，用需要保留的元素的值覆盖掉那些要被删除的元素的值。这种覆盖是通过对那些需要被覆盖的元素的赋值来完成的。

![image-20230424221210405](/media/image/2023-04-09-EffectiveSTL/remove.png)

```cpp
vector<int> intVec{1, 2, 3, 99, 5, 99, 7, 8, 9, 99};
// 不能真正删除，是用需要保留的元素的值覆盖掉那些要被删除的元素的值。
// 结果是：1 2 3 5 7 8 9 8 9 99
//remove(intVec.begin(), intVec.end(), 99);
// 只有容器函数才能删除
// 尾部迭代器改变，通过指针依旧能获取上述数据
intVec.erase(remove(intVec.begin(), intVec.end(), 99), intVec.end());
int value = *(&intVec[0] + 9);
```

### 5.4. 第33条：对包含指针的容器使用remove这一类算法时要特别小心

删除容器中的指针并不能删除该指针所指的对象。

![image-20230425230910570](/media/image/2023-04-09-EffectiveSTL/remove_if.png)

如果无法避免对这种容器使用 `remove`，那么一种可以消除该问题的做法是，在进行 `erase-remove` 习惯用法之前，先把那些指向未被验证过的 `Widget` 的指针删除并置成空，然后清除该容器中所有的空指针。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

class Widget
{
public:
    Widget(){};
    Widget(double weight) : m_weight(weight){};
    Widget& operator=(double weight) { m_weight = weight; };

public:
    double m_weight;
};

// 删除条件
void delAndNullifyUncertified(Widget*& pWidget) {
	if (pWidget->m_weight > 10) {
		delete pWidget;
		pWidget = nullptr;
	}
}

int main()
{
	vector<Widget*> v;
	// 遍历指针对象并删除置空
	for_each(v.begin(), v.end(), delAndNullifyUncertified);
	// 删除空指针
	v.erase(remove(v.begin(), v.end(), nullptr), v.end());

    return 0;
}
```

### 5.5. 第34条：了解哪些算法要求使用排序的区间作为参数

### 5.6. 第35条：通过mismatch或lexicographical_compare实现简单的忽略大小写的字符串比较

### 5.7. 第36条：理解copy_if算法的正确实现

`C++11` 已实现 `copy_if`

```cpp
vector<int> v1{1, 2, 3, 4, 5, 6};
vector<int> v2;
// 拷贝大于 3 的值
auto iter = copy_if(v1.begin(), v1.end(), back_inserter(v2), [](int i) { return i > 3; });
```

### 5.8. 第37条：使用accumulate或者for_each进行区间统计

```cpp
vector<int> v1{1, 2, 3, 4, 5, 6};
// 求和
auto num           = accumulate(v1.begin(), v1.end(), 0);
// 自定义求和函数
auto str           = accumulate(next(v1.begin()),
                                v1.end(),
                                to_string(v1[0]),
                                [](string a, int b) -> string { return std::move(a) + '-' + std::to_string(b); });
// 区间统计，大于2小于5
int num_accumulate = accumulate(v1.begin(), v1.end(), 0, [](int a, int b) { return (b > 2 && b < 5) ? ++a : a; });

// 求和
int sum = 0;
for_each(v1.begin(), v1.end(), [&sum](int a) { sum += a; });
// 区间统计，大于1小于5
int num_for_each = 0;
for_each(v1.begin(), v1.end(), [&num_for_each](int a) { (a > 1 && a < 5) ? num_for_each++ : num_for_each; });
```



## 6. 第6章 函数子、函数子类、函数及其他

### 6.1. 第38条：遵循按值传递的原则来设计函数子类

### 6.2. 第39条：确保判别式是“纯函数”

- 一个判别式（predicate）是一个返回值为 `bool` 类型（或者可以隐式地转换为bool类型）的函数。
- 一个纯函数（pure function）是指返回值仅仅依赖于其参数的函数。
- 判别式类（predicate class）是一个函数子类，它的 `operator()` 函数是一个判别式，也就说是，它的 `operator()` 返回 `true` 或者 `false`。

### 6.3. 第40条：若一个类是函数子，则应使它可配接

### 6.4. 第41条：理解ptr_fun、mem_fun和mem_fun_ref的来由

如果有一个函数f和一个对象x，现在希望在x上调用f，而我们在x的成员函数之外，那么为了执行这个调用，C++提供了3种不同的语法

```cpp
f();		// 语法一：f 是一个非成员函数
x.f();		// 语法二：f 是成员函数，并且 x 是一个对象或对象的引用
p->f();		// 语法三：f 是成员函数，并且 p 是一个指向对象 x 的指针
```

### 6.5. 第42条：确保 `less<T>` 与 `operator<` 具有相同的语义



## 7. 第7章 在程序中使用STL

### 7.1. 第43条：算法调用优先于手写的循环

- 效率。算法通常比程序员自己写的循环效率更高。
- 正确性。自己写循环比使用算法更容易出错。
- 可维护性。使用算法的代码通常比手写循环的代码更加简洁明了。

### 7.2. 第44条：容器的成员函数优先于同名的算法

- 成员函数往往速度快；
- 成员函数通常与容器（特别是关联容器）结合得更加紧密，这是算法所不能比的。

### 7.3. 第45条：正确区分count、find、binary_search、lower_bound、upper_bound和equal_range

| 前提                                   | 使用算法   |                       | 使用成员函数 |                   |
| -------------------------------------- | ---------- | --------------------- | ------------ | ----------------- |
|                                        | 未排序区间 | 排序区间              | set map      | multiset multimap |
| 特定值存在吗？                         | find       | binary_search         | count        | find              |
| 特定值存在吗？存在时第一个值对象在哪里 | find       | equal_range           | find         | find, lower_bound |
| 第一个不超过特定值对象在哪里           | find_if    | lower_bound           | lower_bound  | lower_bound       |
| 第一个特定值之后的对象在哪里           | find_if    | upper_bound           | upper_bound  | upper_bound       |
| 具有多少个特定值对象                   | count      | equal_range(distance) | count        | count             |
| 具有特定之的对象在哪里                 | find       | equal_range           | equal_range  | equal_range       |

### 7.4. 第46条：考虑使用函数对象而不是函数作为STL算法的参数

### 7.5. 第47条：避免产生“直写型”（write-only）的代码

### 7.6. 第48条：总是包含（#include）正确的头文件

- 几乎所有的标准 `STL` 容器都被声明在与之同名的头文件中，比如 `vector` 被声明在 `<vector>` 中，`list` 被声明在 `<list>` 中。但是 `<set>` 中声明了 `set` 和 `multiset`，`<map>` 中声明了 `map` 和 `multimap`。
- 除了4个STL算法以外，其他所有的算法都被声明在 `<algorithm>` 中，这4个算法是 `accumulate`、`inner_product`、`adjacent_difference`和`partial_sum`，它们被声明在头文件 `<numeric>` 中。
- 特殊类型的迭代器，包括 `istream_iterator` 和 `istreambuf_iterator`（见第29条），被声明在 `<iterator>` 中。
- 标准的函数子（比如 `less<T>`）和函数子配接器（比如not1、bind2nd）被声明在头文件 `<functional>` 中。

### 7.7. 第49条：学会分析与STL相关的编译器诊断信息

| 类型        | 原型                                                   |
| ----------- | ------------------------------------------------------ |
| std::string | basic_string<char, char_traits<char>, allocator<char>> |

### 7.8. 第50条：熟悉与STL相关的Web站点

- SGI STL 站点：http://www.sgi.com/tech/stl
- STLport 站点：http://www.stlport.org
- Boost 站点：http://www.boost.org



