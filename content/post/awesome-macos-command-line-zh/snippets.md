---
title: "代码片段"
date: 2019-05-01T14:25:24+08:00
description: ""
hiddenFromHomePage: true
tags: ["Awesome"]
categories: ["Mac"]
---

> 各种有用的 Bash 风格命令，可随时复制、粘贴。
> [Awesome OS X Command Line](https://github.com/nusr/awesome-macos-command-line-zh) 的一部分。

<!--more-->

## 文本操作

### 从文本中提取唯一单词

```bash
grep -o -E '\w+' <file> | sort -u -f
```
