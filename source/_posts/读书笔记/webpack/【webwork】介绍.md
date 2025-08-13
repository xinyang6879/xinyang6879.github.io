---
title: 【webwork】介绍
categories: webwork
tag: webwork
---

## 特性

- 能长时间运行
- 启动性能理想
- 内存消耗理想

## 线程

webwork依赖于浏览器（宿主环境）来实现的，从而实现了浏览器对多线程的支持

> 种类

- dedicated worker（专用线程）：只能被首次生成它的脚本使用
- shared worker（共享线程）：可以同时被多个脚本使用

## 模式和执行流

### 模式

- 主线程提交任务

- WorkerGlobal做子任务分配，分配到某个worker

- worker子线程分配后，将处理结果返回给WorkerGlobal，WorkerGlobal再将结果返回给主线程

### 执行流

- 主线程调用new Worker，内核异步创建worker

- 浏览器内核创建WebCore：Worker

- 主线程调用postMessage通知。

如果当时worker没有创建完成，那么会在浏览器内核中的message队列存放信息，当worker创建完成后，由浏览器内核将消息从队列中放入到worker线程中的执行循环

如果worker创建已经完成了，那么主线程直接通知到worker线程中的onmessage，不会经历浏览器内核

- worker执行完成后，调用postMessage将信息传递给主线程的onmessage回调中。不经历浏览器内核

## api

### 主线程

- 构造
```js
const worker = new Worker(url, options)
```
注：
- 页面不允许启动worker，会引发security error

- 脚本类型只接收mime类型为text/javascript

- url无法解析会导致syntaxerror

