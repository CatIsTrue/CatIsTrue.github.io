---
title: Windows常用指令
date: 2025-12-18 10:12:00
tags: [windows]
categories: windows
---

之前一直在linux下，跨平台移植到windows下的时候发现命令怎么用都不顺手，整理一下常用的命令：对目录/文件的常用增、删、重命名、复制、移动、路径切换命令记录。

由于 Windows 有两个主要的命令行环境：CMD (命令提示符) 和 PowerShell（Git Bash 兼容部分 Linux 命令），它们的指令略有不同。

下面以 **PowerShell** 为例（Git Cmd 也通用）。

---

## 1. 删除操作
### 删除目录

> 删除前最好确认路径，`/s` 会把子目录也删掉,`/q` 静默，不再确认。

```bat
rd /s /q 目录名

REM 示例：
rd /s /q D:\test_folder
```

---

### 删除文件

```bat
del /f /q 文件名   REM /f 强制删除，/q 静默

REM 示例：
del /f /q D:\logs\*.log   REM 删除目录下所有 .log 文件
```

---

## 2. 复制操作

### 复制文件

```bat
copy 源文件 目标文件或目录

REM 示例：
copy D:\a.txt E:\backup\a.txt
copy a.txt D:\backup\   REM 复制到目录，文件名不变
```

---

### 复制目录

推荐用 `robocopy`。

```bat
robocopy 源目录 目标目录 /e REM /e 递归复制所有子目录，包括空的子目录。

REM 示例：
robocopy D:\project E:\project_backup /e
```

---

## 3. 移动操作

### 移动文件

```bat
move 源文件 目标文件或目录

REM 示例：
move a.txt D:\backup\
move D:\a.txt E:\newname.txt
```

---

### 移动目录

```bat
move 源目录 目标目录

REM 示例：
move D:\old_folder D:\new_folder
move D:\project E:\project   REM 整个目录移到 E 盘
```

---

## 4. 快速切换盘符到指定路径

### 切换盘符：

```bat
C:
D:
E:
```

### 进入指定路径：

```bat
cd 路径

REM 示例：
D:
cd \code\myproject
REM 现在路径：D:\code\myproject

REM 示例：
cd /d D:\code\myproject
```

> `cd /d` 可以 **同时** 切换盘符和目录，比较方便。

---

## 5. 快速回到当前磁盘根路径

```bat
cd \

REM 示例：
D:\code\project> cd \
D:\>
```

---

## 6. 创建操作

### 创建目录

```bat
mkdir 目录名
md 目录名

REM 示例：
mkdir new_folder
mkdir D:\code\project

REM 可以一次创建多级目录：
mkdir D:\code\project\src\utils
```

---

### 创建文件

`cmd` 里没有专门的“新建空文件”命令，一般用下面几种方式：

### 空文件（或覆盖）

```bat
type nul > 文件名

REM 示例：
type nul > test.txt

REM  带内容的简单文本文件：
echo hello > file.txt         REM 创建并写入一行
echo world >> file.txt        REM 追加一行
```

### 如果安装了vscode ,可以直接用`code 文件名`创建空文件并打开

---

## 7. 重命名操作

### 重命名目录

```bat
ren 原目录名 新目录名
rename 原目录名 新目录名

REM 示例：（当前在 D:\ 盘）：
ren old_folder new_folder
ren D:\code\project_old project
```

---

### 重命名文件

```bat
ren 原文件名 新文件名

REM 示例：
ren a.txt b.txt
ren D:\logs\old.log app.log
```

---

## 总结
删除目录的指令和linux完全不一样，`rd /s /q 目录名`
删除文件的指令也很奇怪，`del /f /q 文件名`

复制文件和linux也不一样，不能缩写成cp,会不识别，windows下复制文件是`copy 源文件 目标文件或目录`
复制目录更费劲，是`robocopy 源目录 目标目录 /e`

移动文件或目录都是`move 源 目标`指令
创建目录`mkdir`,还好，这个和linux一致(终于有一个一样的了)
创建文件是`type`，如果安装了vscode ,可以直接用`code 文件名`创建空文件并打开
重命名文件或目录都是`ren 原名 新名`指令

两个平台的指令差异还挺大。。