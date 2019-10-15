---
title: "一行命令更新所有 npm 依赖包"
date: 2019-04-21T12:55:27+08:00

description: ""
tags: ["Npm"]
categories: ["技术"]
---

**npm** 包的更新速度很快，为了将项目或者全局依赖更新到最新版本。传统的做法是一个一个更新，比如更新 **react** 到最新版本，命令如下：

```bash
# npm
npm i --save react@latest
# yarn
yarn add react@latest
```

**yarn** 是 facebook 发明的新一代 js 包管理器，支持离线使用。[这是 npm 与 yarn 的 命令对照](https://yarnpkg.com/zh-Hans/docs/migrating-from-npm)。
但是，这种做法相当耗时。有没有更简单的方法呢？

<!--more-->

答案是使用 [npm-check](https://github.com/dylang/npm-check) 或者 [yarn](https://yarnpkg.com/zh-Hans/)。两者都需要全局安装。

```bash
npm i -g yarn
npm i -g npm-check
```

## 使用 npm-check 更新项目依赖

在项目根目录运行

```bash
npm-check -u
```

输出如下：

```
? Choose which packages to update. (Press <space> to select)

 Update package.json to match version installed.
❯◯ chalk     ^1.1.3   ❯  2.4.2   https://github.com/chalk/chalk#readme
 ◯ cheerio   ^0.22.0  ❯  0.22.0  https://github.com/cheeriojs/cheerio#readme
 ◯ debug     ^2.3.3   ❯  4.1.1   https://github.com/visionmedia/debug#readme
 ◯ log4js    ^1.0.1   ❯  4.1.0   https://log4js-node.github.io/log4js-node/
 ◯ mustache  ^2.3.0   ❯  3.0.1   https://github.com/janl/mustache.js
 ◯ request   2.79.0   ❯  2.88.0  https://github.com/request/request#readme
 ◯ unescape  ^0.2.0   ❯  1.0.1   https://github.com/jonschlinkert/unescape
 ◯ yargs     ^6.4.0   ❯  13.2.2  https://yargs.js.org/

 Space to select. Enter to start upgrading. Control-C to cancel.
```

空格切换包是否更新，**Control + C** 取消更新，回车就是执行更新。

## 使用 yarn 更新项目依赖

在项目根目录运行

```bash
yarn upgrade-interactive  --latest
```

输出如下：

```
yarn upgrade-interactive v1.15.2
info Color legend :
 "<red>"    : Major Update backward-incompatible updates
 "<yellow>" : Minor Update backward-compatible features
 "<green>"  : Patch Update backward-compatible bug fixes
? Choose which packages to update. (Press <space> to select, <a> to toggle all,
<i> to invert selection)
 dependencies
   name      range   from       to          url
❯◯ chalk     latest  1.1.3   ❯  2.4.2       https://github.com/chalk/chalk#readm
e
 ◯ cheerio   latest  0.22.0  ❯  1.0.0-rc.3  https://github.com/cheeriojs/cheerio
#readme
 ◯ debug     latest  2.6.9   ❯  4.1.1       https://github.com/visionmedia/debug
#readme
 ◯ log4js    latest  1.1.1   ❯  4.1.0       https://log4js-node.github.io/log4js
-node/
 ◯ mustache  latest  2.3.2   ❯  3.0.1       https://github.com/janl/mustache.js
 ◯ request   latest  2.79.0  ❯  2.88.0      https://github.com/request/request#r
eadme
 ◯ unescape  latest  0.2.0   ❯  1.0.1       https://github.com/jonschlinkert/une
scape
 ◯ yargs     latest  6.6.0   ❯  13.2.2      https://yargs.js.org/
```

**yarn** 提供了全选切换功能，就是按键 **A**，空格切换包是否更新，**Control + C** 取消更新，回车就是执行更新。

**yarn** 的更新命令太长了，谁记得住，这种时候，请合理使用命令行工具的帮助，比如运行 **yarn help**。

## 更新命令对照表

更新全局依赖同上

| 说明                   | yarn                                     | npm-check       |
| ---------------------- | ---------------------------------------- | --------------- |
| 更新项目依赖，没有交互 | yarn upgrade --latest                    | npm-check -y    |
| 更新项目依赖，有交互   | yarn upgrade-interactive --latest        | npm-check -u    |
| 更新全局依赖，没有交互 | yarn global upgrade --latest             | npm-check -g -y |
| 更新全局依赖，有交互   | yarn global upgrade-interactive --latest | npm-check -g -u |

## 检测原理

**yarn** 是根据 **yarn.lock** 文件来检测版本是否是最新的，所以项目是使用 **npm** 安装依赖包，更新前要运行 `yarn install` 一下。

**npm-check** 是检测 **package.json** 文件，项目存在 **node_modules** 文件夹即可更新。

## 更新提醒

没有交互就是将依赖包直接更新到最新版本，推荐使用交互式更新，会有更新的警告信息。

**最新的依赖包，API 可能发生重大改变。为了顺利更新，更新前请 `git commit` 一下，更新失败了也能顺利回退。**

## 不推荐使用 cnpm

为了加快安装依赖的安装速度，可能被同事安利 **cnpm**，但是这样会导致包的依赖安装不正常，项目无法运行。

更好的做法是使用 [nrm](https://github.com/Pana/nrm) 切换下载源。

平时使用 **yarn** 装包，**npm** 运行脚本。

## 安装 nrm

```bash
npm i -g nrm
```

## 查看下载镜像源

```bash
nrm ls
```

输出如下

```
  npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
* taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/
```

## 切换镜像源

```bash
nrm use taobao
```

装包命令不变，比如安装 **react** 。

```bash
# npm
npm i --save react
# yarn
yarn add react
```

体验飞一般的装包速度，再也不是装包一小时，码代码五分钟。
