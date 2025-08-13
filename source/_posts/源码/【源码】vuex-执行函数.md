---
title: vuex-执行函数
time: 2024/12/9
categories: vuex
  源码
tag: vuex
  源码
---

## dispatch

vuex 中有三个关于 dispatch 回调的方法：

- 进行初始化时 Store 中对 dispatch 的重新赋值

  ```js
  this.dispatch = function boundDispatch(type, payload) {
    console.log(this, ref, this == ref);
    return dispatch.call(store, type, payload);
  };
  ```

- Store 的 prototype；

  ```js
  Store.prototype.dispatch = function dispatch(_type, _payload) {};
  ```

- 在走初始化时，注册每个 action
  ```js
  function registerAction(store, type, handler, local) {
    var entry = store._actions[type] || (store._actions[type] = []);
    entry.push(function wrappedActionHandler(payload) {});
  }
  ```
  那么当在项目中调用 dispatch 时，调用顺序是依次调用的，实际调用项目的 action 函数，也是在 registerAction 中进行调用的。平时写 action 函数时的 root，就是在 registerAction 的函数中进行赋值的

### Store 重新赋值

```js
var store = this;
var ref = this;
var dispatch = ref.dispatch;
var commit = ref.commit;
this.dispatch = function boundDispatch(type, payload) {
  return dispatch.call(store, type, payload);
};
```

这里的 dispatch 主要是为了执行 dispatch 时，将其 this 确定绑定到 Store 中

### Store 原型链的 dispatch

- 获取到 action 对应的 type 和参数
  传入的第一个参数可能是个 object，那么需要从 object 中取出 type 和 action 的参数

  ```js
  var ref = unifyObjectStyle(_type, _payload);
  var type = ref.type;
  var payload = ref.payload;

  var action = { type: type, payload: payload };
  ```

- 循环执行 Store 提供的 subscribeAction 回调的 before
  ```js
  try {
    // 拿到所有的subscribeAction列表，过滤掉只存在before的函数并且执行
    this._actionSubscribers
      .slice() // shallow copy to prevent iterator invalidation if subscriber synchronously calls unsubscribe
      .filter(function (sub) {
        // 过滤只存在before的函数
        return sub.before;
      })
      .forEach(function (sub) {
        // 循环执行对应的函数
        return sub.before(action, this$1.state);
      });
  } catch (e) {
    if (process.env.NODE_ENV !== "production") {
      console.warn("[vuex] error in before action subscribers: ");
      console.error(e);
    }
  }
  ```
- 执行 action 函数
  action 是一个数组，重复的 action 名称是会放到同一个 actionType 下的
  实际这里执行的是在 registerAction 对函数进行包装了一层的 action 函数

  ```js
  var result =
    entry.length > 1
      ? Promise.all(
          entry.map(function (handler) {
            return handler(payload);
          })
        )
      : entry[0](payload);
  ```

- 循环执行 Store 提供的 subscribeAction 回调的 after 和 error，并将 action 执行结果传给此函数
  ```js
  return new Promise(function (resolve, reject) {
    // 在action成功执行之后
    result.then(
      function (res) {
        try {
          this$1._actionSubscribers
            .filter(function (sub) {
              // 过滤只含有after的项
              return sub.after;
            })
            .forEach(function (sub) {
              // 循环执行函数
              return sub.after(action, this$1.state);
            });
        } catch (e) {
          if (process.env.NODE_ENV !== "production") {
            console.warn("[vuex] error in after action subscribers: ");
            console.error(e);
          }
        }
        resolve(res);
      },
      // 如果出错了，就需要将错误传给error回调
      function (error) {
        try {
          this$1._actionSubscribers
            .filter(function (sub) {
              return sub.error;
            })
            .forEach(function (sub) {
              return sub.error(action, this$1.state, error);
            });
        } catch (e) {
          if (process.env.NODE_ENV !== "production") {
            console.warn("[vuex] error in error action subscribers: ");
            console.error(e);
          }
        }
        reject(error);
      }
    );
  });
  ```

### registerAction 的 action

这个函数才是真正执行项目中写的 action 回调

```js
function registerAction(store, type, handler, local) {
  // 拿到当前action列表
  var entry = store._actions[type] || (store._actions[type] = []);
  entry.push(function wrappedActionHandler(payload) {
    // 这里是在Store.prototype.dispatch中调用action时，调用的函数
    // handler是项目中的action，将其this指向store
    // 传入root参数
    var res = handler.call(
      store,
      {
        dispatch: local.dispatch,
        commit: local.commit,
        getters: local.getters,
        state: local.state,
        rootGetters: store.getters,
        rootState: store.state,
      },
      payload
    );
    // 如果返回结果不是一个promise，那么也包装为一个promise返回
    // 这里保证了在Store.prototype.dispatch的执行结果必须是个promise
    if (!isPromise(res)) {
      res = Promise.resolve(res);
    }
    if (store._devtoolHook) {
      return res.catch(function (err) {
        store._devtoolHook.emit("vuex:error", err);
        throw err;
      });
    } else {
      return res;
    }
  });
}
```

## commit

和 dispatch 一样，vuex 中关于 commit 的地方也有三个：

- 进行初始化时 Store 中对 commit 的重新赋值

  ```js
  this.commit = function boundCommit(type, payload, options) {
    return commit.call(store, type, payload, options);
  };
  ```

- Store 的 prototype；

  ```js
  Store.prototype.commit = function commit(_type, _payload, _options) {};
  ```

- 在走初始化时，注册每个 action

  ```js
  function registerMutation(store, type, handler, local) {
    var entry = store._mutations[type] || (store._mutations[type] = []);
    entry.push(function wrappedMutationHandler(payload) {
      handler.call(store, local.state, payload);
    });
  }
  ```

  那么当在项目中调用 commit 时，调用顺序是依次调用的，实际调用项目的 commit 函数，也是在 registerMutation 中进行调用的。平时写 commit 函数时的 state，就是在 registerMutation 的函数中进行赋值的

### Store 重新赋值

作用和 dispatch 一样，也是确保执行的 this 指向 Store

```js
this.commit = function boundCommit(type, payload, options) {
  return commit.call(store, type, payload, options);
};
```

### Store.prototype.commit

- 和 dispatch 一样，将传入的参数格式化

  ```js
  var this$1 = this;
  // check object-style commit
  var ref = unifyObjectStyle(_type, _payload, _options);
  var type = ref.type;
  var payload = ref.payload;
  var options = ref.options;

  var mutation = { type: type, payload: payload };
  ```

- 执行 commit
  先获取到 commit 列表，然后执行 commit 函数，和 dispatch，这里执行的也是 registerMutation 中的函数

  ```js
  var entry = this._mutations[type];
  this._withCommit(function () {
    entry.forEach(function commitIterator(handler) {
      handler(payload);
    });
  });
  ```

- \_withCommit
  这里将 Store 的\_committing 置为 true，然后再去对应的 commit
  因此可以通过 Store 的`_committing`字段来判断是否是通过 commit 进行的数据修改
  ```js
  Store.prototype._withCommit = function _withCommit(fn) {
    var committing = this._committing;
    this._committing = true;
    fn();
    // 执行之后重置数据
    this._committing = committing;
  };
  ```
- 执行 subscribe 列表
  commit 执行结束之后，循环执行 subscribe 列表
  ```js
  this._subscribers
    .slice() // shallow copy to prevent iterator invalidation if subscriber synchronously calls unsubscribe
    .forEach(function (sub) {
      return sub(mutation, this$1.state);
    });
  ```
- 输出日志
  如果commit的时候，第三个参数是带有silent的，那么会输出一行日志
  ```js
  if (process.env.NODE_ENV !== "production" && options && options.silent) {
    console.warn(
      "[vuex] mutation type: " +
        type +
        ". Silent option has been removed. " +
        "Use the filter functionality in the vue-devtools"
    );
  }
  ```

### registerMutation
实际情况下，action和commit都是被拉平了的，那么这里如何确保在执行的时候，传入项目的commit函数state的对应module下的呢？

执行registerMutation的函数是在installModule，进行forEachMutation时调用的，在这个时候调用registerMutation传入的module。这个情况下，因为还在对每个模块进行遍历，模块是单独独立的，然后在执行wrappedMutationHandler时，由于闭包的特性，所以访问的local是没有被污染的真实的module
``` js
function registerMutation (store, type, handler, local) {
  var entry = store._mutations[type] || (store._mutations[type] = []);
  entry.push(function wrappedMutationHandler (payload) {
    // 主要是这里执行项目中真正编写的commit函数
    // 也是这里将module的state传入的
    handler.call(store, local.state, payload);
  });
}
```

