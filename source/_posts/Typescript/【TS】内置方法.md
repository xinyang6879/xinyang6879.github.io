---
title:【TS】内置方法
categories: Typescript
---

## 语法

- ?

当前属性或者参数为非必传，参数有默认值时会默认转为非必传

- ()

定义函数类型时，()会作为函数参数类型

```ts
function fn1(a: number) {
  let bb = <T1>function (start: number, v: "") {};
  bb.interval = 0;
  bb.reset = () => {};
}
type T1 = {
  (start: number, v: string): string;
  interval: number;
  reset(): void;
};
```

### abstract

- abstract class{}

  抽象类，做为其它派生类的基类使用。 它们一般不会直接被实例化

- abstract function

  抽象方法不包含具体实现，并且**必须**在子类中实现

#### 例子

```ts
abstract class A {
  abstract hello() {}
}

class B extends A {
  hello() {}
}
// 报错，必须要声明hello方法
class C extends A {}
```

### implements

属于 ts 中的 mixin，用于对多个类的类型继承

#### 异同

- 相同点

  都可以继承父类的类型

  `extends`、`implements`和`abstract`在子类中都可以覆写父类的方法

- 不同点

  > 覆写方法

    `extends`不是强制要求，子类可以继承父类的方法。

    `implements`是强制要求全部覆写父类的方法

    `abstract`是强制要求加了`abstract`的方法或者变量

  > 继承个数

    `extends` 只能继承一个，`implements` 可以继承多个

  > 访问范围

    `extends` 的子类实例可以访问到父类内容

    `implements` 只是定义了一个类型，即使在父类里面有方法，子类实例也不能访问，只能访问到子类自身的方法

#### 使用

```ts
class A {
  a_cb() {}
}
class B {
  b_cb() {}
}
class C implements A, B {
  a_cb() {}
  b_cb() {}
}
```

### `///`指令

作用：告诉编译器在编译过程中需要额外的处理

#### 使用类型

- `/// <reference path="..." />`

  - 作用：声明文件间的依赖。告诉编译器在编译过程中需要引入的额外文件

  当使用`--out`或`--outFile`时，可以做为调整顺序的一种方式。

  - 过滤：使用`--noResolve`

    如果指定了--noResolve 编译选项，三斜线引用会被忽略；它们不会增加新文件，也不会改变给定文件的顺序。

  - `/// <reference types="..." />`

    - 作用：声明了对某个包类型文件的依赖

      例如：`/// <reference types="node" /`表示使用`@types/node/index.d.ts`

  - `/// <reference no-default-lib="true"/>`

    - 作用：把文件标记为默认库

    - 忽略：`--skipDefaultLibCheck`

  - `/// <amd-module name="xxx"/>`

    - 作用：给编译器传入一个可选的模块名

> 注意点：

  - 三斜线指令<font color=red>只能</font>可放在包含它的文件的<font color=red>最顶端</font>

      `///`指令之前只能出现单/多行注释或者`///`指令，如果出现在语句或者声明之后，那么就会被当做是一个注释

  - 引用不存在的文件会报错，自身引用自身也会报错

## 类型

### 过滤

- Exclude<T, U>

  从 T 中剔除可以赋值给 U 的类型。

- Extract<T, U>

  提取 T 中可以赋值给 U 的类型。

- Omit<T, U>

  构造一个除类型 U 中的属性外具有 T 属性的类型。

- NonNullable<T>

  从 T 中剔除 null 和 undefined。

- Record<K, T>

  构造一个具有类型 T 的一组属性 K 的类型

- Pick<T, K extends keyof T>

  从 T 中选择一组键位于并集 K 中的属性

### 描述

- Readonly<T>

  将 T 类型属性全部变为 readonly

- Partial<T>

  将 T 类型属性全部变为可选

- Required<T>

  将 T 类型属性全部变为必选

### 转换

- Uppercase<T extends string>

  将字符串文字类型转换为大写

- Lowercase<T extends string>

  将字符串文字类型转换为小写

- Capitalize<T extends string> / Uncapitalize<T extends string>

  将字符串文字类型的第一个字符转换为大/小写

### 函数相关

- Parameters<T extends (...args: any) => any>

  返回函数的参数

  ```ts
  function fn1(a: string, b = 2) {}
  type T1 = Parameters<typeof fn1>;
  // T1: [a: string, b?: number]
  ```

- ConstructorParameters<T extends abstract new (...args: any) => any>

  获取构造函数的参数

  ```ts
  class fn1 {
    constructor(a: number, b = 2) {}
  }

  type T1 = ConstructorParameters<typeof fn1>;
  // T1: [a: number, b?: number]
  ```

- ReturnType<T extends (...args: any) => any>

  获取函数返回参数

  ```ts
  function fn() {
    return {
      a: 1,
      b: 2,
    };
  }

  type T1 = ReturnType<typeof fn>;
  ```

- InstanceType<T extends abstract new (...args: any) => any>

  获取构造函数函数类型的返回类型

