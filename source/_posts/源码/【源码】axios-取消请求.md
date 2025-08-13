---
title: axios取消请求原理
time: 2024/4/3
categories: 
  axios
  源码
tag: 
  axios
  源码
---

[![pFqWBin.png](https://s21.ax1x.com/2024/04/07/pFqWBin.png)](https://imgse.com/i/pFqWBin)

[![pFqWrR0.png](https://s21.ax1x.com/2024/04/07/pFqWrR0.png)](https://imgse.com/i/pFqWrR0)

## 使用方法

```js
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios
  .get("/get/server", {
    cancelToken: source.token,
  })
  .catch(function (err) {
    if (axios.isCancel(err)) {
      console.log("请求取消", err.message);
    } else {
      // handle error
    }
  });

// 取消函数。
source.cancel("调用cancel方法");
```

## 取消类

### CancelToken

```js
// /lib/cancel/CancelToken.js

function CancelToken(executor) {
  // executor传入的函数，参数是取消函数，调用这个参数可以直接取消请求
  if (typeof executor !== "function") {
    throw new TypeError("executor must be a function.");
  }

  var resolvePromise;
  // 实例化一个Promise
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var token = this;
  // 如果执行了取消，那么就执行监听取消的函数
  this.promise.then(function (cancel) {
    if (!token._listeners) return;

    var i;
    var l = token._listeners.length;
    // 循环执行
    for (i = 0; i < l; i++) {
      token._listeners[i](cancel);
    }
    token._listeners = null;
  });

  // 这个没有在axios里面看到使用过，猜测应该是之前老axios遗留代码
  this.promise.then = function (onfulfilled) {
    var _resolve;
    var promise = new Promise(function (resolve) {
      token.subscribe(resolve);
      _resolve = resolve;
    }).then(onfulfilled);

    promise.cancel = function reject() {
      token.unsubscribe(_resolve);
    };

    return promise;
  };
  // 执行传入的函数，当source.cancel时，就执行这个回调
  executor(function cancel(message) {
    // 如果已经有reason，那么表示是已经取消请求了
    if (token.reason) {
      return;
    }
    // 把CanceledError实例作为一个reason，并且把message作为reason的message
    token.reason = new CanceledError(message);
    // 执行this.promise.then()的回调
    resolvePromise(token.reason);
  });
}

/**
 * 判断是否已经取消，因为只有reason有值时，才表示已经取消请求了
 * 如果已经取消请求，那么直接抛出错误，走入catch
 */
CancelToken.prototype.throwIfRequested = function throwIfRequested() {
  if (this.reason) {
    throw this.reason;
  }
};

/**
 * 添加取消监听函数
 */

CancelToken.prototype.subscribe = function subscribe(listener) {
  // 如果已经执行取消，那么就直接执行监听函数
  if (this.reason) {
    listener(this.reason);
    return;
  }
  // 将监听函数加入数组中
  if (this._listeners) {
    this._listeners.push(listener);
  } else {
    this._listeners = [listener];
  }
};

/**
 * 删除监听函数
 */
CancelToken.prototype.unsubscribe = function unsubscribe(listener) {
  if (!this._listeners) {
    return;
  }
  var index = this._listeners.indexOf(listener);
  if (index !== -1) {
    this._listeners.splice(index, 1);
  }
};

/**
 * Returns an object that contains a new `CancelToken` and a function that, when called,
 * cancels the `CancelToken`.
 * 调用创建取消token的函数，返回一个对象，包含一个取消token和一个取消函数
 * token是返回的函数实例，即可以直接访问函数内部
 * cancel是在CancelToken里面执行executor时，里面包裹的回调函数
 */
CancelToken.source = function source() {
  var cancel;
  // 创建一个实例
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  // 返回实例和取消调用的函数
  return {
    token: token,
    cancel: cancel,
  };
};

module.exports = CancelToken;
```

## 取消调用流程

以下是以浏览器的流程为示例

- `dispatchRequest`中判断一次此请求是否确认取消

  ```js
  // /lib/core/dispatchRequest.js
  function throwIfCancellationRequested(config) {
    // 如果有cancelToken，那么表示这个请求是可被取消的，调用cancelToken的请求取消报错即可结束
    if (config.cancelToken) {
      config.cancelToken.throwIfRequested();
    }

    if (config.signal && config.signal.aborted) {
      throw new CanceledError();
    }
  }
  ```

- 在 xhr 文件中，进行取消监听函数的添加

  ```js
  // 如果配置列表里有cancelToken，那么表示需要走取消监听
  // /lib/adapters/xhr
  if (config.cancelToken || config.signal) {
      // 取消函数的回调
      onCanceled = function(cancel) {
        // 如果这个请求已经进行完成了,那么就不走后面的函数
        if (!request) {
          return;
        }
        // 如果这个请求还没有结束或者取消,那么就使请求回调走入catch中
        reject(!cancel || (cancel && cancel.type) ? new CanceledError() : cancel);
        // 取消请求.调用xhr的取消
        request.abort();
        // 表明请求已经结束,防止重复取消
        request = null;
      };
      // 如果有cancelToken,那么就添加取消函数
      config.cancelToken && config.cancelToken.subscribe(onCanceled);
     // 这个是针对于fetch的取消处理
      if (config.signal) {
        config.signal.aborted ? onCanceled() : config.signal.addEventListener('abort', onCanceled);
      }
  }
  ```

  `onCanceled`这个函数的执行是在调用`source.cancel()`执行的
  

- 请求结束后,在 xhr 的`onloadend`/`onreadystatechange`也进行了处理

  ```js
  // 请求结束之后的回调,不管是失败还是成功都会执行
  function done() {
    // 如果有cancelToken,那么就取消监听订阅的这个函数
    if (config.cancelToken) {
      config.cancelToken.unsubscribe(onCanceled);
    }
    // fetch则是取消监听取消的函数
    if (config.signal) {
      config.signal.removeEventListener("abort", onCanceled);
    }
  }
  if ("onloadend" in request) {
    // Use onloadend if available
    request.onloadend = onloadend;
  } else {
    setTimeout(onloadend);
  }
  function onloadend() {
    // 如果请求已经取消,那么直接结束就行
    if (!request) {
      return;
    }
    var responseHeaders =
      "getAllResponseHeaders" in request
        ? parseHeaders(request.getAllResponseHeaders())
        : null;
    var responseData =
      !responseType || responseType === "text" || responseType === "json"
        ? request.responseText
        : request.response;
    var response = {
      data: responseData,
      status: request.status,
      statusText: request.statusText,
      headers: responseHeaders,
      config: config,
      request: request,
    };
    // 用请求结果去进行判断
    settle(
      function _resolve(value) {
        resolve(value);
        done();
      },
      function _reject(err) {
        reject(err);
        done();
      },
      response
    );
  }
  ```

  ```js
  // /lib/core/settle.js
  function settle(resolve, reject, response) {
    var validateStatus = response.config.validateStatus;
    // 如果请求返回状态是合理的,那么就直接resolve
    if (
      !response.status ||
      !validateStatus ||
      validateStatus(response.status)
    ) {
      resolve(response);
    } else {
      reject(
        new AxiosError(
          "Request failed with status code " + response.status,
          [AxiosError.ERR_BAD_REQUEST, AxiosError.ERR_BAD_RESPONSE][
            Math.floor(response.status / 100) - 4
          ],
          response.config,
          response.request,
          response
        )
      );
    }
  }
  ```

- 请求结束后,在`dispatchRequest`中,也有相对于取消请求的处理

  其实和开始请求之前的逻辑一样

  ```js
  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);
  });
  ```

总结: 

- 对于请求开始前和结束后,都是在`dispatchRequest`文件中调用`cancelToken.throwIfRequested()`进行请求取消的

- 对于已经开始请求的,分为两个

  - 请求结果还未返回: 调用`cancelToken.subscribe()`进行请求取消回调的**添加**

  - 请求结果已经返回: 调用`cancelToken.unsubscribe()`进行请求取消回调的**删除**

  - 取消请求回调具体是在外部调用`source.cancel`进行的,因此会存在请求结果前或者请求结果返回后都存在的情况

## 参链

- [五分钟！让你彻底搞懂axios的请求取消原理！附源码分析](https://juejin.cn/post/7284417436752265277?searchId=202403261358326BBE4B654AB52795AB71)

- [学习 axios 源码整体架构，打造属于自己的请求库](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744604&idx=1&sn=51d8d865c9848fd59f7763f5fb9ce789&chksm=88662490bf11ad86061ae76ff71a1177eeddab02c38d046eecd0e1ad25dc16f7591f91e9e3b2&scene=21#wechat_redirect)

  注:由于这篇文章时间较久,所以里面对于xhr的取消处理是之前的流程,和现在的最新版本不一致

- [AbortController](https://developer.mozilla.org/zh-CN/docs/Web/API/AbortController)

  针对于fetch的取消请求处理控制器