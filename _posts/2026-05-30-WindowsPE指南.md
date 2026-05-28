---
layout: post
title: "WindowsPE指南"
categories: [Windows, PE]
tags: [Windows]
author: August
typora-root-url: ..
---



- [1. PE 文件头](#1-pe-文件头)
  - [1.1. 基本概念](#11-基本概念)
    - [1.1.1. 地址](#111-地址)
    - [1.1.2. 指针](#112-指针)
    - [1.1.3. 数据目录](#113-数据目录)
    - [1.1.4. 节](#114-节)
    - [1.1.5. 对齐](#115-对齐)
    - [1.1.6. Unicode字符串](#116-unicode字符串)
  - [1.2. PE文件结构](#12-pe文件结构)
    - [1.2.1. DOS头](#121-dos头)
    - [1.2.2. PE头](#122-pe头)
      - [1.2.2.1. PE头标识](#1221-pe头标识)
      - [1.2.2.2. 标准PE头](#1222-标准pe头)
      - [1.2.2.3. 扩展PE头](#1223-扩展pe头)
        - [1.2.2.3.1. 数据目录项](#12231-数据目录项)
    - [1.2.3. 节表项](#123-节表项)
- [2. 导入表](#2-导入表)
  - [2.1. 导入表描述符](#21-导入表描述符)
  - [2.2. 延迟加载导入描述符](#22-延迟加载导入描述符)
  - [2.3. 绑定导入表](#23-绑定导入表)
- [3. 导出表](#3-导出表)
  - [3.1. 导出目录](#31-导出目录)
- [4. 栈与重定位表](#4-栈与重定位表)
  - [4.1. 重定位表项](#41-重定位表项)
- [5. 资源表](#5-资源表)
  - [5.1. 资源目录头](#51-资源目录头)
  - [5.2. 资源目录项](#52-资源目录项)
  - [5.3. 资源数据项](#53-资源数据项)
- [6. 线程局部存储](#6-线程局部存储)
  - [6.1. TLS目录结构](#61-tls目录结构)
- [7. 加载配置信息](#7-加载配置信息)
  - [7.1. 加载配置目录](#71-加载配置目录)
- [8. 总结](#8-总结)




该文主要介绍 `Windows PE` 结构。



# WindowsPE指南



## 1. PE 文件头

### 1.1. 基本概念


#### 1.1.1. 地址

虚拟内存地址（Virtual Address, VA）：用户的PE文件被操作系统加载进内存后，PE对应的进程支配了自己独立的虚拟空间；进程本身的VA被解释为：进程的基地址+相对虚拟内存地址；

相对虚拟内存地址（Reverse Virtual Address, RVA）：RVA是相对于模块而言的，VA是相对于整个地址空间而言的。RVA与具体模块相关，它有一个范围，该范围从模块的开始到模块结束，脱离开这个范围的RVA是无效的，称为越界。

![](/media/image/2026-05-30-WindowsPE指南/RVA.png)

文件偏移地址（File Offset Address, FOA）：和内存无关，它是指某个位置距离文件头的偏移。

特殊地址：在PE结构中还有一种特殊地址，其计算方法并不是从文件头算起，也不是从内存的某个模块的基地址算起，而是从某个特定的位置算起。

#### 1.1.2. 指针

PE数据结构中的指针的定义：如果数据结构中某个字段存储的值为一个地址，那么这个字段就是一个指针。

#### 1.1.3. 数据目录

PE中有一个数据结构称为数据目录，其中记录了所有可能的数据类型。这些类型中，目前已定义的有15种，包括导出表、导入表、资源表、异常表、属性证书表、重定位表、调试数据、Architecture、Global Ptr、线程局部存储、加载配置表、绑定导入表、IAT、延迟导入表和CLR运行时头部。

#### 1.1.4. 节

节就是存放不同类型数据（比如代码、数据、常量、资源等）的地方，不同的节具有不同的访问权限。节是PE文件中存放代码或数据的基本单元。

#### 1.1.5. 对齐

PE中规定了三类对齐：数据在内存中的对齐、数据在文件中的对齐、资源文件中资源数据的对齐。

#### 1.1.6. Unicode字符串

Unicode是继ASCII字符编码后的另一种新型字符编码。严格意义上讲，ASCII码的每个字符使用7位表示，Unicode则使用全16位表示一个字符。Unicode字符串中的每个字符均为双字节，所以又称为宽字符串。

### 1.2. PE文件结构

![](/media/image/2026-05-30-WindowsPE指南/PE_program.png)

#### 1.2.1. DOS头

```c++
typedef struct _IMAGE_DOS_HEADER {
    WORD   e_magic;                             // 0000h - EXE标志，“MZ”
    WORD   e_cblp;                              // 0002h - 最后（部分）页中的字节数
    WORD   e_cp;                                // 0004h - 文件中的全部和部分页数
    WORD   e_crlc;                              // 0006h - 重定位表中的指针数
    WORD   e_cparhdr;                           // 0008h - 头部尺寸，以段落为单位
    WORD   e_minalloc;                          // 000ah - 所需的最小附加段
    WORD   e_maxalloc;                          // 000ch - 所需的最大附加段
    WORD   e_ss;                                // 000eh - 初始的SS值（相对偏移量)
    WORD   e_sp;                                // 0010h - 初始的SP值
    WORD   e_csum;                              // 0012h - 补码校验值
    WORD   e_ip;                                // 0014h - 初始的IP值
    WORD   e_cs;                                // 0016h - 初始的Cs值
    WORD   e_lfarlc;                            // 0018h - 重定位表的字节偏移量
    WORD   e_ovno;                              // 001ah - 覆盖号
    WORD   e_res[4];                            // 001ch - 保留字
    WORD   e_oemid;                             // 0024h - OEM标识符（相对e_oeminfo)
    WORD   e_oeminfo;                           // 0026h - OEM信息
    WORD   e_res2[10];                          // 0028h - 保留字
    LONG   e_lfanew;                            // 003ch - PE头相对于文件的偏移地址
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

注释后的偏移是基于 `IMAGE_DOS_HEADER` 头的

#### 1.2.2. PE头

```c++
typedef struct _IMAGE_NT_HEADERS64 {
    DWORD Signature;                            // PE 文件标识， "PE\0\0"
    IMAGE_FILE_HEADER FileHeader;               // PE 标准头
    IMAGE_OPTIONAL_HEADER64 OptionalHeader;     // PE 扩展头
} IMAGE_NT_HEADERS64, *PIMAGE_NT_HEADERS64;
```

##### 1.2.2.1. PE头标识

紧跟在 `DOS Stub` 后面的是PE头标识 `Signature`。与大部分文件格式的头部结构一样，PE 头部信息中有一个四字节的标识，该标识位于指针 `IMAGE_DOS_HEADER.e_lfanew` 指向的位置。其内容固定，对应于 `ASCII` 码的字符串 `PE\0\0`。

##### 1.2.2.2. 标准PE头

标准 PE 头 `IMAGE_FILE_HEADER` 紧跟在 PE 头标识后，即位于 `IMAGE_DOS_HEADER的e_lfanew+4` 的位置。由此位置开始的 20 个字节为数据结构标准 PE 头 `IMAGE_FILE_HEADER` 的内容。该结构在微软的官方文档中被称为标准通用对象文件格式（Common Object File Format，COFF）头。它记录了PE文件的全局属性，如该PE文件运行的平台、PE文件类型（是EXE文件还是DLL文件）、文件中存在的节的总数等

```c++
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;                            // 0004h - 运行平台
    WORD    NumberOfSections;                   // 0006h - PE中节的数量
    DWORD   TimeDateStamp;                      // 0008h - 文件创建日期和时间
    DWORD   PointerToSymbolTable;               // 000ch - 指向符号表（用于调试）
    DWORD   NumberOfSymbols;                    // 0010h - 符号表中的符号数量（用于调试）
    WORD    SizeOfOptionalHeader;               // 0014h - 扩展头结构的长度
    WORD    Characteristics;                    // 0016h - 文件属性
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

注释后的偏移是基于 `IMAGE_NT_HEADERS` 头的

##### 1.2.2.3. 扩展PE头

尽管从名字上看好像该部分数据是可选（optional）的，但在PE文件结构中，它却有着比标准PE头更多的内容，让人感觉似乎它才是真正的PE头。

```c++
typedef struct _IMAGE_OPTIONAL_HEADER64 {
    WORD        Magic;                          // 0018h - 魔术字: 0x10B=32位PE，0x20B=64位PE，0x107=ROM镜像
    BYTE        MajorLinkerVersion;             // 001ah - 链接器主版本号
    BYTE        MinorLinkerVersion;             // 001bh - 链接器次版本号
    DWORD       SizeOfCode;                     // 001ch - 所有含代码的节的总大小
    DWORD       SizeOfInitializedData;          // 0020h - 所有含已初始化数据的节的总大小
    DWORD       SizeOfUninitializedData;        // 0024h - 所有含未初始化数据的节的总大小
    DWORD       AddressOfEntryPoint;            // 0028h - 程序执行入口RVA（相对虚拟地址）
    DWORD       BaseOfCode;                     // 002ch - 代码节的起始RVA
    ULONGLONG   ImageBase;                      // 0030h - 镜像首选加载基址（64位默认0x140000000）
    DWORD       SectionAlignment;               // 0038h - 内存中节的对齐粒度（典型0x1000=4KB）
    DWORD       FileAlignment;                  // 003ch - 文件中节的对齐粒度（典型0x200=512字节）
    WORD        MajorOperatingSystemVersion;    // 0040h - 所需操作系统主版本号
    WORD        MinorOperatingSystemVersion;    // 0042h - 所需操作系统次版本号
    WORD        MajorImageVersion;              // 0044h - 镜像主版本号（由开发者定义）
    WORD        MinorImageVersion;              // 0046h - 镜像次版本号（由开发者定义）
    WORD        MajorSubsystemVersion;          // 0048h - 所需子系统主版本号
    WORD        MinorSubsystemVersion;          // 004ah - 所需子系统次版本号
    DWORD       Win32VersionValue;              // 004ch - Win32版本值（保留，通常为0）
    DWORD       SizeOfImage;                    // 0050h - 内存中整个镜像的总大小（按SectionAlignment对齐）
    DWORD       SizeOfHeaders;                  // 0054h - 所有头部（DOS头+NT头+节表）按FileAlignment对齐后的大小
    DWORD       CheckSum;                       // 0058h - 校验和（用于驱动DLL和系统关键文件验证）
    WORD        Subsystem;                      // 005ch - 子系统类型（如2=GUI，3=控制台）
    WORD        DllCharacteristics;             // 005eh - DLL特征标志（如ASLR、DEP、NX兼容等）
    ULONGLONG   SizeOfStackReserve;             // 0060h - 线程栈保留大小（虚拟地址空间，非实际分配）
    ULONGLONG   SizeOfStackCommit;              // 0068h - 线程栈初始提交大小（实际分配的物理内存）
    ULONGLONG   SizeOfHeapReserve;              // 0070h - 堆保留大小（虚拟地址空间）
    ULONGLONG   SizeOfHeapCommit;               // 0078h - 堆初始提交大小
    DWORD       LoaderFlags;                    // 0080h - 加载器标志（已弃用，通常为0）
    DWORD       NumberOfRvaAndSizes;            // 0084h - 数据目录项数量（通常为16）
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]; // 0088h - 数据目录表（导入/导出/资源/重定位等）
} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;
```

注释后的偏移是基于 `IMAGE_NT_HEADERS` 头的。

###### 1.2.2.3.1. 数据目录项

该字段定义了PE文件中出现的所有不同类型的数据的目录信息。如前所述，应用程序中的数据被按照用途分成很多种类，如导出表、导入表、资源、重定位表等。在内存中，这些数据被操作系统以页为单位组织起来，并赋以不同的访问属性；在文件中，这些数据也同样被组织起来，按照不同类别分别存放在文件的指定位置。该结构就是用来描述这些不同类别的数据在文件（和内存）中的位置及大小的。

```c++
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;                     // 0000h - 数据的起始RVA
    DWORD   Size;                               // 0004h - 数据块的长度
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

| 编号 | 英文名称                               | 中文描述                                                                       |
| :--- | :------------------------------------- | :----------------------------------------------------------------------------- |
| 0    | `IMAGE_DIRECTORY_ENTRY_EXPORT`         | 导出表 - 存储 DLL/EXE 导出的函数、变量、类等信息                               |
| 1    | `IMAGE_DIRECTORY_ENTRY_IMPORT`         | 导入表 - 记录本模块需要调用的外部函数信息                                      |
| 2    | `IMAGE_DIRECTORY_ENTRY_RESOURCE`       | 资源表 - 存储图标、菜单、对话框、字符串等资源数据                              |
| 3    | `IMAGE_DIRECTORY_ENTRY_EXCEPTION`      | 异常处理表 - 用于结构化异常处理（SEH），x64/ARM 平台的展开信息                 |
| 4    | `IMAGE_DIRECTORY_ENTRY_SECURITY`       | 安全证书目录 - 存储数字签名和证书信息（不映射到内存，仅存在于文件中）          |
| 5    | `IMAGE_DIRECTORY_ENTRY_BASERELOC`      | 基址重定位表 - 当首选加载地址被占用时，需要修复的地址偏移列表                  |
| 6    | `IMAGE_DIRECTORY_ENTRY_DEBUG`          | 调试信息表 - 存储调试符号、PDB 文件路径等调试数据                              |
| 7    | `IMAGE_DIRECTORY_ENTRY_ARCHITECTURE`   | 架构保留表 - 已弃用，保留为 0                                                  |
| 8    | `IMAGE_DIRECTORY_ENTRY_GLOBALPTR`      | 全局指针表 - 用于 RISC 平台的全局指针相对寻址（MIPS/Alpha 架构）               |
| 9    | `IMAGE_DIRECTORY_ENTRY_TLS`            | TLS 表 - 线程局部存储，用于多线程环境下每线程独立的全局变量                    |
| 10   | `IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG`    | 加载配置表 - 包含安全特性配置（如 DEP、SEH、控制流保护 CFG、动态安全初始化等） |
| 11   | `IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT`   | 绑定导入表 - 预绑定的导入表，用于加速 DLL 加载（已较少使用）                   |
| 12   | `IMAGE_DIRECTORY_ENTRY_IAT`            | 导入地址表 - 导入函数实际地址表，由加载器填充                                  |
| 13   | `IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT`   | 延迟导入表 - 延迟加载的 DLL 信息，只在首次调用时才真正加载                     |
| 14   | `IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR` | COM+ 描述符 - 用于 .NET / CLI 元数据描述符（CLR 头）                           |
| 15   | `IMAGE_DIRECTORY_ENTRY_RESERVED`       | 保留项 - 未使用，值为 0                                                        |

#### 1.2.3. 节表项

每个节表项记录了PE中与某个特定的节有关的信息，如节的属性、节的大小、在文件和内存中的起始位置等。

```c++
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];      // 节名称（8字节，ANSI字符串，不足补0，不必以\0结尾）".text", ".data", ".rdata", ".rsrc"
    union {
        DWORD   PhysicalAddress;                // 物理地址（仅用于目标文件.obj，不用于PE）
        DWORD   VirtualSize;                    // 未对齐前的实际大小（内存中节的有效数据大小）
    } Misc;
    DWORD   VirtualAddress;                     // 节在内存中的RVA（相对虚拟地址，按SectionAlignment对齐）
    DWORD   SizeOfRawData;                      // 节在磁盘文件中的大小（按FileAlignment对齐，可能是VirtualSize的向上取整）
    DWORD   PointerToRawData;                   // 节在磁盘文件中的偏移量（相对于文件开头）
    DWORD   PointerToRelocations;               // 重定位表在文件中的偏移（仅用于.obj目标文件，PE文件中为0）
    DWORD   PointerToLinenumbers;               // 行号表在文件中的偏移（用于调试信息，PE中很少使用）
    WORD    NumberOfRelocations;                // 重定位表条目数量（仅用于.obj目标文件）
    WORD    NumberOfLinenumbers;                // 行号表条目数量（用于调试信息）
    DWORD   Characteristics;                    // 节属性标志位（描述该节的内存属性：可读/可写/可执行等）
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

注释后的偏移是基于 `IMAGE_SECTION_HEADER` 头的



## 2. 导入表

### 2.1. 导入表描述符

```c++
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;                // 0 表示此为结束标记（最后一个导入描述符）
        DWORD   OriginalFirstThunk;             // RVA，指向原始的未绑定的IAT（即 INT，导入名称表）
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp;                      // 时间戳，用于绑定导入（Bound Import）
    DWORD   ForwarderChain;                     // 转发链索引
    DWORD   Name;                               // RVA，指向以 '\0' 结尾的 DLL 名称字符串
    DWORD   FirstThunk;                         // RVA，指向 IAT（导入地址表）
} IMAGE_IMPORT_DESCRIPTOR, *PIMAGE_IMPORT_DESCRIPTOR;

typedef struct _IMAGE_THUNK_DATA64 {
    union {
        ULONGLONG ForwarderString;              // 转发器字符串的RVA（指向一个ASCII字符串）
        ULONGLONG Function;                     // 函数的内存地址（运行时填充）
        ULONGLONG Ordinal;                      // 导入函数的序号（按序号导入）
        ULONGLONG AddressOfData;                // 指向导入函数名称信息的RVA（按名称导入）
    } u1;
} IMAGE_THUNK_DATA64, *PIMAGE_THUNK_DATA64;
```

### 2.2. 延迟加载导入描述符

```c++
typedef struct _IMAGE_DELAYLOAD_DESCRIPTOR {
    union {
        DWORD AllAttributes;                // 所有属性标志的组合值
        struct {
            DWORD RvaBased : 1;             // 延迟导入版本标识（1表示版本2）
            DWORD ReservedAttributes : 31;  // 保留属性位（必须为0）
        } DUMMYSTRUCTNAME;
    } Attributes;
    DWORD DllNameRVA;                       // RVA，指向目标DLL名称字符串（以NULL结尾的ASCII）
    DWORD ModuleHandleRVA;                  // RVA，指向缓存DLL模块句柄的变量（PHMODULE）
    DWORD ImportAddressTableRVA;            // RVA，指向延迟导入IAT起始位置（PIMAGE_THUNK_DATA）
    DWORD ImportNameTableRVA;               // RVA，指向导入名称表起始位置（指向函数名称RVA）
    DWORD BoundImportAddressTableRVA;       // RVA，指向可选的绑定导入地址表
    DWORD UnloadInformationTableRVA;        // RVA，指向可选的卸载信息表
    DWORD TimeDateStamp;                    // 时间戳（0=未绑定，否则为目标DLL的日期/时间）
} IMAGE_DELAYLOAD_DESCRIPTOR, *PIMAGE_DELAYLOAD_DESCRIPTOR;
```

### 2.3. 绑定导入表

```c++
typedef struct _IMAGE_BOUND_IMPORT_DESCRIPTOR {
    DWORD   TimeDateStamp;                      // 绑定时所依赖 DLL 的时间戳
    WORD    OffsetModuleName;                   // 模块名称的偏移量（相对于本结构起始位置）
    WORD    NumberOfModuleForwarderRefs;        // 转发器引用的数量
} IMAGE_BOUND_IMPORT_DESCRIPTOR, *PIMAGE_BOUND_IMPORT_DESCRIPTOR;

typedef struct _IMAGE_BOUND_FORWARDER_REF {
    DWORD   TimeDateStamp;                      // 转发目标 DLL 的时间戳
    WORD    OffsetModuleName;                   // 转发目标 DLL 名称的偏移量（相对于主描述符起始）
    WORD    Reserved;                           // 保留字段（未使用，通常为 0）
} IMAGE_BOUND_FORWARDER_REF, *PIMAGE_BOUND_FORWARDER_REF;
```



## 3. 导出表

导出表的主要作用是将PE中存在的函数引出到外部，以便其他人可以使用这些函数，实现代码的重用。

### 3.1. 导出目录

```c++
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD   Characteristics;                    // 导出表属性标志（通常为0，保留未用）
    DWORD   TimeDateStamp;                      // 导出表创建时间戳（用于版本校验）
    WORD    MajorVersion;                       // 导出表主版本号
    WORD    MinorVersion;                       // 导出表次版本号
    DWORD   Name;                               // RVA，指向DLL名称字符串（如"kernel32.dll"）
    DWORD   Base;                               // 导出函数序号的起始值（序号=Base+索引）
    DWORD   NumberOfFunctions;                  // 导出函数总数（AddressOfFunctions数组元素个数）
    DWORD   NumberOfNames;                      // 以名称导出的函数数量（AddressOfNames数组元素个数）
    DWORD   AddressOfFunctions;                 // RVA，指向导出函数地址数组（RVA数组）
    DWORD   AddressOfNames;                     // RVA，指向导出函数名称数组（字符串RVA数组）
    DWORD   AddressOfNameOrdinals;              // RVA，指向导出序号数组（与AddressOfNames一一对应）
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```



## 4. 栈与重定位表

### 4.1. 重定位表项

```c++
typedef struct _IMAGE_BASE_RELOCATION {
    DWORD   VirtualAddress;                     // 本重定位块所应用的内存页起始RVA（按页对齐）
    DWORD   SizeOfBlock;                        // 当前重定位块的总字节数（包含本结构体大小）
//  WORD    TypeOffset[1];                      // 可变长数组，每项高4位表示重定位类型，低12位表示偏移量
} IMAGE_BASE_RELOCATION;
```



## 5. 资源表

在程序设计中，总会涉及一些数据。这些数据可能是源代码内部需要用到的常量，比如菜单选项、界面描述等；也可能是源代码外部的，比如程序的图标文件、背景音乐文件、配置文件等，以上这些数据统称为资源。

### 5.1. 资源目录头

```c++
typedef struct _IMAGE_RESOURCE_DIRECTORY {
    DWORD   Characteristics;                    // 资源属性标志（通常为0，保留未用）
    DWORD   TimeDateStamp;                      // 资源数据的时间戳
    WORD    MajorVersion;                       // 资源目录主版本号
    WORD    MinorVersion;                       // 资源目录次版本号
    WORD    NumberOfNamedEntries;               // 使用名称标识的子目录项数量
    WORD    NumberOfIdEntries;                  // 使用数字ID标识的子目录项数量
//  IMAGE_RESOURCE_DIRECTORY_ENTRY DirectoryEntries[];  // 紧随其后的目录项数组
} IMAGE_RESOURCE_DIRECTORY, *PIMAGE_RESOURCE_DIRECTORY;
```

### 5.2. 资源目录项

```c++
typedef struct _IMAGE_RESOURCE_DIRECTORY_ENTRY {
    union {
        struct {
            DWORD NameOffset:31;                // 名称字符串偏移量（相对于资源目录起始）
            DWORD NameIsString:1;               // 名称类型标志（1=使用字符串名称，0=使用整数ID）
        } DUMMYSTRUCTNAME;
        DWORD   Name;                           // 资源名称的RVA（高位标志决定解释方式）
        WORD    Id;                             // 资源整数ID（当高位标志为0时使用）
    } DUMMYUNIONNAME;
    union {
        DWORD   OffsetToData;                   // RVA，指向资源数据（当DataIsDirectory=0时）
        struct {
            DWORD   OffsetToDirectory:31;       // 子目录偏移量（相对于资源目录起始）
            DWORD   DataIsDirectory:1;          // 目录标志（1=指向子目录，0=指向资源数据）
        } DUMMYSTRUCTNAME2;
    } DUMMYUNIONNAME2;
} IMAGE_RESOURCE_DIRECTORY_ENTRY, *PIMAGE_RESOURCE_DIRECTORY_ENTRY;
```

### 5.3. 资源数据项

```c++
typedef struct _IMAGE_RESOURCE_DATA_ENTRY {
    DWORD   OffsetToData;                       // 资源原始数据相对于文件起始的RVA
    DWORD   Size;                               // 资源数据的字节大小
    DWORD   CodePage;                           // 资源的代码页（用于字符串等文本资源，通常为0）
    DWORD   Reserved;                           // 保留字段（必须为0）
} IMAGE_RESOURCE_DATA_ENTRY, *PIMAGE_RESOURCE_DATA_ENTRY;
```



## 6. 线程局部存储

### 6.1. TLS目录结构

```c++
typedef struct _IMAGE_TLS_DIRECTORY64 {
    ULONGLONG StartAddressOfRawData;            // TLS模板数据起始的RVA（内存中）
    ULONGLONG EndAddressOfRawData;              // TLS模板数据结束的RVA（内存中）
    ULONGLONG AddressOfIndex;                   // RVA，指向TLS索引变量（DWORD）
    ULONGLONG AddressOfCallBacks;               // RVA，指向TLS回调函数指针数组（以NULL结尾）
    DWORD SizeOfZeroFill;                       // 初始化数据后填充零的大小（字节）
    union {
        DWORD Characteristics;                  // TLS目录特性标志
        struct {
            DWORD Reserved0 : 20;               // 保留位（必须为0）
            DWORD Alignment : 4;                // TLS数据对齐方式（2的幂次）
            DWORD Reserved1 : 8;                // 保留位（必须为0）
        } DUMMYSTRUCTNAME;
    } DUMMYUNIONNAME;

} IMAGE_TLS_DIRECTORY64, *PIMAGE_TLS_DIRECTORY64;
```



## 7. 加载配置信息

### 7.1. 加载配置目录

```c++
typedef struct _IMAGE_LOAD_CONFIG_DIRECTORY64 {
    DWORD      Size;                                        // 结构体总大小（用于版本识别）
    DWORD      TimeDateStamp;                               // 时间戳（通常为0）
    WORD       MajorVersion;                                // 主版本号
    WORD       MinorVersion;                                // 次版本号
    DWORD      GlobalFlagsClear;                            // 运行时需要清零的全局标志
    DWORD      GlobalFlagsSet;                              // 运行时需要设置的全局标志
    DWORD      CriticalSectionDefaultTimeout;               // 临界区默认超时时间
    ULONGLONG  DeCommitFreeBlockThreshold;                  // 释放内存块的解除提交阈值
    ULONGLONG  DeCommitTotalFreeThreshold;                  // 总释放内存的解除提交阈值
    ULONGLONG  LockPrefixTable;                             // VA，指向锁定前缀表（用于原子操作优化）
    ULONGLONG  MaximumAllocationSize;                       // 最大单次内存分配大小
    ULONGLONG  VirtualMemoryThreshold;                      // 虚拟内存阈值
    ULONGLONG  ProcessAffinityMask;                         // 进程CPU亲和性掩码
    DWORD      ProcessHeapFlags;                            // 进程堆标志
    WORD       CSDVersion;                                  // Service Pack版本号
    WORD       DependentLoadFlags;                          // 依赖项加载标志
    ULONGLONG  EditList;                                    // VA，指向编辑列表
    ULONGLONG  SecurityCookie;                              // VA，指向安全Cookie（/GS保护）
    ULONGLONG  SEHandlerTable;                              // VA，指向SEH异常处理表
    ULONGLONG  SEHandlerCount;                              // SEH异常处理表条目数量
    ULONGLONG  GuardCFCheckFunctionPointer;                 // VA，指向控制流检查函数（/guard:cf）
    ULONGLONG  GuardCFDispatchFunctionPointer;              // VA，指向控制流分发函数
    ULONGLONG  GuardCFFunctionTable;                        // VA，指向CFG有效函数地址表
    ULONGLONG  GuardCFFunctionCount;                        // CFG函数表条目数量
    DWORD      GuardFlags;                                  // CFG标志位
    IMAGE_LOAD_CONFIG_CODE_INTEGRITY CodeIntegrity;         // 代码完整性验证信息
    ULONGLONG  GuardAddressTakenIatEntryTable;              // VA，指向地址被获取的IAT条目表
    ULONGLONG  GuardAddressTakenIatEntryCount;              // 地址被获取的IAT条目数量
    ULONGLONG  GuardLongJumpTargetTable;                    // VA，指向longjmp目标地址表
    ULONGLONG  GuardLongJumpTargetCount;                    // longjmp目标表条目数量
    ULONGLONG  DynamicValueRelocTable;                      // VA，指向动态值重定位表
    ULONGLONG  CHPEMetadataPointer;                         // VA，指向CHPE元数据
    ULONGLONG  GuardRFFailureRoutine;                       // VA，指向返回地址失败处理函数
    ULONGLONG  GuardRFFailureRoutineFunctionPointer;        // VA，指向返回地址失败函数指针
    DWORD      DynamicValueRelocTableOffset;                // 动态值重定位表偏移量
    WORD       DynamicValueRelocTableSection;               // 动态值重定位表所在节索引
    WORD       Reserved2;                                   // 保留字段2
    ULONGLONG  GuardRFVerifyStackPointerFunctionPointer;    // VA，指向栈指针验证函数指针
    DWORD      HotPatchTableOffset;                         // 热补丁表偏移量
    DWORD      Reserved3;                                   // 保留字段3
    ULONGLONG  EnclaveConfigurationPointer;                 // VA，指向Enclave配置（VBS/CFG）
    ULONGLONG  VolatileMetadataPointer;                     // VA，指向易变元数据
    ULONGLONG  GuardEHContinuationTable;                    // VA，指向异常处理延续表
    ULONGLONG  GuardEHContinuationCount;                    // 异常处理延续表条目数量
    ULONGLONG  GuardXFGCheckFunctionPointer;                // VA，指向XFGC检查函数指针
    ULONGLONG  GuardXFGDispatchFunctionPointer;             // VA，指向XFGC分发函数指针
    ULONGLONG  GuardXFGTableDispatchFunctionPointer;        // VA，指向XFGC表分发函数指针
    ULONGLONG  CastGuardOsDeterminedFailureMode;            // VA，指向CastGuard失败处理模式
    ULONGLONG  GuardMemcpyFunctionPointer;                  // VA，指向安全memcpy函数指针
    ULONGLONG  UmaFunctionPointers;                         // VA，指向统一内存访问函数指针表
} IMAGE_LOAD_CONFIG_DIRECTORY64, *PIMAGE_LOAD_CONFIG_DIRECTORY64;
```



## 8. 总结

图上使用的是 32 位记录，目前是 64 位，部分结构体存在区别，以最新的为准。

![](/media/image/2026-05-30-WindowsPE指南/PE_0.jpg)

![](/media/image/2026-05-30-WindowsPE指南/PE_1.png)



# 参考
