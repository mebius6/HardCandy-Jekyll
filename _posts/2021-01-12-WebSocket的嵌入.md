---
layout: post
title: "WebSocket的嵌入"
date: 2021-01-12
tags: 随笔

color: "rgb(154,133,255)"
---

### WebSocket 是什么

> WebSocket 是一种在客户端与服务器之间保持 TCP 长连接的网络协议，这样它们就可以随时进行信息交换。提供了一个双向通讯的功能。

### WebSocket 解决了什么

> WebSocket 解决了传统的 Ajax 只能单向通讯的问题

### WebSocket 的使用

【客户端】

```js
const ws = new WebSocket("ws://localhost:6050", "echo-protocol");
ws.onopen = () => {
  this.send("connect success");
};
ws.onmessage = (event) => {
  console.log(event, "event");
};
```

【服务端】

可以参考 [npm version](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/868b2b71ca8f400ea0a966e6f17dff5e~tplv-k3u1fbpfcp-zoom-1.image)\]([badge.fury.io/js/websocke…](http://badge.fury.io/js/websocket))

### WebSocket 的封装

#### WebSocket 封装的考虑点

- 以类的方式封装
- 异常情况下的断开重连、用户手动断开则不重连
- 消息发送失败的处理，下次连接成功时发送之前失败的内容
- 订阅消息、取消订阅（需要结合发布订阅者模式）
- 根据不同的类型，处理不通的消息
- 销毁（这一步处理的不是很好、各位大佬有什么建议、可以教教小弟）

#### WebSocket 封装的实现

**ws.js**

```js
class Ws {
  // 要连接的URL
  url;
  // 一个协议字符串或一个协议字符串数组。
  // 这些字符串用来指定子协议，这样一个服务器就可以实现多个WebSocket子协议
  protocols;
  // WebSocket 实例
  ws;
  // 是否在重连中
  isReconnectionLoading = false;
  // 延时重连的 id
  timeId = null;
  // 是否是用户手动关闭连接
  isCustomClose = false;
  // 错误消息队列
  errorStack = [];
  // 消息管理中心
  eventCenter = new EventCenter();

  constructor(url, protocols) {
    this.url = url;
    this.protocols = protocols;
    this.createWs();
  }

  createWs() {
    if ("WebSocket" in window) {
      // 实例化
      this.ws = new WebSocket(this.url, this.protocols);
      // 监听事件
      this.onopen();
      this.onerror();
      this.onclose();
      this.onmessage();
    } else {
      console.log("你的浏览器不支持 WebSocket");
    }
  }

  // 监听成功
  onopen() {
    this.ws.onopen = () => {
      console.log(this.ws, "onopen");
      // 发送成功连接之前所发送失败的消息
      this.errorStack.forEach((message) => {
        this.send(message);
      });
      this.errorStack = [];
      this.isReconnectionLoading = false;
    };
  }

  // 监听错误
  onerror() {
    this.ws.onerror = (err) => {
      console.log(err, "onerror");
      this.reconnection();
      this.isReconnectionLoading = false;
    };
  }

  // 监听关闭
  onclose() {
    this.ws.onclose = () => {
      console.log("onclose");

      // 用户手动关闭的不重连
      if (this.isCustomClose) return;

      this.reconnection();
      this.isReconnectionLoading = false;
    };
  }

  // 接收 WebSocket 消息
  async onmessage() {
    this.ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        this.eventCenter.emit(data.type, data.data);
      } catch (error) {
        console.log(error, "error");
      }
    };
  }

  // 重连
  reconnection() {
    // 防止重复
    if (this.isReconnectionLoading) return;

    this.isReconnectionLoading = true;
    clearTimeout(this.timeId);
    this.timeId = setTimeout(() => {
      this.createWs();
    }, 3000);
  }

  // 发送消息
  send(message) {
    // 连接失败时的处理
    if (this.ws.readyState !== 1) {
      this.errorStack.push(message);
      return;
    }

    this.ws.send(message);
  }

  // 手动关闭
  close() {
    this.isCustomClose = true;
    this.ws.close();
  }

  // 手动开启
  start() {
    this.isCustomClose = false;
    this.reconnection();
  }

  // 订阅
  subscribe(eventName, cb) {
    this.eventCenter.on(eventName, cb);
  }

  // 取消订阅
  unsubscribe(eventName, cb) {
    this.eventCenter.off(eventName, cb);
  }

  // 销毁
  destroy() {
    this.close();
    this.ws = null;
    this.errorStack = null;
    this.eventCenter = null;
  }
}
```

#### 事件管理中心（发布订阅者模式）

**eventCenter.js**

```js
class EventCenter {
  // 通过事件类型作为属性来管理不通的事件回调
  eventStack = {};

  constructor() {
    this.eventStack = {};
  }

  on(eventName, cb) {
    const { eventStack } = this;
    const eventValue = eventStack[eventName];

    eventValue ? eventValue.push(cb) : (eventStack[eventName] = [cb]);
  }

  once(eventName, cb) {
    const { eventStack } = this;
    const eventValue = eventStack[eventName];
    // 利用闭包的形式 来模拟一次性监听的功能函数
    const tempCb = () => {
      let isOutOfDate = false;

      return () => {
        if (isOutOfDate) return;
        cb();
        isOutOfDate = true;
      };
    };

    eventValue
      ? eventValue.push(tempCb())
      : (eventStack[eventName] = [tempCb()]);
  }

  off(eventName, cb) {
    const { eventStack } = this;
    const eventValue = eventStack[eventName];

    if (!eventValue) return;

    (eventValue || []).forEach((eventCb, index) => {
      if (eventCb === cb) {
        eventValue.splice(index, 1);
      }
    });
  }

  emit(eventName, data) {
    const { eventStack } = this;
    const eventValue = eventStack[eventName];

    if (!eventValue) return;

    (eventValue || []).forEach((eventCb) => {
      eventCb(data);
    });
  }
}
```

#### 使用

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>
  <body>
    <div id="app">
      {{ msg }}
      <button @click="handleClose">close</button>
      <button @click="handleStart">start</button>

      <button @click="handleUnsubscribe">unsubscribe</button>
      <button @click="handleDestroy">destroy</button>
    </div>

    <script src="./eventCenter.js"></script>
    <script src="./ws.js"></script>
    <script>
      // 第一个连接
      const WS = new Ws("ws://localhost:6050", "echo-protocol");
      WS.send("conent success");
      const handleChat = (data) => console.log(data);
      WS.subscribe("chat", handleChat);

      // 第二个连接
      const WS2 = new Ws("ws://localhost:6050", "echo-protocol");
      WS2.send("conent success");
      const handleChat2 = (data) => console.log(data);
      WS2.subscribe("chat", handleChat2);

      let count = 0;
      let timeId = setInterval(() => {
        ++count;
        WS.send(`count: ${count}`);
      }, 1000);

      let count2 = 0;
      let timeId2 = setInterval(() => {
        ++count2;
        WS2.send(`count2: ${count2}`);

        if (count2 === 20) {
          WS2.unsubscribe("chat", handleChat2);
        }
      }, 1000);

      const APP = new Vue({
        el: "#app",
        data: {
          msg: "...",
        },
        methods: {
          handleClose() {
            clearInterval(timeId);
            WS.close();
          },
          handleStart() {
            WS.start();
          },
          handleUnsubscribe() {
            WS.unsubscribe("chat", handleChat);
          },
          handleDestroy() {
            clearInterval(timeId);
            WS.destroy();
          },
        },
      });
    </script>
  </body>
</html>
```

#### 服务端（没深入，复制的官方代码）

**serve.js**

```js
const http = require("http");
var WebSocketServer = require("websocket").server;

const app = http.createServer((req, res) => {
  console.log("...");
  res.setHeader("Content-type", "application/json");
  res.end("success");
});

app.listen(6050, () => {
  console.log("listen port 6050");
});

const wsServer = new WebSocketServer({
  httpServer: app,
  // You should not use autoAcceptConnections for production
  // applications, as it defeats all standard cross-origin protection
  // facilities built into the protocol and the browser.  You should
  // *always* verify the connection's origin and decide whether or not
  // to accept it.
  autoAcceptConnections: false,
});

function originIsAllowed(origin) {
  // put logic here to detect whether the specified origin is allowed.
  return true;
}

wsServer.on("request", function (request) {
  if (!originIsAllowed(request.origin)) {
    // Make sure we only accept requests from an allowed origin
    request.reject();
    console.log(
      new Date() + " Connection from origin " + request.origin + " rejected."
    );
    return;
  }

  var connection = request.accept("echo-protocol", request.origin);
  console.log(new Date() + " Connection accepted.");
  connection.on("message", function (message) {
    if (message.type === "utf8") {
      console.log("Received Message: " + message.utf8Data);
      connection.sendUTF(
        JSON.stringify({
          type: "chat",
          data: message.utf8Data,
        })
      );
    } else if (message.type === "binary") {
      console.log(
        "Received Binary Message of " + message.binaryData.length + " bytes"
      );
      connection.sendBytes(message.binaryData);
    }
  });
  connection.on("close", function (reasonCode, description) {
    console.log(
      new Date() + " Peer " + connection.remoteAddress + " disconnected."
    );
  });
});
```

**博文推荐**

- [笔记：Vue 常见面试题汇总及解析](https://juejin.im/post/6844904161595621384#heading-33)
- [Vue3.0 中 Object.defineProperty 的代替方案 Proxy](https://juejin.im/post/6844904150560423950)
- [vue 3.0 —— 之初体验一](https://juejin.im/post/6844904151424450573)
- [一张图搞懂原型、原型对象、原型链](https://juejin.im/post/6844904146445811720)
- [Promise 原理篇 = 从 0 到 1 构建一个 Promise](https://juejin.im/post/6844904146420645896)
- [Vue + TypeScript + Element-ui + Axios 搭建前端项目基础框架](https://github.com/liuxinwu/vue-typescript-template)
