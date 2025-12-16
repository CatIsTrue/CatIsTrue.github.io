---
title: linux项目移植到Windows下CMake改造
date: 2025-11-06 17:00:20
tags: [linux, 跨平台移植, windows, c++, CMake]
categories: 技术折腾
---

# linux项目移植到Windows下CMake改造

## 背景
因项目工程需要，将linux下的`AProject`项目迁移到Windows下，对代码进行适配，编译选项进行修改。
通过CMake配置Windows(MSVC编译器)选项，进行跨平台移植。

## Windows下工具链
CMake+Visual Studio 2022

## CMake调整关键点

关键片段
```cmake
if(MSVC)
  option(BUILD_SHARED_LIBS "Build shared libraries (.dll)." OFF)
  add_compile_definitions(_USE_MATH_DEFINES)
  add_compile_definitions(NOMINMAX)
  string(APPEND CMAKE_CXX_FLAGS " /diagnostics:classic /utf-8 /MP /bigobj /EHsc /W3")
  string(APPEND CMAKE_C_FLAGS   " /diagnostics:classic /utf-8 /MP /bigobj /W3")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /STACK:10000000")
  if(MSVC_VERSION GREATER_EQUAL 1914)
    add_compile_options(/Zc:__cplusplus)
  endif() 
endif()
```

这段 CMake 配置代码是专门为 **Windows (MSVC 编译器)** 准备的“大补丁包”。

它的主要目的是：**让 Visual Studio 的行为更像 Linux 下的 GCC/Clang，同时解决 Windows 特有的一些“坑”。**

下面逐条详细解释：

### 1. 库的构建方式
```cmake
option(BUILD_SHARED_LIBS "Build shared libraries (.dll)." OFF)
```
*   **含义**：定义一个开关，默认设置为 `OFF`。
*   **作用**：告诉 CMake 默认编译 **静态库 (.lib)** 而不是 **动态库 (.dll)**。
*   **场景**：如果你不想在运行 exe 时拖着一堆 `.dll` 文件到处跑，或者不想处理复杂的 dll 导出符号（`__declspec(dllexport)`），用静态库是最省事的。

### 2. 预处理器定义 (Preprocessor Definitions)

```cmake
add_compile_definitions(_USE_MATH_DEFINES)
```
*   **含义**：开启数学常量定义。
*   **为什么需要**：标准 C++ 其实不强制要求提供 `M_PI` (圆周率) 这种宏。Linux 的 math.h 默认有，但 Windows 为了严格符合标准默认把它藏起来了。
*   **效果**：加上它，你才能在代码里愉快地使用 `M_PI`、`M_PI_2` 等宏，否则会报错“未声明的标识符”。

```cmake
add_compile_definitions(NOMINMAX)
```
*   **含义**：**这是一个救命的宏。** 禁用 Windows 头文件中的 `min` 和 `max` 宏。
*   **为什么需要**：Windows 的 `<windows.h>` 历史遗留问题，它定义了全局宏 `min(a,b)` 和 `max(a,b)`。这会严重干扰 C++ 标准库的 `std::min` 和 `std::max`。
*   **效果**：如果不加这个，当你写 `std::max(1, 2)` 时，预处理器会把它替换成错误的乱码从而编译失败。

### 3. 编译器标志 (Compile Flags)

这些标志被追加到了 `CMAKE_CXX_FLAGS` (C++) 和 `CMAKE_C_FLAGS` (C) 中。

*   **`/diagnostics:classic`**
    *   **含义**：设置报错信息的格式为经典模式。
    *   **作用**：VS2022 新版有时候报错信息太花哨，而且有时候VS界面读不懂编译器输出把真正的编译错误给吞掉（如`MSB8084`“CL.exe”的结构化输出无效: 无法分析 JsonRpc 通知:“Cannot transcode invalid UTF-8 JSON text to UTF-16 string.”这种报错。），用这个可以让输出格式变回 `文件名(行号): error ...`，更清晰。

*   **`/utf-8`**
    *   **含义**：**强制源文件和执行字符集都为 UTF-8。**
    *   **作用**：**极其重要**。Linux 代码通常是 UTF-8 的，而 Windows 中文环境默认是 GBK。
    *   **场景**：如果不加这个，你的代码里如果有中文字符串（比如 `printf("你好")`），打印出来就是乱码；或者代码里有中文注释，可能会导致编译报错。

*   **`/MP`**
    *   **含义**：**多进程编译 (Multi-Processor)。**
    *   **作用**：**加速神器**。它允许 VS 同时启动多个编译器进程来编译不同的源文件，利用多核 CPU。不加这个，编译速度会慢很多。

*   **`/bigobj`**
    *   **含义**：允许生成更大的对象文件（增加节的数量限制）。
    *   **作用**：**防止 `fatal error C1128`**。
    *   **场景**：如果你的代码里大量使用了模板（Template）、或者某个 `.cpp` 文件特别巨大（几万行），生成的 `.obj` 文件会超过默认限制。加上它就能解决。

*   **`/EHsc`** (仅 C++)
    *   **含义**：启用标准 C++ 异常处理模型。
    *   **作用**：告诉编译器捕捉 C++ 的 `try-catch` 异常，并假设 `extern "C"` 的函数不会抛出异常。这是 VS 编译 C++ 代码的标准姿势。

*   **`/W3`**
    *   **含义**：警告等级 3 (Warning Level 3)。
    *   **作用**：开启大部分常用的警告。等级从 `/W0` (不警告) 到 `/W4` (极度严格)。`/W3` 是一个比较平衡的生产环境选择。

### 4. 链接器标志 (Linker Flags)

```cmake
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /STACK:10000000")
```
*   **含义**：设置主线程栈大小为 ~10MB。
*   **作用**：防止栈溢出（Stack Overflow）。
详细讨论可以看另一篇博客：
[增大函数栈内存空间](cmake选项之CMAKE_EXE_LINKER_FLAGS.md)


### 5. 语言标准修正

```cmake
if(MSVC_VERSION GREATER_EQUAL 1914)
    add_compile_options(/Zc:__cplusplus)
endif()
```
*   **含义**：如果 MSVC 版本大于等于 1914 (VS 2017 15.7)，则启用 `/Zc:__cplusplus`。
*   **为什么需要**：这是一个历史坑。默认情况下，即使你开启了 C++17 或 C++20，MSVC 里的 `__cplusplus` 宏的值依然是 `199711L`（老标准）。这会导致很多跨平台库（如 Boost, Qt）误以为编译器不支持新特性。
*   **效果**：加上这个后，`__cplusplus` 宏就会正确报告版本号（如 C++17 会报告 `201703L`），让代码能正确识别当前的语言标准。

### 总结

这份配置可以解决很多**项目代码无关平台特性强相关**的报错，它**把 Visual Studio 调教得更符合现代 C++ 标准和跨平台开发习惯**。
如果不加这些配置，直接把 Linux 代码拿过来编译，通常会遇到中文乱码、数学库报错、min/max 冲突等一堆琐碎问题。。。