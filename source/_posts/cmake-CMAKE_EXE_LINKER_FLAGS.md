---
title: 如何通过CMake修改 Windows 程序的主线程栈大小
date: 2025-11-06 17:20:20
tags: [Windows, CMake]
categories: 技术折腾
---

# CMake选项之CMAKE_EXE_LINKER_FLAGS

# 如何通过CMake修改 Windows 程序的主线程栈大小

只需要在CMakeLists.txt里添加如下一行
```bash
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /STACK:10000000") 
```

这个命令的作用是 **修改 Windows 程序的主线程栈大小（Stack Size）**。

具体来说：

*   **`/STACK:10000000`**：这是传给 Windows 链接器（Linker）的一个参数。
*   **`10000000`**：单位是字节（Bytes）。
    *   $10,000,000 \text{ Bytes} \approx 9.5 \text{ MB}$。

### 为什么要加这一行？

默认情况下，Windows 程序的栈大小通常只有 **1MB**。如果你的程序中存在以下情况，1MB 就不够用了，会导致 **Stack Overflow（栈溢出）** 崩溃：

1.  **超大的局部数组**：
    ```cpp
    void someFunction() {
        // 这种大数组是分配在栈上的
        // 如果数组太大（比如 double arr[200000]），就会直接撑爆默认的 1MB 栈
        double hugeArray[500000]; 
        // ...
    }
    ```
2.  **极深的递归调用**：
    比如解析复杂的 G 代码结构，或者某些算法（如快速排序的最坏情况、树的遍历）递归层数太深，每一层递归都会占用一点栈空间，累积起来就会溢出。
3.  **复杂的类对象在栈上实例化**：
    如果你的某些类非常巨大（包含很多数据成员），并且你在函数里直接 `MyBigClass obj;` 这样定义，也会消耗大量栈空间。

### 这个修改意味着什么？

你把栈空间从默认的 **1MB** 增加到了 **约 10MB**。

这是一种“暴力但有效”的手段，用来防止因栈空间不足导致的程序闪退。在仿真、图像处理软件中，一般会这样配。

### 总结
添加这行在CMake里就是在告诉编译器：
**“给我这个程序的主线程预留 10MB 的内存栈空间，我有些函数里局部变量很大，或者递归很深，默认的 1MB 不够用。”**