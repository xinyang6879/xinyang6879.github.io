---
title: axios拦截器原理
time: 2024/3/25
categories: 
  axios
  源码
tag:   
  axios
  源码
---

## 拦截器使用

```js
import axios from "axios";

// http request 拦截器
axios.interceptors.request.use(
  (config: any) => {
    console.log("请求拦截器");
    return config;
  },
  (err) => {
    return Promise.reject(err);
  }
);

// http response 拦截器
axios.interceptors.response.use(
  (response) => {
    console.log("响应拦截器");
    return response.data;
  },
  (error) => {
    onError("系统异常！");
  }
);

axios.get(url).then((res) => {
  console.log("请求结果");
});
// 输出结果：
// 请求拦截器
// 请求结果
// 响应拦截器
```

## 入口代码

```js
// /lib/core/axios.js
function Axios(instanceConfig) {
  // 分别设置请求和响应拦截器管理
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager(),
  };
}
Axios.prototype.request = function request(configOrUrl, config) {
  // 不相关代码...

  // filter out skipped interceptors
  // 请求拦截器相关
  var requestInterceptorChain = [];
  var synchronousRequestInterceptors = true;
  this.interceptors.request.forEach(function unshiftRequestInterceptors(
    interceptor
  ) {
    if (
      typeof interceptor.runWhen === "function" &&
      interceptor.runWhen(config) === false
    ) {
      return;
    }

    synchronousRequestInterceptors =
      synchronousRequestInterceptors && interceptor.synchronous;

    requestInterceptorChain.unshift(
      interceptor.fulfilled,
      interceptor.rejected
    );
  });
  // 响应拦截器相关代码
  var responseInterceptorChain = [];
  this.interceptors.response.forEach(function pushResponseInterceptors(
    interceptor
  ) {
    responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
  });

  var promise;

  if (!synchronousRequestInterceptors) {
    var chain = [dispatchRequest, undefined];

    Array.prototype.unshift.apply(chain, requestInterceptorChain);
    chain = chain.concat(responseInterceptorChain);

    promise = Promise.resolve(config);
    while (chain.length) {
      promise = promise.then(chain.shift(), chain.shift());
    }

    return promise;
  }

  var newConfig = config;
  while (requestInterceptorChain.length) {
    var onFulfilled = requestInterceptorChain.shift();
    var onRejected = requestInterceptorChain.shift();
    try {
      newConfig = onFulfilled(newConfig);
    } catch (error) {
      onRejected(error);
      break;
    }
  }

  try {
    promise = dispatchRequest(newConfig);
  } catch (error) {
    return Promise.reject(error);
  }

  while (responseInterceptorChain.length) {
    promise = promise.then(
      responseInterceptorChain.shift(),
      responseInterceptorChain.shift()
    );
  }

  return promise;
};
```

## 拦截器管理

```js
// /lib/core/InterceptorManager.js

var utils = require("./../utils");
// 回调数组
function InterceptorManager() {
  this.handlers = [];
}

/**
 * 将新的拦截器添加到回调中
 * @param {Function} fulfilled The function to handle `then` for a `Promise`
 * @param {Function} rejected The function to handle `reject` for a `Promise`
 * @return {Number} An ID used to remove interceptor later
 */
InterceptorManager.prototype.use = function use(fulfilled, rejected, options) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected,
    synchronous: options ? options.synchronous : false,
    runWhen: options ? options.runWhen : null,
  });
  return this.handlers.length - 1;
};

/**
 * 删除拦截器
 * @param {Number} id The ID that was returned by `use`
 */
InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};

/**
 * 遍历拦截器
 * @param {Function} fn The function to call for each interceptor
 */
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};

module.exports = InterceptorManager;
```

总结：

- `InterceptorManager`是一个管理拦截器的类，它包含一个`handlers`数组，用于存储拦截器。`use`方法用于添加拦截器，`eject`方法用于删除拦截器，`forEach`方法用于遍历拦截器。
- 在使用拦截器时，调用的`use`实际上是调用的`InterceptorManager`的`use`方法

## 请求拦截器

```js
var requestInterceptorChain = [];
var synchronousRequestInterceptors = true;
// this.interceptors指向的是在文件`/lib/core/Axios.js`中Axios方法中定义的`this.interceptors`
// this.interceptors是一个InterceptorManager实例
// 在外部(项目)调用axios.interceptors.request的时候，会把对应的请求回调加入到this.interceptors.request的handlers中
// 这里的this.interceptors.request实际调用的是InterceptorManager的`forEach`方法，所以这里的foreach实际是遍历this.interceptors.request的handlers数组
this.interceptors.request.forEach(function unshiftRequestInterceptors(
  interceptor
) {
  if (
    typeof interceptor.runWhen === "function" &&
    interceptor.runWhen(config) === false
  ) {
    return;
  }
  // 判断请求拦截器中是否有异步的函数
  synchronousRequestInterceptors =
    synchronousRequestInterceptors && interceptor.synchronous;
  // 将请求拦截器的回调方法加入到requestInterceptorChain中，注意这里其实是以request的顺序倒叙了一次
  // 即如果request.handlers = [1,2,3]
  // 那么requestInterceptorChain = [3,2,1]
  requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
});
// 如果请求拦截器中有异步的函数
if (!synchronousRequestInterceptors) {
  // dispatchRequest是实际的请求
    var chain = [dispatchRequest, undefined];
    // chain组成一个[请求拦截器1成功， 请求拦截器1失败，请求拦截器2成功，请求拦截器2失败，实际请求，响应拦截器1成功，响应拦截器1失败，响应拦截器2成功，响应拦截器2失败]
    Array.prototype.unshift.apply(chain, requestInterceptorChain);
    chain = chain.concat(responseInterceptorChain);

    promise = Promise.resolve(config);
    while (chain.length) {
      // 执行顺序应该为：
      // promise.then(请求拦截器1成功， 请求拦截器1失败)
      // promise.then(请求拦截器2成功，请求拦截器2失败)
      // promise.then(实际请求, undefined)
      // promise.then(响应拦截器1成功，响应拦截器1失败)
      // promise.then(响应拦截器2成功，响应拦截器2失败)
      promise = promise.then(chain.shift(), chain.shift());
    }

    return promise;
  }


// 无关代码...
// 将原有的请求配置浅拷贝一份出来
var newConfig = config;
// 循环执行requestInterceptorChain中的回调方法;
while (requestInterceptorChain.length) {
  var onFulfilled = requestInterceptorChain.shift();
  var onRejected = requestInterceptorChain.shift();
  try {
    // 执行新的请求拦截器，并将请求拦截器的结果作为下一次执行请求拦截器的参数
    newConfig = onFulfilled(newConfig);
  } catch (error) {
    // 如果请求拦截器中报错，那么不执行后续的操作
    onRejected(error);
    break;
  }
}
// 执行请求
try {
  promise = dispatchRequest(newConfig);
} catch (error) {
  return Promise.reject(error);
}
```

总结：

- 请求拦截器的执行顺序是与书写顺序相反的
  ```js
  axios.interceptors.request.use(
    (config: any) => {
      console.log("请求拦截器1");
      return config;
    },
    (err) => {
      return Promise.reject(err);
    }
  );
  axios.interceptors.request.use(
    (config: any) => {
      console.log("请求拦截器2");
      return config;
    },
    (err) => {
      return Promise.reject(err);
    }
  );
  // 执行结果：
  // 请求拦截器2
  // 请求拦截器1
  ```
- 请求拦截器中如果报错，那么不执行后续的操作
- 如果请求拦截器中传入`synchronous=false`表示为异步请求，默认都是异步请求处理操作
- 如果是异步请求，axios是和相应拦截器一样的处理，用Promise包裹了一层
- 如果是同步请求，那么是直接循环执行的

## 响应拦截器

```js
  Axios.prototype.request = function request(configOrUrl, config) {
  // ...
  // 响应拦截器数组
  var responseInterceptorChain = [];
  // 按书写顺序将响应拦截器加入到数组中
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
  });

  // 无关代码...
  var promise;
  // 执行真实请求
  try {
    promise = dispatchRequest(newConfig);
  } catch (error) {
    return Promise.reject(error);
  }
  // 按书写顺序执行响应拦截器，并且是以异步执行的
  while (responseInterceptorChain.length) {
    promise = promise.then(responseInterceptorChain.shift(), responseInterceptorChain.shift());
  }

  return promise;
};
```
总结：
- 与请求拦截器执行顺序不同，响应拦截器按书写顺序执行
- 响应拦截器是默认用异步函数包裹了一层，而请求拦截器是将同步和异步分为两种情况的

## 参链

- [学习 axios 源码整体架构，打造属于自己的请求库](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744604&idx=1&sn=51d8d865c9848fd59f7763f5fb9ce789&chksm=88662490bf11ad86061ae76ff71a1177eeddab02c38d046eecd0e1ad25dc16f7591f91e9e3b2&scene=21#wechat_redirect)

- [axios官网](http://www.axios-js.com/zh-cn/docs/#axios-spread-callback)