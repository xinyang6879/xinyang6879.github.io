---
title: 【js】修饰器（Decorator）
time: 2024/03/12
categories: JS
tag: js
---

## 修饰器

修饰器是 ES7 的提案，是一种用于修改类、方法或属性的语法，它可以在不修改原始代码的情况下增强其功能。修饰器可以实现横切关注点（cross-cutting concerns）的功能，例如日志记录、性能分析、缓存等。通过将这些功能与原始代码分离，我们可以更好地组织和维护代码，并实现更高的可重用性和可扩展性。

它的出现可以解决其下两个问题:

- 不同类之间共享方法

- 编译期对类和方法的行为进行改变

## 类修饰器

类修饰器用于修改类的行为和属性。它可以在类定义之前应用，以修改类的构造函数或原型。

```js
function log(target) {
  const originCon = target;
  console.log("log", target);
  function newCon(...args) {
    console.log("newCon", args);
    args[0] = "234"; //修改传给class的参数
    return new originCon(...args);
  }
  return newCon;
}

@log
class classA {
  constructor(props) {
    console.log("classA", props);
  }
}

// 输出：
// log class{...}
const objA = new classA("123");

// 输出：
// new Con ['123']
// classA ['234']
```

- 访问静态属性

  定义了一个修饰器函数 log，然后将这个修饰器应用在 classA，在使用这个修饰器的时候，就会执行修饰器函数 log。

  但是如果需要访问类的静态属性或者方法，会直接报错：

  ```js
  @log
  class classA {
    constructor(props) {
      console.log("classA", props);
    }

    static staticFn() {
      console.log("staticFn");
    }
  }
  classA.staticFn();
  //   报错：TypeError: classA.staticFn is not a function
  ```

  因为 log 返回的新实例，不是原始的类，所以没有 staticFn 方法，导致报错。所以需要在修饰器函数中，将静态属性和方法进行赋值。

  ```js
  function log(target) {
  const originCon = target;
  console.dir(target);
  function newCon(...args) {
      console.log("newCon", args);
      args[0] = "234";
      return new originCon(...args);
  }

  // 辅助函数，用于复制静态方法
  +  function copyStatic(originalConstructor, newConstructor) {
  +    Reflect.ownKeys(originalConstructor).forEach((prop) => {
  +      console.log("prop", prop);
  +      if (prop !== "prototype" && prop !== "length" && prop !== "name") {
  +        newConstructor[prop] = originalConstructor[prop];
  +      }
  +    });
  +  }
  +  copyStatic(originCon, newCon);
  return newCon;
  }

  classA.staticFn();

  输出：
  // staticFn
  ```

- 应用场景

  - **日志记录**：在类的方法执行前后记录日志信息。
  - **验证和授权**：对类的方法进行验证和授权操作。
  - **性能分析**：测量类的方法执行时间，进行性能分析。
  - **依赖注入**：为类的构造函数注入依赖项。

## 方法修饰器

方法修饰器用于修改类的方法行为。它可以在方法定义之前应用，以修改方法的特性和行为。

```js
function log(target, name, descriptor) {
  console.dir(target);
  console.dir(name);
  console.dir(descriptor);
  console.log("初始化调用修饰器");
  return {
    ...descriptor,
    value: function (...args) {
      console.log("修饰器函数的方法");
      const res = descriptor.value.apply(this, args);
      return res * 100;
    },
  };
}
class classA {
  constructor(props) {
    console.log("classA", props);
  }
  @log
  addFn(a, b) {
    console.log("调用了原始函数");
    return a + b;
  }
}
const objA = new classA("123");
const res = objA.addFn(1, 2);
console.log("res", res);

// 输出：
// Object { addFn, constructor: class classA }
// addFn
// {value: f addFn(a, b), enumerable: false, configurable: true, writable: true}
// 初始化调用修饰器
// classA 123
// 修饰器函数的方法
// 调用了原始函数
// res 300
```

方法修饰器函数接收三个参数，分别是`target`（类的原型或构造函数）、`name`（方法名）和`descriptor`（方法的属性描述符）。修改`descriptor.value`可以替换原有的方法

- 应用场景

- **日志记录**：在方法执行前后记录日志信息。
- **验证和授权**：对方法进行验证和授权操作。
- **性能分析**：测量方法执行时间，进行性能分析。
- **缓存**：为方法添加缓存功能，提高性能。

## 属性修饰器

用于修改类的属性行为。它可以在属性定义之前应用，以修改属性的特性和行为

虽然有很多博客说可以直接用类属性修饰器，但是个人实际使用了一下，并未成功

```js
function protoDec(initVal) {
  return function (target, name) {
    target[name] = initVal;
    console.dir(target);
    console.log(name);
    let val = target[name];
    const getter = function () {
      console.log("获取值：", val);
      return val;
    };
    const setter = function (newVal) {
      if (typeof newVal === "number") {
        console.log("设置值：", newVal);
        val = newVal + 1;
      } else {
        throw new Error("设置值必须为数字");
      }
    };
    Object.defineProperty(target, name, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true,
    });
  };
}
class classA {
  @protoDec("0")
  num;

  constructor(props) {
    console.log("classA", props);
  }
}
const objA = new classA("123");
console.log("objA", objA.num);
objA.num = 100;
console.log("objA", objA.num);

// 当前输出：
// Object { num: "0" }
// num
// classA 123
// objA undefined
// objA 100

// 预期输出：
// Object { num: "0" }
// num
// classA 123
// 获取值：0
// objA "0"
// 设置值：100
// objA 101
```

## 参数修饰器

参数修饰器用于修改方法的参数行为。它可以在方法参数声明之前应用，以修改参数的特性和行为

参数装饰器只能用来**监视**一个方法的参数是否被传入，无法修改函数执行结果

参数装饰器表达式会在运行时当作函数被调用，传入下列 3 个参数：

- 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
- 成员的名字。
- 参数在函数参数列表中的索引。

参数装饰器的返回值会被**忽略**。

```js
function paramsDec(target, name, index) {
  console.dir(target)
  console.dir(name)
  console.dir(index)
}
class classA {

  fn(@paramsDec num1, @paramsDec num2) {
    console.log("执行函数")
  };

  constructor(props) {
    console.log("classA", props);
  }
}
const objA = new classA("123");
objA.fn(1, 2)

输出：
Object {
    constructor: class classA,
    fn: ƒ fn(num1, num2)
}
fn
1
Object {
    constructor: class classA,
    fn: ƒ fn(num1, num2)
}
fn
0
classA 123
执行函数
```

## 装饰器执行顺序

当多个装饰器应用在一个声明上时会进行如下步骤的操作：

- 由上至下依次对装饰器表达式求值。
- 求值的结果会被当作函数，由下至上依次调用。

## 参链

[JavaScript修饰器：简化代码，增强功能](https://www.coding-time.cn/js/advance/%E8%A3%85%E9%A5%B0%E5%99%A8.html#_8-%E5%B8%B8%E7%94%A8%E4%BF%AE%E9%A5%B0%E5%99%A8%E5%BA%93%E5%92%8C%E5%B7%A5%E5%85%B7)

[装饰器](https://www.tslang.cn/docs/handbook/decorators.html)