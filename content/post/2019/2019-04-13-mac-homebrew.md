---
title: "解决 mac HomeBrew 下载缓慢的问题"
date: 2019-04-13T15:57:25+08:00
description: ""
tags: ["HomeBrew"]
categories: ["Mac"]
---

> 文章首发 [https://nusr.github.io/](https://nusr.github.io/)

为了更好的管理安装的程序开发包，使用包管理器是最方便的。

**macOS** 自身不提供包管理器，常用的包管理器有

1. [HomeBrew](https://brew.sh/)
2. [MacPorts](https://www.macports.org/)

**MacPorts** 第一次使用要 build 整个基本库，编译时间很长。优点是不怎么依赖系统，更新 **macOS** 不会破坏现有的包，以及可以使用所有的开源包。

**HomeBrew** 尽量使用系统现有的库，减少编译时间，安装简单。缺点是与系统紧密相连。

简单优先原则，使用了 **HomeBrew**。

<!--more-->

## 安装 HomeBrew

安装 **HomeBrew**，安装命令如下

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

控制输入 `brew -v`,输出以下内容表示安装成功。

```
Homebrew 2.0.6
Homebrew/homebrew-core (git revision c432e8; last commit 2019-04-12)
Homebrew/homebrew-cask (git revision 22128; last commit 2019-04-13)
```

## HomeBrew 常用命令：

搜索包： ` brew search [package-name]`` 查看包的信息： `brew info [package-name]`安装包：`brew install [package-name]`更新所有包：`brew update`更新某个包：`brew upgrade [package-name]`卸载某个包：`brew uninstall [package-name]`列出所有安装的包：`brew list `

## 解决安装第三包缓慢

开始没有使用代理安装第三方包，下载超级慢，配置代理后，火箭般的下载速度。

比如安装 **nodejs** ,使用代理前的命令如下：

`brew install node`

使用代理后，安装命令如下：

`ALL_PROXY=socks5://127.0.0.1:1086 brew install node`

不使用镜像源的原因是，不能做到实时同步。安装报错，折腾半天，可以就是因为包有点老。

## HomeBrew Cask

**HomeBrew Cask** 是通过命令行安装、卸载软件的方式，比 **App Store** 好用。安装简单，卸载软件也十分干净。

简单点说，**HomeBrew** 安装开发环境所用的包，而 **HomeBrew Cask** 是安装编译好的应用程序(**.dmg/.pkg**)。

**HomeBrew** 默认集成了 **HomeBrew Cask**，无需额外安装 。

使用 `brew cask`，就是使用 **HomeBrew Cask** 。

命令行输入 `brew cask`,就会列出所有 **HomeBrew Cask** 的命令。

## HomeBrew Cask 安装程序

以下通过 **HomeBrew Cask** 安装 **QQ** 举例。

### 搜索 QQ

搜索 **QQ** ，使用 `brew search [app-nam]`,就会列出含有 **qq** 字段的软件。

```
$ brew search qq
==> Casks
qq ✔            qqbrowser       qqlive          qqmacmgr        qqmusic
```

### 查看软件信息

搜索出的软件特别多，不知道安装源时,使用 `brew cask info [app-name]`

```
$ brew cask info qq
qq: 6.5.3
https://im.qq.com/macqq/
/usr/local/Caskroom/qq/6.5.3 (2 files, 231.2KB)
From: https://github.com/Homebrew/homebrew-cask/blob/master/Casks/qq.rb
==> Name
QQ
==> Artifacts
QQ.app (App)
```

### 安装 QQ

可以确认时腾讯出品，放心安装,使用 `brew cask install [app-name]` 安装

`brew cask install qq`

### 升级所有程序

使用 `brew cask upgrade` 命令，就可以升级所有通过 **HomeBrew Cask** 的应用程序至最新版，这比一个一个软件点击更新，要快上许多。

### 使用代理

安装一些软件时，下载速度比较慢，就可以使用代理了。

`ALL_PROXY=socks5://127.0.0.1:1086 brew cask install qq`

## 额外说明

最好不要将 `export ALL_PROXY=socks5://127.0.0.1:1086` 写入到 **.zshrc** 或者 **.bash_profile** 等配置文件中。

因为这可能会导致其他的工具无法正常使用，每次 **HomeBrew** 使用代理前，加上 `ALL_PROXY=socks5://127.0.0.1:1086` 即可。
