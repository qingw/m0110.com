---
title: "Node.js 种子下载器"
date: 2018-10-11T12:58:44+08:00
description: ""
tags: ["Spider", "Node.js"]
categories: ["技术"]
---

庆祝 2018 国庆，制作了一个 `Node.js` 的种子下载器。爬取页面，根据页面的链接，破解另外一个网站，下载种子文件。项目比较简单，爬取页面没有使用任何爬虫框架。[项目源码](https://github.com/nusr/FBIWarning)

<!--more-->

`Node.js` 的安装请看我的另外一篇文章，[Node.js 的多版本安装](https://www.cnblogs.com/stevexu/p/9734249.html)。

## 项目初始化

新建一个文件夹 `FBIWarning`,在该文件夹下打开命令行 `CMD` 或者 `git bash`。运行 `npm init -y`,该文件夹会生成一个 `package.json` 文件。

## 安装依赖包

安装依赖包 `cnpm install --save cheerio iconv-lite request socks5-http-client`。每个依赖包的功能如下：

- `cheerio` // 解析 DOM
- `iconv-lite` // 解决中文乱码的问题
- `request` // http 请求，图片和种子的下载
- `socks5-http-client` // socks 代理

## 爬取网页策略

网页之间，是靠链接联系在一起的，符合数据结构里面的图状结构。所以，对应有如下两种爬取策略。

1. 爬取所有列表页面的链接后，再去爬取所有详情页面，对应图算法的广度优先遍历。
2. 爬取一部分列表页面，就去爬取详情页面。然后再去爬取列表页面，爬取详情页面，循环进行，对应图算法的深度优先遍历。

因为是国外网站，网络可能随时断开，所以采用第二种策略比较好。同时，也能很快得到种子文件。为了防止重复爬取页面，可以将爬取页面的链接作为索引。

## 请求代理

网站是国外网站，需要使用梯子，否则不能爬取。[代理传送门](https://github.com/nusr/FBIWarning)。`socks5-http-client` 配合 `reqeust` 使用，可以解决代理的问题。但是，该代理只支持 `socks` 代理, `http(s)` 代理暂不支持。

## 解决中文乱码的问题

目标网站的页面编码是 `gbk` ，而 `request` 依赖包的默认编码是 `UTF-8`,使用默认编码解码方式，会导致页面的中文变成乱码。所以得到返回数据前，去掉默认编码，就是设置编码为 `encoding: null`,然后使用 `iconv-lite` 使用 `gbk` 方式解码，这样就可以解决中文编码为乱码的问题，代码如下：

```js
const request = require("request");
// 解析 dom
const cheerio = require("cheerio");
// 中文编码
const iconv = require("iconv-lite");
// 代理
const Agent = require("socks5-http-client/lib/Agent");
const COMMON_CONFIG = require("./config");
/**
 * 请求页面
 * @param {String} requestUrl 请求页面
 */
function requestPage(requestUrl) {
  try {
    return new Promise((resolve, reject) => {
      if (!requestUrl) {
        resolve(false);
      }
      request.get(
        {
          url: requestUrl,
          agentClass: Agent,
          agentOptions: {
            socksPort: 13838, // 代理端口
            socksHost: "127.0.0.1" // 代理 Host
          },
          headers: {
            "User-Agent":
              "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
          },
          // 去掉默认 utf-8 解码，否则解码会乱码
          encoding: null
        },
        function(err, response, body) {
          // 防止解析报错
          try {
            // 统一解决中文乱码的问题
            let content = iconv.decode(body, "gbk");
            let $ = cheerio.load(content);
            resolve($, err, response, body, content);
          } catch (error) {
            resolve(null);
          }
        }
      );
    });
  } catch (error) {
    //如果连续发出多个请求，即使某个请求失败，也不影响后面的其他请求
    Promise.resolve(null);
  }
}
```

## 并发请求

分页请求有很多个，可以使用递归来一个一个请求，但是写法不太好看。所以，可以使用 `ES7+` 里面的 `async` 函数，将同步过程变为异步过程。`async` 要配合 `await` 使用，就可以将同步过程变为异步过程。详细了解 `async` 请看[阮一峰 ES async](http://es6.ruanyifeng.com/#docs/async)

```js
async function innerRecursion() {
  for (let i = 1; i <= 100; i++) {
    let requestUrl = "http://www.baidu.com?page=" + i; // 事例网站，非爬取网站
    let result = await this.requestPage(url);
  }
}
```

一个一个请求比较慢，可以使用 `Promise.all` 实现并发请求。当然，也可以使用 `async` [模块](https://github.com/caolan/async) 提高下载的并发量，有需要的可以自己去了解。这个 `async` 模块并非上面的 `async` 函数。

```js
function innerRecursion() {
  let requestUrls = [];
  for (let i = 1; i <= 100; i++) {
    let requestUrl = "http://www.baidu.com?page=" + i; // 事例网站，非爬取网站
    requestUrls.push(requestUrl);
  }
  let promises = requestUrls.map(url => this.requestPage(url));
  Promise.all(promises)
    .then(results => {
      // results 是一个数组，对应上面每个请求的结果
    })
    .catch(error => {
      // 捕获请求中可能发生的错误
      console.log(error);
    });
}
```

## 图片下载

图片的下载非常简单,代码如下：

```js
/**
 * 下载文件
 * @param {String} url 请求链接
 * @param {String} filePath 文件路径
 */
function downloadFile(url, filePath) {
  // try...catch 防止一个请求出错，导致程序终止 种子的下载相同
  try {
    if (!url || !filePath) {
      return false;
    }
    request
      .get({
        url,
        agentClass: Agent,
        agentOptions: {
          socksPort: 13838, // 代理端口
          socksHost: "127.0.0.1" // 代理 Host
        },
        headers: {
          headers: {
            "User-Agent":
              "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
          }
        }
      })
      .pipe(fs.createWriteStream(filePath));
  } catch (error) {
    console.log(error);
  }
}
```

## 破解网站种子下载

解析详情页面，只能得到类似 `http://www.jandown.com?ref=VENU794` 的链接，需要破解该网站的种子下载。查看网站的种子下载方式，就是一个 `post` 请求，后端就会返回种子文件。刚开始的时候，不熟悉服务端的表单提交方式，导致文件一直得不到，后来详细查看了 `request` 的官文文档，发现是自己写错了。结合上面的图片下载，种子的下载方式自然就有了，代码如下：

```js
/**
 * 下载种子链接
 * @param {String} childDir // 子目录
 * @param {String} downloadUrl  // 下载种子地址
 */
function downloadTorrent(childDir, downloadUrl) {
  try {
    // 解析出链接的 code 值
    let code = querystring.parse(downloadUrl.split("?").pop()).ref;
    if (!code || !childDir) {
      return false;
    }
    // 发出 post 请求，然后接受文件即可
    request
      .post({
        url: "http://www.jandown.com/fetch.php",
        agentClass: Agent,
        agentOptions: {
          socksPort: 13838, // 代理端口
          socksHost: "127.0.0.1" // 代理 Host
        },
        headers: {
          "User-Agent":
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
        },
        formData: {
          code
        }
      })
      .pipe(fs.createWriteStream(childDir + "/" + code + ".torrent"));
  } catch (error) {
    console.log(error);
  }
}
```

## 面向对象

刚开始是使用面向过程的方式写的，后来发现代码太重复了，所以采用 `OOP` 改写了整个代码。详细了解 `javaScript Class` 请看[阮一峰 ES class](http://es6.ruanyifeng.com/#docs/class)

## 总结

1. 学习中文编码为乱码的解决方法
2. 学习了 `request` 的代理以及文件下载功能
3. 破解种子网站的种子下载功能
4. js 面向对象开发
5. 爬虫并发量解决

感谢阅读！
