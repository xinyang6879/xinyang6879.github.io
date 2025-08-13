---
title: 【手写】call、apply、bind
categories: js
---

## _call_

call 可以改变调用函数时的 this 指向，当调用但什么都不传入时，默认是 window 对象

### 内部流程

[es 文档](https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-function.prototype.call)上编写的流程如下:

- 让一个函数 func 指向 this

- 如果这个函数 func 不能被调用，那么抛出 TypeError

- 准备尾调用 PrepareForTailCall

- 返回执行结果

思路

- 判断需要改变的 context 是否为空，如果为空，那么就默认指为 window。保留这个新的 context，作为后面函数调用时需要的 this 指向

- 将当前的 this 指向赋值给上一步保留的 context，作为 context 的一个属性

- 调用第一步保留的 context 的第二步赋值的属性函数，将所需要的参数传递给上一步新赋值的属性

### 实现

- call

```js
function call_handle_writing(fn) {
  // ctx指向obj，因为fn为非空
  const ctx = fn || window;
  //   ctx.cb指向的是fn1
  ctx.cb = this;
  //   获取参数
  const args = [...arguments].slice(1);
  //   调用fn1函数，但是由于是ctx调用的，那么fn1被调用的时候的this指向是ctx
  const res = ctx.cb(...args);
  return res;
}

Function.prototype.call_handle_writing = call_handle_writing;
```

- 使用

```js
function fn1() {
  this.a = 1;
  console.log("out", this, arguments);
}
let obj = {
  a: 100,
};
fn1.call_handle_writing(obj, 1, 2, 3);
```

## apply

### 实现流程

> 1.  Let func be the this value.
> 2.  If IsCallable(func) is false, throw a TypeError exception.
> 3.  If argArray is either undefined or null, then
>     > a. Perform PrepareForTailCall().<br />
>     > b. Return ? <font color=red>Call(func, thisArg).</font>
> 4.  Let argList be ? CreateListFromArrayLike(argArray).
> 5.  Perform PrepareForTailCall().
> 6.  Return ? Call(func, thisArg, argList).

其实就是接受一个数组作为参数，实际最后的调用时，调用的也是 call 方法

apply 调用 call 的时候，会用到扩展运算符，将参数放到 call 的参数中，因此这在一定程度上，导致了 apply 的性能会稍低于 call

### 具体实现

```js
function apply_handle_writing(fn, params) {
  if (Array.isArray(params)) {
    return this.call_handle_writing(fn, ...params);
  }
  return this.call_handle_writing(fn);
}
Function.prototype.apply_handle_writing = apply_handle_writing;
```

## bind

### 内部流程

> 1. Let Target be the this value.
> 2. If IsCallable(Target) is false, throw a TypeError exception.
> 3. Let F be ? BoundFunctionCreate(Target, thisArg, args).
> 4. Let L be 0.
> 5. Let targetHasLength be ? HasOwnProperty(Target, "length").
> 6. If targetHasLength is true, then
>    > a. Let targetLen be ? Get(Target, "length").<br/>
>    > b. If targetLen is a Number, then<br/>
>    >
>    > > i. If targetLen is +∞𝔽, set L to +∞.<br/>
>    > >
>    > > > ii. Else if targetLen is -∞𝔽, set L to 0.<br/>
>    > > > iii. Else,
> 7. Let targetLenAsInt be ! ToIntegerOrInfinity(targetLen).
> 8. Assert: targetLenAsInt is finite.
> 9. Let argCount be the number of elements in args.
> 10. Set L to max(targetLenAsInt - argCount, 0).
> 11. Perform SetFunctionLength(F, L).
> 12. Let targetName be ? Get(Target, "name").
> 13. If targetName is not a String, set targetName to the empty String.
> 14. Perform SetFunctionName(F, targetName, "bound").
> 15. Return F.

考虑场景：

```js
function A() {}
A.prototype.say = function () {};
function B() {}
const C = A.bind(B);
const c = new C();
```

- 调用 bind 后，使用`new`去创建一个实例，那么在`new`的时候不应该改变`this`指向

  在返回的函数中，用`instanceof`判断是否是通过`new`方法进行调用的

- 调用 bind 后，在 c 上面可以调用 A 上 `prototype` 的方法

  采用继承的方式，在返回的函数上，继承 A

### 具体实现

```js
function bind_handle_writing(fn) {
  const that = this;
  const args = [...arguments].slice(1);
  function cb() {
    const params = [...arguments, ...args];
    fn = this instanceof cb ? this : fn;
    return that.call(fn, ...params);
  }
  // 用一个中间函数，在原型链上加一层，防止属性覆盖
  const buf = function () {};
  buf.prototype = this.prototype;
  cb.prototype = Object.create(this.prototype);
  return cb;
}
Function.prototype.bind_handle_writing = bind_handle_writing;
```

### 参考链接

[如何手写一个 bind 方法](https://www.jianshu.com/p/b540e1e17f54)
