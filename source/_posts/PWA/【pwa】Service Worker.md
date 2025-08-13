---
title:【pwa】Service Worker
categories: PWA
---

# service worker

## 特性

- 是一个独立的worker线程，有自己的上下文，独立于当前网页的进程

- 安装成功后，只能手动卸载，否则将永远存在

- service worker节省性能，使用时自动唤醒，不使用时自动休眠

  属于事件驱动型Worker，Service Worker在空闲时进入空闲状态节省内存和处理器使用；当有onfetch/onasync/onmessage/onpush等事件时才会激活线程

- 必须运行在https下或者localhost下

- 注册时必须是当前域名下

## 相关接口

### ServiceWorker

对ServiceWorker线程的引用，可用于获取线程信息和向线程发送信息


### ServiceWorkerRegistration

- 对于ServiceWorker注册实例的引用，可用于注册同步消息，推送消息，通知等

- Service Worker注册成功后的实例，可以控制共享相同源的一个或者多个页面

- 该对象的持久化列表由浏览器维护

- ServiceWorkerContainer和ServiceWorkerGlobalScope可以通过`await navigator.serviceWorker.ready`、`navigator.serviceWorker.getRegistration()`、`self.registration`获取

### ServiceWorkerContaine

- window环境下用于注册、注销Service Worker线程的容器

- 对Service Worker从注册到卸载的整个流程进行控制

- 通过window.navigator.serviceWorker访问Service WorkerRegistration和Service Worker接口
### ServiceWorkerGlobalScope

- Service Worker线程中的Context

- 不支持处理同步请求，只能处理异步

## 生命周期

### 脚本生命周期

- parsed（解析成功）

 当ServiceWorkerContainer.register执行成功后，并不意味着注册的Service Worker文件已经安装或者激活了，而是注册的Service Worker文件解析完成了，符合文件同源及https协议等

- installing（正在安装）

  ServiceWorkerGlobalScope.oninstall事件被触发，可以在这个事件中做一些静态资源的缓存等操作

- installed（安装成功）

  ServiceWorkerGlobalScope.oninstall处理完成后，状态即为installed，此时新的Service Worker线程处于等待状态，可以手动调用self.skipWating或者重新打开页面进行激活

  网站第一次安装时会自动触发激活

- activating（正在激活）

 触发ServiceWorkerGlobalScope.onactivate事件，可以在这个事件中处理一些旧版本的资源删除操作。

 此状态手动调用self.clients.claim()，相关页面会立刻被新的Service Worker线程控制，并触发ServiceWorkerContainer.oncontrollerchange事件

- activated（激活成功）

  ServiceWorkerGlobalScope.onactivate事件中的处理逻辑完成后，状态变为已激活

- redundant（废弃）

  安装失败、激活失败会导致当前注册的Service Worker线程废弃。

  新的Service Worker线程激活成功会导致旧的Service Worker线程废弃


### 线程的生命周期

- STARTING（正在启动）

- RUNNING（正在运行）

- STOPPING（正在停止）

- STOPPED（已停止）


### 线程退出条件

- Service Worker文件中存在异常情况（Js语法错误、worker安装失败/激活失败、Service Worker线程执行时存在未捕获的异常）

- Service Worker线程监听事件函数是否处理完成，变为空闲状态时，Service Worker线程会自动退出

- Service Worker Js执行时间过长（js执行时间超过30s，fetch请求超过5min）

- 启动Service Worker线程的30s后，浏览器会周期性检查线程是否可以退出，关掉超过30s的线程


### 更新Service Worker文件的条件

- 线上运行的文件与浏览器运行的文件有一个字节不同

- 注册的Service Worker文件发生变化时，即使只是换了一个名字，也会认为这是一个新的文件，会触发更新

- 手动调用ServiceWorkerRegistration.update()时，浏览器会主动拉去新的Service Worker文件进行比对，比如发现不一致，那么会触发更新

- importScripts包含进来的js文件内容发生变化时，默认情况下遵循http缓存规则，也可以通过设置updateViaCache配置不走缓存

- 当安装24h后，浏览器会主动无缓存地拉取相关文件进行比较

#### 触发更新后

- 更新的线程和现在的线程一起启动，并且都有自己的install事件

- 如果新的Service Worker线程出现不正常的状态码(4xx/5xx)、解析失败、执行过程中发生错误等，浏览器会丢弃新的Service Worker线程，但当前的Service Worker线程仍处于活跃状态

- 新的Service Worker安装成功后将处于waiting状态，直到当前的Service Worker控制的client为0

- 可以使用self.skipWaiting()防止等待状态，将其立即激活
