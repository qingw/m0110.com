---
title: "Hugo + Github Pages 搭建个人博客"
date: 2019-04-26T13:06:49+08:00
description: ""
tags: ["Hugo", "Blog"]
categories: ["技术"]
---

尝试过 Hexo 、GatsbyJs、 Vuepress 搭建博客后，对这些工具最大的不满，就是运行速度以及打包速度。

后来看到 [Hugo](https://gohugo.io/) ,号称最快的静态站点生成器后。

尝试搭建博客，发现不管是运行速度，还是打包速度超级快，果断将个人博客迁移到 Hugo。
Hugo 官方的定义是：

Hugo is a fast and modern static site generator written in Go, and designed to make website creation fun again.(Hugo 是使用 Go 编写的快速而现代的静态站点生成器，旨在使网站创建变得有趣。)

<!--more-->

## 安装 Hugo

Hugo 搭建个人博客前，需要先安装 Git 和 Go 语言开发环境。

分别在 [https://golang.org/dl/](https://golang.org/dl/) 、[http://git-scm.com/](http://git-scm.com/) 下载二进制安装包，点击默认设置安装即可。

### Mac 安装 Hugo

既可以使用 HomeBrew 安装，也可以在 [https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases) 下载二进制包安装

```bash
brew install hugo
```

### Windows 安装 Hugo

在 [https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases) 下载二进制包安装

安装完成后，在命令行输入以下命令验证安装：

```bash
hugo version
```

输出版本信息表示安装成功，更多安装方式查看 [https://gohugo.io/getting-started/installing](https://gohugo.io/getting-started/installing)

## 建立 Hugo 项目

一个 Hugo 项目就是一个站点，创建命令如下：

```bash
hugo new site [project-name]
```

例如我的站点名称是 **blog**,创建命令如下：

```bash
hugo new site blog
```

创建完成后，在 **blog** 文件夹下会生成以下文件结构：

```
.
├── archetypes # 存放生成博客的模版
├── assets # 存放被 Hugo Pipes 处理的文件
├── config # 存放 hugo 配置文件 支持 JSON YAML TOML 三种格式配置文件
├── content # 存放 markdown 文件
├── data # 存放 Hugo 处理的数据
├── layouts # 存放布局文件
├── static # 存放静态文件 图片 CSS JS文件
└── themes # 存放主题
```

## 添加主题

为了快速搭建博客，可以使用主题。使用主题后，只需要向 **content** 文件夹添加 Markdown 文件即可。

Hugo 有主题市场 [https://themes.gohugo.io/](https://themes.gohugo.io/) ，挑选了半天，选出了两个自认为好看的主题：

1. [https://github.com/olOwOlo/hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even)
1. [https://github.com/yoshiharuyamashita/blackburn](https://github.com/yoshiharuyamashita/blackburn)

第一个更好看、功能更加强大，选择了第一个。进入根目录，克隆主题文件就是安装主题。

```bash
cd blog
git clone https://github.com/olOwOlo/hugo-theme-even themes/even
```

## 启动 Hugo

进入 **blog/themes/even/exampleSite** 文件夹，将 **config.tom** 文件拷贝到项目根目录下，同时将 **blog/themes/even/exampleSite/content** 文件夹覆盖掉根目录下的 **content** 。

命令行输入以下命令，启动 Hugo :

```bash
hugo server
```

在浏览器打开 `http://localhost:1313/` 即可查看效果。

## 配置主题

为了个人使用，需要修改 **config.tom** 文件，根据说明修改配置即可。

[我的主题配置文件](https://github.com/nusr/blog/blob/master/config.toml)

进入 **blog/themes/even** 文件夹，会发现文件结构与新建的 Hugo 项目的文件结构几乎是一样的。这样设置是为了用户的配置可以覆盖掉主题的配置。

比如我要自定义底部的显示，**hugo-theme-even** 底部配置由 **blog/themes/even/layouts/partials/footer.html** 控制。

为了覆盖掉主题的配置，在项目根目录下新建 **blog/layouts/partials/footer.html** 文件，填入自定义内容即可覆盖掉主题配置。其他文件的覆盖是一样的。

**hugo-theme-even** 使用了 Webpack 打包 js,css,并且文件名加入 hash 值，这种 css,js 是无法覆盖的，不过 **hugo-theme-even** 配置中有覆盖这种 css,js 的参数，详情参考我的文件配置。

## 添加新博客

添加新博客命令比较简单，命令如下：

```bash
hugo new post/my-first-blog.md
```

这个命令会使用模板创建文件，首先查找用户的模板文件，没有就会查找主题的模板文件。

**hugo-theme-even** 的模版文件 **blog/themes/even/archetypes/default.md** 比较复杂，新建 **blog/archetypes/default.md** 文件覆盖掉即可。

我的模板配置如下：

```

---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
description: ""
draft: true
tags: []
categories: []
---

<!--more-->

```

**draft** 参数控制网站上该页面是否显示。设置为 false 或者去掉该参数才显示。`<!--more-->` 之前的内容会自动作为页面摘要。

## 打包

为了部署到线上，需要将 Markdown 文件打包成 HTML 文件。打包命令如下，**even** 是主题名：

```bash
hugo -t even
```

## 部署到 Github Pages

打包之后就是纯 HTML 文件，理论上所有支持部署静态页面的网站都是支持的。

我的部署命令如下，更多部署方式查看 [https://gohugo.io/hosting-and-deployment/](https://gohugo.io/hosting-and-deployment/)

```bash
#!/bin/bash
# 部署到 github pages 脚本
# 错误时终止脚本
set -e

# 删除打包文件夹
rm -rf public

# 打包。even 是主题
hugo -t even # if using a theme, replace with `hugo -t <YOURTHEME>`

# 进入打包文件夹
cd public

# Add changes to git.

git init
git add -A

# Commit changes.
msg="building site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# 推送到githu
# nusr.github.io 只能使用 master分支
git push -f git@github.com:nusr/nusr.github.io.git master

# 回到原文件夹
cd ..

```

## 代码

博客文件存放地址 [https://github.com/nusr/blog](https://github.com/nusr/blog)

> 文章首发 [https://nusr.github.io/](https://nusr.github.io/)
