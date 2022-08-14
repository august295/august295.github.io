---
layout: post
title: "boost-序列化"
categories: C/C++
tags: ThirdPart boost serialization
author: August
mathjax: true
typora-root-url: ..
---

* content
{:toc}


本文讲述使用 `boost` 进行序列化。



# boost-序列化

`boost` 是 `C++` 委员会创建的库，优秀的库将加入 `C++` 标准，十分适合 `C++` 工程开发。

- 官方网址  [https://www.boost.org/](https://www.boost.org/)
- 开源网址  [https://github.com/boostorg/boost](https://github.com/boostorg/boost)



## 1 介绍

`Boost C++` 的 序列化 库允许将 `C++` 应用程序中的对象转换为一个字节序列， 此序列可以被保存，并可在将来恢复对象的时候再次加载。 

`Boost.Serialization` 的主要概念是归档。 归档的文件是相当于序列化的 `C++` 对象的一个字节流。 对象可以通过序列化添加到归档文件，相应地也可从归档文件中加载。 为了恢复和之前存储相同的 `C++` 对象，需假定数据类型是相同的。



## 2 侵入式序列化

### 2.1 people.h

```c++
#ifndef __PEOPLE_H__
#define __PEOPLE_H__

#include <iostream>
#include <sstream>

#include <boost/archive/text_iarchive.hpp>
#include <boost/archive/text_oarchive.hpp>
#include <boost/serialization/access.hpp>
#include <boost/serialization/string.hpp>
#include <boost/serialization/vector.hpp>

template <class T>
std::string toString(const T& theClass)
{
    std::ostringstream            oss;
    boost::archive::text_oarchive oa(oss);
    oa << theClass;
    return oss.str();
}
template <class T>
T fromString(const std::string& theString)
{
    T                             theClass;
    std::istringstream            iss(theString);
    boost::archive::text_iarchive ia(iss);
    ia >> theClass;
    return theClass;
}

class Software
{
public:
    Software();
    void print();

private:
    friend class boost::serialization::access;
    template <class Archive>
    void serialize(Archive& ar, const unsigned int version)
    {
        ar& m_type;
    }

private:
    int m_type;
};

class Phone
{
public:
    Phone();
    void print();

private:
    friend class boost::serialization::access;
    template <class Archive>
    void serialize(Archive& ar, const unsigned int version)
    {
        ar& m_name;
        for (size_t i = 0; i < m_software.size(); i++)
        {
            ar& m_software[i];
        }
    }

private:
    std::string           m_name;
    std::vector<Software> m_software;
};

class People
{
public:
    People();
    void print();

private:
    friend class boost::serialization::access;
    template <class Archive>
    void serialize(Archive& ar, const unsigned int version)
    {
        ar& m_age;
        ar& m_name;
        for (size_t i = 0; i < m_phones.size(); i++)
        {
            ar& m_phones[i];
        }
    }

private:
    int                m_age;
    std::string        m_name;
    std::vector<Phone> m_phones;
};

#endif

```

### 2.2 people.cpp

```cpp
#include "people.h"

// 软件
Software::Software() { m_type = -1; }

void Software::print() { std::cout << m_type << " "; }

// 手机
Phone::Phone()
{
    m_name = "xiaomi";
    Software s;
    m_software.push_back(s);
    m_software.push_back(s);
    m_software.push_back(s);
}

void Phone::print()
{
    std::cout << m_name << " ";
    for (size_t i = 0; i < m_software.size(); i++)
    {
        m_software[i].print();
    }
}

// 人
People::People()
{
    m_age  = 18;
    m_name = "august";
    Phone p;
    m_phones.push_back(p);
    m_phones.push_back(p);
}

void People::print()
{
    std::cout << m_age << " ";
    std::cout << m_name << " ";
    for (size_t i = 0; i < m_phones.size(); i++)
    {
        m_phones[i].print();
    }
    std::cout << std::endl;
}
```

### 2.3 main.cpp

```cpp
#include <iostream>

#include "people.h"

void demo()
{
    People      st;
    std::string str = toString<People>(st);
    st.print();

    std::cout << "\n"
              << str << "\n"
              << std::endl;

    // 低版本序列化后，高版本可以反序列化，反之不行
    People new_st = fromString<People>(str);
    new_st.print();
}

// g++ -o main main.cpp people.cpp -lboost_serialization
int main()
{
    demo();

    return 0;
}

/**
 * boost-1.56 
 *  22 serialization::archive 11 0 0 18 6 august 3 0 0 6 xiaomi 0 0 -1 -1 -1 6 xiaomi -1 -1 -1 6 xiaomi -1 -1 -1
 * boost-1.69
 *  22 serialization::archive 17 0 0 18 6 august 3 0 0 6 xiaomi 0 0 -1 -1 -1 6 xiaomi -1 -1 -1 6 xiaomi -1 -1 -1
 * boost-1.77
 *  22 serialization::archive 19 0 0 18 6 august 3 0 0 6 xiaomi 0 0 -1 -1 -1 6 xiaomi -1 -1 -1 6 xiaomi -1 -1 -1
 */
```



## 3 注意事项

- 序列化使用 `boost` 二次封装的 `STL` 库，不然可能序列化和反序列化失败。同时尽量不使用指针，这可能会导致多成嵌套反序列化失败。

```cpp
// array, list, map, queue, set, stack, string, vector
#include <boost/serialization/xxx.hpp>
```

- 高版本可以反序列化低版本序列化的数据，但是低版本不能反序列化高版本。
