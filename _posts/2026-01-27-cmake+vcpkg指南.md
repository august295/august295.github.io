---
layout: post
title: "cmake+vcpkg指南"
categories: [Tool, vcpkg]
tags: [vcpkg]
author: August
typora-root-url: ..
---

* content
{:toc}

本文介绍 `cmake` 和 `vcpkg` 有网情况下管理项目。



# cmake+vcpkg 指南

## 1. cmake

从 `CMake3.15` 版本开始普及 `Toolchain`，其中 `Toolchain = 告诉 CMake 用什么编译器 + 为哪个平台 + 怎么编`，存在以下功能：

- 跨平台 / 交叉编译
- 包管理器接管
- 构建结果可复现

`vcpkg` 可以使用工具链做以下事情：

- 决定 triplet（x64-windows / x64-windows-static / x64-linux）
- 决定 CRT / linkage
- 把安装目录塞进 CMake 搜索路径

这些必须在 `project()` 之前生效，最好安装使用 `CMake3.15+` 版本

## 2. vcpkg

vcpkg介绍请看：[vcpkg指南](2022-03-27-vcpkg指南.md)

## 3. cmake + vcpkg[^1]

`C/C++` 外部引入三方库一直比较麻烦，`cmake` 自身也有下载三方库机制，但是目前不能获取依赖

### 3.1. find_package

这个需要系统安装或者自己手动编译且包含 `cmake` 配置

```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject)

# 查找 Boost，要求至少 1.82
find_package(Boost 1.82 REQUIRED COMPONENTS filesystem system)

add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE Boost::filesystem)
```

### 3.2. FetchContent

对于小型库比较友好

```cmake
include(FetchContent)

# 拉取 spdlog 版本 1.12.0
FetchContent_Declare(
    spdlog
    GIT_REPOSITORY https://github.com/gabime/spdlog.git
    GIT_TAG v1.12.0
)
FetchContent_MakeAvailable(spdlog)

add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE spdlog::spdlog)
```

### 3.3. ExternalProject_Add

```cmake
include(ExternalProject)

ExternalProject_Add(fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 10.1.0
    PREFIX ${CMAKE_BINARY_DIR}/external
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)

# 使用导入库时要额外处理 include/link
```

### 3.4. vcpkg

从 `2024` 年后的 `vcpkg` 可以更多的定制化，基本满足版本管理便捷、`CMake` 快速集成

示例代码可以参考： [EasyPluginFramework](https://github.com/august295/EasyPluginFramework)

目前 `vcpkg` 可以通过 `vcpkg.josn` 指定安装三方库和版本，最主要是会自动查找和编译依赖库

#### 3.4.1. vcpkg.json

在 `cmake` 目录下编写 `vcpkg.json` 清单文件，然后可以在 `CMakeLists.txt` 指定目录，这里以 `openssl` 和 `sqlite3` 库作为示例，在配置时当版本输入不对时，还会列出所有可选择版本

```json
{
    "dependencies": [
        "openssl",
        "sqlite3"
    ],
    "builtin-baseline": "6f29f12e82a8293156836ad81cc9bf5af41fe836",
    "overrides": [
        {
            "name": "openssl",
            "version": "3.5.4"
        },
        {
            "name": "sqlite3",
            "version": "3.50.4"
        }
    ]
}
```

#### 3.4.2. module_vcpkg.cmake

> 如果在 CMAKE_TOOLCHAIN_FILE 文件中设置 CMakeList.txt，请确保在调用 project() 之前设置变量。

在 `cmake` 目录下编写 `module_vcpkg.cmake` 文件，并在 `project()` 前使用

```cmake
# 设置 vcpkg 配置
if(CMAKE_HOST_SYSTEM_NAME MATCHES "Windows")
    if(NOT DEFINED VCPKG_ROOT)
        if(DEFINED ENV{VCPKG_ROOT})
            set(VCPKG_ROOT "$ENV{VCPKG_ROOT}")
        else()
            set(VCPKG_ROOT "C:/dev/vcpkg")
        endif()
    endif()

    # 设置工具链文件
    set(CMAKE_TOOLCHAIN_FILE "${VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")
    set(VCPKG_TARGET_TRIPLET "x64-windows")
    set(VCPKG_MANIFEST_DIR "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
endif()
message(STATUS "VCPKG_ROOT: ${VCPKG_ROOT}")
message(STATUS "CMAKE_TOOLCHAIN_FILE: ${CMAKE_TOOLCHAIN_FILE}")
message(STATUS "VCPKG_TARGET_TRIPLET: ${VCPKG_TARGET_TRIPLET}")
message(STATUS "VCPKG_MANIFEST_DIR: ${VCPKG_MANIFEST_DIR}")

################################################################################
# 3RDPARTY
################################################################################
macro(VCPKG_LOAD_3RDPARTY)
    set(CMAKE_PREFIX_PATH "${CMAKE_BINARY_DIR}/vcpkg_installed/${VCPKG_TARGET_TRIPLET}")
    message(STATUS "Loading 3rd party libraries from vcpkg...")

    # openssl
    find_package(OpenSSL REQUIRED)
    if (OpenSSL_FOUND)
        message(STATUS "Found OpenSSL: ${OPENSSL_INCLUDE_DIR}")
        message(STATUS "Found OpenSSL: ${OPENSSL_LIBRARIES}")
    else()
        message(FATAL_ERROR "Could not find OpenSSL")
    endif()

    # sqlite3
    find_package(unofficial-sqlite3 CONFIG REQUIRED)
    if (unofficial-sqlite3_FOUND)
        message(STATUS "Found SQLite3")
    else()
        message(FATAL_ERROR "Could not find SQLite3")
    endif()
endmacro()
```

#### 3.4.3. 使用

在最外层 `CMakeLists.txt` 的 `project()` 前引入即可，其中 `unofficial` 表示三方库未实现 `cmake` 配置，是由 `vcpkg` 人员设置的

```cmake
# 要求最低Cmake版本
cmake_minimum_required(VERSION 3.15.0)

include(cmake/module_vcpkg.cmake)

# 解决方案名称
set(UseProjectName "EasyPluginFramework")
project(${UseProjectName})

# 三方库
VCPKG_LOAD_3RDPARTY()

# 需要使用 vcpkg 库，实际项目名称
target_link_libraries(${ProjectName} PRIVATE OpenSSL::SSL)
target_link_libraries(${ProjectName} PRIVATE OpenSSL::Crypto)
target_link_libraries(${ProjectName} PRIVATE unofficial::sqlite3::sqlite3)
```

进行项目配置（构建），安装过程需要科学上网，有时依赖编译环境（vcpkg自己维护strawberry下载并编译）会导致第一次配置会比较久，会在 `{VCPKG_ROOT}/installed` 中构建三方库，并且拷贝到编译目录（这里是 build）`build/vcpkg_installed`，不用担心删除 `build` 目录再次构建花费时间，但是修改三方库版本和 `VCPKG_TARGET_TRIPLET` 就会重新下载构建

```cmake
cmake -B"build" -G"Visual Studio 17 2022"
```

安装完成会提示怎么使用

```cmake
[cmake] Packages installed in this vcpkg installation declare the following licenses:
[cmake] Apache-2.0
[cmake] MIT
[cmake] blessing
[cmake] openssl is compatible with built-in CMake targets:
[cmake] 
[cmake]   find_package(OpenSSL REQUIRED)
[cmake]   target_link_libraries(main PRIVATE OpenSSL::SSL)
[cmake]   target_link_libraries(main PRIVATE OpenSSL::Crypto)
[cmake] 
[cmake] sqlite3 provides pkgconfig bindings.
[cmake] sqlite3 provides CMake targets:
[cmake] 
[cmake]     find_package(unofficial-sqlite3 CONFIG REQUIRED)
[cmake]     target_link_libraries(main PRIVATE unofficial::sqlite3::sqlite3)
[cmake] 
[cmake] Completed submission of openssl:x64-windows@3.5.4 to 1 binary cache(s) in 2.8 s
```

项目编译后动态库还会自动拷贝到编译目录

```cmake
cmake --build build
```

### 3.5. 总结

| 方法                  | 优点               | 缺点                     | 适用场景             |
| --------------------- | ------------------ | ------------------------ | -------------------- |
| `find_package`        | 简单，依赖系统库   | 系统库版本不可控         | 稳定环境，系统库丰富 |
| `FetchContent`        | 自动拉取、锁定版本 | 编译时间增加             | 中小型项目           |
| `ExternalProject_Add` | 独立构建库         | 配置复杂                 | 大型项目或多库       |
| `vcpkg`               | 版本管理便捷       | 依赖外部工具，第一次耗时 | 跨平台、多库         |




# 参考

[^1]: [CMake 项目中的 vcpkg](https://learn.microsoft.com/zh-cn/vcpkg/users/buildsystems/cmake-integration)