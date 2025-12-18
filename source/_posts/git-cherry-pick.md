---
title: git cherry-pick选择想要合并的提交
date: 2020-01-11 20:00:01
tags: [git]
categories: git
---

`git cherry-pick` 用来把某些特定提交从一个分支“拣选”到当前分支，相当于“只搬你想要的那几次提交”，而不是把整个分支合并过来。

就是说：我只要这几个 `commit`，不要整条分支历史。

---

## 一、什么是 `cherry-pick`？

简单来说，`cherry-pick` 就是将某个分支上的**特定提交（commit）**，复制并应用到当前分支上。

它与 Merge 的区别在于：
*   **Merge**：把另一个分支的所有改动一股脑合过来。
*   **Cherry-pick**：只把特定的某次（或某几次）提交合过来，其他的不要。

> **注意**：Cherry-pick 会产生一个新的提交 SHA-1 值，虽然内容一样，但在 Git 看来这是一个全新的提交。

---

## 二、什么时候用 `cherry-pick`（使用场景）

### 1. 热修复回补：把线上修复同步到其它分支
常见流程：
- 在 `release/hotfix` 上修了一个 `bug` 并发布
- 需要把同样的修复回补到 `main`、`develop`（或多个维护分支）
这时 `cherry-pick` 比 `merge` 更直接：只带走“修复提交”，不带走其它分支差异。

### 2. 挑选单个功能/提交：从别的分支拿一小段变更
例如某个 feature 分支上有一个独立的小优化提交，你当前分支也需要，但你不想把整个 feature 合并过来（因为它还没完成/包含其它改动）。

### 3. 修复“提交到了错误分支”
你本来应该把提交放到 `feature/a`，结果在 `feature/b` 上提交了：
- 用 `cherry-pick` 把提交拣到正确分支
- 然后在错误分支上回滚/删除（看团队策略）

### 4. 分支策略限制：禁止 merge，只允许线性回补
一些团队对 `release` 分支要求很严格：只允许挑选经过验证的修复提交，禁止直接合并开发分支。`cherry-pick` 很适合这种“审计式回补”。

---

## 三、基本操作指南

### 1. 挑选单个提交（最常用）

假设你想把 `feature` 分支上的提交 `a1b2c3d` 应用到 `main` 分支。

1.  **切换到目标分支**：
    ```bash
    git switch main
    ```
2.  **执行 cherry-pick**：
    ```bash
    git cherry-pick a1b2c3d
    ```

### 2. 挑选多个提交

#### 挑选不连续的多个提交：
```bash
git cherry-pick commit_id_1 commit_id_2
```
这会按顺序一次应用这两个提交。

#### 挑选连续的一段提交（区间）：
```bash
git cherry-pick start_commit_id^..end_commit_id
```
*   注意中间是两个点 `..`。
*   `start_commit_id^` 表示包含起始提交（如果不加 `^` 就不包含起始提交，这是 Git 区间的特性）。

### 3. 挑选提交但不立即提交（只拿代码）

如果你只想把改动拿过来，放在暂存区（Staged），不想自动生成 Commit，可以加 `-n` 或 `--no-commit`：

```bash
git cherry-pick -n a1b2c3d
```
*   **场景**：你想拿过来改点东西再提交，或者你想把多个 cherry-pick 的改动合并成一个新的提交。

---

## 四、冲突处理

Cherry-pick 本质上也是一种合并操作，所以**非常容易遇到冲突**。

当执行 `git cherry-pick` 遇到冲突时，Git 会暂停操作。你需要解决冲突：

1.  **查看冲突文件**：
    ```bash
    git status
    ```
2.  **手动解决冲突**（编辑代码，保留你想要的部分）。
3.  **标记冲突已解决**：
    ```bash
    git add <path/to/conflict-file>
    ```
4.  **继续执行 cherry-pick**：
    ```bash
    git cherry-pick --continue
    ```
    *(此时不需要 `git commit`，该命令会自动弹窗让你编辑提交信息并生成提交)*

#### 如果想放弃 cherry-pick：
```bash
git cherry-pick --abort
```
这会回到操作前的状态。

---

## 五、技巧与注意事项

### 1. 保留原作者信息
Cherry-pick 默认会保留原提交的作者（Author）信息，但提交者（Committer）会变成你。这通常是期望的行为。

### 2. 加上来源标记 (`-x`)
如果你想在提交信息里记录这个提交是从哪里摘过来的，可以加 `-x` 参数：
```bash
git cherry-pick -x a1b2c3d
```
提交信息末尾会自动添加一行：`(cherry picked from commit a1b2c3d...)`。
> **建议**：在团队协作中，加上 `-x` 是个好习惯，方便溯源。

### 3. 避免频繁 Cherry-pick
虽然好用，但不要滥用。如果你发现自己在频繁地在分支间倒腾提交，说明你的**分支策略**可能出了问题。频繁 Cherry-pick 会导致产生大量重复内容的提交（Duplicate Commits），增加未来合并时的冲突风险。

---

## 六、总结

| 命令                               | 作用                               |
| :--------------------------------- | :--------------------------------- |
| `git cherry-pick <commit-hash>`    | 把指定提交应用到当前分支           |
| `git cherry-pick -x <commit-hash>` | 应用并自动追加来源说明（推荐）     |
| `git cherry-pick -n <commit-hash>` | 应用改动但不生成提交（只放暂存区） |
| `git cherry-pick --continue`       | 解决冲突后继续                     |
| `git cherry-pick --abort`          | 放弃操作，回退                     |

`Cherry-pick`，能让你在处理紧急修复、特定功能迁移时非常方便。