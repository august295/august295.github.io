---
layout: post
title: "qt-serialization"
categories: C/C++
tags: qt
author: August
typora-root-url: ..
---

* content
{:toc}


本文讲述使用 `qt` 进行序列化和反序列化。



# qt-序列化

`Qt` 是一个跨平台 `C++` 图形用户界面应用程序开发框架。

- 官网网址： [https://www.qt.io/](https://www.qt.io/)
- 下载网址： [https://download.qt.io/](https://download.qt.io/)

| 下载镜像         | 网址                                     |
| ---------------- | ---------------------------------------- |
| 中国科学技术大学 | http://mirrors.ustc.edu.cn/qtproject/    |
| 清华大学         | https://mirrors.tuna.tsinghua.edu.cn/qt/ |



## 1. 介绍

`QDataStream` 类实现了序列化 `C++` 的基本数据类型的功能，比如 `char`，`short`，`int`，`char*` 等等,不但如此还可以直接序列化 `QMap` ,`QVector`之类的容器(需要保证容器内的元素是基本类型元素)。

| 类型    | 类型       | 类型    | 类型    |
| ------- | ---------- | ------- | ------- |
| bool    | float      | double  | char *  |
| qint8   | qint16     | qint32  | qint64  |
| quint8  | quint16    | quint32 | quint64 |
| qfloat  | QByteArray | QDate   | QTime   |
| QString | QVector    | QList   | QMap    |
| ...     | ...        | ...     | ...     |

但是往往程序中包含了复杂的数据结构，此时就不能直接进行序列化了。因此我们需要将复杂数据类型分解成独立的基本数据类型分别进行序列化。利用`QDataStream` 不能直接实现序列化，必须重载 `<<` 和 `>>` 操作符，只有重载完之后才可以按我们的要求实现序列化。

```cpp
QDataStream &operator<<(QDataStream &, const xxx &);
QDataStream &operator>>(QDataStream &, xxx &);
```



## 2. 使用

### 2.1. serialize.h

类型声明文件 

```cpp
#include <iosfwd>

#include <QByteArray>
#include <QDataStream>
#include <QMap>
#include <QString>
#include <QVector>

enum class PhoneType {
    MOBILE = 0,
    HOME   = 1,
    WORK   = 2,
};

// 手机
class Phone {
public:
    Phone();
    Phone(QString number, PhoneType type);

    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const Phone& phone);

    // 重载 (QDataStream <<) 进行序列化
    friend QDataStream& operator<<(QDataStream& stream, const Phone& phone);
    friend QDataStream& operator>>(QDataStream& stream, Phone& phone);

public:
    QString   _Number;
    PhoneType _Type;
};

// 人
class Person {
public:
    Person();
    Person(QString name, int id, QString email, double salary, int phoneNum, QVector<Phone> phone);

    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const Person& person);

    // 重载 (QDataStream <<) 进行序列化
    friend QDataStream& operator<<(QDataStream& stream, const Person& person);
    friend QDataStream& operator>>(QDataStream& stream, Person& person);

public:
    QString        _Name;
    int            _Id;
    QString        _Email;
    double         _Salary;
    int            _PhoneNum;
    QVector<Phone> _Phone;
};

// 地址簿
class AddressBook {
public:
    AddressBook();
    void Insert(QString personName, Person person);

    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const AddressBook& addressBook);

    // 重载 (QDataStream <<) 进行序列化
    friend QDataStream& operator<<(QDataStream& stream, const AddressBook& addressBook);
    friend QDataStream& operator>>(QDataStream& stream, AddressBook& addressBook);

public:
    QMap<QString, Person> _Persons;
};

```

### 2.2. serialize.cpp

类型实现文件

```cpp
#include "serialize.h"

// 手机
Phone::Phone()
{
}

Phone::Phone(QString number, PhoneType type)
    : _Number(number)
    , _Type(type)
{
}

std::ostream& operator<<(std::ostream& stream, const Phone& phone)
{
    stream << "\t\t" << phone._Number.toStdString() << std::endl;
    stream << "\t\t" << static_cast<int>(phone._Type) << std::endl;
    return stream;
}

QDataStream& operator<<(QDataStream& stream, const Phone& phone)
{
    stream << phone._Number << static_cast<int>(phone._Type);
    return stream;
}

QDataStream& operator>>(QDataStream& stream, Phone& phone)
{
    int type = 0;
    stream >> phone._Number >> type;
    phone._Type = static_cast<PhoneType>(type);
    return stream;
}

// 人
Person::Person()
{
}

Person::Person(QString name, int id, QString email, double salary, int phoneNum, QVector<Phone> phone)
    : _Name(name)
    , _Id(id)
    , _Email(email)
    , _Salary(salary)
    , _PhoneNum(phoneNum)
    , _Phone(phone)
{
}

std::ostream& operator<<(std::ostream& stream, const Person& person)
{
    stream << "\t" << person._Name.toStdString() << std::endl;
    stream << "\t" << person._Id << std::endl;
    stream << "\t" << person._Email.toStdString() << std::endl;
    stream << "\t" << person._Salary << std::endl;
    stream << "\t" << person._PhoneNum << std::endl;
    for (auto p : person._Phone) {
        stream << p;
    }
    return stream;
}

QDataStream& operator<<(QDataStream& stream, const Person& person)
{
    stream << person._Name << person._Id << person._Email << person._Salary << person._PhoneNum << person._Phone;
    return stream;
}

QDataStream& operator>>(QDataStream& stream, Person& person)
{
    stream >> person._Name >> person._Id >> person._Email >> person._Salary >> person._PhoneNum >> person._Phone;
    return stream;
}

// 地址簿
AddressBook::AddressBook()
{
}

void AddressBook::Insert(QString personName, Person person)
{
    _Persons.insert(personName, person);
}

std::ostream& operator<<(std::ostream& stream, const AddressBook& addressBook)
{
    for (auto p = addressBook._Persons.begin(); p != addressBook._Persons.end(); ++p) {
        stream << p.key().toStdString() << p.value();
    }
    return stream;
}

QDataStream& operator<<(QDataStream& stream, const AddressBook& addressBook)
{
    stream << addressBook._Persons;
    return stream;
}

QDataStream& operator>>(QDataStream& stream, AddressBook& addressBook)
{
    stream >> addressBook._Persons;
    return stream;
}

```

### 2.3. main.cpp

测试文件

```c++
#include <iostream>

#include "serialize.h"

AddressBook InitAddressBook()
{
    QVector<Phone> phoneVec;
    Phone          phone("123", PhoneType::MOBILE);
    phoneVec.push_back(phone);
    Person person("august", 1, "123@qq.com", 1234.5, 1, phoneVec);

    QVector<Phone> phoneVec2;
    Phone          phone2("456", PhoneType::WORK);
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
    QByteArray  qba;
    QDataStream qds(&qba, QIODevice::WriteOnly);
    qds.setVersion(QDataStream::Qt_5_9);
    qds << addressBook;

    // 反序列化
    AddressBook addressBookNew;
    QDataStream qdsNew(&qba, QIODevice::ReadOnly);
    qdsNew.setVersion(QDataStream::Qt_5_9);
    qdsNew >> addressBookNew;
    std::cout << addressBookNew << std::endl;

    return 0;
}
```



## 3. 注意事项

- 重载 `<<` `>>` 使用友元函数
- `C++11` 新特性，枚举添加作用域，不能隐式转换为 `int` 类型，需要强制类型转换。
