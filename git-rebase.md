---
title: GitRebase与交互式变基
date: 2020-01-07 20:42:01
tags: [git]
categories: git
---


在 Git 的世界里，`git rebase`（变基）大概是最常用的命令之一。

很多人因为害怕“把代码搞丢”而只敢用 `git merge`，导致提交历史里充斥着无意义的 `Merge branch 'master' into feature` 节点，或者像蚯蚓一样弯弯曲曲的分支线。

但是如果使用 `git rebase`，就会把你的提交历史变成一条优雅的直线。

rebase 的核心作用：把一串提交“搬家”到新的基底上，从而让提交历史更线性、更易读。你可以把它理解为：把分支上的提交“重放”到另一个分支（或另一个提交）之后。

---

## 一、 核心概念：Rebase 到底是在干什么？

简单来说，**Rebase = 重新（Re）定义起点（Base）。**

想象一下，你从 `master` 分支切出了一个 `feature` 分支写代码。
在你写代码的几天里，同事向 `master` 推送了新代码。此时，你的 `feature` 分支的“地基”已经过时了。

### Merge vs Rebase

*   **Merge (合并)**：保留所有历史，创建一个新的“合并节点”。就像把两条河流汇聚在一起，虽然真实，但如果合并频繁，历史线会变得像蜘蛛网一样乱。
*   **Rebase (变基)**：把你在这个分支上的所有修改“剪”下来，然后贴到 `master` 的最新位置后面。就像你时光倒流，假装你的代码是**刚刚**基于最新的 `master` 写出来的。

**结果：你的历史线变成了一条干净的直线。**

---

## 二、 场景一：同步上游代码 (不用 `-i`)

**场景**：你在开发 `feature` 分支，准备提 Pull Request，但发现 `master` 已经更新了。为了避免冲突，或者为了让提交历史好看，你需要把 `master` 的新代码同步过来。

### 操作步骤

```bash
# 1. 切换到master，把最新的代码同步过来
git checkout master
git pull

# 2. 再切换到你的开发分支
git checkout feature

# 3. 执行变基（把 feature 接到 master 的最前面）
git rebase master
```

### 可能会发生什么？

如果一切顺利，Git 会自动搞定。但通常会遇到冲突（Conflict）。此时 Git 会停下来让你解决。

**解决冲突流程：**

1.  打开代码编辑器，手动解决冲突文件。
2.  将解决后的文件放入暂存区：
    ```bash
    git add <文件名>
    ```
3.  **注意：不要 commit！** 而是告诉 rebase 继续：
    ```bash
    git rebase --continue
    ```

(如果你中途后悔了，想放弃变基，执行 `git rebase --abort` 即可回到操作前。)

---

## 三、 场景二：整理提交历史 (使用 `-i`)

这是 Rebase 最神的地方。

**场景**：你在开发某个功能，为了保存进度，你可能提交了这样的 Log：

*   `feat: 完成登录功能`
*   `fix: 修复一个拼写错误`
*   `wip: 还没写完，先存一下`
*   `fix: 刚才有个 bug 没改对`

这堆乱七八糟的提交如果推送到公司仓库，会被同事鄙视的。你需要把这 4 个提交合并成 **1 个** 完美的提交。

### 操作步骤

假设我们要整理最近的 4 次提交：

```bash
git rebase -i HEAD~4
```

这里的 `-i` 代表 **Interactive（交互式）**。

### 交互界面怎么玩？

输入命令后，Git 会自动打开 Vim（或你配置的编辑器），内容大概长这样：

```text
pick 1a2b3c4 feat: 完成登录功能
pick 5d6e7f8 fix: 修复一个拼写错误
pick 9g0h1i2 wip: 还没写完，先存一下
pick 3j4k5l6 fix: 刚才有个 bug 没改对

# Commands:
# p, pick = use commit (保留该提交)
# r, reword = use commit, but edit the commit message (保留提交，但修改注释)
# s, squash = use commit, but meld into previous commit (合并到上一个提交)
# f, fixup = like "squash", but discard this commit's log message (合并但丢弃注释)
# d, drop = remove commit (删除该提交)
```

**你的任务是修改每一行开头的单词：**

我们想把后面 3 个都“挤”进第 1 个里，所以修改如下：

```text
pick 1a2b3c4 feat: 完成登录功能
s 5d6e7f8 fix: 修复一个拼写错误
s 9g0h1i2 wip: 还没写完，先存一下
s 3j4k5l6 fix: 刚才有个 bug 没改对
```

保存并退出（Vim 中按 `Esc` 然后输入 `:wq`）。

Git 会弹出第二个编辑器窗口，让你编写合并后的 **最终 Commit Message**。修改完保存退出，你的 4 个提交就变成 1 个了！

---

## 四、 黄金法则：绝对不要 Rebase 公共分支！

这是使用 Rebase 唯一的、绝对的红线。

**原则：**
> 只有当这个分支 **只有你一个人在用**（还是本地分支）时，你才能随便 Rebase。

**为什么？**
Rebase 会修改提交的 Hash ID（因为它重写了历史）。
如果你把 `master` 分支或者同事正在开发的 `feature-shared` 分支给 Rebase 了，同事那边的代码就会和远程仓库彻底对不上号，导致灾难级的冲突。

**一句话总结：已推送到远程且有人协作的分支，老老实实用 Merge；本地自己玩的分支，大胆用 Rebase。**

---

## 五、 总结

| 需求                 | 命令                   | 效果                                        |
| :------------------- | :--------------------- | :------------------------------------------ |
| **同步 master 代码** | `git rebase master`    | 你的分支基于最新的 master，历史是一条直线。 |
| **合并琐碎提交**     | `git rebase -i HEAD~N` | 进入交互模式，使用 `squash` 压缩 N 个提交。 |
| **修改某次提交信息** | `git rebase -i HEAD~N` | 进入交互模式，使用 `reword` 修改注释。      |
| **删除某次提交**     | `git rebase -i HEAD~N` | 进入交互模式，使用 `drop` 丢弃某次提交。    |
| **变基失败想重来**   | `git rebase --abort`   | 回到变基之前的状态，无事发生。              |

学会 Git Rebase，是提高使用git效率的必经之路~
