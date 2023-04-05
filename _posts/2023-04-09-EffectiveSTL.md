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



























