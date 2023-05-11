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



## 第1章 类型推导

C++98有一套类型推导的规则：用于函数模板的规则。C++11修改了其中 的 一 些 规 则 并 增 加 了 两 套 规 则 ， 一 套 用 于 auto ， 一 套 用 于decltype。C++14扩展了auto和decltype可能使用的范围。

### 条款一：理解模板类型推导

- 在模板类型推导时，有引用的实参会被视为无引用，他们的引用会被忽略
- 对于通用引用的推导，左值实参会被特殊对待
- 对于传值类型推导，const和/或volatile实参会被认为是nonconst的和non-volatile的
- 在模板类型推导时，数组名或者函数名实参会退化为指针，除非它们被用于初始化引用











