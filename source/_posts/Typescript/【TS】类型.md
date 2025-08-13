---
title:【TS】类型
categories: Typescript
tag: Typescript
---

ts除了基础类型，还有通过ts的语法来定义的很多类型

## 基础类型

- String
- Number
- Boolean
- symbol: 表示唯一且不可变的值，通常用作对象属性的键。
- bigint：大整数
- Null：表示 null 值，未赋值
- Undefined：表示未定义

## 对象结构类型

- Object
- Function： 函数类型
- 接口	interface User { id: number }	结构化契约
- 构造函数类型	new (name: string) => Animal	类构造函数
- 索引签名	{ [key: string]: number }	动态键值类型

## 内置对象类型

- Array  Array<T> 数组
- Date	new Date()	日期对象
- RegExp	/ab+c/	正则表达式
- Error	new Error()	错误对象
- Map<K,V>	new Map<string, number>()	ES6 Map
- Set<T>	new Set<number>()	ES6 Set
- Promise<T>	Promise.resolve(42)	异步操作

## 特殊基础类型

- Never：永远也不出现的值，一般表示为错误
- any：任何类型
- void: 表示空，一般用于函数返回
- unknown：表示未知类型，比any更安全，因为不能直接对其进行操作

## 复合类型

- enum： 枚举
- tuple：元组，固定长度和类型的数组
    - [T, ...T[]]: 至少有一个元素（T），长度不固定（...T[]）
- 联合类型： type T = string | number | boolean;
- 可辨识联合类型： type T =  { kind: "circle"; radius: number } | { kind: "square"; size: number };例如redux的action
- 交叉类型： type T = { a: string } & { b: number };
- 字面量类型：type T = "success" | "fail" | "loading"

可辨识联合类型和联合类型区别

- 可辨识联合类型**必须有共同字面量类型标签属性**； 联合类型**没有**可辨识联合类型
- 因此可辨识联合类型可以通过标签属性自动精确区分， 而联合类型不行
    ``` ts
    type Shape = 
    { kind: "circle"; radius: number } 
    | { kind: "square"; size: number };

    // 普通联合类型无法直接区分具体类型
    function getArea(shape: Shape) {
    // ❌ 错误：TS 不知道 shape 是 circle 还是 square
     return shape.radius * shape.radius * Math.PI;
    }

    // 2. 使用标签属性进行类型收窄
    function getArea(shape: DiscriminatedShape): number {
    switch (shape.kind) {
        case "circle":
        // ✅ 自动识别为 circle 类型
        return Math.PI * shape.radius ** 2;
        case "square":
        // ✅ 自动识别为 square 类型
        return shape.size ** 2;
    }
    }
    ```
- 可辨识联合需要用switch/case手动收窄，联合类型需要手动类型断言
- 可辨识联合安全性高于联合类型

## 高级类型

### 泛型


泛型（Generics）是 TypeScript 的核心类型编程特性，它允许创建可重用的类型/函数组件，这些组件可以动态适应不同数据类型而不丢失类型安全

泛型本质：类型参数化

``` ts
// 基础语法：在尖括号中声明类型变量
function identity<T>(arg: T): T {
  return arg;
}

// 使用：显式或隐式指定类型
const num = identity<number>(42);     // 显式
const str = identity("hello");        // 隐式推断为 string

```

- 命名规范：使用单字母大写（T, U, K）或描述性名称（TData）
- 避免 any：优先用泛型替代 any 保持类型安全
- 渐进复杂：从简单约束开始，逐步增加类型操作
- 组合使用：泛型 + 条件类型 + 映射类型 = 强大类型系统

#### 函数泛型

自动推断函数的参数和返回的类型

``` ts
function fun(parans: T[]): T[] | undefined {
    return parans[];
}
const num = firstElement([1, 2, 3]);  // number | undefined
const str = firstElement(["a", "b"]); // string | undefined
```

#### 接口泛型

定义灵活数据结构

``` ts
type TResponse<T> = {
  data: T
  code: number
  msg: string
}
type TRequest1 = TResponse<{
  list: number[]
}>  
// TRequest1 = {
//     data: {
//         list: number[];
//     };
//     code: number;
//     msg: string;
// }
```

#### 类泛型

创建可复用组件

``` ts
// 泛型栈类（支持任意元素类型）
class Stack<T> {
  private items: T[] = [];
  
  push(item: T) {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);  // ✅ 合法

const stringStack = new Stack<string>();
stringStack.push("text"); // ✅ 合法
stringStack.push(123);    // ❌ 类型错误
```

使用原型属性推断并约束构造函数与类实例的关系

``` ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```

#### 类型约束

限制泛型范围

``` ts
// 要求 T 必须包含 length 属性
type Lengthwise = {
  length: number
}

function logLength<T extends Lengthwise>(obj: T) {
  console.log(obj.length);
}

logLength("hello"); // ✅ 合法 (length=5)
logLength(42);      // ❌ 错误：数字没有 length

```
####  默认类型参数

提供备用类型

``` ts
// 默认泛型类型为 number
class Pagination<T = number> {
  currentPage: T;
  constructor(page: T) {
    this.currentPage = page;
  }
}

const page1 = new Pagination(1);     // T = number
const page2 = new Pagination<string>("home"); // T = string

```

#### 类型推断

``` ts
// 提取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type FnReturn = ReturnType<() => number>; // number

```

#### 过滤筛选

``` ts
// 过滤出函数类型
type FunctionType<T> = T extends (...args: any[]) => any ? T : never;

type Mixed = string | (() => void) | number;
type FunctionsOnly = FunctionType<Mixed>; // () => void

```

####  keyof 操作符

``` ts
function fun<T, K extends keyof T>(params: T, key: K): T[K] {
  return params[key]
}
fun({ a: 1 }, 'a')
fun({ a: 1 }, 'b')//错误，b不在params的属性中
```

### 条件类型

基于类型条件来定义类型的机制，使用extends关键字。

``` ts
type IsString<T> = T extends string ? true : false;
type A = IsString<"abc">;
```

可以依据传入的参数而返回不同的类型，比如判断传入的参数是否为字符串，如果是字符串则返回 true，否则返回 false

#### infer

infer是只能和extends一起搭配使用，因为infer 的设计初衷是在类型匹配的过程中提取信息，它依赖于 extends 所提供的类型匹配机制。

``` ts
type Example<T> = T extends infer U ? U : never;
```

使用规则：

- infer 必须出现在extends 条件类型的左侧
- 作用是在类型系统中推断某个子类型

和`typeof`区别：

- `typeof`: 获取一个值的类型，即某个值/函数的类型是已经确定好了的，在运行时获取这个值的类型

    ``` ts
    const name = 'hello';
    type T = typeof name; // string

    const user = { name: 'Alice', age: 30 };
    type T = typeof user; // { name: string; age: number }

    function add(a: number, b: number) {
    return a + b;
    }
    type T = typeof add; // (a: number, b: number) => number
    ```

    - 适用场景：
        - 获取变量的类型
        - 获取函数的类型签名
        - 获取对象的结构类型
        - 用于类型守卫中做运行时判断

- `infer`: 推断某个类型的子类型，只能和`extends`一起使用

    ``` ts
    type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
    type R = GetReturnType<() => string>; // string

    type GetElementType<T> = T extends Array<infer E> ? E : never;

    type E = GetElementType<number[]>; // number

    type UnwrapPromise<T> = T extends Promise<infer P> ? P : T;

    type P = UnwrapPromise<Promise<string>>; // string
    ```
    -  适用场景：
        - 提取函数返回值类型（如 ReturnType<T>）
        - 提取数组元素类型（如 ExtractArrayElement<T>）
        - 提取 Promise 内部类型（如 UnwrapPromise<T>）
        - 类型编程中做类型转换和提取


|特性|	typeof	|infer|
|------|------|------|
|作用	|获取值的类型	| 在类型中提取子类型|
|使用位置|	可用于任意类型声明|	只能在 extends 条件类型中使用|
|作用对象|	运行时值（变量、对象、函数等）|	类型（函数类型、数组类型、Promise 等）|
|是否用于类型推断	|是，基于值推断类型|	是，基于类型推断子类型|
|是否影响运行时|	否|	否|
|典型用途	|类型复用、类型守卫	|类型提取、类型转换、泛型编程|

#### 分布式条件类型

分布式条件类型是 TypeScript 中的一种特殊行为，当条件类型作用于泛型类型参数时，如果该类型是 联合类型（A | B），则条件会 分布 到每一个联合成员上，分别计算，再将结果合并成一个新的联合类型。

``` ts
type ToFlag<T> = T extends number ? "num" : "other";

type R = ToFlag<number | string | boolean>;
// 分布后：
// ToFlag<number> | ToFlag<string> | ToFlag<boolean>
// => 'num' | 'other' | 'other'
// => 'num' | 'other'
// => R='num' | 'other'
```

#### 裸类型参数

裸类型参数，就是指这个类型参数 T 被直接写在 extends 的左侧，没有被任何结构包裹住

``` ts
T extends string ? true : false;
```

这里的`T`就是裸类型参数，它没有被放进数组、对象或别的东西里

条件类型中，如果 T 是裸类型参数，并且 T 是联合类型，TypeScript 会将其拆分每个成员进行判断；一旦我们用结构包裹了 T，它就失去了这种分布能力。

- 分布式特性

当我们传入的泛型类型是联合类型，比如 number | string | boolean，而条件类型的写法又是 T extends U ? X : Y 这种形式时，TypeScript 会自动对联合类型的每个成员**分别**进行判断。

``` ts
type ToFlag<T> = T extends number ? true : false;
type Num = ToFlag<number>;// true
type R = ToFlag<number | string | boolean>; // boolean(true | false)
```

但只有当T是裸类型参数时，才会进行分布式判断。

``` ts
type ToFlag<T> = [T] extends [number] ? true : false
type Num = ToFlag<number> // true
type Arr = ToFlag<[number]>; // false
type R = ToFlag<number | string | boolean>; // false
```

因为这里的T已经被包裹起来的，所以是对整体进行一个判断，如果满足条件，则返回true，否则返回false。

条件类型中，如果 T 是裸类型参数，并且 T 是联合类型，TypeScript 会将其拆分每个成员进行判断；一旦我们用结构包裹了 T，它就失去了这种分布能力。

#### 用法

##### 类型推断

对无子类型进行推断：

``` ts
type IsString<T> = T extends string ? true : false;
type GetStyle<T> = IsString<T> extends true ? T : never;
type Test1 = GetStyle<'abc'>;   // 'abc'
type Test2 = GetStyle<number>;  // never
```

对带子类型进行推断：

``` ts
type GetChildType<T> = T extends Array<infer E> ? E : never;
const numbers = [1, 2, 3]; // 类型为number[]
type NumType = GetChildType<typeof numbers>; // NumType 推断为 number
```

##### 类型过滤

从一个联合类型中，筛选出需要的类型

``` ts
type FunctionFilter<T> = T extends (...args) => void ? T : never;S
type Mixed = string | (() => void) | number;
type OnlyFunctions = FunctionFilter<Mixed>; // OnlyFunctions 为 () => void

// 定义条件类型：提取键名，如果属性值是字符串类型
type ExtractStringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never
}[keyof T]; // 映射类型过滤，并提取非 never 的键
```

##### 条件映射

在映射类型中使用条件，例如创建只读或可写版本、选择或排除满足条件的属性。

``` ts
// 将联合类型合并为一个类型，例如 {a: string} & {b: number} => {a: string, b: number}
type Combine<T> = { [K in keyof T]: T[K] }

// 将类型中，string类型的数据转为只读类型，其他类型不变
type ReadonlyIfString<T> = Combine<
  {
    // 筛选出string类型的属性，将其加上readonly变为只读
    readonly [K in keyof T as T[K] extends string ? K : never]: T[K]
  } & {
    // T[K] extends string ? never : K  排除掉string类型
    // K in keyof T  循环非string类型的属性
    // T[K] extends object如果是对象，那么递归处理，当然如果不需要递归，可以直接省略掉这里的extends，直接返回T[K]
    [K in keyof T as T[K] extends string ? never : K]: T[K] extends object
      ? ReadonlyIfString<T[K]>
      : T[K]
  }
>
```

##### 泛型约束

依据不同的输入返回不同的结果

```ts
// 定义条件类型：基于输入类型返回不同输出
type ConvertedType<T> = T extends string ? number : T extends number ? string : never;

// 泛型函数实现（使用类型断言简化）
function convert<T>(input: T): ConvertedType<T> {
  if (typeof input === 'string') {
    return parseInt(input) as ConvertedType<T>; // 实际实现需处理转换
  } else if (typeof input === 'number') {
    return input.toString() as ConvertedType<T>;
  }
  throw new Error('Unsupported type');
}

// 使用示例
const strResult: number = convert("123"); // 输入 string, 输出 number
const numResult: string = convert(123); // 输入 number, 输出 string
```

- 函数重载简化

用条件类型替代多个函数重载，使类型定义更简洁。

``` ts
// 传统函数重载（多个声明）
function process(input: string): number;
function process(input: number): string;
function process(input: any): any {
  // 实现略
}

// 用条件类型简化：定义单一泛型函数
type ProcessedType<T> = T extends string ? number : T extends number ? string : never;

function process<T>(input: T): ProcessedType<T> {
  if (typeof input === 'string') {
    return input.length as ProcessedType<T>; // 示例：返回字符串长度
  } else if (typeof input === 'number') {
    return input.toString() as ProcessedType<T>;
  }
  throw new Error('Invalid input');
}

// 使用示例
const result1: number = process("hello"); // 输入 string, 输出 number
const result2: string = process(42); // 输入 number, 输出 string

```

### 映射类型

允许你基于现有类型动态创建新类型，通过遍历键集合并应用转换规则实现

``` ts
// 基础语法：遍历键集合（K in KeyType）并转换值类型
type MappedType<T> = {
  [P in keyof T]: NewType;
};

```

#### 通过映射实现ts部分使用工具类型

| 工具类型 | 作用 |	映射类型实现 |
| ------ | ------ | ------ |
| Partial<T> | 将所有属性变为可选 |	`[P in keyof T]?: T[P]` |
| Required<T> | 将所有属性变为必选 |	`[P in keyof T]-?: T[P]` |
| Readonly<T> | 将所有属性变为只读 |	`[P in keyof T]: T[P]` |
| Pick<T,K> | 从T中选择一组键位于并集K中的属性 |	`[P in K]: T[P]` |
| Record<K,T> | 构造一个具有类型T的一组属性K的类型 |	`[P in K]: T` |

`-?`：移除可选标记

#### 类型值转换

``` ts
// 所有属性转为 boolean
type Flags<T> = {
  [P in keyof T]: boolean;
};

// 所有属性转为 Promise
type AsyncData<T> = {
  [P in keyof T]: Promise<T[P]>;
};

type UserFlags = Flags<User>; 
// { name: boolean; age: boolean }

type AsyncUser = AsyncData<User>;
// { name: Promise<string>; age: Promise<number> }

```

#### 键名重映射

``` ts
// 添加前缀（模板字面量类型）
type AddPrefix<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

type UserAccessors = AddPrefix<User>;
// { 
//   getName: () => string; 
//   getAge: () => number 
// }

```

#### 深度递归映射

``` ts
// 递归所有层级添加 readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object 
    ? DeepReadonly<T[P]> 
    : T[P];
};

type NestedData = {
  user: {
    name: string;
    permissions: string[];
  };
};

type LockedData = DeepReadonly<NestedData>;
// {
//   readonly user: {
//     readonly name: string;
//     readonly permissions: readonly string[];
//   }
// }

```

#### 枚举键的映射

``` ts
enum LogLevel {
  DEBUG,
  INFO,
  ERROR
}

// 创建基于枚举键的类型
type LogConfig = {
  [Level in keyof typeof LogLevel]: {
    output: "console" | "file";
  };
};

// 结果：
// {
//   DEBUG: { output: "console" | "file" };
//   INFO: { output: "console" | "file" };
//   ERROR: { output: "console" | "file" };
// }

```

### 模板字面量

基于字符串字面量创建新类型，支持模式匹配和动态字符串类型生成

- 动态生成字符串字面量类型。
- 结合联合类型，可以创建所有可能的字符串组合。
- 常用于事件名称、路径模式、CSS类名等场景。
- 类型层面的模式匹配

但是这些都只能是在编译时的校验，无法使用动态的数据校验

#### 遍历组合生成新类型

- 事件名称

``` ts
type EventType = 'click' | 'scroll';
// Capitalize 函数将字符串的第一个字符大写
type EventName = `on${Capitalize<EventType>}`;  // "onClick" | "onScroll"
const handleEvent = (event: EventName) => {
  console.log(`Handling event: ${event}`);
};

handleEvent("onClick");  // ✅
handleEvent("onScroll"); // ✅
handleEvent("onhover");  // ❌ 错误，因为'onhover'不在允许的类型中
```

- props：例如弹窗位置的ts定义

``` ts
type VerticalAlignment = "top" | "middle" | "bottom";
type HorizontalAlignment = "left" | "center" | "right";

type Alignment = `${VerticalAlignment}-${HorizontalAlignment}`;
// 结果为： "top-left" | "top-center" | "top-right" 
//        | "middle-left" | "middle-center" | "middle-right"
//        | "bottom-left" | "bottom-center" | "bottom-right"
```

- 对象取键

``` ts
type NestedKeyPaths<T, Prefix extends string = ''> = 
  T extends object 
    ? {
        [K in keyof T]: K extends string 
          ? (Prefix extends '' ? K : `${Prefix}.${K}`) | NestedKeyPaths<T[K], Prefix extends '' ? K : `${Prefix}.${K}`>
          : never;
      }[keyof T]
    : never;

interface Messages {
  user: {
    login: string;
    logout: string;
  };
  error: {
    notFound: string;
  };
}

type MessageKeys = NestedKeyPaths<Messages>; 
// "user" | "user.login" | "user.logout" | "error" | "error.notFound"

```

#### 结合infer进行模式匹配

结合infer和条件类型，可以从字符串中提取部分内容。


- 提取路由
``` ts
type ExtractRouteParams<T extends string> = T extenhs `${string}/:${infer Param}/${infer Param2}` ? Param | ExtractRouteParams<Param2> : T extends `${string}/:${infer Param}` ? Param : never; 
type Params = ExtractRouteParams<'/user/:userId/post/:postId'>;
// 结果为: "userId" | "postId"

function createRoute<Path extends string>(path: Path) {
  return {
    match: (url: string) => url === path ? {} : null,
    params: {} as ExtractRouteParams<Path>
  };
}
const userRoute = createRoute("/user/:id/post/:postId");
type Params = typeof userRoute.params;
// 自动推断为: { id: string; postId: string }

userRoute.match("/user/123/post/456")?.params.id; // ✅ "123"
```

- 提取url参数

``` ts
// 1. 定义 URL 模式
type URLPattern = `${"http" | "https"}://${string}.${"com" | "org" | "net"}/${string}`;

// 2. 提取域名部分
type ExtractDomain<T> = 
  T extends `${string}://${infer Domain}.${string}/${string}` 
    ? Domain 
    : never;

// 3. 使用
type BlogDomain = ExtractDomain<"https://blog.example.com/posts">;
// 推断为 "blog.example"

// 4. 应用：安全路由处理
function handleRoute<Path extends URLPattern>(path: Path) {
  const domain = path.split("://")[1].split(".")[0];
  // 自动推断 domain 类型为 string
}
```

#### 递归模板

自动生成文档结构

``` ts
type GenerateMarkdown<Depth extends number> = 
  Depth extends 1 ? `# ${string}` :
  Depth extends 2 ? `## ${string}` :
  Depth extends 3 ? `### ${string}` :
  never;

function createHeading<D extends number>(depth: D, text: string)
  : GenerateMarkdown<D> 
{
  return `${"#".repeat(depth)} ${text}` as any;
}

const h1 = createHeading(1, "Title"); // "# Title"
const h3 = createHeading(3, "Subtitle"); // "### Subtitle"
const invalid = createHeading(4, "Error"); // ❌ 类型错误

```

#### SQL校验

``` ts
// 1. 定义安全查询模式
type SafeSelectQuery<Table extends string> = 
  `SELECT ${string} FROM ${Table} WHERE ${string} = ${string}`;

// 2. 执行类型安全查询
function query<T extends string>(sql: SafeSelectQuery<T>) {
  // 执行数据库操作
}

// 使用
query(`SELECT * FROM users WHERE email = 'user@example.com'`); // ✅
query(`DELETE FROM users`); // ❌ 不符合模式
query(`SELECT * FROM products WHERE id = 123`); // ❌ 值未加引号

// 3. 扩展：带 JOIN 的复杂查询
type JoinQuery = 
  `SELECT ${string} FROM ${string} JOIN ${string} ON ${string} = ${string}`;
```

---

#### 5. **API 路径参数自动提取（RESTful 服务）**
**场景**：自动推断路由参数类型  
```typescript
// 1. 定义路由模式
type RoutePattern<Path extends string> = 
  Path extends `${infer Start}/:${infer Param}/${infer Rest}`
    ? { [K in Param]: string } & RoutePattern<`/${Rest}`>
    : Path extends `${string}/:${infer Param}`
      ? { [K in Param]: string }
      : {};

// 2. 实现类型安全路由
function createRoute<Path extends string>(path: Path) {
  return {
    match: (url: string) => url === path ? {} : null,
    params: {} as RoutePattern<Path>
  };
}

// 3. 使用
const userRoute = createRoute("/user/:id/post/:postId");
type Params = typeof userRoute.params;
// 自动推断为: { id: string; postId: string }

userRoute.match("/user/123/post/456")?.params.id; // ✅ "123"
```

---

#### 6. **编译时正则表达式验证（高级模式匹配）**
**场景**：确保字符串符合复杂正则规则  
```typescript
// 1. 创建类型安全邮箱格式
type ValidEmail = 
  `${string}@${string}.${string}` & { 
    _pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ 
  };

// 2. 使用条件类型验证
function assertEmail<T extends string>(
  value: T extends ValidEmail ? T : never
): ValidEmail {
  return value as ValidEmail;
}

// 3. 使用
const valid = assertEmail("user@example.com"); // ✅
const invalid = assertEmail("invalid-email"); // ❌ 类型错误

// 4. 扩展：密码强度验证
type StrongPassword = string & {
  _pattern: /^(?=.*[A-Z])(?=.*[!@#$%^&*])(?=.{8,})/
};

function setPassword(pwd: StrongPassword) {
  // ...
}

```

### 类型谓词/类型收窄：in

允许我们在函数中编写逻辑来判断某个值是否属于特定类型，并在条件分支中缩小类型范围。

用于自定义类型保护，通过返回布尔值的函数来缩小类型范围，增强类型安全性

语法：`parameterName is Type`

- 在函数中执行运行时检查，并告诉TypeScript编译器在某个作用域内变量的具体类型。
- 自定义复杂的类型保护逻辑。

#### 判断数组是否是非空

利用了元组的特性，再加上`in`关键词

```ts
type NonEmptyArray<T> = [T, ...T[]];

function isNonEmptyArray<T>(arr: T[]): arr is NonEmptyArray<T> {
  return arr.length > 0;
}

const arr1: number[] = [1, 2, 3];
const arr2: number[] = [];

if (isNonEmptyArray(arr1)) {
  console.log('First element:', arr1[0]); // 这里arr1被识别为NonEmptyArray<number>
}

if (isNonEmptyArray(arr2)) {
  console.log('First element:', arr2[0]);
} else {
  console.log('Array is empty'); // 会执行这个分支
}

```

#### 类型谓词工厂

创建可配置的类型守卫，避免重复代码

``` ts
// 1. 定义工厂函数
function isType<T>(typeName: string): (value: unknown) => value is T {
  return (value): value is T => {
    return Object.prototype.toString.call(value) === `[object ${typeName}]`
  }
}

// 2. 创建特定类型守卫
const isArray = isType<Array<any>>('Array')
const isDate = isType<Date>('Date')

// 3. 使用
const data: unknown = [1, 2, 3]
if (isArray(data)) {
  data.push(4) // ✅ data 被识别为 any[]
}

const input: unknown = new Date()
if (isDate(input)) {
  console.log(input.getFullYear()) // ✅ 识别为 Date
}
```

#### 联合类型收窄（处理复杂联合类型）

当联合类型成员结构相似但无公共判别属性时，使用类型谓词精确区分

类似于`switch case`语法

``` ts
// 1. 定义易混淆的联合类型
type Result = 
  | { success: true; data: string }
  | { error: true; code: number }
  | { status: "pending"; id: string };

// 2. 没有公共可辨识属性，需要自定义守卫
function isSuccess(result: Result): result is { success: true; data: string } {
  return "success" in result && result.success === true;
}

function isError(result: Result): result is { error: true; code: number } {
  return "error" in result && result.error === true;
}

function handle(result: Result) {
  if (isSuccess(result)) {
    console.log(result.data); // ✅ 收窄到成功类型
  } else if (isError(result)) {
    console.error(result.code); // ✅ 收窄到错误类型
  } else {
    console.log(result.id); // ✅ 收窄到 pending 类型
  }
}
```

#### 递归类型守卫（处理嵌套数据结构）
验证递归结构的数据（如树形结构、嵌套对象）。
```typescript
// 1. 定义树节点类型
type TreeNode = {
  value: number;
  children?: TreeNode[];
};

// 2. 创建递归类型守卫
function isTreeNode(value: unknown): value is TreeNode {
  if (typeof value !== "object" || value === null) return false;

  const obj = value as Record<string, unknown>;
  // 检查 value 属性
  if (typeof obj.value !== "number") return false;
  // 递归检查 children
  if ("children" in obj) {
    if (!Array.isArray(obj.children)) return false;
    return obj.children.every(isTreeNode); // 递归验证每个子节点
  }
  return true;
}

// 3. 使用
const data: unknown = {
  value: 1,
  children: [
    { value: 2 },
    { value: 3, children: [{ value: 4 }] }
  ]
};

if (isTreeNode(data)) {
  // ✅ 识别为 TreeNode 类型
  console.log(data.children?.[0]?.value); // 2
}
```

#### 异步类型守卫（结合 Promise）
在异步流程中验证数据形状（如 API 响应）。
```typescript
// 1. 定义远程数据类型
type RemoteData = { ok: true; data: string } | { ok: false; error: string };

// 2. 创建异步类型守卫
async function isRemoteDataSuccess(response: Promise<RemoteData>)
  : Promise<response is Promise<{ ok: true; data: string }>> 
{
  const result = await response;
  return result.ok === true;
}

// 3. 使用
async function fetchData() {
  const response: Promise<RemoteData> = fetchAPI(); // 模拟 API 调用

  if (await isRemoteDataSuccess(response)) {
    const resolved = await response;
    console.log(resolved.data); // ✅ 类型收窄为 { ok: true; data: string }
  } else {
    const resolved = await response;
    console.error(resolved.error); // ✅ 类型收窄为 { ok: false; error: string }
  }
}
```

#### 品牌化类型（Nominal Typing）验证
在结构类型系统中模拟名义类型检查（如验证 UUID 格式）。
``` ts
// 1. 定义品牌化类型
type UUID = string & { readonly brand: unique symbol };

// 2. 类型守卫验证格式
function isUUID(value: string): value is UUID {
  const regex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
  return regex.test(value);
}

// 3. 使用
function saveId(id: UUID) {
  // 保存操作
}

const input: string = "550e8400-e29b-41d4-a716-446655440000";
if (isUUID(input)) {
  saveId(input); // ✅ 允许：input 被识别为 UUID
} else {
  throw new Error("Invalid UUID");
}
```

#### 类型谓词与断言函数结合

- 类型谓词（Type Predicate）：value is string 
    - 返回 boolean 值，用于类型收窄
- 断言函数（Assertion Function）：asserts value is string 
    - 不返回值，而是抛出异常或继续执行

``` ts
// 1. 定义断言函数（类型谓词 + throw）
// 运行时行为：如果 condition 为 false，函数会抛出错误
// 编译时行为：如果函数成功返回（没有抛出错误），TypeScript 会认为 condition 为 true，并相应地缩小类型范围
function assert(condition: boolean, msg?: string): asserts condition {
  if (!condition) throw new Error(msg);
}

// 2. 自定义类型断言
function assertString(value: unknown): asserts value is string {
  assert(typeof value === "string", "Value must be string");
}

// 3. 使用
function process(input: unknown) {
  assertString(input);
  console.log(input.toUpperCase()); // ✅ input 被识别为 string
}

// 4. 组合类型谓词
function validateUser(user: unknown): asserts user is { name: string; age: number } {
  if (
    typeof user === "object" && 
    user !== null && 
    "name" in user && 
    typeof user.name === "string" &&
    "age" in user &&
    typeof user.age === "number"
  ) {
    return;
  }
  throw new Error("Invalid user");
}

```

## 内置高级类型

|类型	|示例|	说明|
|-------| -------| ------- |
|`Partial<T>`	    | Partial<User>	    | 所有属性变为可选|
|`Required<T>`	| Required<User>	| 所有属性变为必选|
|`Readonly<T>`	| Readonly<User>	| 所有属性变为只读|
|`Record<K,T>`	| Record<'id', string>	    | 键值映射|
|`Pick<T,K>`	    | Pick<User, 'name'>	    | 选择部分属性|
|`Omit<T,K>`	    | Omit<User, 'password'>	| 排除部分属性|
|`Exclude<T,U>`	| Exclude<string|number, number>	| 从联合类型排除|
|`Extract<T,U>`	| Extract<string|number, number>	| 从联合类型提取|
|`NonNullable<T>`	| NonNullable<string|null>	排除 null/undefined|
|`Parameters<T>`	| Parameters<(a: string) => void>	| 获取函数参数类型元组|
|`ReturnType<T>`	| ReturnType<() => number>	| 获取函数返回值类型|
|`ConstructorParameters<T>`	| ConstructorParameters<Error>	| 获取构造函数参数类型|


## 参链

- [不是只有服务能分布，类型也能分布：解密 TypeScript 分布式条件类型](https://juejin.cn/post/7511568225987346467)