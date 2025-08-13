---
title:【TS】模块解析
categories: Typescript
---

## 模块

在文件中，如果使用 import 导入或者 export 导出，那么这个文件就会被认为是一个模块

如果一个文件中没有 import 或者 export，那么在里面声明的变量或者函数会被视为全局。即如果有多个文件都不包含导入导出语法，那么当这几个文件中有相同命名的变量或者函数时，就会报错

```ts
// A.ts
const a = 1;

// B.ts
const a = 1; //报错:Cannot redeclare block-scoped variable 'a'
```

在有 import 或者 export 时

```ts
// A.ts
const a = 1;

// B.ts
import "xxx"
const a = 1; //不报错
```

### 特点语法

`export =` 和 `import = require()`

为了兼容Commonjs和Amd的exports语法，ts提供的特有语法

若使用 export =导出一个模块，则必须使用 TypeScript 的特定语法 import module = require("module")来导入此模块。

```js
export = {};
import a = require("./a");
```


## 模块解析流程

分为`Node`和`Classic`，使用 `--moduleResolution`标记来指定使用哪种模块解析策略。若未指定，那么在使用了 `--module AMD` | `System` | `ES2015`时的默认值为`Classic`，其它情况时则为`Node`。

### Classic

#### 相对路径解析

相对于导入它的文件进行解析的

`/root/src/folder/A.ts`文件里的`import { b } from "./moduleB"`

查找流程：

- /root/src/folder/moduleB.ts

- /root/src/folder/moduleB.d.ts

#### 绝对路径解析

从包含导入文件的目录开始`依次向上`级目录遍历

`/root/src/folder/A.ts`文件里的`import { b } from "moduleB"`

查找流程：

- `/root/src/folder/moduleB.ts`
- `/root/src/folder/moduleB.d.ts`
- `/root/src/moduleB.ts`
- `/root/src/moduleB.d.ts`
- `/root/moduleB.ts`
- `/root/moduleB.d.ts`
- `/moduleB.ts`
- `/moduleB.d.ts`

<h1 />

### Node

ts是仿照[Node](https://nodejs.org/api/modules.html#modules_all_together)解析机制

#### Node 大概解析方案

##### 相对路径

`/root/src/folder/A.ts`文件里的`require("./moduleB")`

查找流程：

- 检查`/root/src/moduleB.js`文件是否存在。

- 检查`/root/src/moduleB`目录是否包含一个`package.json`文件，且`package.json`文件指定了一个`"main"`模块。

  - 如果 Node.js 发现文件 `/root/src/moduleB/package.json`包含了`{ "main": "lib/mainModule.js" }`，那么 Node.js 会引用`/root/src/moduleB/lib/mainModule.js`

- 检查`/root/src/moduleB`目录是否包含一个`index.js`文件。 这个文件会被隐式地当作那个文件夹下的"main"模块

##### 绝对路径

`/root/src/folder/A.ts`文件里的`require("moduleB")`

查找流程：

- `/root/src/node_modules/moduleB.js`
- `/root/src/node_modules/moduleB/package.json `(如果指定了"main"属性)
- `/root/src/node_modules/moduleB/index.js`

- `/root/node_modules/moduleB.js`
- `/root/node_modules/moduleB/package.json` (如果指定了"main"属性)
- `/root/node_modules/moduleB/index.js`

- `/node_modules/moduleB.js`
- `/node_modules/moduleB/package.json` (如果指定了"main"属性)
- `/node_modules/moduleB/index.js`


#### TS解析

TypeScript是模仿Node.js运行时的解析策略来在编译阶段定位模块定义文件

> 不同点：

- TypeScript在Node解析逻辑基础上增加了TypeScript源文件的扩展名（ .ts，.tsx和.d.ts）

- 在 package.json里使用字段"types"来表示类似"main"的意义 - 编译器会使用它来找到要使用的"main"定义文件

##### 相对路径

`/root/src/folder/A.ts`文件里的`require("./moduleB")`

查找流程：

- `/root/src/moduleB.ts`
- `/root/src/moduleB.tsx`
- `/root/src/moduleB.d.ts`
- `/root/src/moduleB/package.json` (如果指定了"types"属性)
- `/root/src/moduleB/index.ts`
- `/root/src/moduleB/index.tsx`
- `/root/src/moduleB/index.d.ts`

##### 绝对路径

`/root/src/folder/A.ts`文件里的`require("moduleB")`

- `/root/src/node_modules/moduleB.ts`
- `/root/src/node_modules/moduleB.tsx`
- `/root/src/node_modules/moduleB.d.ts`
- `/root/src/node_modules/moduleB/package.json` (如果指定了"types"属性)
- `/root/src/node_modules/moduleB/index.ts`
- `/root/src/node_modules/moduleB/index.tsx`
- `/root/src/node_modules/moduleB/index.d.ts`

- `/root/node_modules/moduleB.ts`
- `/root/node_modules/moduleB.tsx`
- `/root/node_modules/moduleB.d.ts`
- `/root/node_modules/moduleB/package.json` (如果指定了"types"属性)
- `/root/node_modules/moduleB/index.ts`
- `/root/node_modules/moduleB/index.tsx`

- `/node_modules/moduleB.ts`
- `/node_modules/moduleB.tsx`
- `/node_modules/moduleB.d.ts`
- `/node_modules/moduleB/package.json` (如果指定了"types"属性)
- `/node_modules/moduleB/index.ts`
- `/node_modules/moduleB/index.tsx`
- `/node_modules/moduleB/index.d.ts`

### 路径映射

``` json
// tsconfig.json
    {
        "paths": {
          "packageName": "packagePath"
        }
    }
```

- `paths`是相对于`baseUrl`进行解析