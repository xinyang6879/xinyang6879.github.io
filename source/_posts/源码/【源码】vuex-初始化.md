---
title: vuex-流程
time: 2024/11/20
categories: vuex
  源码
tag: vuex
  源码
---

源码位置：`vuex/dist/vuex.esm.js`，该文件默认导出了`{ Store, createLogger, createNamespacedHelpers, install, mapActions, mapGetters, mapMutations, mapState }`，在使用 vuex 时，用的`vue.use(Vuex)`实际执行的是`install`方法

## 引入 vuex

### install

- 首先判断传入的 vue 是否和之前的 vue 一致，如果一致的话且非 prd 环境，则会抛出异常
  ```js
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== "production") {
      console.error(
        "[vuex] already installed. Vue.use(Vuex) should be called only once."
      );
    }
    return;
  }
  ```
- 将传入的 vue 赋值给 vuex 的全局变量
    用于判断是否已经装载和减少全局作用域查找
  `Vue = _Vue;`
- 利用 vue3 以下的 mixin，将 vuex 挂载到 vm 的全局上

### mixin

- 首先判断 vue 的版本，如果是在 v2 以上，那么直接调用`mixin`方法
  ` Vue.mixin({ beforeCreate: vuexInit });`
- 若非 vue2 以上，那么就调用 vue 的 init 方法。将初始化的函数加入 vue 的 init 方法列表中，并进行循环调用
  ```js
  options.init = options.init ? [vuexInit].concat(options.init) : vuexInit;
  _init.call(this, options);
  ```
  - vuexInit 方法：判断当前 vue 是否是 root（用是否有 parent 进行判断）。若是 root，那么就将 store 赋值到 rootVue 的`$store`中，若不是，那么就将父 Vue 的`$store`作为当前 vue 的`$store`
  ```js
  if (options.store) {
    this.$store =
      typeof options.store === "function" ? options.store() : options.store;
  } else if (options.parent && options.parent.$store) {
    this.$store = options.parent.$store;
  }
  ```

## 创建 store

创建 store 时，执行的是`Store`方法

### Store

- 首先进行一堆初始值赋值

  ```js
  var plugins = options.plugins;
  if (plugins === void 0) plugins = [];
  var strict = options.strict;
  if (strict === void 0) strict = false;

  // store internal state
  this._committing = false; //是否是修改state
  this._actions = Object.create(null);
  this._actionSubscribers = [];
  this._mutations = Object.create(null);
  this._wrappedGetters = Object.create(null);
  this._modules = new ModuleCollection(options); //初始化注册module
  this._modulesNamespaceMap = Object.create(null);
  this._subscribers = [];
  this._watcherVM = new Vue(); //用于store的watch使用
  this._makeLocalGettersCache = Object.create(null);
  ```

- 将 dispatch 重新赋值执行
  ```js
  var ref = this;
  var dispatch = ref.dispatch;
  this.dispatch = function boundDispatch(type, payload) {
    return dispatch.call(store, type, payload);
  };
  ```
  这里调用的还是 store 上面的 dispatch 方法
- 将 commit 重新赋值执行
  ```js
  this.commit = function boundCommit(type, payload, options) {
    return commit.call(store, type, payload, options);
  };
  ```
- 初始化 module（installModule）
  这里的 state，是经过了`new ModuleCollection`流程的，取用的是 module 根节点的 state，实际就是项目中 store 的 module 配置列表
  ```js
  var state = this._modules.root.state;
  installModule(this, state, [], this._modules.root);
  ```
- 对 store 的`_vm`重新赋值
  `resetStoreVM(this, state);`
- 执行 plugins 列表

### module 相关

#### new ModuleCollection

创建了一个 module 的对象，store 里面的 module 并不是直接归属 store 下的一个对象，而是 ModuleCollection 函数，这个是一个独立的对象

而 ModuleCollection 又是由一个个 Module 对象集合而成的

ModuleCollection 有一个根节点，根节点创建完成之后，后续的 module 都会通过根节点(Module 对象)的 addChild 函数进行添加

modules 结构如下：
[![pAWHwB4.png](https://s21.ax1x.com/2024/11/21/pAWHwB4.png)](https://imgse.com/i/pAWHwB4)

可以看到有一个根节点，然后其下的 state 才是保存的实际的项目中的 storeModule

- 在 vuex 初始化时执行这个函数,进入 register 流程
  ```js
  var ModuleCollection = function ModuleCollection(rawRootModule) {
    // register root module (Vuex.Store options)
    this.register([], rawRootModule, false);
  };
  ```
- 创建 module 的根节点
  这里由于是第一次调用的话，path 默认就是个空数组，所以调用 ModuleCollection 的 register 都会先创建一个 modules 的根节点

  ```js
  var newModule = new Module(rawModule, runtime);
  // 由于调用register的时候传入的path是个空数组，所以这里会把第一个创建的module作为根节点
  if (path.length === 0) {
    this.root = newModule;
  } else {
    var parent = this.get(path.slice(0, -1));
    parent.addChild(path[path.length - 1], newModule);
  }
  ```

- 循环调用 register，将项目中的 store 循环放在根节点的 children 中
  ```js
  // register nested modules
  if (rawModule.modules) {
    forEachValue(rawModule.modules, function (rawChildModule, key) {
      // 这里执行的也是ModuleCollection的register函数，不过由于path里面已经有数据了，所以这里走的是上一步的else流程
      this$1.register(path.concat(key), rawChildModule, runtime);
    });
  }
  ```

### installModule

这个函数主要是用于将上一步得到的\_modules 进行处理，然后循环注册 module 的 getter、action、mutation 等

- 前置处理
  判断当前是否是根节点，如果是非根节点，即项目中的每个 module，把这些放在根节点的 state 中

  ```js
  // 判断是否是根节点，实际只有从Store中初始化的入口，其他进入的都是非根节点
  var isRoot = !path.length;
  var namespace = store._modules.getNamespace(path);

  // register in namespace map
  if (module.namespaced) {
    if (
      store._modulesNamespaceMap[namespace] &&
      process.env.NODE_ENV !== "production"
    ) {
      console.error(
        "[vuex] duplicate namespace " +
          namespace +
          " for the namespaced module " +
          path.join("/")
      );
    }
    store._modulesNamespaceMap[namespace] = module;
  }

  // set state
  if (!isRoot && !hot) {
    var parentState = getNestedState(rootState, path.slice(0, -1));
    var moduleName = path[path.length - 1];
    store._withCommit(function () {
      if (process.env.NODE_ENV !== "production") {
        if (moduleName in parentState) {
          console.warn(
            '[vuex] state field "' +
              moduleName +
              '" was overridden by a module with the same name at "' +
              path.join(".") +
              '"'
          );
        }
      }
      // 将module的state都挂到根节点的state上
      Vue.set(parentState, moduleName, module.state);
    });
  }
  ```

- 循环注册每个模块的 getter、action、mutation

  ```js
  module.forEachMutation(function (mutation, key) {
    var namespacedType = namespace + key;
    registerMutation(store, namespacedType, mutation, local);
  });

  module.forEachAction(function (action, key) {
    var type = action.root ? key : namespace + key;
    var handler = action.handler || action;
    registerAction(store, type, handler, local);
  });

  module.forEachGetter(function (getter, key) {
    var namespacedType = namespace + key;
    registerGetter(store, namespacedType, getter, local);
  });
  ```

- 循环执行每个 module
  对项目中的每个 module 每次都走一个第一步
  ```js
  module.forEachChild(function (child, key) {
    installModule(store, rootState, path.concat(key), child, hot);
  });
  ```

### resetStoreVM(this, state);

这个主要是用于初始化 store 上的`_vm`，以及将 gettter 放到 store 的`getters`上，即走了这一步，store 里面才有 getter。dispatch、commit 等是直接挂载在 Store.prototype 上的

- 挂载 getters
  这里是将\_wrappedGetters 的数据挂载到 store 里面的 getter 上，\_wrappedGetters 是执行的函数，getters 是执行的结果
  ```js
  var oldVm = store._vm;
  console.log(store);
  // bind store public getters
  store.getters = {};
  // reset local getters cache
  store._makeLocalGettersCache = Object.create(null);
  var wrappedGetters = store._wrappedGetters;
  var computed = {};
  forEachValue(wrappedGetters, function (fn, key) {
    // use computed to leverage its lazy-caching mechanism
    // direct inline function use will lead to closure preserving oldVm.
    // using partial to return function with only arguments preserved in closure environment.
    computed[key] = partial(fn, store);
    Object.defineProperty(store.getters, key, {
      get: function () {
        return store._vm[key];
      },
      enumerable: true, // for local getters
    });
  });
  ```
- 对 store 的\_vm 重新赋值
  这里的 state 是\_modules 根节点的 state

  ```js
  var silent = Vue.config.silent;
  // 暂时将Vue设为静默模式，避免报出用户加载的某些插件触发的警告
  Vue.config.silent = true;
  store._vm = new Vue({
    data: {
      $$state: state,
    },
    computed: computed,
  });
  // 恢复Vue的模式
  Vue.config.silent = silent;
  // 如果设置了严格模式
  if (store.strict) {
    enableStrictMode(store);
  }
  ```

- 销毁老的 vm
  ```js
  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(function () {
        oldVm._data.$$state = null;
      });
    }
    Vue.nextTick(function () {
      return oldVm.$destroy();
    });
  }
  ```

### 模块函数注册
在installModule中，会对每个模块进行mutation、action、getter的注册

- registerMutation
    调用方：
    module.forEachMutation是将_modules中的mutation取出来，然后循环执行回调函数
    ``` js
    module.forEachMutation(function (mutation, key) {
        var namespacedType = namespace + key;
        registerMutation(store, namespacedType, mutation, local);
    });
    // 等同于
    // this._rawModule.mutations.foreach()
    ```
    获取到对应的muation，这里的创建store的_muation的地方

    如果没有同名的muation，那么就创建出；然后再将项目的muation包装一下，增加到store的_mutation中
    
    ``` js
    function registerMutation (store, type, handler, local) {
    //    创建muation的地方 store._mutations[type] = []
    var entry = store._mutations[type] || (store._mutations[type] = []);
    entry.push(function wrappedMutationHandler (payload) {
        // handler就是实际项目中的的muation，将执行的this绑定到store中，
        // local是`module.context`，是在创建Module的时候，传入的 
        // installModule 中的 var local = module.context = makeLocalContext(store, namespace, path);
        // 即在上一步中，每个Module创建时的
        //  ModuleCollection根节点，其下_children里面的Module
        handler.call(store, local.state, payload);
    });
    }
    ```
    [![pAf1YiF.png](https://s21.ax1x.com/2024/11/22/pAf1YiF.png)](https://imgse.com/i/pAf1YiF)

    注意，vuex中的muation和dispatch都是数组，只有getter是非数组

- 平时在执行commit时的顺序
    - Store初始化中的`this.commit`
    - Store原型链中的`Store.prototype.commit`
    - 注册mutation中（上一步）的`wrappedMutationHandler`

- registerAction
    module.forEachAction是将_modules中的action（action.handler || action）取出来，然后循环执行回调函数
    ``` js
      function registerAction (store, type, handler, local) {
      // 获取到对应的action函数，即项目中实际写的dispatch回调
      var entry = store._actions[type] || (store._actions[type] = []);
      // 因为action和mutation都是一个数组，所以用的是push方法，将对应的dispatchType放到数组中
      // 这里就是调用项目中dispatch回调，在调用之前包了一层，将执行作用于绑定到store中，
      // dispatch中的两个参数也是在这里进行传递的
      entry.push(function wrappedActionHandler (payload) {
        // dispath的root就是第二个参数，调用时传递的数据是payload
        var res = handler.call(store, {
          dispatch: local.dispatch,
          commit: local.commit,
          getters: local.getters,
          state: local.state,
          rootGetters: store.getters,
          rootState: store.state
        }, payload);
        // dispatch是promise的处理
        if (!isPromise(res)) {
          res = Promise.resolve(res);
        }
        if (store._devtoolHook) {
          return res.catch(function (err) {
            store._devtoolHook.emit('vuex:error', err);
            throw err
          })
        } else {
          return res
        }
      });
    }
  ```

- 平时在执行dispatch时的顺序
    - Store初始化中的`this.dispatch`
    - Store原型链中的`Store.prototype.dispatch`
    - 注册dispatch中（上一步）的`registerAction`中entry包过的函数


- registerGetter
    module.forEachAction是将_modules中的getters取出来，然后循环执行回调函数
    rawGetter和action第三个参数一样，都是getter的回调
    ``` js
      function registerGetter (store, type, rawGetter, local) {
        // gettter不能有重复命名的，所以会报错，这个是和action、mutation不同的点
      if (store._wrappedGetters[type]) {
        if ((process.env.NODE_ENV !== 'production')) {
          console.error(("[vuex] duplicate getter key: " + type));
        }
        return
      }
      // store就是Store，是初始创建的
      store._wrappedGetters[type] = function wrappedGetter (store) {
        return rawGetter(
          local.state, // local state
          local.getters, // local getters
          store.state, // root state
          store.getters // root getters
        )
      };
    }
    ```

- 平时在执行getter时的顺序
    - 先遍历执行一次`module.forEachGetter`
    - 调用的时候执行resetStoreVm中，将getter调用`store._vm[key]`
        ``` js
          Object.defineProperty(store.getters, key, {
            get: function () { 
              debugger
              return store._vm[key];
            },
            enumerable: true // for local getters
          });
        ```
    - 执行registerGetter中封装好的getter




