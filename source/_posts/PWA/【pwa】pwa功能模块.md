---
title:【pwa】pwa功能模块
categories: PWA
---

# pwa 功能模块

## fetch

对 http 接口进行抽象，包含`Request`、`Response`、`Headers`、`Body`模块。Fetch Api 是异步化接口，所有请求接口都以`Promise`结果返回

`fetch()`方法由`Content Security Policy`的`connect-src`指令控制，而不是他请求的资源。fetch()接收到一个错误的 http 状态码时（如 4xx、5xx），并不会标记为`Promise.reject`，而是`Promise.reslove`，但返回的 Response 接口对象的 ok 属性会被置为`false`。

仅在网络出现故障或者请求被浏览器取消时才会返回`Promise.reject`
<h1></h1>

### Request

Fetch Api 的 Request 接口表示资源请求，和 fetch()方法拥有相同的构造器

- 构造器

  ```js
  const req = new Request(url, options);
  ```

- 方法

  > `clone()`: 用于创建请求对象的副本

- 例子

  ```js
  fetch(
    new Request("xxx", {
      mode: "no-cors",
    })
  );
  ```
<h1></h1>

### Headers

用于构造 Request 的 headers 属性，主要用于 http 请求和响应头的各种操作，可以通过 Request.headers 和 Response.headers 属性获取 Headers 对象

- 构造器

  ```js
  const headers = new Headers(options);
  ```

- 方法

  - append(name, value)

    将新值附加到 Headers 接口对象上，主要用于为一个属性添加多个值，如果没有，则添加该字段及值

  - delete(name):

    删除 Header 接口对象上的标头字段

  - entries():

    返回一个包含 Header 接口对象字段和值的迭代器

  - forEach(callback(values, name)):

    为 Header 接口对象的遍历器增加回调函数

  - get(name):

    获取 Header 中键为 name 的值

  - has(name):

    判断 Header 是否有键为 name 的

  - keys():

    返回 Header 接口对象所有字段的迭代器

  - set(name, value):

    为 Headers 接口对象设置字段和值

  - values():

    返回 Header 接口对象所有值的迭代器

- 例子

  ```js
  fetch(
    new Request("xxx", {
      mode: "no-cors",
      headers: new Headers({
        "Content-Type": "xxx",
      }),
    })
  );
  ```
<h1></h1>

### Response

请求响应类型为 Reponse，通过 Reponse 接口可以直接创建一个新的 Reponse 接口对象

Reponse 接口对象的正文信息只能使用一次，多次使用需要进行复制

- 构造器

  ```js
    const res = new Response(body: TBody, options)
  ```

  ```ts
  type TBody =
    | "Blob"
    | "BufferSource"
    | "FormData"
    | "ReadableStream"
    | "URLSearchParams"
    | string;
  ```

- 方法

  - clone()

    创建响应对象的副本，响应内容完全一样

  - error()

    用于返回与网络错误关联的新的 Response 对象

  - redirect(url, status)

    用于返回一个重定向到指定的 Url 的 Response 对象

- 例子

  ```js
  new Response(JSON.stringify({ data: 1 }), {
    headers: new Headers({}),
  });
  ```
<h1></h1>

### Body

由 Response 和 Request 接口对象实现，为这些接口对象提供了主体流和是否使用标志及 MIME 类型

- 方法

  - arrayBuffer():

    将 Response 流读完并返回一个 ArrayBuffer 类型的 Promise 对象

  - blob():

    将 Response 流读取完成并返回一个 blob 类型的 Promise 对象。当 Response.type=opaque 时，则生成的 Blob.size=0，Blob.type 为空，使用 URL.createObjectURL 会报错

  - formData():

    将 Response 流读取完成并返回一个 FormData 类型的 Promise 对象。

  - json():

    将 Response 流读取完成并返回一个 json 类型的 Promise 对象。如果 Response 流的内容不符合 json 字符串规则，报错

  - text():

    将 Response 流读取完成并返回一个字符串类型的 Promise 对象，使用 UTF-8 解码
<h1></h1>

## Notification

用于配置和显示用户的桌面通知。

- 必须在 https 或者本地域名下使用
<h1></h1>

### 消息展示流程

- 判断是否支持 Notification

- 支持的情况下判断是否授权，未授权会进行授权确认

- 授权成功后就可以展示通知信息
<h1></h1>

### 构造

```js
const not = new Notification(title, options);
```

- options:

  - actions: TNotificationActions[]，表示在显示通知用户可用的操作选项

    当用户选择这些操作项后，Service Worker 会通过 onnotificationclick 事件获取用户的选择操作，此属性只在 Service Worker 下有效。

    ```ts
    type TNotificationActions = {
      action: string; //显示在通知上的action标志
      title: string; //显示在通知上的action标题
      icon: string; //显示在action上的icon Url
    };
    ```

  - badge: 在没有足够空间显示消息时，显示 badge 设置的图片

  - body： 通知中显示的内容消息

  - data：用于消息通知的数据传递。通过 e.currentTarget.data 获取

  - dir：设置显示通知的方向

  - icon：消息通知中显示的图标的 url

  - lang：设置通知中的语言，必须是有效 bcp 47 语言标记

  - renotify：Boolean，新通知替换旧通知时是否通知用户，默认为 false，表示不会通知

  - requireInteraction：通知应该保持活动状态，直到用户单击或者关闭它，而不是自动关闭。必须带 tag 才有效果

  - silent：消息是否是静默通知

  - tag：给消息 tag，用于进行消息分组

  - timestamp：设置创建通知的时机

  - vibrate：收到消息通知时的震动模式。以毫秒为单位的时间数组。如[100,200,300]表示震动 100ms，暂停 200ms，然后震动 300ms
<h1></h1>

### 例子

```js
// index.html
navigator.serviceWorker.ready.then(() => {
  console.log("ready");
  //   打开授权
  Notification.requestPermission().then(() => {
    const not = new Notification("test", {
      body: "测试内容内容11111",
      icon: "qr-code-fill.png",
      requireInteraction: true,
    });
  });
});

navigator.serviceWorker.ready.then((swReg) => {
  console.log("ready");
  Notification.requestPermission().then(() => {
    // 使用actions属性，不能用new来创建了
    swReg.showNotification("test", {
      body: "测试内容内容11111",
      icon: "qr-code-fill.png",
      requireInteraction: true,
      actions: [
        {
          action: "1",
          title: "傻狗",
        },
        {
          action: "2",
          title: "laoto",
        },
      ],
    });
  });
});
```
<h1></h1>

## Sync后台同步

Sync Api可以在用户处理一些数据上传的操作时，无需关心网络环境，所有相关操作均会在合适的时机去完成数据同步

### SyncManager

- 获取

  ``` js
    navigator.serviceWorker.ready.then(swReg=>{
        const SyncManager = swReg.sync
    })
  ```

### 方法

- register(tag)
  
  注册一个Sync tag。tag值自定义，注册成功后，当网络成功时，会立即触发onsync事件

- getTags()

  获取已注册但未完成的Sync tag
<h1></h1>

### 事件

Sync注册成功后，当前网络会立即触发onsync事件，可在该事件里处理Sync tag是否完成。此事件需要在ServiceWorkerGlobalScope中监听
<h1></h1>

### Sync流程

- Registered sync: 注册sync

- Dispatched sync event：发出event事件

- Sync completed： Sync完成

Sync tag是否完成取决于SyncEvent.waitUntil中的Promise是否返回reject。如果不是reject则立即完成；如果是reject，目前chrome浏览器会最多尝试3此onsync事件的触发，每次周期间隔至少5min
<h1></h1>

## Cache

- Cache Api与其他存储的主要区别
  
  - Cache存储虽然也是键值对进行存储，主要存储流式数据，但键是Request，值为Response。

  - Cache存储是异步化的

Cache主要涉及CacheStorage接口和Cache接口
<h1></h1>

### CacheStorage

- 用于管理Cache接口对象

- 一个域名浏览器指挥创建一个CacheStorage，底层会创建相应的目录，相关的Cache会被存储在该目录下。
<h1></h1>

### Cache

- 键是Request，值为Response

- Cache数据生成后，缓存数据会一直存在，修改删除操作需要api方法调用

- Cache只能存储Get请求的Request，Cache的添加方法进行相同的请求时会覆盖之前的
<h1></h1>

### opaque响应缓存问题

不透明响应：采用mode:no-cors的不允许跨域的跨域请求，Response的类型为opaque

这类响应的status为0，body长度不可读，因此不能确认响应完整性及正确性，所以缓存下来无法查看其长度和内容

Chrome浏览器对这种响应做了一层数据填充，来保证不透明响应的数据安全性
<h1></h1>

## Push消息推送

### 涉及接口

- PushManager：推送服务器的订阅和订阅信息的获取等

- PushSubscription：订阅成功后的对象，主要用于获取订阅信息

- PushMessageData：push事件推送的消息对象

<h1></h1>

### 推送

#### 浏览器订阅

需要应用服务器的公钥，通过公钥和浏览器的用户通知得到授权，然后由浏览器向关联的消息推送服务器进行订阅，获取包含消息推送服务器的信息，最后将这些信息发送到应用服务器进行保存。

``` js
nagigator.serviceWorker.ready.then(swReg=>{
    swReg.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlB64ToUint8Array("xx")
    }).then(pushSub=>{
        fetch("https://server-address", {
            method: "post",
            bdy: JSON.stringify(pushSub.toJSON())
        })
    })
})
```
<h1></h1>

#### 应用服务器推送

应用服务器安装web push协议将消息发送到订阅信息中的消息推送服务器即可，无需关心浏览器端

<h1></h1>

#### 浏览器端接收

消息推送服务器接收到消息后会进行校验，通过后会向订阅的浏览器客户端进行消息推送。

此时将收到的消息通过Ntification Api进行展示
``` js
self.addEventListener("push", e=>{
    const data = e.data.json()
    self.registration.showNotification(data.title, {
        body: data.body
    })
})
```