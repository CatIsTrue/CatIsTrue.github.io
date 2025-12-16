---
title: Windows 会话 ID (Session ID) 与 Session 0 隔离穿透指南
date: 2021-03-18 08:11:39
tags: [windows, Session ID, c++]
categories: c++
---


## 1. 背景：开机自启动程序无法弹出界面？

在开发 Windows 后台服务（Service）或开机自启程序时，经常遇到一个棘手的问题：

**场景描述：**
有一个 GUI 程序 `getCode.exe` 需要开机自启动并与用户交互（例如弹出窗口让用户输入）。
*   **尝试 1**：最初直接使用 `nssm` 将 `getCode.exe` 注册为 Windows 服务。
    *   **结果**：程序启动了，但在后台默默运行，用户界面完全看不到，对键盘鼠标无反应。
*   **尝试 2**：开机后手动双击运行 `getCode.exe`。
    *   **结果**：程序正常显示，交互正常。

**原因分析：**
通过上网翻找资料，查看任务管理器后发现：
*   通过 `nssm` 启动的进程，其 **会话 ID (Session ID)** 为 `0`。
*   用户手动启动的进程，其 **会话 ID** 为 `1`（或更高）。
*   当前登录用户正在操作的桌面属于 Session 1，而 Session 0 的程序无法直接在 Session 1 的桌面上显示界面。

原来这就是 Windows 的 **Session 0 隔离机制** ——服务运行在 Session 0，而用户交互桌面在 Session 1/2/...；Session 0 中创建的界面用户看不见，也无法接收用户输入。

**解决方案思路：**
考虑了项目的使用场景，最后决定编写一个“启动器”服务（例如 `setExeSID.exe`），它的职责不是自己显示界面，而是探测当前用户所在的 Session ID，然后使用 Windows API “穿透”隔离，将目标程序 `getCode.exe` 注入到用户的 Session 中运行。

**大概逻辑为：**
1. `setExeSID.exe` 由 nssm 注册为服务，开机自启（因此它在 **Session 0**）。
2. `setExeSID.exe` 获取当前活动用户的 Session ID（通常是 **1**，也可能是 2/3...）。
3. `setExeSID.exe` 以该 Session 的用户 Token 为上下文，调用 `CreateProcessAsUser` 在用户会话中启动 `getCode.exe`（运行在用户桌面 `winsta0\default`）。


**预期效果**：
仍然实现开机自启动（靠服务）
`getCode.exe` 实际运行在当前登录用户的 Session 中，可以正常显示 UI 并交互

---

## 2. 查看会话ID

打开“任务管理器” -> “详细信息”选项卡 -> 右键表头“选择列” -> 勾选“会话 ID”。

![look_sid.jpg](/images/look_sid.jpg)

---

## 3. Session 0 隔离（为什么服务无法交互）

### 机制
*   **Windows XP 时代**：系统服务和第一个登录的用户都运行在 Session 0。服务可以直接弹出窗口（`MessageBox`），用户能看到并交互。这带来了巨大的安全风险（例如 Shatter Attack，恶意程序向服务窗口发送消息提升权限）。
*   **Windows Vista / 7 / 10 / 11**：引入了 **Session 0 隔离**。
    *   **Session 0**：**专用于系统服务**。非交互式，没有用户界面。
    *   **Session 1, 2...**：**用户会话**。第一个登录的用户分配 Session 1，远程桌面用户分配 Session 2 等。

### 为什么普通启动会失败？
当服务（Session 0）尝试启动一个带 UI 的进程时，该进程默认继承父进程的 Session ID (0)。由于 Session 0 无法访问用户的显卡驱动和桌面环境（`winsta0\default`），UI 渲染会失败，或者被系统拦截在不可见的后台。

---

## 4. 核心做法：如何穿透？

要实现从 Session 0 启动进程到 Session 1，不能使用简单的 `CreateProcess` 或 `system()`，必须使用 **Windows API** 进行令牌（Token）操作。

### 关键步骤流程
1.  **定位目标**：找到当前正在活动的（用户正在看的）Session ID (`WTSGetActiveConsoleSessionId`)。
2.  **获取令牌**：拿到该 Session 中用户的身份令牌 (`WTSQueryUserToken`)。
3.  **复制令牌**：将令牌复制一份，并转换为主令牌 (`DuplicateTokenEx`)。
4.  **处理环境**：为新进程创建正确的环境变量块 (`CreateEnvironmentBlock`)，否则程序可能找不到路径。
5.  **跨界启动**：使用 **`CreateProcessAsUser`** 启动进程，并显式指定运行在交互式桌面 (`winsta0\default`)。

### 涉及到的主要 API

涉及到的主要 API 及其作用

- `GetCurrentProcess`  
  获取当前进程伪句柄（常用于配合 `OpenProcessToken`）。

- `OpenProcessToken`  
  打开进程访问令牌（Token），用于查询/调整权限。

- `LookupPrivilegeValue` + `AdjustTokenPrivileges`  
  为当前进程启用某些特权。  
  常见原因：`CreateProcessAsUser` / 资源配额调整等需要特权。  
  注意：**前提是账户本身拥有该特权**，只是默认未启用。

- （会话枚举/定位）
  - `WTSEnumerateSessions`：枚举会话列表（可选）
  - `WTSGetActiveConsoleSessionId`：获取当前物理控制台活动会话（常用）
  - `WTSQueryUserToken`：获取指定 Session 的用户 Token（常用）

- `DuplicateTokenEx`  
  将 Token 复制为可用于创建进程的 **Primary Token**（常见必做步骤）。

- `CreateEnvironmentBlock`  
  为目标用户创建环境变量块（否则新进程可能继承服务的环境，导致路径/变量异常）。

- `CreateProcessAsUser`  
  以指定用户的安全上下文创建进程。  
  关键点：默认新进程可能是非交互桌面，需要在 `STARTUPINFO.lpDesktop` 指定交互桌面。
  `STARTUPINFO.lpDesktop = L"winsta0\\default"`
  否则可能创建在不可见桌面，表现为“进程在跑但看不见/不能交互”。


## 5. 常见坑与注意事项

1.  **服务权限问题**：
    你的“启动器服务”必须以 **LocalSystem (本地系统)** 账户运行。如果以 `LocalService` 或 `NetworkService` 运行，调用 `WTSQueryUserToken` 会直接返回 `ERROR_PRIVILEGE_NOT_HELD` (错误码 1314)。

2.  **无人登录时的情况**：
    如果电脑刚开机，停留在登录界面（LogonUI），此时 `WTSGetActiveConsoleSessionId` 可能返回 Session 1，但并没有即时用户。如果在此时启动程序，程序会在登录界面后台运行，用户输入密码进入桌面后可能反而看不到了。
    最好在服务中做一个轮询，检测到有真实用户登录（Session ID > 0 且状态为 Active）后再启动。

3.  **用户文件访问权限**：
    虽然进程在用户 Session 中运行，但如果涉及到读写特定的网络共享路径或加密文件夹，仍需注意令牌的权限范围。

## 5. 总结

当 Windows 上的程序需要：

- **开机自启动**（常由服务实现）
- 同时又需要**用户交互（UI/输入）**

就必须考虑 **Session 0 隔离**。服务所在的 Session 0 与用户交互会话隔离，直接启动的 UI 程序往往不可交互。常见做法是：服务在 Session 0 中运行，引导在当前用户 Session 中创建进程（`WTSQueryUserToken` + `CreateProcessAsUser`），或采用服务与用户端代理的架构通过 IPC 完成交互。

这样就可以完美实现开机自启服务与用户桌面的无缝交互。（比如开发远程协助软件）


## 6. 参考资料

[关于突破SESSION 0隔离创建进程](https://cloud.tencent.com/developer/article/1424933)

[总结了用户权限设置和进程权限提升，提权demo也可参考](https://blog.csdn.net/yockie/article/details/17029293)

[总结了window API](http://yfvb.com/help/win32sdk/index.htm?page=html/_qx5ll.htm)
