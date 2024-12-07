> [dmjcb个人博客](https://dmjcb.github.io/)
> [原文地址](https://dmjcb.github.io/cmake/使用)

> [cmake使用详细教程(日常使用这一篇就足够了)](https://blog.csdn.net/iuu77/article/details/129229361)

## 安装

### linux

#### 软件安装

```shell
sudo apt install cmake -y
```

#### 源码安装

- 下载

```shell
wget https://cmake.org/files/v3.25/cmake-3.25.0-linux-aarch64.tar.gz

tar -xvzf cmake-3.25.0-linux-aarch64.tar.gz                                      
```

- 编译

```shell
cd cmake-3.25.0-linux-aarch64 && mkdir build && cd build 

# 配置编译环境
../bootstrap

# 编译安装
make && make install        
```

- 查看cmake版本

```shell
cmake --version
```

## 开发

CMake是C/C++构建文件生成工具, 通过CMakeList.txt生成项目构建文件

```mermaid
graph TB
    S(开始)-->A[编写]--> A1[/CMakeLists.txt/]--> B[生成构建文件]-->B1[/Makefile/]-->C[构建]--> C1[/可执行程序/库/]-->D[安装]-->F(结束)
```

### 编写

```c
// Main.cpp
#include<iostream>
int main() {
    std::cout << "Hello World" << std::endl;
    return 0;
}
```

```c
# CMakeList.txt
# 设置CMake的最低版本要求
cmake_minimum_required(VERSION 3.10)
# 设置项目名称
project(Main)

# 设置C++标准(C++11)
set(CMAKE_CXX_STANDARD 11)

# 定义可执行文件
add_executable(Main Main.cpp)
```

### 生成

cmake读取解析CMakeLists.txt, 检查系统环境、依赖库、编译器等设置, 生成对应平台构建文件

例如在Unix系统上会生成Makefile

#### 命令

- 读取当前目录CMakeList.txt, 在当目录生成

该方法会在当前目录下产生大量中间文件, 不推荐

```shell
cmake .
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241027_184942.jpg)

- 读取当前目录CMakeList.txt, 在Build/下生成

```shell
cmake -B Build

# 或者
cd Build
cmake . .
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241027_202125.jpg)

- 读取Source/CMakeList.txt, 在Build/下生成

```shell
cmake -S Source -B Build
```

### 构建

构建工具(如make)调用构建文件进行编译和链接

#### 命令

```shell
cmake --build 构建目录
```

- 当前目录构建

```shell
cmake --build .
```

- 在Build/下构建

```shell
cmake --build Build
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241027_202211.jpg)

### 安装

cmake将构建产物安装CMakeLists.txt设置安装到指定位置

#### 命令

```shell
cmake --install 构建目录 --prefix 安装路径
```
## 开发

### 单CMakeLists.txt

#### 单文件

最简单情况, 将单个.cpp生成可执行文件

```c
// Main.cpp
#include<iostream>
int main() {
    std::cout << "Hello World" << std::endl;
    return 0;
}
```

```mermaid
graph TB
    X(开始)-->A[add_executable]-->B[/可执行文件/]-->Z(结束)
```

```c
# CMakeLists.txt
# 最低版本号
cmake_minimum_required(VERSION 3.30)

# 项目名
project(Main)

# 生成可执行文件
add_executable(Main Main.cpp)
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241022204809.png)
![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241022205030.png)

#### 多目录

将多个目录下.cpp文件生成可执行文件

```shell
.
├── CMakeLists.txt
├── Include_1
│   └── TestFunc_1.hpp
├── Include_2
│   └── TestFunc_2.hpp
├── Main.cpp
├── Source_1
│   └── TestFunc_1.cpp
└── Source_2
    └── TestFunc_2.cpp
```

```c
// Include_1/TestFunc_1.hpp
#include<iostream>
void Hello_1();
```

```c
// Include_2/TestFunc_2.hpp
#include<iostream>
void Hello_2();
```

```c
// Source_1/TestFunc_1.cpp
#include "TestFunc_1.hpp"
void Hello_1() {
    std::cout << "Hello_1" << std::endl;
}
```

```c
// Source_2/TestFunc_2.cpp
#include "TestFunc_2.hpp"
void Hello_12() {
    std::cout << "Hello_2" << std::endl;
}
```

```c
// Main.cpp
#include "TestFunc_1.hpp"
#include "TestFunc_2.hpp"
int main() {
    Hello_1();
    Hello_2();
    return 0;
}
```

```mermaid
graph TB
    X(开始)
    A[target_include_directories<br>添加头文件目录]
    B[target_sources<br>添加源文件目录]
    C[add_executable]
    D[/可执行文件/]
    Z(结束)
    X-->A-->B-->C-->D-->Z
```

```c
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(Main)

add_executable(${PROJECT_NAME} "")
# 添加可执行文件依赖头文件目录
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_SOURCE_DIR}/Include_1
    ${CMAKE_SOURCE_DIR}/Include_2
)
# 添加可执行文件依赖源文件
target_sources(${PROJECT_NAME} PRIVATE
    ${CMAKE_SOURCE_DIR}/Source_1/TestFunc_1.cpp
    ${CMAKE_SOURCE_DIR}/Source_2/TestFunc_2.cpp
    ${CMAKE_SOURCE_DIR}/Main.cpp
)
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241024224619.png)

#### 生成库

将目标.cpp生成库文件

```shell
.
├── CMakeLists.txt
├── Include
│   └── API.hpp
├── Main.cpp
└── Source
└── API.cpp
```

```c
// Include/API.hpp
#ifndef __INCLUDE_API_HPP__
#define __INCLUDE_API_HPP__

#include<iostream>
#ifdef _WIN32
    #define __EXPORT __declspec(dllexport)
#else
    #define __EXPORT __attribute__((visibility("default")))
#endif

#ifdef __cplusplus
extern "C" {
#endif
    void Display();
    int Add(int x, int y);
#ifdef __cplusplus
}
#endif
#endif
```

```c
// Source/API.cpp
#include "API.hpp"
void Display() {
    std::cout << "Print API success!" << std::endl;
}
int Add(int x, int y) {
    return x + y;
}
```

```mermaid
graph TB
    X(开始)
    A[target_include_directories<br>添加头文件目录]
    B[target_sources<br>添加源文件目录]
    C[add_library<br>生成库]
    D[/库文件/]
    Z(结束)
    X-->A-->B-->C-->D-->Z
```

```c
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(API)

# 设置库生成目录为工程目录下Library
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/Library)

# 生成库文件
add_library(${PROJECT_NAME}_Shared SHARED "")
add_library(${PROJECT_NAME}_Static STATIC "")
# 将动态库与静态库名称存在LIB_NAME变量
foreach(LIB_NAME ${PROJECT_NAME}_Shared ${PROJECT_NAME}_Static)
    target_include_directories(${LIB_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Include)
    target_sources(            ${LIB_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Source/API.cpp)
    # 设置库依赖项, 将库名称设置为PROJECT_NAME值
    set_target_properties(     ${LIB_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
endforeach()
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241024221856.png)

#### 链接库

设Library/下已有libAPI库, Include/API.hpp是其头文件, 生成可执行文件并链接

```c
// Main.cpp
#include "API.hpp"
int main(void) {
    int res = Add(1, 2);
    std::cout << "res = " << res << std::endl;
    Display();
    return 0;
}
```

```mermaid
graph TB
    X(开始)
    A[target_include_directories<br>添加头文件目录]
    B[target_sources<br>添加源文件目录]
    C[target_link_libraries<br>添加库目录]
    D[add_executable<br>生成可执行文件]
    E[/可执行文件/]
    Z(结束)
    X-->A-->B-->C-->D-->E-->Z
```

```c
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(Main)

# 设置可执行文件生成目录
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/Bin)

# 将从Library目录中API库路径与名称值存储于FUNC_LIB
find_library(FUNC_LIB API ${PROJECT_SOURCE_DIR}/Library)

add_executable(${PROJECT_NAME} "")
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Include)
target_sources(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Main.cpp )
# 链接库
target_link_libraries(${PROJECT_NAME} PRIVATE ${FUNC_LIB})
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241024222701.png)

### 多CMakeLists.txt

#### 生成库链接

在Hello/ 中生成libAPI库, 并和根目录下Main.cpp生成可执行文件链接

```shell
.
├── CMakeLists.txt
├── Hello
│   ├── CMakeLists.txt
│   ├── Include
│   │   └── API.hpp
│   ├── Source
│   └── API.cpp
└── Main.cpp
```

```c
# Hello/CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(Hello)

set(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/Library)

# 生成动态库
add_library(${PROJECT_NAME} SHARED "")
# 添加目标文件所引用头文件
target_include_directories(${PROJECT_NAME} PRIVATE ${PROJECT_SOURCE_DIR}/Include)
# 添加目标文件所引用源文件
target_sources(${PROJECT_NAME} PRIVATE${PROJECT_SOURCE_DIR}/Source/API.cpp)
```

```c
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(Main)

# 设置可执行文件输出路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/Bin)
# 设置预链接库名称
set(EXTRA_LIBS ${EXTRA_LIBS} API)

# 添加子目录CMake执行
add_subdirectory(API)

# 生成可执行文件
add_executable(${PROJECT_NAME} "")
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/API/Include)
target_sources(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Main.cpp)
target_link_libraries(${PROJECT_NAME} ${EXTRA_LIBS})
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241022231348.png)

### 三方库

#### 源码编译

若三方库有CMake配置文件, 且能访问源码, 可直接添加到项目并用add_subdirectory编译

```c
add_subdirectory(SomeLibrary)
target_link_libraries(MyExecutable PRIVATE SomeLibrary)
```

#### FetchContent

CMake 3.11及以上版本引入FetchContent模块, 可直接下载第三方库编译

例如编译fmt库, 可通过以下两种方式实现

- 方式一, 主CMakeLists.txt调用

```shell
.
├── CMakeLists.txt
├── Main.cpp
└── Extern
```

```c
# CMakeLists.txt
cmake_minimum_required(VERSION 3.17)
project(Main)

set(CMAKE_CXX_STANDARD 14)

# 引入FetchContent
include(FetchContent)
FetchContent_Declare(fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 9.1.0
    # 源码存放位置
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/Extern/fmt
)
# 构建库
FetchContent_MakeAvailable(fmt)

add_executable(${PROJECT_NAME} "")
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Extern/fmt/include)
target_sources(            ${PROJECT_NAME} PUBLIC Main.cpp)
target_link_libraries(     ${PROJECT_NAME} PRIVATE fmt::fmt)
```

```c
// Main.cpp
#include "fmt/core.h"
int main(){
    std::string world = fmt::format("Hello {0}", "World");
    fmt::print("{}\n", world);
}
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241030_003207.jpg)
![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241030_003358.jpg)

发现第三方库已经下载

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241030_205530.jpg)

- 方式二, .cmake模块调用

可新建CMake/FMT.cmake, 将fmt库安装逻辑与主CMakeLists.txt解耦

```shell
.
├── CMakeLists.txt
├── Main.cpp
├── CMake
│   └── FMT.cmake
└── Extern
```

```c
# FMT.cmake
include(FetchContent)

set(FMT_LIB fmt)

FetchContent_Declare(${FMT_LIB}
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 9.1.0
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/Extern/${FMT_LIB}
)
FetchContent_MakeAvailable(${FMT_LIB})
```

```c
# CMakeLists.txt
cmake_minimum_required(VERSION 3.17)
project(Main)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/CMake)

# 导入FMT.cmake模块
include(FMT)

add_executable(${PROJECT_NAME} "")
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/Extern/fmt/include)
target_sources(            ${PROJECT_NAME} PUBLIC ${CMAKE_SOURCE_DIR}/Main.cpp)
target_link_libraries(     ${PROJECT_NAME} PRIVATE ${FMT_LIB}::${FMT_LIB})
```

![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241112_224124.jpg)
![](https://raw.githubusercontent.com/dmjcb/self_assets/main/image/20241112_224242.jpg)