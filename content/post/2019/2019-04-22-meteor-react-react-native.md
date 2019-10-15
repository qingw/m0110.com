---
title: "React 与 React-Native 使用同一个 meteor 后台"
date: 2019-04-22T09:07:06+08:00

description: ""
tags: ["Meteor", "React", "React-Native"]
categories: ["技术"]
---

[meteor](https://www.meteor.com/) 可以快速构建 pc,移动端，桌面端应用。

最大的优点是：数据库的数据发生变化时，可以实时推送到前端，非常适用于实时展示的应用开发。

在 react,react-native 应用中，可以仅使用同一个 meteor 后台，实时向前端推送数据。

[github 代码地址](https://github.com/nusr/meteor)

<!--more-->

## metaor 安装

### windows 安装 meteor

官方推荐 [chocolatey](https://chocolatey.org/install) 安装 meteor。

1. 先从 [chocolatey](https://chocolatey.org/install) 安装 chocolatey
2. 然后在命令行中运行 `choco install meteor`

但是 meteor 安装速度非常慢,一顿搜索之后，找到了 Windows 下通过二进制包安装的下载地址 [https://install.meteor.com/windows](https://install.meteor.com/windows),搜索来源 [https://github.com/meteor/docs/blob/version-NEXT/long-form/alternate-windows-installation.md](https://github.com/meteor/docs/blob/version-NEXT/long-form/alternate-windows-installation.md)

### OSX/Linux 安装 meteor

安装非常简单

```bash
curl https://install.meteor.com/ | sh
```

### 验证安装

命令行输入

```bash
meteor --version
```

输出版本号，表示安装成功

```bash
Meteor 1.8.1
```

## mongodb 安装

### windows 安装 mongodb

[https://www.mongodb.com/](https://www.mongodb.com/) 下载安装包安装

### OSX 安装 mongodb

```bash
brew install mongod
```

或者下载二进制包安装

是 **mongod** ,不是 **mongodb**

### mongodb 图形界面

推荐 [https://robomongo.org/](https://robomongo.org/), 易于使用，也是免费的。

## meteor DDP

react,react-native 使用同一个 meteor 后台，所以 meteor 后台要与前端应用分开编写。

这就涉及到 meteor 中后台与前端的数据交互,meteor 中定义了一个 **[DDP 协议](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md)**。

DDP 协议定义了 meteor 后台发布数据，客户端订阅数据的操作。

本应用使用已经编写好了的 DDP 协议库，地址如下： [https://github.com/mondora/ddp.js](https://github.com/mondora/ddp.js)。

## 创建 meteor 项目

```bash
meteor create --bare [project-name]
```

更多创建参数

```bash
meteor help create
```

## meteor 连接 mongodb

meteor 项目启动命令如下：

```bash
meteor run
```

配置端口

```bash
meteor run --port 9090
```

## meteor 连接自己的 mongodb

meteor 安装包中集成了 mongodb，默认启动的是集成的 mongodb。
为了连接自己的 mongodb，需要传入参数

```bash
MONGO_URL=mongodb://username:password@localhost:27017/[database-name]?authSource=admin meteor  run --port 9090
```

刚开始没加上 **authSource=admin** 参数，一直连接不上 mongodb，加上之后就好了，根据需要加。

[更多连接参数](https://docs.mongodb.com/manual/reference/connection-string/)

## 编写 meteor 后台

```js
import { Meteor } from "meteor/meteor";
// mongodb 的 todo collection
const Todo = new Meteor.Collection("todo");
// 发布数据，前端就可以调用
Meteor.publish("todo", () => {
  return Todo.find();
});
/**
 * 定义前端调用的方法
 */
Meteor.methods({
  // 查找一条数据
  getTodo(id) {
    return Todo.findOne(id);
  },
  // 查找所有数据
  getAllTodo() {
    return Todo.find().fetch();
  },
  // 新增
  addTodo(item) {
    return Todo.insert(item);
  },
  // 删除
  removeTodo(id) {
    return Todo.remove({ _id: id });
  },
  // 编辑
  editTodo(item) {
    return Todo.update({ _id: item.id }, { $set: item });
  },
  /**
   *
   * @param {number 当前页面 从 1 开始} currentPage
   * @param {number 单次请求总条数} pageSize
   */
  getPageTodo(currentPage = 1, pageSize = 10) {
    if (page < 1) {
      return null;
    }
    // meteor 对 mongodb 的操作方法做了封装
    // 更多操作请查看 meteor 官方文档
    const total = Todo.find().count();
    const list = Todo.find(
      {},
      {
        skip: (currentPage - 1) * pageSize,
        limit: pageSize
      }
    ).fetch();
    return { total, data: list };
  }
});
// 定义对 mongodb 的操作权限
// 若没有定义，则是允许所有增删改查操作
Todo.deny({
  // 是否允许 mongodb 的新增操作， 返回 true 表示允许，否则不允许
  insert() {
    return true;
  },
  update() {
    return true;
  },
  remove() {
    return true;
  }
});

export default Todo;
```

## 前端调用

### 定义高阶组件

为了代码复用，定义了高阶组件，react 与 react-native 可以共用

```js
// meteor.js
import React, { Component } from "react";
import DDP from "ddp.js";
/**
 * meteor 连接选项
 */
const meteorOptions = {
  endpoint: "ws://192.168.31.121:9090/websocket", // react-native 不支持 localhost,127.0.0.1,请替换为自己的 IPv4 地址
  SocketConstructor: WebSocket,
  reconnectInterval: 10000, // 重连间隔
  autoConnect: true, // 是否自动连接
  autoReconnect: true // 是否自动重连
};
const PUBLIC_EVENTS = [
  // 'ready',
  // 'nosub',
  "added",
  "changed",
  "removed"
  // 'result',
  // 'updated',
  // 'error',
];
export default (WrapperComponent, { collectionName, methodName }) => {
  class MeteorWrapper extends Component {
    ddp = new DDP(meteorOptions);
    lockRequest = false;
    recordSubscriptions = {};
    state = {
      meteorList: [],
      initOver: false
    };

    componentDidMount() {
      if (!this.ddp) {
        console.error(`数据推送未连接上服务器！`);
        return;
      }
      // 添加订阅
      this.addSubscription();
    }

    componentWillUnmount() {
      // 取消订阅
      this.removeSubscription();
      // 断开连接
      this.ddp.disconnect();
    }

    getDataResult() {
      // 防止初始化请求次数过多
      if (this.lockRequest) {
        return;
      }
      this.lockRequest = true;
      const { ddp } = this;
      const self = this;
      /**
       * 调用后台定义的方法， 前端传递数组参数，meteor 后台接受到的是列表参数
       */
      ddp.method(methodName, [1, 10]);
      ddp.on("result", data => {
        const { result } = data;
        console.log(data);
        self.setState({
          meteorList: result,
          initOver: true
        });
        self.lockRequest = false;
      });
    }

    componentDidCatch(error, info) {
      console.error(error, info);
    }

    addSubscription() {
      if (!collectionName) {
        console.error("mongodb collection 为空！");
        return;
      }
      const { ddp } = this;
      const self = this;
      // 订阅数据
      self.recordSubscriptions[collectionName] = ddp.sub(collectionName);
      PUBLIC_EVENTS.forEach(event => {
        ddp.on(event, () => {
          console.log(event);
          self.getDataResult();
        });
      });
      ddp.on("error", error => {
        console.error(`服务器推送数据错误,错误消息：${error}`);
      });
      ddp.on("ready", () => {
        self.getDataResult();
      });
    }

    removeSubscription() {
      this.ddp.unsub(this.recordSubscriptions[collectionName]);
    }

    render() {
      return <WrapperComponent {...this.props} {...this.state} />;
    }
  }

  return MeteorWrapper;
};
```

### react 使用示例

```js
import React, { Component } from "react";
import { List, Skeleton } from "antd";
import "./App.css";
import MeteorWrapper from "./meteor";

function App(props) {
  const { meteorList = [], initOver } = props;
  return (
    <div className="App">
      <List
        itemLayout="horizontal"
        dataSource={meteorList}
        renderItem={item => (
          <List.Item key={item.id}>
            <Skeleton loading={!initOver} active avatar>
              <List.Item.Meta title={item.name} description={item.desc} />
            </Skeleton>
          </List.Item>
        )}
      />
    </div>
  );
}

export default MeteorWrapper(App, {
  collectionName: "todo",
  methodName: "getAllTodo"
});
```

### react-native 使用示例

```js
import React from "react";
import { StyleSheet, Text, View, FlatList } from "react-native";
import MeteorWrapper from "./meteor";

function App(props) {
  const { meteorList = [], initOver } = props;
  return (
    <View style={styles.container}>
      <FlatList
        data={meteorList}
        renderItem={({ item }) => (
          <View style={styles.item}>
            <View style={styles.name}>
              <Text>{item.name}</Text>
            </View>
            <View style={styles.desc}>
              <Text>{item.desc}</Text>
            </View>
          </View>
        )}
      />
    </View>
  );
}

export default MeteorWrapper(App, {
  collectionName: "todo",
  methodName: "getAllTodo"
});
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    fontSize: 14,
    lineHeight: 2
  },
  item: {
    padding: 10,
    borderColor: "#ccc",
    borderBottomWidth: 1,
    borderStyle: "solid"
  },
  name: {
    color: "#000",
    fontWeight: "900",
    fontSize: 24
  },
  desc: {
    color: "#666"
  }
});
```

## 开启远程调试

运行命令

```bash
adb shell input keyevent 82
```

点击 **dev setting**
然后点击 **Debug server host & port for device**
设置为 **127.0.0。1:8081**

再次运行

```bash
adb shell input keyevent 82
```

点击 **Debug js remote** ，就会自动弹出调试页面。

## IOS 运行报错

错误信息如下，请查看[解决 React-Native mac10.14.4 运行报错 error Failed to build iOS project](https://juejin.im/post/5cbc2ee5f265da035a1f1b79)

```bash
error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. To debug build logs further, consider building your app with Xcode.app, by opening reactNative.xcodeproj
```
