---
title: 从零开始：使用 Docker + Hexo + Vercel + 腾讯云域名搭建个人博客
date: 2025-12-10 12:00:00
tags: [Hexo, 博客搭建, 教程, Docker]
categories: 技术折腾
---

终于拥有了自己的独立域名博客！🎉

这篇博客记录了我从零开始搭建 `catistrue.com` 的全过程。

在这场折腾中，我本来想直接用**Hexo+GitHub Page**方案。但是考虑到我有多台电脑，有时候会随机打开某一台开始工作，难道我要每一台都配置**Node.js**等等各种环境？
经过我的研究，对于我这种情况，最完美的方案是采用 “本地 Docker 预览 + GitHub Actions 自动构建 + 阿里云加速镜像” 的架构。

**这个时候，我的核心思路：CI/CD 自动化流**

不要在任何一台电脑上执行 hexo d（部署命令）。所有的部署工作都交给云端自动化完成。
    多台电脑：只负责写 Markdown 文章，然后 git push。
    GitHub：负责存储源码，并通过 GitHub Actions 自动生成静态页面。
    阿里云：作为国内访问的“加速节点”或“镜像站”。

为了保证我在任何一台电脑上都能无缝切换，同时利用阿里云加速，我开始按照 **“本地环境 -> 代码仓库 -> 自动化构建 -> 双路部署”** 的顺序来搭建。

---

### 第一阶段：初始化项目与本地环境
**目标：** 在我顺手打开的这台电脑上，用 Docker 初始化 Hexo，并建立 Git 仓库。

#### 1. 准备工作
我的电脑上已经安装了：
*   **Docker Desktop**（最好配置Docker镜像加速器，否则非常慢）
*   **Git**
*   拥有一个 GitHub 账号

#### 2. 初始化 Hexo 文件（使用 Docker）
打开终端（Windows 下建议用 PowerShell 或 Git Bash），进入我想存放博客的目录：

```bash
# 1. 创建博客目录
mkdir my-blog
cd my-blog

# 2. 使用临时 Docker 容器初始化 Hexo (无需本地安装 Node)
# 注意：这一步会下载 node 镜像并安装 hexo，可能需要一点时间
# 最好预先配置一下 Docker 镜像加速器
docker run --rm -v "$(pwd):/app" -w /app node:18-alpine sh -c "npm install hexo-cli -g && hexo init ."

# 3. 补充安装 npm 依赖
docker run --rm -v "$(pwd):/app" -w /app node:18-alpine npm install
```

#### 3. 创建本地环境配置
在 `my-blog` 根目录下创建一个 `docker-compose.yml` 文件。这是你**多台电脑同步**的核心：

```yaml
version: '3'
services:
  hexo:
    image: node:18-alpine
    container_name: hexo-dev
    working_dir: /app
    volumes:
      - ./:/app
    ports:
      - "4000:4000"
    # 启动时自动安装新依赖，并开启预览服务器
    command: sh -c "npm install && npx hexo server -p 4000 -i 0.0.0.0"
```

#### 4. 测试本地运行
在终端运行：
```bash
docker-compose up
```
访问 `http://localhost:4000`。如果看到 Hexo 的默认页面，说明本地环境搭建成功。按 `Ctrl+C` 停止。

---

### 第二阶段：推送到 GitHub 并配置自动化
**目标：** 将源码上传，并让 GitHub Actions 接管构建任务。

#### 1. 创建 GitHub 仓库
*   在 GitHub 创建一个新仓库，命名为 `你的用户名.github.io` (要设置为`Public`，我一开始设置为私有仓库，结果发现免费账号无法直接在私有仓库中使用 GitHub Pages)。

#### 2. 提交代码
在 `my-blog` 目录下：
```bash
# 生成 .gitignore (Hexo 初始化时通常已有，确认包含 node_modules, public, db.json)
git init
git branch -M main
git remote add origin <你的仓库SSH地址>
git add .
git commit -m "Initialize blog"
git push -u origin main
```

#### 3. 配置 GitHub Pages 部署流程
在项目根目录创建路径 `.github/workflows/deploy.yml`，填入以下内容。
*注意：此时我们先只配置 GitHub Pages，确保跑通后再加阿里云/腾讯云。*

```yaml
name: Deploy Blog

on:
  push:
    branches:
      - main # 监听 main 分支的改动

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install Dependencies
        run: npm install

      - name: Build Hexo
        run: npx hexo generate

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public

```

**提交这个文件并 Push。**

去 GitHub 仓库的 `Actions` 标签页，查看是否构建成功。

![如果构建失败](/images/image-1.png)

为了看更详细的失败原因，可以去 GitHub 仓库的 Actions 标签页，点击那个红色的失败任务，点击左侧的 build。
![查看详细失败原因](/images/image-2.png)

网上查了一下这个报错 `sh: 1: hexo: Permission denied` 结合 `Exit code 127`，发现是因为**把 Windows 下生成的 `node_modules` 文件夹上传到 GitHub 了。**

##### 研究了一下为什么会报错？
1.  我在本地（Windows 或 Docker 挂载的 Windows 目录）生成了 `node_modules`。
2.  Windows 下的可执行文件权限和 Linux（GitHub Actions 运行环境）是不兼容的。
3.  当我把这些文件推送到 GitHub，Actions 在 Linux 环境下尝试运行 `node_modules` 里的 `hexo` 命令时，发现文件权限不对，或者格式不对，于是报“拒绝访问”。

##### 解决方法

如果你也遇到了一样的问题，解决方法也很简单，只需要**从 Git 仓库中删除 `node_modules`，并让 GitHub Actions 自己重新下载安装。**

打开**本地电脑**的终端，进入博客目录，依次执行以下命令：

（1）第一步：修改 .gitignore
确保你的目录里有一个 `.gitignore` 文件。
如果没有，新建一个；如果有，确保里面包含 `node_modules`。
可以使用命令追加：
```powershell
# Windows PowerShell 下追加内容（如果文件不存在会自动创建）
Add-Content .gitignore "node_modules/"
Add-Content .gitignore "public/"
Add-Content .gitignore "db.json"
```

（2）第二步：从 Git 记录中删除这些文件
这一步很重要，**不要直接在文件管理器删文件夹**，我们要的是从“Git 的追踪记录”里删除，但保留你本地的文件。

```bash
# 1. 从 Git 暂存区删除 node_modules (不会删除你本地的实体文件)
git rm -r --cached node_modules

# 2. 顺便把生成的 public 也删了，源码库不需要存生成后的网页
git rm -r --cached public

# 3. 提交更改
git add .
git commit -m "Fix: remove node_modules from git tracking"
git push
```

（3）第三步：去 GitHub Actions 查看
当你执行完 `git push` 后，GitHub Actions 会自动触发一次新的构建。
这次，它在 `Install Dependencies` 这一步会下载全新的、适配 Linux 环境的依赖包，`hexo generate` 就不会报错了。

---


成功后，可以去仓库 `Settings -> Pages`，将 source 改为 `gh-pages` 分支。
此时我的博客已经可以通过 `https://catistrue.github.io/` 访问了。

---

### 第三阶段：抉择时刻：阿里云or腾讯云

鉴于我的具体需求: “个人博客 + Docker 部署 + 需要国内访问加速” ，到底是选择腾讯云还是阿里云，我纠结了一下。经过我翻看各大网友的建议，作为个人博主，我最后决定选择腾讯云，它够轻量也够用了，省下来的钱和带宽，对个人博客来说才是实打实的。

而且，我可以通过 腾讯云 DNS + Vercel ，个人版够用。

于是，我现在的架构是 “基于 Serverless 的现代化博客架构：本地 Hexo + Git 版本控制 + Vercel 边缘网络自动部署 + 腾讯云 DNS 解析” （据说是全球最流行的 JAMstack 架构，在技术圈里是非常时髦的！）（这个说法我还没有考证过，欢迎各位小伙伴解答。）


---

### 第四阶段：腾讯云购买域名的保姆级教程

整个过程大概需要 **15-20 分钟**，主要时间花在“实名认证”的审核上。

#### （1） 准备工作

1.  **账号准备**：注册并登录腾讯云账号（可以用微信直接扫码登录，最方便）。
2.  **身份认证**：这是国家规定必须做的。
    *   登录后，系统通常会提示你进行 **“实名认证”**。
    *   做个人博客，所以我选择 **“个人认证”** 
    *   *注意：这一步是绑定你的身份信息，为了合规。*

---

#### （2）挑选并购买域名

1.  **进入域名注册页面**：
    *   在腾讯云官网首页搜索栏输入 **“域名注册”**，点击进入。
    *   或者直接访问：`https://dnspod.cloud.tencent.com/`

2.  **搜索心仪的域名**：
    *   在搜索框输入你想好的名字（比如 `CatIsTrue`）。
    *   点击查询，系统会列出哪些后缀（.com / .cn / .net）还能买，以及价格。

3.  **选择与购买**：
    *   **强烈建议**：首选 **`.com`**（最通用、看着专业），如果为了便宜也可以选 `.cn`（必须实名且有些限制），或者 `.net`。
    *   看到“未注册”字样，点击右侧的 **“加入购物车”** -> **“立即购买”**。
    *   对于个人博客网站来说，只买`.com`就够了，不用看它绑定的一堆组合（省杯咖啡钱）

4.  **填写域名信息模板（关键步骤）**：
    *   购买界面会让你选择 **“域名信息模板”**。如果像我一样是第一次买，还需要点击 **“创建新模板”**。
    *   **填写信息**：填写真实的姓名、邮箱、地址、手机号。
    *   **实名核验**：这里需要再次上传你的**身份证正面照片**（只正面就行了）。
    *   提交后，腾讯云会审核这个模板（通常 1-10 分钟内完成）。
    *   *审核通过后，回到购买页面，勾选这个模板。*
    *   *注意：如果是还在审核中，是不能购买下单的，等它审核结束就行。差不多3-5分钟就完成了，审核很快*

5.  **隐私保护（重要！）**：
    *   在结算页面，留意有没有 **“域名隐私保护”** 或者 **“开启隐私保护”** 的勾选框。
    *   腾讯云现在大部分后缀是默认免费赠送并开启的，确认一下即可。这能防止你的手机号被公开查到。

6.  **支付**：
    *   确认金额（我买的时候 .com 首年 83 元左右），然后直接使用微信支付即可。

7.  **验证隐私保护设置是否开启成功（保护隐私比较重要）**
    *  购买成功后，可以打开 whois.cloud.tencent.com (或者直接百度搜 "whois查询")。
    *  输入你的域名。
    *  看查询结果里的 “注册人/Registrant”。
    *  如果显示的是你的真名 -> 没开成功。
    *  如果显示的是 “通过表单联系域名所有者” 这种乱七八糟的代码 -> 开成功了。
---

#### （3）配置解析（让域名指向你的博客）

搞定域名后告一段落！
但如果现在要访问 http://catistrue.com/ 会发现还是打不开。

这是完全正常的！别慌。

买完域名到能正常访问，中间缺了最关键的**搭桥**环节。

我们在腾讯云买了 `catistrue.com`，这就像刚领了个**车牌号**。但是：
1.  你还没把这个车牌号挂在你的**车子**（Vercel 或 GitHub）上。
2.  即使挂上了，送信的邮递员（DNS服务器）还没来得及更新地图。

现在的空白，说明这个域名**还不知道该去哪里**。

在解决这个问题前，先插播一段：我需要配置`Vercel`

因为我不想在接下来换电脑的时候反复配环境，我的预期是新的电脑只写文章 (CI/CD 自动化)利用 GitHub Actions，把“生成网页”这个苦力活交给 GitHub 的服务器去做。并且希望 **国内外都能访问**，所以我最后采用 **“GitHub 存储代码 + Vercel 自动构建/托管”** 的组合。

**为什么要这样组合？**
*   **GitHub Pages**：在国内访问经常抽风，有时候慢到打不开。
*   **Vercel**：它自带了全球 CDN（包括针对大陆地区的优化线路），速度比 GitHub Pages 快得多，而且极其稳定。
*   **自动化**：Vercel 会自动监听你的 GitHub 仓库。你只要往 GitHub 传了 `.md` 文件，Vercel 就会立刻感知到，自动在云端帮你执行 `hexo g` 生成网页并发布。**你连 GitHub Actions 脚本都不用写！**

---

下面是全流程的操作步骤

##### 第一步：在 Vercel 上“认领”你的代码
1.  去 [Vercel 官网](https://vercel.com/) 注册一个账号（直接用 GitHub 账号登录）。
2.  登录后，点击 **"Add New..."** -> **"Project"**。
3.  它会列出你 GitHub 里的仓库，找到你的 **Hexo 博客仓库**，点击 **Import**。
4.  **关键配置（Vercel 足够聪明，通常会自动填好）：**
    *   **Framework Preset**: 选 `Hexo`。
    *   **Build Command**: `hexo generate` (或者 `hexo g`)。
    *   **Output Directory**: `public`。
    *   点击 **Deploy**。

等几十秒，你会看到满屏庆祝的彩带，说明 Vercel 已经成功把你的博客在云端构建出来了！

##### 第二步：在 Vercel 端设置（让国内外都能访问）
这一步就是把刚才我们在腾讯云买的域名，指引到 Vercel 上。

1.  **在 Vercel 端设置**：
    进入你刚才创建的项目 -> **Settings** -> **Domains**。

2.  **输入框**：
    填好 `catistrue.com` （刚才在腾讯云注册的域名）

3.  **Redirect catistrue.com to www.catistrue.com (Recommended)**：
    **务必勾选（默认已经勾选了）！**

    *   **为什么要勾选？**
        这是 Vercel 的一个最佳实践。它会自动帮你配置两个域名：`catistrue.com` (根域名) 和 `www.catistrue.com` (带www的域名)。
        勾选后，当别人访问 `catistrue.com` 时，会自动跳转到 `www.catistrue.com`。这对 SEO（搜索引擎优化）和 CDN 缓存都更友好。如果不勾选，两个域名是独立的，虽然也能访问，但不够规范。


**接着直接点击右下角的黑底白字按钮 "Add" (或者 "Save") 即可！**

点完之后，它会跳回原来的界面，并显示两个红色的或者黄色的提示（Invalid Configuration），这是正常的，因为你还没去腾讯云（DNSPod）那边改解析记录。（戳开这俩提示，可以看到对应的`@`、`www`信息，后续要用）


##### 第三步：在腾讯云（DNSPod）端设置（最关键的一步）
**回到腾讯云**
*   回到腾讯云控制台 -> **DNS 解析 DNSPod**。
*   点击你的域名，添加（或修改）以下两条记录：

请务必**完全按照** Vercel提供的信息填写，不要自己发挥：

**第一条记录：给根域名 `catistrue.com` 用的**
*   **主机记录**：`@`
*   **记录类型**：`A`
*   **记录值**：`xx.xx.xx.xx` (以 Vercel 显示的为准)

**第二条记录：给 `www` 子域名用的**
*   **主机记录**：`www`
*   **记录类型**：`CNAME`
*   **记录值**：`xxx.com` (以 Vercel 显示的为准)

---

操作完毕后
回到 Vercel 这个界面，等待几分钟（有时候秒级生效，有时候要等10分钟）。

那个红色的 **Invalid Configuration** 会自动变成蓝色的 **Valid** 或者对勾。
![配置完成](/images/image-3.png)


##### 第四步：给 Hexo 本地加个保险（可选但推荐）
为了防止 Hexo 在生成的时候不知道自己的域名变了，建议修改你本地博客的配置文件。

1.  打开你电脑上博客根目录下的 `_config.yml` 文件。
2.  找到 `url:` 这一行。
3.  改成：`url: https://catistrue.com`
4.  把这个修改推送到 GitHub。

---

激动人心的时刻来了，现在，打开电脑，除了可以访问`https://catistrue.github.io/` ,还可以访问`https://catistrue.com` , `https://www.catistrue.com`。 
个人博客网站可以打开啦！（太棒了！🎉 恭喜我们！）

**ps**
如果到这一步还是能打开个人网站，不用担心，现在打不开，只剩下一个原因：SSL 证书还没颁发好，或者本地 DNS 缓存没更新。可以再等一会儿，或者在终端输入`ipconfig /flushdns`刷新一下DNS解析缓存看看。

---

### 第五阶段：日常使用流程

恭喜！我们已经完成全部的搭建过程了。

1.  **以后的工作流**：
    *   在任何电脑上写好 `xxx.md` 文章。
    *   扔进 `source/_posts` 文件夹。
    *   通过 Git 推送到 GitHub。
    *   **结束！** Vercel 会自动构建，两分钟后我的域名 `catistrue.com` 上就有新文章了。

2.  **访问体验**：
    *   **国外用户**：飞快，直接连 Vercel 的海外节点。
    *   **国内用户**：比较快，走 Vercel 优化的亚洲节点（通常是香港或新加坡），比直连 GitHub 快10倍以上。

---

如果你也想拥有一个酷炫的独立博客，希望这篇教程能帮到你。
