---
title: cmake入门
comments: true
category: [c++, cmake]
keywords: [cmake]
date: 2021-10-07 21:30:40
update: 2021-10-07 21:30:40
description: cmake
tags: cmake

---

## 一、什么是cmake

- 允许开发者编写一种平台无关的 CMakeList.txt 文件来定制整个编译流程
- 根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件
- CMake 是一个比 make 更高级的编译配置工具。

## 二、单个文件DEMO

<!--more-->

单个文件xxx在目录xxx下，编写 CMakeLists.txt 文件，并保存在与 该 源文件同个目录下：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo1)

# 指定生成目标
add_executable(Demo main.cc)
```

CMakeLists.txt 的语法:

由命令、注释和空格组成，其中命令是不区分大小写的。符号 `#` 后面的内容被认为是注释。命令由命令名称、小括号和参数组成，参数之间使用空格进行间隔。

命令：

- `cmake_minimum_required`：指定运行此配置文件所需的 CMake 的最低版本；
- `project`：参数值是 `Demo1`，该命令表示项目的名称是 `Demo1` 。
- `add_executable`： 将源文件编译成一个名称为 Demo 的可执行文件。

在当前目录执行 `cmake .` ，得到 Makefile 后再使用 `make` 命令编译得到 Demo1 可执行文件。

## 三、多个源文件

### 1.同一目录下多个源文件

```
./Demo2
|
+--- main.cc
|
+--- MathFunctions.cc
|
+--- MathFunctions.h
```

CMakeLists.txt 可以改成如下的形式：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo2)

# 指定生成目标
add_executable(Demo main.cc MathFunctions.cc)
```

唯一的改动只是在 `add_executable` 命令中增加了一个 `MathFunctions.cc` 源文件。

如果源文件很多，把所有源文件的名字都加进去将是一件烦人的工作。更省事的方法是使用 **aux_source_directory**命令：该命令会查找指定目录下的所有源文件，然后将结果存进指定变量名。

其语法如下：

```
aux_source_directory(<dir> <variable>)
```

修改 CMakeLists.txt 如下：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo2)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
```

CMake 会将当前目录所有源文件的文件名赋值给变量 `DIR_SRCS` ，再指示变量 `DIR_SRCS` 中的源文件需要编译成一个名称为 Demo 的可执行文件。

### 2.多个目录，多个源文件

```
./Demo3
|
+--- main.cc
|
+--- math/
|
+--- MathFunctions.cc
|
+--- MathFunctions.h
```

需要分别在项目根目录 Demo3 和 math 目录里各编写一个 CMakeLists.txt 文件。为了方便，我们可以先将 math 目录里的文件编译成静态库再由 main 函数调用。根目录中的 CMakeLists.txt ：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo3)
# 查找当前目录下的所有源文件

# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 添加 math 子目录
add_subdirectory(math)

# 指定生成目标
add_executable(Demo main.cc)

# 添加链接库
target_link_libraries(Demo MathFunctions)
```

命令 `target_link_libraries` 指明可执行文件 main 需要连接一个名为 MathFunctions 的链接库 。

子目录中的 CMakeLists.txt：

```cmake
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library (MathFunctions ${DIR_LIB_SRCS})
```

命令 `add_library` 将 src 目录中的源文件编译为静态链接库。

## 四、自定义编译选项

CMake 允许为项目增加编译选项，从而可以根据用户的环境和需求选择最合适的编译方案。

例如，可以将 MathFunctions 库设为一个可选的库，如果该选项为 `ON` ，就使用该库定义的数学函数来进行运算。否则就调用标准库中的数学函数库。

### 1.修改 CMakeLists 文件

在顶层的 CMakeLists.txt 文件中添加该选项：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo4)

# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
"${PROJECT_SOURCE_DIR}/config.h.in"
"${PROJECT_BINARY_DIR}/config.h"
)

# 是否使用自己的 MathFunctions 库
option (USE_MYMATH "Use provided math implementation" ON)
# 是否加入 MathFunctions 库
if (USE_MYMATH)
include_directories ("${PROJECT_SOURCE_DIR}/math")
add_subdirectory (math)
set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})
target_link_libraries (Demo ${EXTRA_LIBS})
```

- `configure_file` 命令用于加入一个配置头文件 config.h ，这个文件由 CMake 从 [config.h.in](http://config.h.in/) 生成，通过这样的机制，将可以通过预定义一些参数和变量来控制代码的生成。
- `option` 命令添加了一个 `USE_MYMATH` 选项，并且默认值为 `ON` 。
- 根据 `USE_MYMATH` 变量的值来决定是否使用我们自己编写的 MathFunctions 库。

### 2.修改 [main.cc](http://main.cc/) 文件

修改 [main.cc](http://main.cc/) 文件，让其根据 `USE_MYMATH` 的预定义值来决定是否调用标准库还是 MathFunctions 库：

```c
#include
#include
#include "config.h"
#ifdef USE_MYMATH
#include "math/MathFunctions.h"
#else
#include
#endif
int main(int argc, char *argv[])
{
if (argc < 3){
printf("Usage: %s base exponent \n", argv[0]);
return 1;
}
double base = atof(argv[1]);
int exponent = atoi(argv[2]);
#ifdef USE_MYMATH
printf("Now we use our own Math library. \n");
double result = power(base, exponent);
#else
printf("Now we use the standard library. \n");
double result = pow(base, exponent);

#endif
printf("%g ^ %d is %g\n", base, exponent, result);
return 0;
}
```

### 3.编写 [config.h.in](http://config.h.in/) 文件

上面引用了一个 config.h 文件，这个文件预定义了 `USE_MYMATH` 的值。但我们并不直接编写这个文件，为了方便从 CMakeLists.txt 中导入配置，我们编写一个 [config.h.in](http://config.h.in/) 文件，内容如下：

```cmake
#cmakedefine USE_MYMATH
```

这样 CMake 会自动根据 CMakeLists 配置文件中的设置自动生成 config.h 文件。

### 4.编译项目

编译一下这个项目，为了便于交互式的选择该变量的值，可以使用 `ccmake` 命令 ，也可以使用 `cmake -i` 命令，该命令会提供一个会话式的交互式配置界面，从中可以找到刚刚定义的 `USE_MYMATH` 选项，按键盘的方向键可以在不同的选项窗口间跳转，按下 `enter` 键可以修改该选项。修改完成后可以按下 `c` 选项完成配置，之后再按 `g` 键确认生成 Makefile 。ccmake 的其他操作可以参考窗口下方给出的指令提示。

### 5.安装和测试

CMake 也可以指定安装规则，以及添加测试。这两个功能分别可以通过在产生 Makefile 后使用 `make install` 和 `make test` 来执行。

#### 1.定制安装规则

先在 math/CMakeLists.txt 文件里添加下面两行：

```cmake
# 指定 MathFunctions 库的安装路径
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```

指明 MathFunctions 库的安装路径。之后同样修改根目录的 CMakeLists 文件，在末尾添加下面几行：

```cmake
# 指定安装路径
install (TARGETS Demo DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/config.h"
DESTINATION include)
```

通过上面的定制，生成的 Demo 文件和 MathFunctions 函数库 libMathFunctions.o 文件将会被复制到 `/usr/local/bin` 中，而 MathFunctions.h 和生成的 config.h 文件则会被复制到 `/usr/local/include` 中。

这里的 `/usr/local/` 是默认安装到的根目录，可以通过修改 `CMAKE_INSTALL_PREFIX` 变量的值来指定这些文件应该拷贝到哪个根目录。

### 6.为工程添加测试

CMake 提供了一个称为 CTest 的测试工具。我们要做的只是在项目根目录的 CMakeLists 文件中调用一系列的 `add_test` 命令。

```cmake
# 启用测试
enable_testing()

# 测试程序是否成功运行
add_test (test_run Demo 5 2)

# 测试帮助信息是否可以正常提示
add_test (test_usage Demo)
set_tests_properties (test_usage
PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")

# 测试 5 的平方
add_test (test_5_2 Demo 5 2)
set_tests_properties (test_5_2 PROPERTIES PASS_REGULAR_EXPRESSION "is 25")

# 测试 10 的 5 次方
add_test (test_10_5 Demo 10 5)
set_tests_properties (test_10_5 PROPERTIES PASS_REGULAR_EXPRESSION "is 100000")

# 测试 2 的 10 次方
add_test (test_2_10 Demo 2 10)
set_tests_properties (test_2_10 PROPERTIES PASS_REGULAR_EXPRESSION "is 1024")
```

 `PASS_REGULAR_EXPRESSION` 用来测试输出是否包含后面跟着的字符串。

如果要测试更多的输入数据，像上面那样一个个写测试太繁琐。这时可以通过编写宏来实现：

```cmake
# 定义一个宏，用来简化测试工作
macro (do_test arg1 arg2 result)
add_test (test_${arg1}_${arg2} Demo ${arg1} ${arg2})
set_tests_properties (test_${arg1}_${arg2}
PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)

# 使用该宏进行一系列的数据测试
do_test (5 2 "is 25")
do_test (10 5 "is 100000")
do_test (2 10 "is 1024")
```

 CTest 的更详细的用法可以通过 `man 1 ctest` 参考 CTest 的文档。

## 五、支持gdb

让 CMake 支持 gdb 的设置也很容易，只需要指定 `Debug` 模式下开启 `-g` 选项：

```c++
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

## 六、添加环境检查

有时候可能要对系统环境做点检查，例如要使用一个平台相关的特性的时候。在这个例子中，我们检查系统是否自带 pow 函数。如果带有 pow 函数，就使用它；否则使用我们定义的 power 函数。

### 1.添加 CheckFunctionExists 宏

在顶层 CMakeLists 文件中添加 CheckFunctionExists.cmake 宏，并调用 `check_function_exists` 命令测试链接器是否能够在链接阶段找到 `pow` 函数。

```cmake
# 检查系统是否支持 pow 函数
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
check_function_exists (pow HAVE_POW)
```

将上面这段代码放在 `configure_file` 命令前。

### 2.预定义相关宏变量

修改 [config.h.in](http://config.h.in/) 文件，预定义相关的宏变量：

```cmake
// does the platform provide pow function?
#cmakedefine HAVE_POW
```

### 3.在代码中使用宏和函数

修改 [main.cc](http://main.cc/) ，在代码中使用宏和函数：

```c
#ifdef HAVE_POW
printf("Now we use the standard library. \n");
double result = pow(base, exponent);
#else
printf("Now we use our own Math library. \n");
double result = power(base, exponent);
#endif
```

## 七、添加版本号

给项目添加和维护版本号是一个好习惯，这样有利于用户了解每个版本的维护情况，并及时了解当前所用的版本是否过时，或是否可能出现不兼容的情况。

首先修改顶层 CMakeLists 文件，在 `project` 命令之后加入如下两行：

```cmake
set (Demo_VERSION_MAJOR 1)
set (Demo_VERSION_MINOR 0)
```

分别指定当前的项目的主版本号和副版本号。为了在代码中获取版本信息，我们可以修改 [config.h.in](http://config.h.in/) 文件，添加两个预定义变量：

```cmake
// the configured options and settings for Tutorial
#define Demo_VERSION_MAJOR @Demo_VERSION_MAJOR@
#define Demo_VERSION_MINOR @Demo_VERSION_MINOR@
```

这样就可以直接在代码中打印版本信息了。

## 八、生成安装包

配置生成各种平台上的安装包，包括二进制安装包和源码安装包。需要用到 CPack ，它同样也是由 CMake 提供的一个工具，专门用于打包。

### 1.修改CMakeLists.txt文件

在顶层的 CMakeLists.txt 文件尾部添加下面几行：

```cmake
# 构建一个 CPack 安装包
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${Demo_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Demo_VERSION_MINOR}")
include (CPack)
```

- 导入 InstallRequiredSystemLibraries 模块，以便之后导入 CPack 模块；

- 设置一些 CPack 相关变量，包括版权信息和版本信息，其中版本信息用了上一节定义的版本号；

- 导入 CPack 模块。

### 2.执行cpack命令

- 生成二进制安装包

  ```cmake
  cpack -C CPackConfig.cmake
  ```

- 生成源码安装包

  ```cmake
  cpack -C CPackSourceConfig.cmake
  ```

- 在生成项目后，执行 `cpack -C CPackConfig.cmake` 命令，会在该目录下创建 3 个不同格式的二进制包文件，这 3 个二进制包文件所包含的内容是完全相同的。我们可以执行其中一个。此时会出现一个由 CPack 自动生成的交互式安装界面。

关于 CPack 的更详细的用法可以通过 `man 1 cpack` 参考 CPack 的文档。

## 九、迁移其他平台项目到CMake



## 十、参考文档

1. [超详细的cmake教程][https://blog.csdn.net/zhuiyunzhugang/article/details/88142908]

[https://blog.csdn.net/zhuiyunzhugang/article/details/88142908]

---
