---
layout: post
title: "protobuf-c"
categories: C/C++
tags: ThirdPart
author: August
typora-root-url: ..
---

* content
{:toc}
本文主要介绍 `protobuf` 对 `C` 结构体序列化的相互转换。



# protobuf-c



## 1. 网址

- 官网 [https://developers.google.com/protocol-buffers/](https://developers.google.com/protocol-buffers/)

+ 开源网址  [https://github.com/protocolbuffers/protobuf](https://github.com/protocolbuffers/protobuf)


## 2. 扩展

很多时候在构建数据时不是使用 `protobuf` 创建的类，而是使用原生结构体仅仅是在需要序列化的时候使用，根据 `protobuf` 的创建的相同结构体类可以获得结构体的字段个数和字段类型，再根据字段偏移实现原生结构体序列化。

<font color=red>因为是根据偏移实现，所以结构体必须使用 `#pragma pack(1)` 将结构体紧凑填充。</font>

### 2.1. PbConvertor

可以参考：[https://github.com/hardxuyp/PbConvertor](https://github.com/hardxuyp/PbConvertor)

#### 2.1.1. PbConvertor.h

```cpp
#ifndef PBCONVERTOR_H
#define PBCONVERTOR_H

/**
 * @file PbConvertor.h
 * @brief PbConvertor is used to convert protobuf message to C struct or C struct to protobuf message.
 * @author xuyp
 * @date 2019.12
 */

#include <google/protobuf/message.h>
#include <string>
#include <vector>

class PbConvertor
{
public:
    struct MemTree
    {
        char*                pMem;
        size_t               memSize;
        std::vector<MemTree> vecChild;

        MemTree()
        {
            pMem    = NULL;
            memSize = 0;
        }

        MemTree(char* pMem, size_t memSize)
        {
            this->pMem    = pMem;
            this->memSize = memSize;
        }

        void release()
        {
            for (auto& child : vecChild)
                child.release();
            if (NULL != pMem)
                free(pMem);
        }
    };

    /**
     * @brief convert C struct to protobuf message.
     * @param pStruct pointer to C struct.
     * @param pPb pointer to protobuf message.
     * @return whether successful.
     */
    static bool struct2Pb(const char*& pStruct, google::protobuf::Message* pPb);

    /**
     * @brief convert C struct to serialized protobuf message.
     * @param pStruct pointer to C struct.
     * @param pbTypeName protobuf message type name.
     * @param pSerializedPb pointer to serialized protobuf message which must be released by user using delete [].
     * @param serializedPbSize serialized protobuf message size.
     * @return whether successful.
     */
    static bool struct2serializedPb(const char*        pStruct,
                                    const std::string& pbTypeName,
                                    char*&             pSerializedPb,
                                    size_t&            serializedPbSize);

    /**
     * @brief convert protobuf message to C struct.
     * @param pPb pointer to protobuf message.
     * @param stru memory tree related to C struct, and pointer to C struct equals to stru.pMem.
     *        Memory tree must be released by user using stru.release().
     * @return whether successful.
     */
    static bool pb2struct(const google::protobuf::Message* pPb, MemTree& stru);

    /**
     * @brief convert serialized protobuf message to C struct.
     * @param pbTypeName protobuf message type name.
     * @param pSerializedPb pointer to serialized protobuf message.
     * @param serializedPbSize serialized protobuf message size.
     * @param stru memory tree related to C struct, and pointer to C struct equals to stru.pMem.
     *        Memory tree must be released by user using stru.release().
     * @return whether successful.
     */
    static bool serializedPb2struct(const std::string& pbTypeName,
                                    const char*        pSerializedPb,
                                    const size_t       serializedPbSize,
                                    MemTree&           stru);

private:
    static bool getSize(const google::protobuf::Reflection*      pReflection,
                        const google::protobuf::Message*         pPb,
                        const google::protobuf::FieldDescriptor* pFieldDescriptor,
                        size_t&                                  size);
};

#endif // PBCONVERTOR_H
```

#### 2.1.2. PbConvertor.cpp

```cpp
#include "PbConvertor.h"

bool PbConvertor::struct2Pb(const char*& pStruct, google::protobuf::Message* pPb)
{
    if (NULL == pStruct || NULL == pPb)
        return false;

    const google::protobuf::Descriptor* pDescriptor = pPb->GetDescriptor();
    if (NULL == pDescriptor)
        return false;
    const google::protobuf::Reflection* pReflection = pPb->GetReflection();
    if (NULL == pReflection)
        return false;

    for (size_t i = 0; i < pDescriptor->field_count(); ++i)
    {
        const google::protobuf::FieldDescriptor* pFieldDescriptor = pDescriptor->field(i);
        if (NULL == pFieldDescriptor)
            return false;

        switch (pFieldDescriptor->cpp_type())
        {
#define _CONVERT(type, pbType, setMethod, addMethod)                                                 \
    case google::protobuf::FieldDescriptor::pbType:                                                  \
    {                                                                                                \
        if (pFieldDescriptor->is_repeated())                                                         \
        {                                                                                            \
            if (i > 0)                                                                               \
            {                                                                                        \
                size_t size;                                                                         \
                if (getSize(pReflection, pPb, pDescriptor->field(i - 1), size))                      \
                {                                                                                    \
                    if (size > 0)                                                                    \
                    {                                                                                \
                        type* addr = *(type**)pStruct;                                               \
                        if (NULL != addr)                                                            \
                        {                                                                            \
                            for (size_t j = 0; j < size; ++j)                                        \
                                pReflection->addMethod(pPb, pFieldDescriptor, *(addr + j));          \
                        }                                                                            \
                    }                                                                                \
                }                                                                                    \
            }                                                                                        \
            pStruct += sizeof(void*);                                                                \
        }                                                                                            \
        else                                                                                         \
        {                                                                                            \
            if (google::protobuf::FieldDescriptor::CPPTYPE_STRING != pFieldDescriptor->cpp_type() || \
                NULL != *(type*)pStruct)                                                             \
                pReflection->setMethod(pPb, pFieldDescriptor, *(type*)pStruct);                      \
            pStruct += sizeof(type);                                                                 \
        }                                                                                            \
        break;                                                                                       \
    }

            _CONVERT(double, CPPTYPE_DOUBLE, SetDouble, AddDouble)
            _CONVERT(float, CPPTYPE_FLOAT, SetFloat, AddFloat)
            _CONVERT(int32_t, CPPTYPE_INT32, SetInt32, AddInt32)
            _CONVERT(int64_t, CPPTYPE_INT64, SetInt64, AddInt64)
            _CONVERT(uint32_t, CPPTYPE_UINT32, SetUInt32, AddUInt32)
            _CONVERT(uint64_t, CPPTYPE_UINT64, SetUInt64, AddUInt64)
            _CONVERT(bool, CPPTYPE_BOOL, SetBool, AddBool)
            _CONVERT(char*, CPPTYPE_STRING, SetString, AddString)
            _CONVERT(int, CPPTYPE_ENUM, SetEnumValue, AddEnumValue)
#undef _CONVERT

        case google::protobuf::FieldDescriptor::CPPTYPE_MESSAGE:
        {
            if (pFieldDescriptor->is_repeated())
            {
                if (i > 0)
                {
                    size_t size;
                    if (getSize(pReflection, pPb, pDescriptor->field(i - 1), size))
                    {
                        if (size > 0)
                        {
                            const char* addr = *(char**)pStruct;
                            for (size_t j = 0; j < size; ++j)
                            {
                                if (!struct2Pb(addr, pReflection->AddMessage(pPb, pFieldDescriptor)))
                                    return false;
                            }
                        }
                    }
                }
                pStruct += sizeof(void*);
            }
            else
            {
                if (!struct2Pb(pStruct, pReflection->MutableMessage(pPb, pFieldDescriptor)))
                    return false;
            }
            break;
        }

        default:
        {
            break;
        }
        }
    }
    return true;
}

bool PbConvertor::struct2serializedPb(const char*        pStruct,
                                      const std::string& pbTypeName,
                                      char*&             pSerializedPb,
                                      size_t&            serializedPbSize)
{
    if (NULL == pStruct)
    {
        pSerializedPb = NULL;
        return false;
    }

    const google::protobuf::Descriptor* pDescriptor =
        google::protobuf::DescriptorPool::generated_pool()->FindMessageTypeByName(pbTypeName);
    if (NULL == pDescriptor)
        return false;
    const google::protobuf::Message* pPrototype =
        google::protobuf::MessageFactory::generated_factory()->GetPrototype(pDescriptor);
    if (NULL == pPrototype)
        return false;
    google::protobuf::Message* pPb = pPrototype->New();
    if (NULL == pPb)
        return false;

    bool bRet = false;
    if (struct2Pb(pStruct, pPb))
    {
        serializedPbSize = pPb->ByteSizeLong();
        pSerializedPb    = new char[serializedPbSize];
        if (pPb->SerializeToArray(pSerializedPb, serializedPbSize))
            bRet = true;
        else
        {
            delete[] pSerializedPb;
            pSerializedPb = NULL;
        }
    }
    delete pPb;
    return bRet;
}

bool PbConvertor::pb2struct(const google::protobuf::Message* pPb, MemTree& stru)
{
    if (NULL == pPb)
        return false;

    const google::protobuf::Descriptor* pDescriptor = pPb->GetDescriptor();
    if (NULL == pDescriptor)
        return false;
    const google::protobuf::Reflection* pReflection = pPb->GetReflection();
    if (NULL == pReflection)
        return false;

    for (size_t i = 0; i < pDescriptor->field_count(); ++i)
    {
        const google::protobuf::FieldDescriptor* pFieldDescriptor = pDescriptor->field(i);
        if (NULL == pFieldDescriptor)
            return false;

        switch (pFieldDescriptor->cpp_type())
        {
#define _CONVERT(type, pbType, getMethod, getRepeatedMethod)                                            \
    case google::protobuf::FieldDescriptor::pbType:                                                     \
    {                                                                                                   \
        if (pFieldDescriptor->is_repeated())                                                            \
        {                                                                                               \
            size_t newMemSize = stru.memSize + sizeof(void*);                                           \
            char*  p          = (char*)realloc(stru.pMem, newMemSize);                                  \
            if (NULL == p)                                                                              \
                return false;                                                                           \
            *(type**)(p + stru.memSize) = NULL;                                                         \
            if (i > 0)                                                                                  \
            {                                                                                           \
                size_t size;                                                                            \
                if (getSize(pReflection, pPb, pDescriptor->field(i - 1), size))                         \
                {                                                                                       \
                    if (size > 0)                                                                       \
                    {                                                                                   \
                        size_t memSize = size * sizeof(type);                                           \
                        char*  p1      = (char*)malloc(memSize);                                        \
                        if (NULL == p1)                                                                 \
                            return false;                                                               \
                        for (size_t j = 0; j < size; ++j)                                               \
                            ((type*)p1)[j] = pReflection->getRepeatedMethod(*pPb, pFieldDescriptor, j); \
                        *(type**)(p + stru.memSize) = (type*)p1;                                        \
                        stru.vecChild.emplace_back(p1, memSize);                                        \
                    }                                                                                   \
                }                                                                                       \
            }                                                                                           \
            stru.pMem    = p;                                                                           \
            stru.memSize = newMemSize;                                                                  \
        }                                                                                               \
        else                                                                                            \
        {                                                                                               \
            size_t newMemSize = stru.memSize + sizeof(type);                                            \
            void*  p          = realloc(stru.pMem, newMemSize);                                         \
            if (NULL == p)                                                                              \
                return false;                                                                           \
            *(type*)(p + stru.memSize) = pReflection->getMethod(*pPb, pFieldDescriptor);                \
            stru.pMem                  = (char*)p;                                                      \
            stru.memSize               = newMemSize;                                                    \
        }                                                                                               \
        break;                                                                                          \
    }

            _CONVERT(double, CPPTYPE_DOUBLE, GetDouble, GetRepeatedDouble)
            _CONVERT(float, CPPTYPE_FLOAT, GetFloat, GetRepeatedFloat)
            _CONVERT(int32_t, CPPTYPE_INT32, GetInt32, GetRepeatedInt32)
            _CONVERT(int64_t, CPPTYPE_INT64, GetInt64, GetRepeatedInt64)
            _CONVERT(uint32_t, CPPTYPE_UINT32, GetUInt32, GetRepeatedUInt32)
            _CONVERT(uint64_t, CPPTYPE_UINT64, GetUInt64, GetRepeatedUInt64)
            _CONVERT(bool, CPPTYPE_BOOL, GetBool, GetRepeatedBool)
            _CONVERT(int, CPPTYPE_ENUM, GetEnumValue, GetRepeatedEnumValue)
#undef _CONVERT

        case google::protobuf::FieldDescriptor::CPPTYPE_STRING:
        {
            if (pFieldDescriptor->is_repeated())
            {
                size_t newMemSize = stru.memSize + sizeof(void*);
                char*  p          = (char*)realloc(stru.pMem, newMemSize);
                if (NULL == p)
                    return false;
                *(char***)(p + stru.memSize) = NULL;
                if (i > 0)
                {
                    size_t size;
                    if (getSize(pReflection, pPb, pDescriptor->field(i - 1), size))
                    {
                        if (size > 0)
                        {
                            size_t memSize = size * sizeof(char*);
                            char*  p1      = (char*)malloc(memSize);
                            if (NULL == p1)
                                return false;
                            *(char***)(p + stru.memSize) = (char**)p1;
                            stru.vecChild.emplace_back(p1, memSize);
                            MemTree& memTree = stru.vecChild.back();
                            for (size_t j = 0; j < size; ++j)
                            {
                                std::string str;
                                str = pReflection->GetRepeatedStringReference(*pPb, pFieldDescriptor, j, &str);
                                size_t memSize2 = str.size() + 1;
                                char*  p2       = (char*)calloc(memSize2, 1);
                                if (NULL == p2)
                                    return false;
                                strcpy(p2, str.c_str());
                                ((char**)p1)[j] = p2;
                                memTree.vecChild.emplace_back(p2, memSize2);
                            }
                        }
                    }
                }
                stru.pMem    = p;
                stru.memSize = newMemSize;
            }
            else
            {
                size_t newMemSize = stru.memSize + sizeof(char*);
                void*  p          = realloc(stru.pMem, newMemSize);
                if (NULL == p)
                    return false;
                std::string str;
                str            = pReflection->GetStringReference(*pPb, pFieldDescriptor, &str);
                size_t memSize = str.size() + 1;
                char*  p1      = (char*)calloc(memSize, 1);
                if (NULL == p1)
                    return false;
                strcpy(p1, str.c_str());
                *(char**)(p + stru.memSize) = p1;
                stru.vecChild.emplace_back(p1, memSize);
                stru.pMem    = (char*)p;
                stru.memSize = newMemSize;
            }
            break;
        }

        case google::protobuf::FieldDescriptor::CPPTYPE_MESSAGE:
        {
            if (pFieldDescriptor->is_repeated())
            {
                size_t newMemSize = stru.memSize + sizeof(void*);
                char*  p          = (char*)realloc(stru.pMem, newMemSize);
                if (NULL == p)
                    return false;
                *(void**)(p + stru.memSize) = NULL;
                if (i > 0)
                {
                    size_t size;
                    if (getSize(pReflection, pPb, pDescriptor->field(i - 1), size))
                    {
                        if (size > 0)
                        {
                            stru.vecChild.emplace_back((char*)NULL, 0);
                            MemTree& memTree = stru.vecChild.back();
                            for (size_t j = 0; j < size; ++j)
                            {
                                const google::protobuf::Message& childPb =
                                    pReflection->GetRepeatedMessage(*pPb, pFieldDescriptor, j);
                                if (!pb2struct(&childPb, memTree))
                                    return false;
                            }
                            *(void**)(p + stru.memSize) = (void*)memTree.pMem;
                        }
                    }
                }
                stru.pMem    = p;
                stru.memSize = newMemSize;
            }
            else
            {
                const google::protobuf::Message& childPb = pReflection->GetMessage(*pPb, pFieldDescriptor);
                if (!pb2struct(&childPb, stru))
                    return false;
            }
            break;
        }

        default:
        {
            break;
        }
        }
    }
    return true;
}

bool PbConvertor::serializedPb2struct(const std::string& pbTypeName,
                                      const char*        pSerializedPb,
                                      const size_t       serializedPbSize,
                                      MemTree&           stru)
{
    if (NULL == pSerializedPb)
        return false;

    const google::protobuf::Descriptor* pDescriptor =
        google::protobuf::DescriptorPool::generated_pool()->FindMessageTypeByName(pbTypeName);
    if (NULL == pDescriptor)
        return false;
    const google::protobuf::Message* pPrototype =
        google::protobuf::MessageFactory::generated_factory()->GetPrototype(pDescriptor);
    if (NULL == pPrototype)
        return false;
    google::protobuf::Message* pPb = pPrototype->New();
    if (NULL == pPb)
        return false;

    bool bRet = false;
    if (pPb->ParseFromArray(pSerializedPb, serializedPbSize))
        bRet = pb2struct(pPb, stru);
    delete pPb;
    return bRet;
}

bool PbConvertor::getSize(const google::protobuf::Reflection*      pReflection,
                          const google::protobuf::Message*         pPb,
                          const google::protobuf::FieldDescriptor* pFieldDescriptor,
                          size_t&                                  size)
{
    if (NULL == pReflection || NULL == pFieldDescriptor)
        return false;

    bool bRet = true;
    if (google::protobuf::FieldDescriptor::CPPTYPE_INT32 == pFieldDescriptor->cpp_type())
        size = pReflection->GetInt32(*pPb, pFieldDescriptor);
    else if (google::protobuf::FieldDescriptor::CPPTYPE_INT64 == pFieldDescriptor->cpp_type())
        size = pReflection->GetInt64(*pPb, pFieldDescriptor);
    else if (google::protobuf::FieldDescriptor::CPPTYPE_UINT32 == pFieldDescriptor->cpp_type())
        size = pReflection->GetUInt32(*pPb, pFieldDescriptor);
    else if (google::protobuf::FieldDescriptor::CPPTYPE_UINT64 == pFieldDescriptor->cpp_type())
        size = pReflection->GetUInt64(*pPb, pFieldDescriptor);
    else
        bRet = false;
    return bRet;
}
```

### 2.2. 测试

#### 2.2.1. serialize.h

类型声明文件 

```cpp
#pragma once

#include <iostream>

enum class PhoneType
{
    MOBILE = 0,
    HOME   = 1,
    WORK   = 2,
};

#pragma pack(1)
struct Phone
{
    char*     m_number;
    PhoneType m_type;

    // 构造
    Phone();
    Phone(char* number, PhoneType type);
    Phone(const Phone& phone);
    ~Phone();

    // 重载
    Phone operator=(const Phone& phone);
    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const Phone& phone);
};

struct Person
{
    char*  m_name;
    int    m_id;
    char*  m_email;
    double m_salary;
    int    m_phone_num;
    Phone* m_phone;

    // 构造
    Person();
    Person(char* name, int id, char* email, double salary, int phone_num, Phone* phone);
    Person(const Person& person);
    ~Person();

    // 重载
    Person operator=(const Person& person);
    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const Person& person);
};

struct AddressBook
{
    int     m_people_num;
    Person* m_people;

    // 构造
    AddressBook();
    AddressBook(int people_num, Person* m_people);
    AddressBook(const AddressBook& addressBook);
    ~AddressBook();

    // 重载
    AddressBook operator=(const AddressBook& addressBook);
    // 重载 (std::cout <<) 进行打印
    friend std::ostream& operator<<(std::ostream& stream, const AddressBook& addressBook);
};
#pragma pack()

```

#### 2.2.2. serialize.cpp

类型实现文件

```cpp
#include <memory.h>
#include <string.h>

#include "serialize.h"

#define MY_ALLOC(dest, src)           \
    {                                 \
        size_t len = strlen(src) + 1; \
        dest       = new char[len];   \
        strcpy_s(dest, len, src);     \
    }

#define MY_FREE(src)  \
    if (src)          \
    {                 \
        delete[] src; \
    }

Phone::Phone()
{
    m_number = nullptr;
}

Phone::Phone(char* number, PhoneType type)
{
    MY_ALLOC(m_number, number);
    m_type = type;
}

Phone::Phone(const Phone& phone)
{
    MY_ALLOC(m_number, phone.m_number);
    m_type = phone.m_type;
}

Phone::~Phone()
{
    MY_FREE(m_number);
}

Phone Phone::operator=(const Phone& phone)
{
    MY_FREE(m_number);

    MY_ALLOC(m_number, phone.m_number);
    m_type = phone.m_type;
    return *this;
}

std::ostream& operator<<(std::ostream& stream, const Phone& phone)
{
    stream << "\t\t" << phone.m_number << std::endl;
    stream << "\t\t" << static_cast<int>(phone.m_type) << std::endl;
    return stream;
}

Person::Person()
{
    m_name  = nullptr;
    m_email = nullptr;
    m_phone = nullptr;
}

Person::Person(char* name, int id, char* email, double salary, int phone_num, Phone* phone)
{
    MY_ALLOC(m_name, name);
    m_id = id;
    MY_ALLOC(m_email, email);
    m_salary    = salary;
    m_phone_num = phone_num;
    m_phone     = new Phone[m_phone_num];
    for (int i = 0; i < m_phone_num; i++)
    {
        m_phone[i] = phone[i];
    }
}

Person::Person(const Person& person)
{
    MY_ALLOC(m_name, person.m_name);
    m_id = person.m_id;
    MY_ALLOC(m_email, person.m_email);
    m_salary    = person.m_salary;
    m_phone_num = person.m_phone_num;
    m_phone     = new Phone[m_phone_num];
    for (int i = 0; i < m_phone_num; i++)
    {
        m_phone[i] = person.m_phone[i];
    }
}

Person::~Person()
{
    MY_FREE(m_name);
    MY_FREE(m_email);
    MY_FREE(m_phone);
}

Person Person::operator=(const Person& person)
{
    MY_FREE(m_name);
    MY_FREE(m_email);
    MY_FREE(m_phone);

    MY_ALLOC(m_name, person.m_name);
    m_id = person.m_id;
    MY_ALLOC(m_email, person.m_email);
    m_salary    = person.m_salary;
    m_phone_num = person.m_phone_num;
    m_phone     = new Phone[m_phone_num];
    for (int i = 0; i < m_phone_num; i++)
    {
        m_phone[i] = person.m_phone[i];
    }
    return *this;
}

std::ostream& operator<<(std::ostream& stream, const Person& person)
{
    stream << "\t" << person.m_name << std::endl;
    stream << "\t" << person.m_id << std::endl;
    stream << "\t" << person.m_email << std::endl;
    stream << "\t" << person.m_salary << std::endl;
    stream << "\t" << person.m_phone_num << std::endl;
    for (int i = 0; i < person.m_phone_num; i++)
    {
        stream << person.m_phone[i];
    }
    return stream;
}

AddressBook::AddressBook()
{
    m_people = nullptr;
}

AddressBook::AddressBook(int people_num, Person* people)
{
    m_people_num = people_num;
    m_people     = new Person[m_people_num];
    for (int i = 0; i < m_people_num; i++)
    {
        m_people[i] = people[i];
    }
}

AddressBook::AddressBook(const AddressBook& addressBook)
{
    m_people_num = addressBook.m_people_num;
    m_people     = new Person[m_people_num];
    for (int i = 0; i < m_people_num; i++)
    {
        m_people[i] = addressBook.m_people[i];
    }
}

AddressBook::~AddressBook()
{
    MY_FREE(m_people);
}

AddressBook AddressBook::operator=(const AddressBook& addressBook)
{
    MY_FREE(m_people);

    m_people_num = addressBook.m_people_num;
    m_people     = new Person[m_people_num];
    for (int i = 0; i < m_people_num; i++)
    {
        m_people[i] = addressBook.m_people[i];
    }
    return *this;
}

std::ostream& operator<<(std::ostream& stream, const AddressBook& addressBook)
{
    for (int i = 0; i < addressBook.m_people_num; i++)
    {
        stream << addressBook.m_people[i];
    }
    return stream;
}

```

#### 2.2.3. main.cpp

测试文件

```cpp
#pragma once

#include "PbConvertor.h"
#include "address_book.pb.h"
#include "serialize.h"

AddressBook InitAddressBook()
{
    Phone phone1("12345678910", PhoneType::MOBILE);
    Phone phone2("10987654321", PhoneType::HOME);
    Phone phone3("11122233344", PhoneType::WORK);
    Phone phone4("77788899944", PhoneType::WORK);
    Phone phoneVec1[] = {phone1, phone2};
    Phone phoneVec2[] = {phone3, phone4};

    Person person1("august", 1, "123@qq.com", 1234.5, 2, phoneVec1);
    Person person2("tom", 2, "456@qq.com", 4567.8, 2, phoneVec2);
    Person personVec[] = {person1, person2};

    AddressBook addressBook(2, personVec);
    return addressBook;
}

void demo_auto_pbconvertor()
{
    AddressBook addressBook = InitAddressBook();

    // 序列化
    char*  pSerializedPb    = nullptr;
    size_t serializedPbSize = 0;
    bool   ret              = false;
    if (ret = PbConvertor::struct2serializedPb(
            (const char*)&addressBook, "TEST.AddressBook", pSerializedPb, serializedPbSize))
    {
        // 反序列化 - 反序列化为 proto 格式
        TEST::AddressBook address_book;
        ret = address_book.ParseFromArray(pSerializedPb, serializedPbSize);
        std::cout << (ret ? "success" : "fail") << std::endl;

        // 反序列化 - 反序列化为 struct 格式
        PbConvertor::MemTree stru;
        ret = PbConvertor::serializedPb2struct("TEST.AddressBook", pSerializedPb, serializedPbSize, stru);
        AddressBook* addressBookNew = (AddressBook*)stru.pMem;
        std::cout << (ret ? "success" : "fail") << std::endl;
        std::cout << (*addressBookNew);
        // 释放内存
        stru.release();
    }
}

int main()
{
	// C 结构体转 C++ protobuf 序列化
    demo_auto_pbconvertor();

    return 0;
}
```



# 参考

[1] [转换C结构体](https://github.com/hardxuyp/PbConvertor)

