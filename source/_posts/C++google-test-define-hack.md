---
title: C++ 单元测试黑魔法：`#define private public`
date: 2025-12-15 15:01:39
tags: [c++, 单元测试, google test]
categories: c++
---


在 C++ 单元测试的世界里，一直流传着一个“邪道”技巧。

当你面对一个庞大的遗留类，想要测试其中一个复杂的 `private` 辅助函数，或者验证某个 `private` 成员变量的状态，但又不想（或不能）修改原始头文件去添加 `friend` 声明时，很多人的第一反应是使用那个著名的“黑魔法”：

```cpp
#define private public
#define protected public
#include "MyLegacyClass.h"
#undef private
#undef protected
```

## 一、 黑魔法的原理：欺骗编译器

这个技巧的核心逻辑非常简单粗暴：**预处理器的宏替换**。

C++ 的编译过程是分阶段的。在编译器真正开始语法分析之前，预处理器会先处理所有的 `#include` 和 `#define`。

当我们写下：

```cpp
// Test.cpp
#define private public
#include "MyLegacyClass.h"
```

预处理器在展开 `MyLegacyClass.h` 时，会将其中的所有 `private` 关键字替换为 `public`。
对于编译器来说，在编译 `Test.cpp` 这个单元时，`MyLegacyClass` 的所有成员**确实就是公有的**。因此，测试代码可以直接调用 `MyLegacyClass::PrivateFunc()` 而不会报“访问权限错误”。

这是编译期的欺骗，很完美，对吧？

## 二、 在 Linux (GCC/Clang) 上屡试不爽

在 Linux 环境下，GCC 和 Clang 遵循 **Itanium C++ ABI**（应用程序二进制接口）标准。

在该标准下，函数的**符号修饰（Name Mangling）**主要包含函数名、命名空间和参数类型等信息，但通常**不包含访问控制级别（public/private/protected）**。

也就是说，对于下面这个函数：

```cpp
class MyLegacyClass {
private:
    void PrivateFunc(int type);
};
```

无论它是 `private` 还是 `public`，GCC 生成的符号名可能都是类似 `_ZN7Scanner11UpdateTokenEi` 的样子。

1. **库的编译**：`MyLegacyClass.cpp` 正常编译，`PrivateFunc` 是 private，生成符号 `_ZN7Scanner11UpdateTokenEi`。
2. **测试的编译**：`Test.cpp` 用了黑魔法，编译器以为 `PrivateFunc` 是 public，生成调用指令，寻找符号 `_ZN7Scanner11UpdateTokenEi`。
3. **链接**：链接器发现两个符号名字一样，**链接成功**！

在实践中，Itanium ABI 往往能“宽容”地让它跑通。因此，这一招在 Linux 环境下（使用 GCC 或 Clang）屡试不爽，属于“快速通关”的秘籍。

不过，需要注意的是，当你试图将代码移植到 Windows 环境，使用 Visual Studio (MSVC) 编译时，黑魔法就会失效——**链接错误 (LNK2019)**。

---

## 三、 Windows (MSVC) 的滑铁卢：LNK2019

如果在 Windows 上使用 MSVC 编译器做同样的事情，你会收到类似这样的错误：

> **error LNK2019**: 无法解析的外部符号  "public: bool __cdecl MyLegacyClass::PrivateFunc(int)" (?PrivateFunc@MyLegacyClass@@QEAAXH@Z)，函数 "private: virtual void __cdecl ATest_APrivateFuncCase_Test::TestBody(void)"(?TestBody@ATest_APrivateFuncCase_Test@@EEAAXXZ) 中引用了该符号

这表明你使用 `#define private public` 这种“黑魔法”虽然欺骗了编译器（Compiler），让你在测试代码中可以调用私有函数，但它改变不了链接器（Linker）的事实。

### 根本原因：MSVC 的符号修饰包含访问级别

微软的 C++ ABI 与 Itanium ABI 不同。MSVC 在生成函数的修饰名（Mangled Name）时，**将函数的访问控制权限（Access Specifier）编码进了符号名里**。

我们来看一下区别：

| 代码定义                | 访问权限    | MSVC 生成的符号名 (大致示意)           |
| :---------------------- | :---------- | :------------------------------------- |
| `void PrivateFunc(int)` | **private** | `?PrivateFunc@MyLegacyClass@@AEAAXH@Z` |
| `void PrivateFunc(int)` | **public**  | `?PrivateFunc@MyLegacyClass@@QEAAXH@Z` |

我们注意到:
*   Private 版本包含 **`A`** (`AEAA...`)
*   Public 版本包含 **`Q`** (`QEAA...`)

### 流程

1.  **源文件编译 (`MyLegacyClass.cpp`)**：
    你编译项目源代码时，没有加黑魔法。编译器看到的是 `private`，生成的 `MyLegacyClass.obj` 里，函数的符号是 **带 A 的 (Private 版)**。

2.  **测试文件编译 (`Test.cpp`)**：
    你使用了 `#define private public`。编译器被欺骗了，它认为 `PrivateFunc` 是 `public` 的。于是它在生成 `Test.obj` 时，生成了一个**寻找 带 Q 的 (Public 版)** 符号的指令。

3.  **链接阶段**：
    链接器开始工作。测试代码大喊：“给我一个 `...QEAA...` (Public) 的函数！”
    由于只有 `MyLegacyClass.obj`，它回答：“我只有 `...AEAA...` (Private) 的版本。”
    **链接器：不匹配，报错，LNK2019。**

这就是为什么`MSVC`下加了黑魔法，却仍然死活链接不上的原因。

## 四、 更推荐的解决方案

一般来说，我们 `#define private public` 是未定义行为且在 Windows 上不可用，我们应该如何测试私有成员呢？

### 1. 使用 Google Test 的 `FRIEND_TEST` (推荐)

这是最标准、最安全的方法。它利用了 C++ 的 `friend` 机制，专门为测试开放白名单。

**在头文件中：**

```cpp
#include <gtest/gtest_prod.h> // 引入 gtest 里的这个头文件

class MyLegacyClass {
public:
    // ...
private:
    void PrivateFunc(int type);

    // 允许 ATest 类的 APrivateFuncCase 测试用例访问私有成员
    FRIEND_TEST(ATest, APrivateFuncCase);
};
```

**在测试文件中：**

```cpp
TEST(ATest, APrivateFuncCase) {
    MyLegacyClass scan;
    // 直接访问，合法的！
    scan.PrivateFunc(1); 
}
```

这种方式生成的符号名是完全一致的，无论在 Linux 还是 Windows 都能完美运行。

### 2. 也是一种思路：Pimpl 模式

如果你的私有逻辑非常复杂以至于需要大量测试，这通常意味着该逻辑应该被提取到一个独立的类中（Impl 类）。你可以将这个 Impl 类设为 public（或在内部头文件中定义），然后单独对其进行测试。

## 总结

`#define private public` 就像是程序员的禁术。它在 Linux/GCC 的宽容下或许能让你尝到甜头，但在 Windows/MSVC 严谨的 ABI 规则面前，它会罢工报警。
还是权衡一下使用场景再决定要不要用吧。