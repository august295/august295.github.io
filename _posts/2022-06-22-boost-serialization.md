---
layout: post
title: "boost-serialization"
categories: C/C++
tags: boost
author: August
typora-root-url: ..
---

* content
{:toc}


本文讲述使用 `boost` 进行序列化。



# boost-序列化

`boost` 是 `C++` 委员会创建的库，优秀的库将加入 `C++` 标准，十分适合 `C++` 工程开发。

- 官方网址  [https://www.boost.org/](https://www.boost.org/)
- 开源网址  [https://github.com/boostorg/boost](https://github.com/boostorg/boost)



## 1. 介绍

`Boost C++` 的 序列化 库允许将 `C++` 应用程序中的对象转换为一个字节序列， 此序列可以被保存，并可在将来恢复对象的时候再次加载。 

`Boost.Serialization` 的主要概念是归档。 归档的文件是相当于序列化的 `C++` 对象的一个字节流。 对象可以通过序列化添加到归档文件，相应地也可从归档文件中加载。 为了恢复和之前存储相同的 `C++` 对象，需假定数据类型是相同的。



## 2. 使用

### 2.1. serialize.h

类型声明文件

```cpp
#pragma once

#include <iostream>
#include <sstream>

#include <boost/archive/binary_iarchive.hpp>
#include <boost/archive/binary_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <boost/archive/text_oarchive.hpp>
#include <boost/serialization/access.hpp>
#include <boost/serialization/map.hpp>
#include <boost/serialization/string.hpp>
#include <boost/serialization/vector.hpp>

enum class PhoneType
{
    MOBILE = 0,
    HOME   = 1,
    WORK   = 2,
};

// 手机
class Phone
{
public:
    Phone();
    Phone(std::string number, PhoneType type);

    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const Phone& phone);

public:
    std::string _Number;
    PhoneType   _Type;
};

// 人
class Person
{
public:
    Person();
    Person(std::string name, int id, std::string email, double salary, int phoneNum, std::vector<Phone> phone);

    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const Person& person);

public:
    std::string        _Name;
    int                _Id;
    std::string        _Email;
    double             _Salary;
    int                _PhoneNum;
    std::vector<Phone> _Phone;
};

// 地址簿
class AddressBook
{
public:
    AddressBook();
    void Insert(std::string personName, Person person);

    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const AddressBook& addressBook);

    // 侵入式序列化
    friend class boost::serialization::access;
    template <class Archive>
    void serialize(Archive& ar, const unsigned int version)
    {
        ar& _Persons;
    }

public:
    std::map<std::string, Person> _Persons;
};

// 非侵入式序列化
namespace boost
{
    namespace serialization
    {
        template <class Archive>
        void serialize(Archive& ar, Phone& phone, const unsigned int version)
        {
            ar& phone._Number;
            ar& phone._Type;
        }

        template <class Archive>
        void serialize(Archive& ar, Person& person, const unsigned int version)
        {
            ar& person._Name;
            ar& person._Id;
            ar& person._Email;
            ar& person._Salary;
            ar& person._PhoneNum;
            ar& person._Phone;
        }
    } // namespace serialization
} // namespace boost

template <class T>
std::string toBinary(const T& theStruct)
{
    std::ostringstream              oss;
    boost::archive::binary_oarchive oa(oss);
    oa << theStruct;
    return oss.str();
}
template <class T>
T fromBinary(const std::string& theText)
{
    T                               theStruct;
    std::istringstream              iss(theText);
    boost::archive::binary_iarchive ia(iss);
    ia >> theStruct;
    return theStruct;
}

template <class T>
std::string toText(const T& theStruct)
{
    std::ostringstream            oss;
    boost::archive::text_oarchive oa(oss);
    oa << theStruct;
    return oss.str();
}
template <class T>
T fromText(const std::string& theText)
{
    T                             theStruct;
    std::istringstream            iss(theText);
    boost::archive::text_iarchive ia(iss);
    ia >> theStruct;
    return theStruct;
}

```

### 2.2. serialize.cpp

类型实现文件

```cpp
#include "serialize.h"

// 手机
Phone::Phone()
{}

Phone::Phone(std::string number, PhoneType type) : _Number(number), _Type(type)
{}

std::ostream& operator<<(std::ostream& stream, const Phone& phone)
{
    stream << "\t\t" << phone._Number << std::endl;
    stream << "\t\t" << static_cast<int>(phone._Type) << std::endl;
    return stream;
}

// 人
Person::Person()
{}

Person::Person(std::string name, int id, std::string email, double salary, int phoneNum, std::vector<Phone> phone) :
    _Name(name), _Id(id), _Email(email), _Salary(salary), _PhoneNum(phoneNum), _Phone(phone)
{}

std::ostream& operator<<(std::ostream& stream, const Person& person)
{
    stream << "\t" << person._Name << std::endl;
    stream << "\t" << person._Id << std::endl;
    stream << "\t" << person._Email << std::endl;
    stream << "\t" << person._Salary << std::endl;
    stream << "\t" << person._PhoneNum << std::endl;
    for (auto p : person._Phone)
    {
        stream << p;
    }
    return stream;
}

// 地址簿
AddressBook::AddressBook()
{}

void AddressBook::Insert(std::string personName, Person person)
{
    _Persons.emplace(personName, person);
}

std::ostream& operator<<(std::ostream& stream, const AddressBook& addressBook)
{
    for (auto p = addressBook._Persons.begin(); p != addressBook._Persons.end(); ++p)
    {
        stream << (*p).first << (*p).second;
    }
    return stream;
}
```

### 2.3. main.cpp

测试文件

```cpp
#include <iostream>

#include "serialize.h"

AddressBook InitAddressBook()
{
    std::vector<Phone> phoneVec;
    Phone              phone("123", PhoneType::MOBILE);
    phoneVec.push_back(phone);
    Person person("august", 1, "123@qq.com", 1234.5, 1, phoneVec);

    std::vector<Phone> phoneVec2;
    Phone              phone2("456", PhoneType::WORK);
    phoneVec2.push_back(phone2);
    Person person2("tom", 2, "456@qq.com", 4567.8, 1, phoneVec2);

    AddressBook addressBook;
    addressBook.Insert(person._Name, person);
    addressBook.Insert(person2._Name, person2);
    return addressBook;
}

int main()
{
    AddressBook addressBook = InitAddressBook();

    // 序列化
    std::string text        = toText<AddressBook>(addressBook);
    std::cout << text << std::endl;

    // 反序列化
    AddressBook addressBookNew;
    addressBookNew = fromText<AddressBook>(text);
    std::cout << addressBookNew << std::endl;

    return 0;
}
```



## 3. 注意事项

- 序列化使用 `boost` 二次封装的 `STL` 库，不然可能序列化和反序列化失败。

```cpp
// array, list, map, queue, set, stack, string, vector
#include <boost/serialization/xxx.hpp>
```

- 高版本可以反序列化低版本序列化的数据，但是低版本不能反序列化高版本。



# 参考

[1] [boost 之序列化和反序列化](https://zhuanlan.zhihu.com/p/432784584)

[2] [boost_xml](https://www.boost.org/doc/libs/1_52_0/libs/serialization/example/demo_xml.cpp)
