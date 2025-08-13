---
title: 【云托管】socket解决h5和小程序通信问题
time: 2025-04-03 14:00:00
categories: 小程序
---

## 云托管

https://developers.weixin.qq.com/miniprogram/dev/wxcloudservice/wxcloudrun/src/

云托管是微信小程序出的，无需后端即可部署服务端

- 基于容器技术，支持用户自定义镜像或代码构建容器，提供全托管的容器服务。
- 支持常驻运行的服务形态，适合需要长期运行或有复杂依赖的传统业务
- 提供弹性扩缩、多活容灾等企业级能力
- 支持长连接
- 自定义部署支持：nodejs、GoLang、Python、Java

[收费标准](https://developers.weixin.qq.com/miniprogram/dev/wxcloudservice/wxcloudrun/src/Billing/price.html)，前三个月免费

## websocket 解决通信问题

找到小程序和 h5 的共同点，这里由于 h5 和小程序用的是同一个 token，因此使用 token 作为链接标识

小程序示例里面推荐`express-ws`

### 服务端

#### 关联 h5 和小程序

思路：

- 声明一个 Map，token 作为 key，h5 和小程序作为 value
- 从请求中拿到 token 和 env
- 如果对应 token 和 env 已经有连接了，那么需要将其关闭

```js
const WebSocket = require("ws");
const connections = new Map(); // 全局存储连接

function useH5WxConnectSocket(ws, req) {
  // 从参数获取到token和env
  const { token, env } = req.query;
  //   查看是否有历史连接
  const connection = connections.get(token) || new Map();
  console.log(
    `[连接更新] token=${token} env=${env}`,
    `新连接ID: ${ws._socket?.remotePort}`, // 查看实际连接ID
    `旧连接存在: ${connection.has(env)}`
  );
  // 清理旧连接
  if (connection.has(env)) {
    const oldWs = connection.get(env);
    connection.delete(env);
    if (oldWs.readyState === WebSocket.OPEN) {
      oldWs.close(); // 显式关闭旧连接
    }
  }
  //   设置环境对应的连接
  connection.set(env, ws);
  //   设置token对应的h5和小程序的连接
  connections.set(token, connection);
  //   socket接收到消息
  ws.on("message", function (msg) {
    console.log("message==", msg);
    sendTo(token, msg, env);
  });

  // 需要添加关闭事件监听
  ws.on("close", () => {
    // 获取到对应的环境
    const currentConnection = connections.get(token);
    if (currentConnection?.get(env) === ws) {
      // 确保只删除当前实例
      currentConnection.delete(env);
      if (currentConnection.size === 0) connections.delete(token);
    }
  });
}
```

#### 互通消息

思路：

- 接收到消息时，获取到对应的 token 和 env
- 遍历对应 token 对应的所有环境
- 除了接收到消息的 env，给其他连接发送消息

```js
// 发送给指定连接
function sendTo(token, message, excludeEnv) {
  // 拿到token对应的所有连接
  const connectionMap = connections.get(token);
  //   遍历所有连接
  for (const [env, ws] of connectionMap) {
    console.log(
      `连接状态检查 env=${env} 状态码=${ws.readyState} 连接ID: ${ws._socket?.remotePort}`
    );
    // 遍历Map结构
    if (env === excludeEnv) continue; // 跳过排除的env
    // 如果有连接且该连接处于开启状态
    if (ws && ws.readyState === WebSocket.OPEN) {
      try {
        ws.send(JSON.stringify(message));
      } catch (err) {
        console.error(`发送到env=${env}失败:`, err);
      }
    }
  }
}
```

#### 启动

在`index.js`中引入对应的 socket 文件，并启动服务

```js
const express = require("express");
const { useH5WxConnectSocket } = require("./socket");

const app = express();
// 引入socket，挂在到express上
var expressWs = require("express-ws")(app);
// 设置socket方法
app.ws("/socket", function (ws, req) {
  useH5WxConnectSocket(ws, req);
});
```

### 小程序

小程序调用云托管函数，是通过 sdk 调用的，首先调用`wx.cloud.init()`后，才能进行云托管的其他函数调用

- 初始化云托管

```js
// App.vue
wx.cloud.init();
```

- 连接 socket

[![pE6E481.png](https://s21.ax1x.com/2025/04/03/pE6E481.png)](https://imgse.com/i/pE6E481)

```js
const onConnect = () => {
    const { socketTask } = await wx.cloud.connectContainer({
    config: { env: "微信云托管环境ID" },
    service: "服务名", // 服务列表对应服务的名称
    path: `/socket?token=${Store.roleStore.getToken}&env=miniapp`, // 请求路径，参数通过url传递
    });
}

```

- 重连 socket

socket 最长 1min 后，会关闭连接，因此需要重新进行连接

```js
socketTask.onClose(async function () {
  console.log("【WEBSOCKET】全局连接关闭，3秒后重连", new Date().getTime());
  setTimeout(onConnect, 3000);
});
```

- 当 socket 相关抽离时，多个文件引入时需要用同一个连接，且 socket 接收到消息时，需要触发所有引用 socket 的 message 方法

```js
let _socketTask = null;
let messageHandlers: ((data: any) => void)[] = [];
const onConnect = async () => {
    const { socketTask } = await wx.cloud.connectContainer({
    config: { env: "微信云托管环境ID" },
    service: "服务名", // 服务列表对应服务的名称
    path: `/socket?token=${Store.roleStore.getToken}&env=miniapp`, // 请求路径，参数通过url传递
    });
    _socketTask = socketTask;
    // 接收到消息时
    _socketTask.onMessage(function (res) {
    let info = JSON.parse(JSON.parse(res.data));
    if (typeof info === "string") {
        info = JSON.parse(info);
    }
    console.log("【WEBSOCKET】收到消息----", info, typeof info);
    messageHandlers.forEach((fn) => fn(info?.type, info?.data));
    });
};
const onMessage = (handler: (data: any) => void) => {
    messageHandlers.push(handler);
    // 返回取消订阅函数
    return () => {
        messageHandlers = messageHandlers.filter((h) => h !== handler);
    };
},
```

全部代码

```js
// hooks/index
// 声明是为了防止多个页面引入同一个hooks时，每次都会生成一个新的socket连接
let _socketTask = null;
let messageHandlers: ((data: any) => void)[] = [];

export const getSocketTask = () => {
  const onOpen = () => {
    _socketTask.onOpen(async function (res) {
      console.log("【WEBSOCKET】", "链接成功！");
    });
  };
  const onError = () => {
    _socketTask.onError(async function (res) {
      console.log("【WEBSOCKET】", "连接失败!!，", res);
    });
  };
  // socket重新连接
  const onClose = () => {
    _socketTask.onClose(async function () {
      console.log("【WEBSOCKET】全局连接关闭，3秒后重连", new Date().getTime());
      setTimeout(onConnect, 3000);
    });
  };
  const onConnect = async () => {
    const { socketTask } = await wx.cloud.connectContainer({
      config: { env: "微信云托管环境ID" },
      service: "服务名", // 服务列表对应服务的名称
      path: `/socket?token=${Store.roleStore.getToken}&env=miniapp`, // 请求路径，参数通过url传递
    });
    _socketTask = socketTask;
    // 接收到消息时
    _socketTask.onMessage(function (res) {
      let info = JSON.parse(JSON.parse(res.data));
      if (typeof info === "string") {
        info = JSON.parse(info);
      }
      console.log("【WEBSOCKET】收到消息----", info, typeof info);
      messageHandlers.forEach((fn) => fn(info?.type, info?.data));
    });
    onOpen();
    onClose();
    onError();
  };

  if (!_socketTask) {
    onConnect();
  }
  const onSend = (msg) => {
    _socketTask.send({
      data: "这是小程序消息",
    });
  };

  return {
    socket: _socketTask,
    onSend,
    onMessage: (handler: (data: any) => void) => {
      messageHandlers.push(handler);
      // 返回取消订阅函数
      return () => {
        messageHandlers = messageHandlers.filter((h) => h !== handler);
      };
    },
  };
};
```

- 使用

```js
const { onMessage } = getSocketTask();
onMessage((type, data) => {
  if (type === "addMaterial") {
    console.log("onMessage", type, data, list);
  }
});
```

### h5

h5 可以直接使用 socket 连接，不需要引入 sdk，socket 的连接从在线调试-》socket 获取

```js
const url = `wss://socketURL/socket?token=${token}&env=h5`;
// 销毁旧连接（如果存在）
if (wsInstance) {
  wsInstance.close();
  wsInstance = null;
}
// 创建新实例
wsInstance = new WebSocket(url);
```
依旧需要考虑所有的事件和多文件引入使用同一个连接

全部代码：
```js
// 模块级变量（单例核心）
let wsInstance: WebSocket | null = null;
let messageHandlers: ((data: any) => void)[] = [];

const initWebSocket = (token: string) => {
  if (
    wsInstance &&
    [WebSocket.OPEN, WebSocket.CONNECTING].includes(wsInstance?.readyState)
  ) {
    return;
  }

  const url = `wss://socketURL/socket?token=${token}&env=h5`;

  // 销毁旧连接（如果存在）
  if (wsInstance) {
    wsInstance.close();
    wsInstance = null;
  }

  // 创建新实例
  wsInstance = new WebSocket(url);

  // 统一事件监听（只需注册一次）
  wsInstance.onopen = () => {
    console.log("全局连接建立成功");
  };

  wsInstance.onmessage = (evt) => {
    try {
      const data = JSON.parse(evt.data);
      messageHandlers.forEach((handler) => handler(data));
    } catch (e) {
      console.error("消息解析失败", e);
    }
  };

  wsInstance.onerror = (err) => {
    console.error("全局连接错误:", err);
  };

  wsInstance.onclose = () => {
    console.log("全局连接关闭，3秒后重连");
    setTimeout(() => initWebSocket(token), 3000);
  };
};

export const useCloudSocket = () => {
  const { token } = useGetIdsFromUrl();

  // 初始化检查
  if (!wsInstance) {
    initWebSocket(token);
  }

  wsInstance.onclose = function () {
    console.log("链接已经关闭");
    initWebSocket(token);
  };

  wsInstance.onerror = function (err) {
    console.log("socket报错信息：", err);
  };
  wsInstance.onmessage = function (evt) {
    messageHandlers.forEach((ele) => ele(evt.data));
  };

  return {
    send: (type = "init", data: any) => {
      if (wsInstance?.readyState === WebSocket.OPEN) {
        wsInstance.send(
          JSON.stringify({
            type,
            data,
          })
        );
      }
    },
    onMessage: (handler: (data: any) => void) => {
      messageHandlers.push(handler);
      // 返回取消订阅函数
      return () => {
        messageHandlers = messageHandlers.filter((h) => h !== handler);
      };
    },
    getInstance: () => wsInstance,
  };
};
```
