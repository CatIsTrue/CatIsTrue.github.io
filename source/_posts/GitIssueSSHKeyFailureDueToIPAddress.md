---
title:  Git 多账号配置下 Pull 需要输入密码的问题记录
date: 2025-12-26 10:29:01
tags: [git]
categories: git
---


## 1. 背景

在本地配置了多个 Git 账号（使用 SSH Key 管理）的情况下，从 GitLab 拉取三个仓库的代码。
- **仓库 A & B：** `git pull` 正常，直接走 SSH 验证。
- **仓库 C：** `git pull` **失败**，提示需要输入密码 (Enter password)。
  
[关于具体的配置过程可以看这个文档](https://www.catistrue.com/posts/aedc60d4.html)


## 2. 排查过程

通过查看远程仓库地址发现差异：

**正常的仓库 (A/B):**
使用域名连接：
```bash
git remote -v
origin  git@gitlab.namexxx.com:pjname/projectA.git (fetch)
```

**异常的仓库 (C):**
使用 IP 地址连接：
```bash
git remote -v
origin  git@10.10.10.xxx:pjname/projectC.git (fetch)
```

## 3. 原因分析

本地的 SSH 配置文件 (`~/.ssh/config`) 是基于**域名 (Host)** 来匹配私钥的。配置通常如下所示：

```text
# ~/.ssh/config 示例
Host gitlab.namexxx.com
    HostName gitlab.namexxx.com
    User git
    IdentityFile ~/.ssh/id_rsa_work
```

- 当 Git 访问 `gitlab.namexxx.com` 时，SSH 代理能成功匹配到 `id_rsa_work` 私钥。
- 当 Git 访问 `10.10.10.xxx` 时，SSH 代理**无法**将这个 IP 匹配到上述规则，导致找不到对应的私钥。
- SSH 机制在无法使用密钥登录时，会自动降级为密码验证，因此提示输入密码。

## 4. 解决方案

将异常仓库的远程地址从 **IP** 修改为 **域名**。

**执行命令：**

```bash
# 1. 进入项目目录
cd D:\projectC

# 2. 修改 remote url (将 IP 替换为域名)
git remote set-url origin git@gitlab.namexxx.com:pjname/projectC.git

# 3. 验证修改结果
git remote -v
# 输出应为: origin git@gitlab.namexxx.com:pjname/projectC.git (fetch)...

# 4. 测试拉取
git pull
```

## 5. 总结

在多 Git 账号环境下，**必须严格保证 Remote URL 的 Host 部分与 `~/.ssh/config` 中的 Host 定义一致**。不要混用 IP 地址和域名，否则 SSH 规则将失效。
