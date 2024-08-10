---
layout: post
title: "boost-json"
categories: C/C++
tags: boost
author: August
typora-root-url: ..
---

* content
{:toc}
本文讲述使用 `boost` 解析和创建 `json` 格式。



# boost-json

`boost` 是 `C++` 委员会创建的库，优秀的库将加入 `C++` 标准，十分适合 `C++` 工程开发。

- 官方网址  [https://www.boost.org/](https://www.boost.org/)
- 开源网址  [https://github.com/boostorg/boost](https://github.com/boostorg/boost)



## 1 介绍

`Boost` 就已经有能够解析 `JSON` 的库了，名字叫做 `Boost.PropertyTree`。`Boost.PropertyTree` 不仅仅能够解析 `JSON`，还能解析 `XML`，`INI` 和 `INFO` 格式的文件。但是由于成文较早及需要兼容其他的数据格式，相比较于其他的 `C++` 解析库，其显得比较笨重，使用的时候有很多的不方便。

`Boost.JSON` 相对于 `Boost.PropertyTree` 来说，其只能支持 `JSON` 格式的解析，但是其使用方法更为简便。

| 名称               | 最低支持版本 | 参考                                                                |
| ------------------ | ------------ | ------------------------------------------------------------------- |
| Boost.PropertyTree | V1.41.0      | https://www.boost.org/doc/libs/1_41_0/doc/html/property_tree.html   |
| Boost.JSON         | V1.75.0      | https://www.boost.org/doc/libs/1_75_0/libs/json/doc/html/index.html |



## 2 代码编写

### 2.1 说明

#### 2.1.1 Boost.PropertyTree

解析 `json` 很简单，命名空间为 `boost::property_tree`，`reson_json` 函数将文件流、字符串解析到 `ptree`，`write_json` 将 `ptree` 输出为字符串或文件流。其余的都是对 `ptree` 的操作。

解析json需要加头文件：

```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
```

#### 2.1.2 Boost.JSON

有两种方法使用 `Boost.JSON`，一种是动态链接库，此时引入头文件 `boost/json.hpp`，同时链接对应的动态库；第二种是使用 `header only` 模式，此时只需要引入头文件 `boost/json/src.hpp` 即可。两种方法各有优缺点，酌情使用。

### 2.2 测试代码

#### 2.2.1 BoostTest.h

```c++
#ifndef __BOOSTTEST_H__
#define __BOOSTTEST_H__

#include <iostream>
#include <sstream>
#include <utility>
#include <fstream>

#include <boost/property_tree/ptree.hpp>		// boost1.41.0
#include <boost/property_tree/json_parser.hpp>	// boost1.41.0
#include <boost/json.hpp>						// boost1.75.0

class Boost1_41_0
{
public:
	Boost1_41_0() {}
	~Boost1_41_0() {}

	static void ParseJsonString(std::string str)
	{
		std::stringstream ss(str);
		boost::property_tree::ptree pt;
		// std::stringstream 就是字符串解析
		boost::property_tree::read_json(ss, pt);

		// put 修改或增加一个 key-value，key不存在则增加
		pt.put("age", 20);

		// get 读取
		std::string name = pt.get<std::string>("name");
		int age = pt.get<int>("age");
		boost::property_tree::ptree addr = pt.get_child("addr");
		std::string country = addr.get<std::string>("country");
		std::string address = addr.get<std::string>("address");
		boost::property_tree::ptree skills = pt.get_child("skills");

		// 打印
		std::cout << "name  = " << name << std::endl;
		std::cout << "age   = " << age << std::endl;
		std::cout << "addr  = " << country << " " << address << std::endl;
		std::cout << "skills= [";
		for (auto & skill : skills)
		{
			std::cout << skill.second.get_value<std::string>() << ",";
		}
		std::cout << "\b]" << std::endl;
	}

	static void ParseJsonFile()
	{
		boost::property_tree::ptree pt;
		// std::string 就是文件解析
		boost::property_tree::read_json("read.json", pt);

		std::string name = pt.get<std::string>("name");
		int age = pt.get<int>("age");
		boost::property_tree::ptree addr = pt.get_child("addr");
		std::string country = addr.get<std::string>("country");
		std::string address = addr.get<std::string>("address");
		boost::property_tree::ptree skills = pt.get_child("skills");

		std::cout << "name  = " << name << std::endl;
		std::cout << "age   = " << age << std::endl;
		std::cout << "addr  = " << country << " " << address << std::endl;
		std::cout << "skills= [";
		for (auto & skill : skills)
		{
			std::cout << skill.second.get_value<std::string>() << ",";
		}
		std::cout << "\b]" << std::endl;
	}

	static void CreateJson()
	{
		// add 添加节点
		boost::property_tree::ptree ptRoot;
		ptRoot.add("name", "张三");
		ptRoot.add("age", 18);

		boost::property_tree::ptree ptAddr;
		ptAddr.add("country", "中国");
		ptAddr.add("address", "四川");
		ptRoot.add_child("addr", ptAddr);

		// 数组其实是空的 key
		boost::property_tree::ptree ptSkills;
		ptSkills.push_back(std::make_pair("", boost::property_tree::ptree("C++")));
		ptSkills.push_back(std::make_pair("", boost::property_tree::ptree("Java")));
		ptSkills.push_back(std::make_pair("", boost::property_tree::ptree("Python")));
		ptRoot.add_child("skills", ptSkills);

		// 获取字符串
		std::stringstream ss;
		boost::property_tree::write_json(ss, ptRoot);
		std::cout << ss.str() << std::endl;

		// 写入文件
		boost::property_tree::write_json("write.json", ptRoot);
	}
};

class Boost1_75_0
{
public:
	Boost1_75_0() {}
	~Boost1_75_0() {}

	static void ParseJsonString(std::string str)
	{
		boost::json::object jsonValue = boost::json::parse(str).as_object();

		// 修改值
		jsonValue["age"] = 20;
		jsonValue["addr"].as_object()["address"] = std::string("重庆");

		// at 读取并指定类型
		std::string name = jsonValue.at("name").as_string().c_str();
		int age = jsonValue.at("age").as_int64();
		
		boost::json::value addr = jsonValue.at("addr");
		std::string country = addr.at("country").as_string().c_str();
		std::string address = addr.at("address").as_string().c_str();
		//address = boost::locale::conv::from_utf(address.c_str(), std::string("gb2312"));
		boost::json::value skills = jsonValue.at("skills");

		std::cout << "name  = " << name << std::endl;
		std::cout << "age   = " << age << std::endl;
		std::cout << "addr  = " << country << " " << address << std::endl;
		std::cout << "skills= [";
		for (auto & skill : skills.as_array())
		{
			std::cout << skill.as_string() << ",";
		}
		std::cout << "\b]" << std::endl;
	}

	static void ParseJsonFile()
	{
		// 没有提供直接读取文件，可以使用流方式读取转字符串
		std::ifstream inFile("read.json");
		std::stringstream ss;
		ss << inFile.rdbuf();
		boost::json::value jsonValue = boost::json::parse(ss.str());

		// at 读取并指定类型
		std::string name = jsonValue.at("name").as_string().c_str();
		int age = jsonValue.at("age").as_int64();

		boost::json::value addr = jsonValue.at("addr");
		std::string country = addr.at("country").as_string().c_str();
		std::string address = addr.at("address").as_string().c_str();
		boost::json::value skills = jsonValue.at("skills");

		std::cout << "name  = " << name << std::endl;
		std::cout << "age   = " << age << std::endl;
		std::cout << "addr  = " << country << " " << address << std::endl;
		std::cout << "skills= [";
		for (auto & skill : skills.as_array())
		{
			std::cout << skill.as_string() << ",";
		}
		std::cout << "\b]" << std::endl;
	}

	static void CreateJson()
	{
		boost::json::object jsonRoot;
		
		// 直接放入
		jsonRoot["name"] = "张三";
		jsonRoot["age"] = 18;
		// 构造放入
		boost::json::object jsonAddr;
		jsonAddr["country"] = "中国";
		jsonAddr["address"] = "四川";
		jsonRoot.emplace("addr", jsonAddr);
		jsonRoot.emplace("skills", boost::json::array({"C++","Java", "Python"}));
		
		std::string jsonStr = boost::json::serialize(jsonRoot);
		std::cout << jsonStr << std::endl;
	}
};

#endif
```

#### 2.2.2 main.cpp

```cpp
#ifdef _WIN32  
#pragma execution_character_set("utf-8")  // 文件编码 utf8
#endif

#include <iostream>

#include "BoostTest.h"

std::string str = "{\
		\"name\": \"张三\",\
		\"age\" : 18,\
		\"addr\" : {\
		    \"country\": \"中国\",\
			\"address\" : \"四川\"\
		},\
		\"skills\" : [\"C++\", \"Java\", \"Python\"]\
	}";

int main(int argc, char** argv)
{
	system("chcp 65001");	// 控制台输出 utf8

	Boost1_41_0::ParseJsonString(str);
	Boost1_41_0::ParseJsonFile();
	Boost1_41_0::CreateJson();

	Boost1_75_0::ParseJsonString(str);
	Boost1_75_0::ParseJsonFile();
	Boost1_75_0::CreateJson();

	return 0;
}
```



## 3 注意事项

### 3.1 boost::property_tree

1. 用 `boost::property_tree` 解析字符串遇到 `"\/"` 时解析失败，而 `jsoncpp` 可以解析成功，要知道 `'/'` 前面加一个 `'\'` 是 `JSON` 标准格式。

2. `boost::property_tree` 的 `read_json` 和 `write_json` 在多线程中使用会引起崩溃。

针对1，可以在使用 `boost::property_tree` 解析前写个函数去掉 `"\/"` 中的 `'\'`；

针对2，在多线程中同步一下可以解决。
