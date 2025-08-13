---
title: axios请求流程
time: 2024/3/22
categories: 
  axios
  源码
tag: 
  axios
  源码
---

## 入口

axios的入口是`/lib/axios.js`，实际主要的流程在`/lib/core/Axios.js`中

在`/lib/axios.js`中，创建了一个`/lib/core/Axios.js`实例，并继承了其父类的方法，其中就包括`get`/`post`/`delete`等方法

`axios`实际是从`request`方法发起全部请求的，只是封装了`get`/`post`/`delete`等方法，

``` js
// /lib/core/Axios.js
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}

// /lib/axios.js
var Axios = require('./core/Axios');
function createInstance(defaultConfig) {
  // context的prototype上已经包含了get等方法
  var context = new Axios(defaultConfig);
  // 直接调用instance时，实际就是调用了request方法
  // 这就是为什么在调用axios时，除了可以通过get/post进行请求，也可以直接作为一个函数进行直接调用
  var instance = bind(Axios.prototype.request, context);

  utils.extend(instance, Axios.prototype, context);

  // Copy context to instance
  utils.extend(instance, context);

  // Factory for creating new instances
  instance.create = function create(instanceConfig) {
    return createInstance(mergeConfig(defaultConfig, instanceConfig));
  };

  return instance;
}
```

## 请求流程

主要的请求流程都在`request`方法里，方法存在于`/lib/core/Axios.js`

``` js
// /lib/core/Axios.js
Axios.prototype.request = function request(configOrUrl, config) {
  // 适配无额外配置，直接传入url的情况
  // 比如 axios('example/url')
  // Allow for axios('example/url'[, config]) a la fetch API
  if (typeof configOrUrl === 'string') {
    config = config || {};
    config.url = configOrUrl;
  } else {
    config = configOrUrl || {};
  }
  // 将默认配置和传入的配置合并
  config = mergeConfig(this.defaults, config);

  // 设置请求方法
  if (config.method) {
    config.method = config.method.toLowerCase();
  } else if (this.defaults.method) {
    config.method = this.defaults.method.toLowerCase();
  } else {
    config.method = 'get';
  }

  var transitional = config.transitional;

  if (transitional !== undefined) {
    validator.assertOptions(transitional, {
      // 版本兼容配置-返回值转换为 Json 出错时是否置为 null 返回
      silentJSONParsing: validators.transitional(validators.boolean),
      // 版本兼容配置-responseType 设置非 json 类型时是否强制转换成 json 格式
      forcedJSONParsing: validators.transitional(validators.boolean),
      // 版本兼容配置-请求超时时是否默认返回 ETIMEDOUT 类型错
      clarifyTimeoutError: validators.transitional(validators.boolean)
    }, false);
  }

  // filter out skipped interceptors
  // 请求拦截器数组
  var requestInterceptorChain = [];
  var synchronousRequestInterceptors = true;
  // 循环访问拦截器的请求数组，将其放在请求真实执行之前
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
      return;
    }

    synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;

    requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  // 响应拦截器数组
  // 循环访问拦截器的响应数组，将其放在请求真实执行之后
  var responseInterceptorChain = [];
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
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
  // 一步步执行请求拦截器，并将其执行结果覆盖请求配置
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

  // 执行请求
  try {
    promise = dispatchRequest(newConfig);
  } catch (error) {
    return Promise.reject(error);
  }
  
  // 一步步执行响应拦截器
  while (responseInterceptorChain.length) {
    promise = promise.then(responseInterceptorChain.shift(), responseInterceptorChain.shift());
  }

  return promise;
};
```

这里包含的流程如下：
- 合并配置，包括默认配置和传入的配置
- 设置请求方法
- 请求拦截器 requestInterceptorChain
- 响应拦截器 responseInterceptorChain
- 执行请求（dispatchRequest），且是在请求拦截器之后，响应拦截器之前

注意点：
- 这里的执行是放在一个数组里面的，最开始只有实际的请求，然后在数组开头加上请求的拦截器函数，在数组最后加上请求的响应拦截器处理函数
- 拦截器的函数的参数是从上一个拦截器中返回的结果
- 默认请求头是`get`

## 派发请求

``` js
// /lib/core/dispatchRequest.js
// 派发请求
module.exports = function dispatchRequest(config) {
  // 取消请求相关
  throwIfCancellationRequested(config);

  // Ensure headers exist
  config.headers = config.headers || {};

  // Transform the data for a request or a response
  config.data = transformData.call(
    config,
    config.data,//方法参数，如果是多个方法，那么方法2的参数是由方法1的返回值决定的
    config.headers,//请求头
    config.transformRequest//执行的方法，具体是在axios里面自己定义了，在/lib/defaults.js文件中，主要是用于请求头的修改
  );

  // 合并拉平请求头
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers
  );

  // 如果是这些请求，那么就删除掉对应的请求头
  utils.forEach(
    ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
    function cleanHeaderConfig(method) {
      delete config.headers[method];
    }
  );

  var adapter = config.adapter || defaults.adapter;
  // 执行适配器
  return adapter(config).then(function onAdapterResolution(response) {
    // 取消相关请求的
    throwIfCancellationRequested(config);

    // Transform response data
    response.data = transformData.call(
      config,
      response.data,
      response.headers,
      config.transformResponse//执行的方法，具体是在axios里面自己定义了，在/lib/defaults.js文件中，主要是用于响应结果的修改
    );

    return response;
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

      // Transform response data
      if (reason && reason.response) {
        reason.response.data = transformData.call(
          config,
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }

    return Promise.reject(reason);
  });
};
```
流程：
- 统一处理请求头
- 合并拉平请求头
- 某些请求会删除请求头的`config.headers[method]`
- 执行请求
- 获取到响应数据后，统一处理响应结果

注意点：
- `['delete', 'get', 'head', 'post', 'put', 'patch', 'common']`这些请求axios会删除请求头的`config.headers[method]`
- 关于请求头的统一额外处理，在这个阶段进行
- 关于响应结果的解析，是在这个阶段执行的

## 实际请求发送

适配器解释了为什么axios可以在nodejs和浏览器里面都发送请求

``` js
// /lib/defaults/index,js
  function getDefaultAdapter() {
  var adapter;
  // 如果是浏览器的环境，那么就用的是xhr
  if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    adapter = require('../adapters/xhr');
  // 如果是非浏览器的环境（例如nodejs），那么就用的是http的文件
  } else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // For node use HTTP adapter
    adapter = require('../adapters/http');
  }
  return adapter;
}
```

``` js
// /lib/adapters/xhr.js
module.exports = function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    // 拿到请求配置项
    var requestData = config.data;
    var requestHeaders = config.headers;
    var responseType = config.responseType;
    var onCanceled;
    function done() {
      // 如果有取消的，那么走取消请求
      if (config.cancelToken) {
        config.cancelToken.unsubscribe(onCanceled);
      }

      if (config.signal) {
        config.signal.removeEventListener('abort', onCanceled);
      }
    }
    // 设置form请求的请求头
    if (utils.isFormData(requestData) && utils.isStandardBrowserEnv()) {
      delete requestHeaders['Content-Type']; // Let the browser set it
    }
    // 创建xhr请求
    var request = new XMLHttpRequest();

    // 如果有设置授权相关的，那么配置其请求头
    if (config.auth) {
      var username = config.auth.username || '';
      var password = config.auth.password ? unescape(encodeURIComponent(config.auth.password)) : '';
      requestHeaders.Authorization = 'Basic ' + btoa(username + ':' + password);
    }

    var fullPath = buildFullPath(config.baseURL, config.url);

    request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);

    // Set the request timeout in MS
    request.timeout = config.timeout;

    function onloadend() {
      if (!request) {
        return;
      }
      // Prepare the response
      var responseHeaders = 'getAllResponseHeaders' in request ? parseHeaders(request.getAllResponseHeaders()) : null;
      var responseData = !responseType || responseType === 'text' ||  responseType === 'json' ?
        request.responseText : request.response;
      var response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config: config,
        request: request
      };

      settle(function _resolve(value) {
        resolve(value);
        done();
      }, function _reject(err) {
        reject(err);
        done();
      }, response);

      // Clean up request
      request = null;
    }

    if ('onloadend' in request) {
      // Use onloadend if available
      request.onloadend = onloadend;
    } else {
      // Listen for ready state to emulate onloadend
      request.onreadystatechange = function handleLoad() {
        if (!request || request.readyState !== 4) {
          return;
        }

        // The request errored out and we didn't get a response, this will be
        // handled by onerror instead
        // With one exception: request that using file: protocol, most browsers
        // will return status as 0 even though it's a successful request
        if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
          return;
        }
        // readystate handler is calling before onerror or ontimeout handlers,
        // so we should call onloadend on the next 'tick'
        setTimeout(onloadend);
      };
    }

    // Handle browser request cancellation (as opposed to a manual cancellation)
    request.onabort = function handleAbort() {
      if (!request) {
        return;
      }

      reject(new AxiosError('Request aborted', AxiosError.ECONNABORTED, config, request));

      // Clean up request
      request = null;
    };

    // Handle low level network errors
    request.onerror = function handleError() {
      // Real errors are hidden from us by the browser
      // onerror should only fire if it's a network error
      reject(new AxiosError('Network Error', AxiosError.ERR_NETWORK, config, request, request));

      // Clean up request
      request = null;
    };

    // Handle timeout
    request.ontimeout = function handleTimeout() {
      var timeoutErrorMessage = config.timeout ? 'timeout of ' + config.timeout + 'ms exceeded' : 'timeout exceeded';
      var transitional = config.transitional || transitionalDefaults;
      if (config.timeoutErrorMessage) {
        timeoutErrorMessage = config.timeoutErrorMessage;
      }
      reject(new AxiosError(
        timeoutErrorMessage,
        transitional.clarifyTimeoutError ? AxiosError.ETIMEDOUT : AxiosError.ECONNABORTED,
        config,
        request));

      // Clean up request
      request = null;
    };

    // Add xsrf header
    // This is only done if running in a standard browser environment.
    // Specifically not if we're in a web worker, or react-native.
    if (utils.isStandardBrowserEnv()) {
      // Add xsrf header
      var xsrfValue = (config.withCredentials || isURLSameOrigin(fullPath)) && config.xsrfCookieName ?
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }

    // Add headers to the request
    if ('setRequestHeader' in request) {
      utils.forEach(requestHeaders, function setRequestHeader(val, key) {
        if (typeof requestData === 'undefined' && key.toLowerCase() === 'content-type') {
          // Remove Content-Type if data is undefined
          delete requestHeaders[key];
        } else {
          // Otherwise add header to the request
          request.setRequestHeader(key, val);
        }
      });
    }

    // Add withCredentials to request if needed
    if (!utils.isUndefined(config.withCredentials)) {
      request.withCredentials = !!config.withCredentials;
    }

    // Add responseType to request if needed
    if (responseType && responseType !== 'json') {
      request.responseType = config.responseType;
    }

    // Handle progress if needed
    if (typeof config.onDownloadProgress === 'function') {
      request.addEventListener('progress', config.onDownloadProgress);
    }

    // Not all browsers support upload events
    if (typeof config.onUploadProgress === 'function' && request.upload) {
      request.upload.addEventListener('progress', config.onUploadProgress);
    }
    // 取消请求相关
    if (config.cancelToken || config.signal) {
      // Handle cancellation
      // eslint-disable-next-line func-names
      onCanceled = function(cancel) {
        if (!request) {
          return;
        }
        reject(!cancel || (cancel && cancel.type) ? new CanceledError() : cancel);
        request.abort();
        request = null;
      };

      config.cancelToken && config.cancelToken.subscribe(onCanceled);
      if (config.signal) {
        config.signal.aborted ? onCanceled() : config.signal.addEventListener('abort', onCanceled);
      }
    }

    if (!requestData) {
      requestData = null;
    }

    var protocol = parseProtocol(fullPath);

    if (protocol && [ 'http', 'https', 'file' ].indexOf(protocol) === -1) {
      reject(new AxiosError('Unsupported protocol ' + protocol + ':', AxiosError.ERR_BAD_REQUEST, config));
      return;
    }


    // Send the request
    request.send(requestData);
  });
};
```
注意点：
- `auth`是表示授权相关的请求头，axios会默认加上配置
- 如果是form表单请求，axios会删除`Content-Type`请求头
- 在这里也做了取消请求相关的，利用的是xhr的`abort`函数进行请求取消

## 参链

- [学习 axios 源码整体架构，打造属于自己的请求库](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744604&idx=1&sn=51d8d865c9848fd59f7763f5fb9ce789&chksm=88662490bf11ad86061ae76ff71a1177eeddab02c38d046eecd0e1ad25dc16f7591f91e9e3b2&scene=21#wechat_redirect)

- [axios官网](http://www.axios-js.com/zh-cn/docs/#axios-spread-callback)