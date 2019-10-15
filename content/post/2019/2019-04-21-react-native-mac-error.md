---
title: "解决 React-Native 在 Mac 下启动报 error code 65"
date: 2019-04-21T16:55:06+08:00

description: ""
tags: ["React-Native"]
categories: ["技术"]
---

React-Native 开发的项目，Android 方面没有任何问题，IOS 就是无法跑起来，报错信息如下：

```bash
error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. To debug build logs further, consider building your app with Xcode.app, by opening reactNative.xcodeproj
```

Mac 版本是 10.14.4 ，xcode 版本是 10.2.1

<!--more-->

## 解决方法

1. 删除项目依赖包以及 yarn 缓存

```bash
rm -rf node_modules && yarn cache clean
```

1. 重新装包

```bash
yarn install
```

1. 清除 React-Native 缓存

```bash
rm -rf ~/.rncache
```

1. 下载 React-Native IOS 运行依赖

直接运行下载脚本，若直接下载完成，后面的步骤就不用看了，直接运行项目 `react-native run-ios` 即可

```bash
node_modules/react-native/scripts/ios-install-third-party.sh
```

## 解决 IOS 运行依赖下载缓慢的问题

国内下载是相当的慢，这时候就要合理使用下载工具了。我使用的是 [proxy-down](https://github.com/proxyee-down-org/proxyee-down),开源下载工具，可以配置代理。

查看要下载的文件

```bash
cat node_modules/react-native/scripts/ios-install-third-party.sh
```

输出如下

```bash
#!/bin/bash
# 多余的东西不显示
fetch_and_unpack glog-0.3.5.tar.gz https://github.com/google/glog/archive/v0.3.5.tar.gz 61067502c5f9769d111ea1ee3f74e6ddf0a5f9cc "\"$SCRIPTDIR/ios-configure-glog.sh\""
fetch_and_unpack double-conversion-1.1.6.tar.gz https://github.com/google/double-conversion/archive/v1.1.6.tar.gz 1c7d88afde3aaeb97bb652776c627b49e132e8e0
fetch_and_unpack boost_1_63_0.tar.gz https://github.com/react-native-community/boost-for-react-native/releases/download/v1.63.0-0/boost_1_63_0.tar.gz c3f57e1d22a995e608983effbb752b54b6eab741
fetch_and_unpack folly-2018.10.22.00.tar.gz https://github.com/facebook/folly/archive/v2018.10.22.00.tar.gz f70a75bfeb394363d2049a846bba118ffb3b368a
```

四个下载链接是关键，使用工具下载这四个文件。**下载链接与 React-Native 版本紧密关联，请仔细查看文件版本。**

1. https://github.com/google/glog/archive/v0.3.5.tar.gz
1. https://github.com/google/double-conversion/archive/v1.1.6.tar.gz
1. https://github.com/react-native-community/boost-for-react-native/releases/download/v1.63.0-0/boost_1_63_0.tar.gz
1. https://github.com/facebook/folly/archive/v2018.10.22.00.tar.gz

下载完成后，新建文件夹 **.rncache**，将下载的四个文件移入到目录下

```bash
mkdir ~/.rncache
```

再次运行安装脚本，因为使用本地下载文件，所有运行起来很快。

```bash
node_modules/react-native/scripts/ios-install-third-party.sh
```

运行项目就可以跑起来了，首次启动会有点慢，耐心等会就行。

```bash
react-native run-ios
```

## 参考

1. [https://github.com/facebook/react-native/issues/20774](https://github.com/facebook/react-native/issues/20774)
