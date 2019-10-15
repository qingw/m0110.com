---
title: "解决 mac 10.14.4 无法 sublime text 3207 安装 Package Control,以及安装第三方包报错 `Package Control There are no packages available for installation`"
date: 2019-04-13T14:58:13+08:00

description: ""
tags: ["Sublime"]
categories: ["Mac"]
---

下载最新的 sublime text 3207，无法安装 **Package Control**。

根据[官方提示](https://packagecontrol.io/installation)，手动安装 **Package Control**。

手动安装 **Package Control** 后，无法安装第三方包。

一顿搜索以后，发现配置代理就可以解决这个问题。

<!--more-->

## 手动安装 Package Control

1. 下载 package control 安装包，[下载地址](https://packagecontrol.io/Package%20Control.sublime-package)。
2. 进入 sublime 的安装目录，默认安装目录是 `/Users/[userName]/Library/Application Support/Sublime Text 3`,或者点击屏幕顶部的 **Sublime Text -> Preferences -> Browse Packages**，就可以找到安装目录。
3. 将第一步下载的 **Package%20Control.sublime-package** ，移动到 sublime 安装目录下 **Installed Packages** 文件夹中
4. 重启 sublime ，**Sublime Text -> Preferences** 中就出现了 **Package Control** 的选项。

## 安装第三方包

手动安装 **Package Control** 后，**Sublime Text -> Preferences -> Package Control** ，或者使用快捷键 **command + shift + p**，就会弹出输入框。

选择 **Package Control:Install Package** ,选择任意第三方包或者搜索任意第三方包，就会弹窗提示：

```
Package Control

There are no packages available for installation

Please see https://packagecontrol.io/docs/troubleshooting for help
```

控制台错误如下(ctrl + `)：

```
Package Control: Unable to download https://packagecontrol.io/channel_v3.json after 3 attempts
```

## 配置代理

**Sublime Text -> Preferences -> Package Settings -> Package Control -> Settings-User**，加入 **"http_proxy": "127.0.0.1:1087"**。

**127.0.0.1:1087** 替换成自己可以使用的代理。

```
{
    "bootstrapped": true,
    "http_proxy": "127.0.0.1:1087",
    "installed_packages":
    [
        "Package Control"
    ]
}

```

再次安装第三方包，尽情享受装包的乐趣。
