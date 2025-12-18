---
title: 本地配置多个git账号
date: 2025-12-03 20:42:01
tags: [git]
categories: git
---

有时候，需要在一个电脑上管理多个 Git 账号（比如一个是 GitLab，一个是 GitHub，或者两个都是 GitHub 账号）是一个比较常见的场景。

如果不配置好，很容易出现“用错账号提交代码”或者“没权限推送代码”的尴尬情况。

最优雅、最推荐的方案是使用 **SSH Config + 文件夹别名** 的方式。

下面是手把手的配置指南：

## 核心思路
1.  **生成两把钥匙**：为每个账号生成不同的 SSH Key。
2.  **告诉 SSH 怎么选**：配置 `~/.ssh/config` 文件，让 SSH 知道访问不同主机时用哪把钥匙。
3.  **告诉 Git 怎么选**：(可选但推荐) 通过文件夹路径自动切换用户名和邮箱。

---

### 第一步：生成两对 SSH Key

打开终端（Terminal 或 Git Bash），分别生成两个 SSH Key。
**注意要给文件起不同的名字！**

```bash
# 1. 生成账号1的 Key (假设是 GitHub)
ssh-keygen -t ed25519 -C "你的邮箱1@gmail.com" -f ~/.ssh/id_ed25519_gmail

# 2. 生成账号2的 Key (假设是 GitLab)
ssh-keygen -t ed25519 -C "你的邮箱2@mail.163.com" -f ~/.ssh/id_ed25519_163
```
*(如果系统不支持 ed25519，可以用 `-t rsa -b 4096` 代替)*

现在你的 `~/.ssh/` 目录下应该有 4 个文件（两个私钥无后缀，两个公钥带 `.pub`）。

### 第二步：把公钥添加到对应的平台
1.  **账号1**：复制 `id_ed25519_gmail.pub` 的内容 -> 去 GitHub -> Settings -> SSH and GPG keys -> New SSH key。
2.  **账号2**：复制 `id_ed25519_163.pub` 的内容 -> 去 GitLab -> Settings -> SSH Keys。

### 第三步：配置 SSH Config (最关键的一步)

在 `~/.ssh/` 目录下创建一个名为 `config` 的文件（如果没有的话）。

```bash
# 创建或编辑 config 文件
code ~/.ssh/config
```

**将以下内容填入文件中：**

```text
# --- 账号1 (GitHub) ---
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gmail

# --- 账号2 (假设是 GitLab) ---
# 注意：这里的 Host 起了个别名，叫 gitlab-123
Host gitlab-123
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_ed25519_163
```

> **注意：** 如果两个账号都是 GitHub，你需要给第二个 Host 起个别名，比如 `Host github-123`，下面的 `HostName` 依然填 `github.com`。

### 第四步：解决用户名和邮箱自动切换 (可选但更方便)

一种方法是你每次 clone 下来后手动设置 `git config user.name`，不过这很容易忘掉这个步骤。

另一种方法就是接下来，我们可以利用 Git 的 **"includeIf"** 功能，根据文件夹路径自动切换配置。

1.  **规划文件夹**：
    *   在电脑里建一个文件夹叫 `~/Code/GMail` (放项目1)
    *   建另一个文件夹叫 `~/Code/Mail163` (放项目2)

2.  **创建特定的 .gitconfig 文件**：
    *   创建 `~/.gitconfig-gmail`，内容如下：
        ```ini
        [user]
            name = 昵称1
            email = 邮箱1@gmail.com
        ```
    *   创建 `~/.gitconfig-123`，内容如下：
        ```ini
        [user]
            name = 昵称2
            email = 邮箱2@mail.163.com
        ```

3.  **修改主配置文件 `~/.gitconfig`**：
    打开全局的 `~/.gitconfig`，在最下面加入：

    ```ini
    # 如果路径包含 ~/Code/GMail/，就引用配置1
    [includeIf "gitdir:~/Code/GMail/"]
        path = ~/.gitconfig-gmail

    # 如果路径包含 ~/Code/Mail163/，就引用配置2
    [includeIf "gitdir:~/Code/Mail163/"]
        path = ~/.gitconfig-123
    ```

### 第五步：验证效果

1.  **测试连接**：
    ```bash
    ssh -T git@github.com
    # 应该显示：Hi [账号1]! You've successfully authenticated...

    ssh -T git@gitlab-123
    # 应该显示：Welcome to GitLab, [账号2]!
    ```

2.  **平时使用**：
    *   **项目1**：正常 clone 即可。
        `git clone git@github.com:user/repo.git`
    *   **项目2**：如果你在 config 里用了别名（比如上面把公司 GitLab 叫 `gitlab-123`），clone 的时候要稍微改一下地址：
        `git clone git@gitlab-123:group/project.git`
        *(把原本的域名换成你在 config 里写的 Host 别名)*

这样配置后，只要你在 `~/Code/GMail` 目录下操作，提交记录自动就是邮箱1；在 `~/Code/Mail163` 下操作，自动就是邮箱2。完美兼容！