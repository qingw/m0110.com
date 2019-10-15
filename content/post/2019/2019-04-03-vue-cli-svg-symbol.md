---
title: " @vue/cli 3.0 使用 svg-sprite-loader 加载本地 SVG 文件"
date: 2019-04-03T16:34:59+08:00

description: ""
tags: ["Vue", "Svg-Symbol"]
categories: ["技术"]
---

Vue 脚手架升级 3.0，项目也随之升级。[Vue cli 3.0](https://cli.vuejs.org/) 使用 svg-symbol(svg 雪碧图)，整个 [iconfont](https://www.iconfont.cn) 的图标库为你所用，加快开发速度。Ant-Design 也使用这种方式引入图标。

<!--more-->

## 运行

1. 装包 `yarn` or `npm i`
2. 运行 `npm run serve`

## 使用

使用 **svg-symbol** 引入图标

svg 文件会导入到 HTML 文件中，无需使用两张 png 切换

`./src/main.js` 引入 `import "./icons";`

在 `./src/icons/svg` 文件夹下的 svg 文件会自动导入项目

1. [https://www.iconfont.cn](https://www.iconfont.cn) 下载图标
2. 下载的 svg 图标放在 `./src/icons/svg` 文件夹下
3. 为了使用当前环境下的颜色，去掉 svg 的 fill 属性
4. 调用 `<svg-icon icon-name="home" class-name="test"></svg-icon>`

## 配置 **svg-sprite-loader**

```js
// ./vue.config.js
module.exports = {
  chainWebpack: config => {
    const svgRule = config.module.rule("svg");
    svgRule.uses.clear();
    svgRule
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({
        symbolId: "icon-[name]",
        include: ["./src/icons"]
      });
  }
};
```

## 调用当前环境下的颜色

> 当前环境的颜色改变，图标颜色也会跟着改变

```html
<!--SvgIcon-->
<style>
  .svg-icon {
    width: 1em;
    height: 1em;
    fill: currentColor; /*关键*/
    overflow: hidden;
  }
</style>
```

## props

| props     | 说明                     | type   |
| --------- | ------------------------ | ------ |
| iconName  | svg 文件名，不要加上后缀 | string |
| className | 图标样式类               | string |

## 代码

[online demo](https://nusr.github.io/vue-svg/dist)
[代码地址](https://github.com/nusr/vuecli3.0-svg)
