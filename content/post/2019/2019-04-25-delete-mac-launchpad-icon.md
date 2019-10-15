---
title: "解决 Mac launchpad 启动台 Gitter 图标无法删除的问题"
date: 2019-04-25T21:12:56+08:00
description: ""
tags: ["Gitter"]
categories: ["Mac"]
---

Mac 删除应用非常简单，将应用拖到回收站就删除了。或者进入应用程序文件夹，选中程序，**command + delete** 就删除了应用，这也是删除文件的快捷键。

但是，安装 **Gitter** 后，删除了应用程序，图标一直删除不了。

我的 Mac 版本是 10.14.4 ，尝试以下方法，均无法删除 **Gitter** 图标。

1. 拖到回收站删除，无效
2. 长按图标，图标抖动后， **Gitter** 图标没有出现 **X**，无效
3. 命令行运行如下命令,无效

<!--more-->

```bash
defaults write com.apple.dock ResetLaunchPad -bool true;killall Dock
```

## 解决方法

最后找到如下命令，成功删除了 **Gitter** 图标。

```bash
sqlite3 $(find /private/var/folders \( -name com.apple.dock.launchpad -a -user $USER \) 2> /dev/null)/db/db "DELETE FROM apps WHERE title='Gitter';" && killall Dock
```

将命令中 **Gitter** 替换成你要删除的图标名称，大小写敏感，写成 **gitter** 就没有删除成功，名称换成 **Gitter** ，就删除成功了，大功告成。

## 参考

1. [https://apple.stackexchange.com/questions/144756/how-to-remove-an-icon-from-launchpad-that-does-not-appear-in-the-finder](https://apple.stackexchange.com/questions/144756/how-to-remove-an-icon-from-launchpad-that-does-not-appear-in-the-finder)

> 文章首发 [https://nusr.github.io/](https://nusr.github.io/)
