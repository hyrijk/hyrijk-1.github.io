---
title: "使用 Github Actions 部署 Hugo 博客"
date: 2020-07-22T09:47:40+08:00
draft: true
---

### 安装 Hugo

macOS 使用 brew 安装，其他系统见 [Install Hugo](https://gohugo.io/getting-started/installing)

```bash
brew install hugo
```

##### 创建博客项目

```bash
hugo new site blog && cd blog
```

##### 初始化 Git 项目

```bash
git init
```

##### 忽略掉一些构建文件

```bash
echo '.DS_Store
public
_gen' > .gitignore
```

##### 第一次提交

```bash
git add .
git commit -m 'init'
```

##### 安装一个主题

以 [meme主题](https://github.com/reuixiy/hugo-theme-meme) 为例

```bash
git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
```

