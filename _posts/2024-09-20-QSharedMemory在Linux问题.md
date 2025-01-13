---
layout: post
title: "QSharedMemory在Linux问题"
categories: Qt
tags: Qt
author: August
typora-root-url: ..
---


- [1. 问题](#1-问题)
- [2. 原因](#2-原因)
- [3. 解决方案](#3-解决方案)



该文是介绍 `QSharedMemory` 在 `Linux` 问题。



# QSharedMemory在Linux问题



## 1. 问题

为了实现只运行一个 `qt` 程序，采用 `QSharedMemory` 共享内存判断是否已经有程序正在运行，存在则不开启新程序。

```cpp
QSharedMemory shared("XXXXX_QSharedMemory_Running");
if (shared.attach())
{
    return 0;
}
shared.create(1);
```



## 2. 原因

`QSharedMemory` 提供了被多线程和多进程共享的一段内存的访问。它也提供了方法，为单线程或单进程锁定内存以实现互斥访问。

当使用这个类的时候，要知道以下不同平台的差异[^1]：

- Windows系统：`QSharedMemory` 并不“拥有”这段共享内存。当所有“拥有一个 `QSharedMemory` 的实例从而附着于某一段共享内存”的进程或线程销毁了它们的 `QSharedMemory` 实例或者退出了，`Windows` 内核会自动释放这一段共享内存。
- Unix系统：`QSharedMemory` “拥有”这段共享内存。当最后一个拥有附着于某段共享内存的线程或进程通过销毁 `QSharedMemory` 实例从而与这段共享内存分离后，`Unix` 内核会释放这段共享内存。但是如果最后这个线程或进程崩溃了而没有运行 `QSharedMemory` 的析构函数时，这段共享内存会逃过本次崩溃而没有被释放。



## 3. 解决方案

先 `create`，如果失败了，然后根据错误码判断已经存在，然后 `attach` 一次，将当前进程附着在这个共享内存上，然后 `detach`。最后再创建，如果还是失败，说明进程确实在运行。

```cpp
QSharedMemory shared("XXXXX_QSharedMemory_Running");    
if (!shared.create(1))
{
    printf("[warn ]: %s\n", shared.errorString().toStdString().c_str());
    if (shared.error() == QSharedMemory::AlreadyExists)
    {
        shared.attach();
        shared.detach();
        if (!shared.create(1))
        {
            printf("[error]: %s\n", shared.errorString().toStdString().c_str());
            return 0;
        }
    }
}
printf("[info ]: app start\n");
```



# 参考

[^1]: [QSharedMemory在linux下异常崩溃导致的bug](https://blog.csdn.net/c1s2d3n4cs/article/details/129409954)
