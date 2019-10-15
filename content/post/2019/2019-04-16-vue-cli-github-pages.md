---
title: "vue cli 3.x 项目部署到 github pages"
date: 2019-04-16T21:20:39+08:00

description: ""
tags: ["Vue", "GitHub Pages"]
categories: ["技术"]
---

github pages 是 github 免费为用户提供的服务，写博客，或者部署一些纯静态项目。

最近将 vue cli 3.x 初始化项目部署到 github pages，踩了一些坑，记录如下。

<!--more-->

## 1. vue-router 不要开启 history 模式

路径中的 **#** 比较丑，就开启了 vue-router 的 history 模式，去掉了 **#**。平时做项目也是默认开启 history 模式。折腾了半天发现，我这是部署到 github pages ，服务器没有配置，导致页面一直请求不到。

## 2. 配置 **publicUrl**

打包路径也要单独配置，否则也是请求不到页面。

比如我的项目地址是 `https://github.com/nusr/resume-vue`

我要部署到 `https://nusr.github.io/resume-vue`，那么 **publicPath** 要配置为 **/resume-vue**

```js
// vue.config.js
module.exports = {
  publicPath: process.env.NODE_ENV === "production" ? "/resume-vue" : "/"
};
```

## 3. css 引入图片错误

css 引入背景图片时，开发环境没有任何问题，但是一旦部署后，就无法获取图片了。

稍微改下 **App.vue** 的代码，暂时解决了这个问题。

```html
<!--App.vue-->
<template>
  <div id="app">
    <router-view />
  </div>
</template>
<script>
  export default {
    name: "App",
    mounted() {
      /**
       * 解决 css 引入图片在 github pages 无法获取的问题
       */
      const { NODE_ENV } = process.env;
      document.documentElement.className = NODE_ENV;
    }
  };
</script>
<style lang="less">
  @import "~@/assets/global.less";
</style>
```

给 **html** 标签配置了一个顶级类，写上不同的 css 解决了这个问题。

真时的部署环境不是这样的，部署文件夹就是根目录，但 github pages 部署文件夹不是根目录，就有这个问题。

```css
// global.less
.development {
  background-image: url(/background.png);
}

.production {
  background-image: url(/resume-vue/background.png);
}
```

## 4. 自动部署脚本

根目录下新建 **deploy.sh** 文件，文件内容如下。

```bash
# deploy.sh
# 错误时停止
set -e

# 打包
npm run build

# 进入目标文件夹
cd dist

# 提交到本地仓库

git init
git add -A
git commit -m 'deploy'

# 提交到 https://github.com:nusr/resume-vue 项目的 gh-pages 分支
git push -f git@github.com:nusr/resume-vue.git master:gh-pages

cd -
```

部署命令是 `bash deploy.sh`

## 开启 github pages

建立仓库，仓库名称是 **username.github.io** ,必须是这种格式。

比如我的用户名是 **nusr**,建立的仓库就是 **nusr.github.io**。

github pages 默认开启分支是 **gh-pages**,可以进入该仓库页面，点击 **Settings -> GitHub Pages**,切换展示的分支。

GitHub Pages 支持定制域名,支持 **jsonp** 请求。

## 代码

[代码地址](https://github.com/nusr/resume-vue)
[预览地址](https://nusr.github.io/resume-vue/)

## 参考

1. [github pages 官方说明](https://pages.github.com/)
2. [vue cli 3.x 部署官方说明](https://cli.vuejs.org/zh/guide/deployment.html#%E9%80%9A%E7%94%A8%E6%8C%87%E5%8D%97)
