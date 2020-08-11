---
title: "使用 Github Actions 部署 Hugo 博客"
date: 2020-07-22T09:47:40+08:00
---

#### 安装 Hugo

macOS 使用 brew 安装，其他系统见 [Install Hugo](https://gohugo.io/getting-started/installing)

```bash
brew install hugo
```

#### 创建博客项目

```bash
hugo new site blog && cd blog
```

#### 初始化 Git 项目

```bash
git init
```

#### 忽略掉一些构建文件

```bash
echo '.DS_Store
public
_gen' > .gitignore
```

#### 第一次提交

```bash
git add .
git commit -m 'init'
```

#### 安装一个主题

以 [meme 主题](https://github.com/reuixiy/hugo-theme-meme) 为例

```bash
git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
rm config.toml && cp themes/meme/config-examples/en/config.toml config.toml
cp themes/meme/data/Socials.toml data/Socials.toml
```

执行完以上命令安装主题，然后编辑 `config.toml` 和 `data/Socials.toml` 配置主题

#### Hello world

写篇博客看看效果。

```bash
hugo new posts/hello-world.md
```

执行以下命令，然后用编辑器打开 `posts/hello-world.md` 添加点内容。

```bash
hugo server -D
```

启动 Hugo server，浏览器访问 [http://localhost:1313](http://localhost:1313)。

`hugo new` 生成的文件带有一个`draft: true` 的标识（文件第四行），表示这是一篇还没完成的草稿。`-D` 参数会构建 `draft: true` 的文章，方便本地看效果。

文章写完之后不要忘记将 `draft: true` 改为 `false` 。

```bash
git add posts/hello-world.md
git commit -m 'hello world'
```

#### 修改 git 分支

由于 Github 的限制，形为 `username.github.io`的仓库只能使用 `master` 分支开启 `Github Pages`功能，所以把 `master` 分支留出来放 Hugo 构建出来的静态文件，另外创建一个 `md` 分支存放博客源码

```bash
git checkou	-B md
git branch -D master
```

#### 同步到 Github

根据 Github 用户名创建一个`username.github.io` 的仓库，比如我的 [hyrijk.github.io](https://github.com/hyrijk/hyrijk.github.io)。然后页面提示 push `md` 分支的代码

```bash
git remote add origin https://github.com/your-username/your-username.github.io.git
git push --set-upstream origin md
```

#### GitHub Actions 自动构建

##### 添加 Deploy keys

```shell
ssh-keygen -t rsa -b 4096 -C "your email" -f gh-pages -N ""
```

执行上面的命令会得到一对 RSA 密钥对，`gh-pages` 的内容是私钥，`gh-pages.pub` 是公钥。把**公钥**的内容粘贴到 GitHub 项目 Settings 的 Deploy Keys 里面。⚠️ 注意要把 ”Allow write access“ 勾上。保存好之后任意拥有对应的私钥的人都有权限给这个仓库提交代码了。公钥一般公开了也没关系，但是私钥一定要保管好。

![Deploy keys](https://img-1251760823.file.myqcloud.com/2020/08/20200806082752.png)

私钥我们需要给 GitHub Actions 自动提交构建文件使用，加密存储到项目的 Secrets 中，命名为 `ACTIONS_DEPLOY_KEY`。

![Secrets](https://img-1251760823.file.myqcloud.com/2020/08/20200806092105.png)

##### 添加 GitHub Actions 配置

新建一个文件 `.github/workflows/main.yml`，内容如下：

```yml
name: github pages

on:
  push:
    branches:
      - md

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0 # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.74.2"
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          publish_dir: ./public
          publish_branch: master
```

然后

```shell
git add .github/workflows/main.yml
git commit -m 'add actions config'
git push
```

完工。

如无意外，每次往 `md` 分支 `push`代码，都会触发 GitHub Actions 的自动构建，点击`Actions`可以看到构建进度，等完成之后就可以访问仓库同名 URL 了。

![](https://img-1251760823.file.myqcloud.com/2020/08/20200811083735.png)
