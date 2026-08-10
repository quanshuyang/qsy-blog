---
title: Windows 系统下搭建个人博客
tags: [Hexo, GitHub, Netlify, CloudFlare, 教程]
categories: [博客搭建]
date: 2026-08-03
description: 从零到上线，手把手教你用 Hexo + GitHub + Netlify + CloudFlare 在 Windows 下搭建一个带自定义域名和 HTTPS 的免费博客。
---

本文记录我在 Windows 下从零搭建一个 Hexo 静态博客，并配合 GitHub、Netlify、CloudFlare 完成上线、配域名、加 HTTPS、CDN 加速的完整流程。

整个过程 **完全免费**（除域名年费），只要你有：一个能上网的 Windows 电脑 + 一个稳定的 VPN + 一个 Google 账号 + 一个 GitHub 账号。

---

## 0. 前置准备

### 0.1 必备工具

| 工具 | 用途 | 下载地址 |
|------|------|----------|
| VPN（稳定） | 访问 GitHub、Netlify、CloudFlare | 自备 |
| Google 账号 | 注册 Netlify / CloudFlare | https://google.com |
| GitHub 账号 | 代码托管 + 触发 Netlify 自动部署 | https://github.com |
| **Git** | 推送代码到 GitHub | https://git-scm.com/downloads/win |
| **Node.js** | Hexo 运行环境 + npm | https://nodejs.org/en/download |

> 注：较老版本的 Hexo 依赖 Go 语言环境，目前的 Hexo（v7+）已经不再需要。如果你用的版本较早，可去 https://go.dev/doc/install 安装 Go。

### 0.2 基础命令行速查（cmd）

后续操作全部在命令行中完成。打开方式：`Win + R` → 输入 `cmd` → 回车。

| 命令 | 说明 | 示例 |
|------|------|------|
| `盘符 + 冒号` | 切换盘符 | `E:` |
| `dir` | 查看当前路径下的内容 | `dir` |
| `cd 目录` | 进入单级目录 | `cd Blog` |
| `cd 目录1\目录2\...` | 进入多级目录 | `cd C:\Users\qsy\Blog` |
| `cd ..` | 退回上一级目录 | `cd ..` |
| `cd \` | 退回盘符根目录 | `cd \` |
| `cls` | 清屏 | `cls` |
| `exit` | 退出终端 | `exit` |

### 0.3 验证安装

`Win + R` → `cmd` 打开终端，依次执行：

```bash
node -v
npm -v
git --version
```

只要都出现版本号，就说明环境 OK 了。

---

## 一、安装 Hexo

Hexo 是一个基于 Node.js 的静态博客生成器，支持 Markdown 写作，生成纯静态 HTML。

全局安装 Hexo 命令行工具：

```bash
npm install hexo-cli -g
```

### 常见问题：npm 镜像源

下载过程中如果报错或速度极慢，大概率是 npm 镜像源的问题。

```bash
# 查看当前镜像源
npm config get registry

# 如果返回的不是下面两个之一，需要手动设置
npm config set registry https://registry.npmjs.org/           # 官方源（推荐，确保包最新）
npm config set registry https://registry.npmmirror.com         # 国内淘宝源（下载更快）
```

> ⚠️ **这两个源只能保留一个**，不要混用。设置好后重新执行 `npm install hexo-cli -g`。

看到正常下载日志即成功。

---

## 二、初始化博客项目

### 2.1 创建博客目录

在 C 盘用户目录下新建一个 `Blog` 文件夹。

> 建议直接放 C 盘。其他盘符可能存在权限限制，反而增加不必要折腾。

### 2.2 初始化

```bash
hexo init "C:\Users\你的用户名\Blog"
```

执行后会自动从 GitHub 克隆 `hexo-starter` 模板并安装依赖：

```
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
INFO  Install dependencies
INFO  Start blogging with Hexo!
```

看到 `Start blogging with Hexo!` 就说明初始化成功了。

### 2.3 安装其余依赖

```bash
cd "C:\Users\你的用户名\Blog"
npm install
```

### 2.4 本地预览

```bash
hexo server
```

浏览器打开 http://localhost:4000 ，看到 Hexo 默认的首页界面，说明本地已经跑起来了。

### 2.5 添加建站脚本

打开 Blog 文件夹里的 `package.json`，在 `scripts` 中加上 `"netlify"` 这一行：

```json
"scripts": {
  "build": "hexo generate",
  "clean": "hexo clean",
  "deploy": "hexo deploy",
  "server": "hexo server",
  "netlify": "npm run clean && npm run build"
}
```

这条命令本质是先清缓存再生成静态文件，后面 Netlify 建站时会直接调用它，避免缓存污染导致构建异常。

---

## 三、推送到 GitHub

### 3.1 创建 GitHub 仓库

打开 https://github.com ，右上角 `+` → `New repository`，给仓库起个名字（建议英文小写，如 `my-blog`），点击 **Create repository**。

### 3.2 本地推送到远程

```bash
# 进入博客目录（如果已在当前路径则跳过）
cd "你的博客目录路径"

# 初始化 Git 仓库
git init

# 添加所有文件
git add .

# 提交
git commit -m "my blog first commit"

# 添加远程仓库（替换为你的仓库地址）
git remote add origin https://github.com/你的用户名/你的仓库名.git

# 重命名默认分支为 main
git branch -M main

# 推送
git push -u origin main
```

### 3.3 首次推送认证

第一次推送时 GitHub 会要求登录。根据你的远程地址类型：

- **HTTPS 地址**（`https://github.com/...`）：终端提示输入 GitHub 用户名和密码（推荐用 Personal Access Token 代替密码）
- **SSH 地址**（`git@github.com:...`）：如果你配置过 SSH 密钥，直接通过

### 3.4 常见坑

**坑 1：远程仓库没添加成功**

```bash
git remote -v
```

如果没显示你的 GitHub 地址，重新执行：

```bash
git remote add origin https://github.com/你的用户名/你的仓库名.git
git push -u origin main
```

正确结果类似：

```
origin  https://github.com/quanshuyang/my-blog (fetch)
origin  https://github.com/quanshuyang/my-blog (push)
```

**坑 2：开了 VPN 也推送失败**

让 Git 走代理（端口根据你的 VPN 客户端调整，Clash 默认是 7890）：

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

然后重新 `git push -u origin main`。

认证通过后终端出现推送成功日志，说明代码已经上云了。

---

## 四、用 Netlify 建站

Netlify 是一个免费的静态网站托管平台，能自动从 GitHub 拉取代码并构建部署。**每次你往 GitHub 推送新代码，Netlify 都会自动重新构建你的博客**，真正做到「写完文章，push 就发布」。

### 4.1 注册并登录

打开 https://www.netlify.com/ ，用 GitHub 账号一键登录（需要 VPN）。

### 4.2 新建站点

1. 进入控制台，点击 `Add new site` → `Import an existing project`
2. 选择 `GitHub`，授权 Netlify 访问你的仓库
3. 选中你创建的博客仓库

### 4.3 配置构建命令

这一步很关键，填错就构建失败：

| 配置项 | 填写内容 |
|--------|----------|
| Base directory | 留空 |
| **Build command** | `npm run netlify` |
| **Publish directory** | `public` |

> `npm run netlify` 就是前面在 `package.json` 里加的那条命令，本质是 `npm run clean && npm run build`。

### 4.4 首次部署

点击 **Deploy**，等待几分钟后 Netlify 会自动构建完成，并分配一个免费域名，比如 `my-blog-xxxxx.netlify.app`。

打开这个地址，看到 Hexo 默认首页——**博客上线了！**

---

## 五、绑定自定义域名

免费的 `.netlify.app` 能用，但太长了。买个自己的短域名，花小钱换大面子。

### 5.1 购买域名

推荐**阿里云万网**：https://wanwang.aliyun.com/

选一个喜欢的后缀（`.cn`、`.com`、`.top` 等），价格从几块到几十块/年不等。

> 购买后会要求填写模板并提交审核。注意：这**不是实名认证**，完成后还要单独做域名实名认证。

### 5.2 配置 DNS 解析

阿里云域名控制台 → 域名列表 → 找到你的域名 → 解析设置：

| 主机记录 | 记录类型 | 记录值 |
|----------|----------|--------|
| `@` | CNAME | `你的站点名.netlify.app` |
| `www` | CNAME | `你的站点名.netlify.app` |

> DNS 全球同步需要时间，快则几分钟，慢则几小时，耐心等待。

### 5.3 在 Netlify 添加自定义域名

Netlify 站点设置 → `Domain management` → `Custom domains` → 添加你的域名。

因为前面已经做好 DNS 解析，一路默认下一步即可。

### 5.4 启用 Netlify 全球 CDN（可选）

在域名设置中点 `Netlify DNS` → 启用。这能让博客通过 Netlify 的海外节点分发，海外访问更快。

---

## 六、用 CloudFlare 加速

Netlify 的服务器在海外，国内直接访问有时会很慢。CloudFlare 提供免费的 CDN 加速，不需要备案。

### 6.1 注册 CloudFlare

打开 https://www.cloudflare.com/zh-cn/plans/ ，注册并登录。

### 6.2 添加站点

输入你的域名，选择 **免费套餐**。

### 6.3 添加 DNS 记录

CloudFlare 通常会自动扫描到已有记录。重点确认 **CNAME 记录**（指向 Netlify 的那条）存在，缺失的话手动补上。

### 6.4 更改名称服务器

CloudFlare 会给出两个名称服务器地址（如 `xxx.ns.cloudflare.com`），需要去域名服务商那里替换：

1. 打开阿里云域名控制台 → 域名列表
2. 找到你的域名 → DNS 修改
3. 把默认的阿里云 DNS 服务器替换为 CloudFlare 提供的地址
4. 提交

配置完成后 CloudFlare 会发邮件通知。

---

## 七、配置 HTTPS

回到 Netlify，在 `Domain settings` → `HTTPS` 中，Netlify 会通过 Let's Encrypt 自动签发证书。

等一段时间后看到绿色对勾 ✅，就说明 HTTPS 已生效。从此你的博客是 `https://你的域名` 而不是不安全的 `http`。

---

## 八、验证成果

在**不开 VPN** 的情况下访问你的域名（如 https://blog.quanshuyang.cn ），能看到 Hexo 的默认界面的博客，就说明大功告成了！

---

## 九、写在最后

到此，一个独立的、带自定义域名 + HTTPS + CDN 加速的个人博客就彻底搭建完成了。整个流程**零成本**（除了域名年费），非常适合作为长期输出的载体。

### 后续可以做什么

- **换主题**：Hexo 官方生态丰富，Butterfly、Next、Yilia 等主题在 `_config.yml` 中一行切换即可。
- **写文章**：执行 `hexo new "文章名"`，在 `source/_posts/` 下编辑 markdown 文件。
- **配评论系统**：接入 Waline、Giscus、Twikoo 等，让读者可以互动。
- **图床**：用 GitHub + jsDelivr 或腾讯云 COS 做图片托管，解决 markdown 插图问题。

Hexo 的配置、主题、插件体系非常庞大，建议多翻阅 [Hexo 官方文档](https://hexo.io/zh-cn/docs/)，遇到问题再搜索解决。

我以后也会不定期更新博客，分享学到的知识和所见所闻。**欢迎大家持续关注！**

---

## 附录：常用命令速查

| 操作 | 命令 |
|------|------|
| 新建文章 | `hexo new "标题"` |
| 本地预览 | `hexo server` |
| 清理缓存 | `hexo clean` |
| 生成静态文件 | `hexo generate` |
| 部署到 Netlify | `npm run netlify` |
| 推送代码到 GitHub | `git add . && git commit -m "msg" && git push` |

> 📌 **初次搭建一键命令合集**：
>
> ```bash
> # 环境安装
> npm install hexo-cli -g
> hexo init "C:\Users\你的用户名\Blog"
> cd "C:\Users\你的用户名\Blog"
> npm install
>
> # 本地预览
> hexo server
>
> # 推送到 GitHub
> git add .
> git commit -m "update"
> git push
> ```
