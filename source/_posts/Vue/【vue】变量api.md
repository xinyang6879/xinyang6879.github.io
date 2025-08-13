---
title: 【vue】变量api
categories: vue
---

### ref 和 reactive 区别

根本区别：

- `ref`只用于定义`typeof val !== 'object'`的数据类型，且内部是基于 class 实现的。如果 ref 传入的是一个 object 类型，那么也会通过 reactive 的方法去执行

  ```js
  const toReactive = (value) => isObject(value) ? reactive(value) : value;
  class RefImpl {
    constructor(value, __v_isShallow) {
        this.__v_isShallow = __v_isShallow;
        this.dep = undefined;
        this.__v_isRef = true;
        this._rawValue = __v_isShallow ? value : toRaw(value);
        // 判断传入的初始数据是否为对象，如果是那么就转为reactive的实现
        this._value = __v_isShallow ? value : toReactive(value);
      }
      get value() {
          trackRefValue(this);
          return this._value;
      }
      set value(newVal) {
          newVal = this.__v_isShallow ? newVal : toRaw(newVal);
          if (hasChanged(newVal, this._rawValue)) {
              this._rawValue = newVal;、
              // 在赋新值的时候也会执行一次
              this._value = this.__v_isShallow ? newVal : toReactive(newVal);
              triggerRefValue(this, newVal);
          }
      }
  }
  ```

- `reactive`用于定义`typeof val === 'object'`的数据类型，且内部基于 proxy 实现

  ```js
  function createReactiveObject(
    target,
    isReadonly,
    baseHandlers,
    collectionHandlers,
    proxyMap
  ) {
    if (!isObject(target)) {
      {
        console.warn(`value cannot be made reactive: ${String(target)}`);
      }
      return target;
    }
    // target is already a Proxy, return it.
    // exception: calling readonly() on a reactive object
    if (
      target["__v_raw" /* RAW */] &&
      !(isReadonly && target["__v_isReactive" /* IS_REACTIVE */])
    ) {
      return target;
    }
    // target already has corresponding Proxy
    const existingProxy = proxyMap.get(target);
    if (existingProxy) {
      return existingProxy;
    }
    // only specific value types can be observed.
    const targetType = getTargetType(target);
    if (targetType === 0 /* INVALID */) {
      return target;
    }
    // 通过proxy实现数据拦截
    const proxy = new Proxy(
      target,
      targetType === 2 /* COLLECTION */ ? collectionHandlers : baseHandlers
    );
    proxyMap.set(target, proxy);
    return proxy;
  }
  ```

其他区别：

- ref 使用时需要通过`.value`访问（但是在 template 中不需要），reactive 不需要
- 在使用过程中，ref 可以使用任意数据类型，reactive 只能使用对象、数组、map 等
- ref 可以直接替换整个数据，也不会丢失响应式，reactive 会丢失

  ```ts
  const data = ref({ a: 1 });
  data.value = { b: 2 }; //不会丢失响应式

  let data2 = reactive({ a: 1 });
  data2 = { b: 2 }; //会丢失
  // data2 = reactive({b: 2}) //会丢失
  ```

- 解构
  - ref 解构后依旧可以保持响应式
  - reactive 需要通过 toRefs 才可以保持
- 父子数据传递
  - ref 在子组件可以通过`update:xxx`更新到父组件的值
  - reactive 无法通过此方法更新，父组件也无法通过 watch 进行获取最新的值

### shallowRef

和ref的流程一样，只是不会走判断如果是对象，那么就会走reactive的逻辑

``` js
  function shallowRef(value) {
        // 这里的true表示不需要走reactive的逻辑（即使是个对象
      return createRef(value, true);
  }
    class RefImpl {
      constructor(value, __v_isShallow) {
          this.__v_isShallow = __v_isShallow;
          this.dep = undefined;
          this.__v_isRef = true;
        //   不走reactive的逻辑
          this._rawValue = __v_isShallow ? value : toRaw(value);
          this._value = __v_isShallow ? value : toReactive(value);
      }
      get value() {
          trackRefValue(this);
          return this._value;
      }
      set value(newVal) {
          newVal = this.__v_isShallow ? newVal : toRaw(newVal);
          if (hasChanged(newVal, this._rawValue)) {
              this._rawValue = newVal;
            //   即使是重新赋值，也不会触发
              this._value = this.__v_isShallow ? newVal : toReactive(newVal);
              triggerRefValue(this, newVal);
          }
      }
  }
```

