---
title: 记录
categories: Typescript
tag: Typescript
---

### 协变与逆变

   ┌───────────┐       ┌───────────┐
   │  父类型    │       │  子类型    │
   │ (宽泛)    │       │ (具体)    │
   └─────┬─────┘       └─────┬─────┘
         │                   │        
         ▼ 协变（同向）       ▼ 逆变（反向）
┌────────────────┐     ┌────────────────┐
│ 集合/返回值类型 │     │ 函数参数类型    │
│   协变领域     │     │   逆变领域      │
└────────────────┘     └────────────────┘

```
{
  "compilerOptions": {
    "strictFunctionTypes": true //严格函数参数（启用逆变检查）
  }
}
```


#### 协变

如果类型A是类型B的子类型，那么由A组成的复杂类型（比如数组、泛型等）也是由B组成的复杂类型的子类型。换句话说，子类型关系与复杂类型构造器保持同向。

``` ts
class Animal {}
class Dog extends Animal {}
const dog: Dog = new Dog();
const animal: Animal = dog;// ✅ 允许：Dog → Animal

// 协变示例：返回值类型
type AnimalFactory = () => Animal;
type DogFactory = () => Dog;

let makeDog: DogFactory = () => new Dog();
let makeAnimal: AnimalFactory = makeDog; // ✅ 允许
```

#### 逆变

如果类型A是类型B的子类型，那么在某些情况下，由B组成的复杂类型反而成为由A组成的复杂类型的子类型。在函数参数中，这种关系是逆变的。

``` ts
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}
type AnimalCallback = (animal: Animal) => void;
type DogCallback = (dog: Dog) => void;

let animalCallback: AnimalCallback = (animal: Animal) => { };
let dogCallback: DogCallback = (dog: Dog) => { };

// 函数参数是逆变的：因为Dog是Animal的子类型，但AnimalCallback是DogCallback的子类型
dogCallback = animalCallback; // 允许，因为AnimalCallback可以赋值给DogCallback
// animalCallback = dogCallback; // 错误，DogCallback不能赋值给AnimalCallback

// 考虑一个函数dogCallback，它接受一个Dog参数。如果我们把一个animalCallback赋值给它，那么dogCallback在调用时传入一个Dog（它是Animal的子类型）是安全的，因为animalCallback可以处理任何Animal，包括Dog。
// 反过来，如果我们将dogCallback赋值给animalCallback，那么当我们用animalCallback去处理一个Cat（也是Animal）时，实际上会调用dogCallback，而它只处理Dog，这样就会出错。
```

### 重载

函数重载允许单个函数根据不同的输入类型或数量提供不同的返回类型和实现逻辑。与传统的面向对象语言不同，TS 重载是类型系统层面的解决方案。

``` ts
// 重载签名（类型声明）
function processInput(input: string): string[];
function processInput(input: number): number;
function processInput(input: boolean): boolean;

// 实现签名（实际实现）
function processInput(input: unknown): unknown {
  if (typeof input === "string") {
    return input.split(""); // 返回字符串数组
  } else if (typeof input === "number") {
    return input * 2; // 返回数字
  } else if (typeof input === "boolean") {
    return !input; // 返回布尔值
  }
  throw new Error("Invalid input");
}

// 使用
const strResult = processInput("hello"); // string[] 类型
const numResult = processInput(10);      // number 类型
const boolResult = processInput(true);   // boolean 类型

```

- 精确的类型推断（根据输入返回对应类型）
- 参数多态（支持不同参数组合）
- 接口契约强化（明确调用约束）
- 代码自文档化（声明即文档）

### implements

有点类似于class的`extends`，只能用于class，是类的专属语法糖

用于**强制类遵循特定接口或类的契约**，确保类结构符合预定义规范。

#### 使用

- class可以继承多个，也可以继承单个

``` ts
interface Serializable {
  serialize(): string;
}

interface Deserializable {
  deserialize(data: string): void;
}

class Config implements Serializable, Deserializable {
  private settings: object = {};
  
  serialize() {
    return JSON.stringify(this.settings); // ✅
  }
  
  deserialize(data: string) {
    this.settings = JSON.parse(data); // ✅
  }
}

```

- 动态接口

``` ts
type FeatureFlags = Record<string, boolean>;

class FeatureService implements FeatureFlags {
  darkMode = true;     // ✅
  analytics = false;   // ✅
  // ❌ 缺少 string 类型的数据
}

```

- 泛型接口

``` ts
interface Repository<T> {
  getById(id: number): T | undefined;
  save(entity: T): void;
}

class UserRepository implements Repository<User> {
  private users = new Map<number, User>();
  
  getById(id: number) {
    return this.users.get(id); // ✅ 返回 User 类型
  }
  
  save(user: User) {
    this.users.set(user.id, user); // ✅
  }
}

```

#### 和extends区别

extends和implements都类似，都用于class上，都需要加上已定义的对应的属性和函数

| 特性	    |implements |	extends |
| ---------| --------- | -------- |
| 继承关系	|实现接口契约（非继承）	| 继承父类属性和方法|
| 多继承	|支持多接口（,分隔）	| 仅支持单类继承|
| 方法实现	|必须重写所有接口方法	| 可选择重写父类方法|
| 构造函数	|不调用父构造函数	| 必须调用super()|
| 类型兼容性	|确保符合接口形状	|创建父子类型关系|

### value is string和value as string类型断言区别

#### `value is string`

属于`类型谓词`，用于自定义类型保护函数

特点：

    - 在类型收窄中使用
    - 返回 boolean 值
    - 用于函数返回类型声明
    - 执行运行时检查

``` ts
// 类型谓词函数
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function processInput(input: unknown) {
  if (isString(input)) {
    // 在这个代码块中，TypeScript 知道 input 是 string 类型
    console.log(input.toUpperCase()); // ✅ 安全调用字符串方法
  } else {
    // input 被收窄为非 string 类型
    console.log('Input is not a string');
  }
}
```

####  `value as string`

属于类型断言，强制将某个值视为特定类型

特点：

    - 直接应用于值
    - 不进行运行时检查
    - 仅影响编译时类型检查
    - 强制转换类型（可能不安全）

使用场景

- 创建类型守卫函数
- 需要运行时类型检查
- 在条件语句中收窄类型
- 构建类型安全的验证函数

``` ts
let someValue: any = "this is a string";

// 类型断言 - 告诉编译器 someValue 应该被视为 string 类型
let strLength: number = (someValue as string).length;

// 或者使用尖括号语法（在 JSX 中不推荐）
let strLength2: number = (<string>someValue).length;
```

使用场景

- 确定值的类型但 TypeScript 不知道
- 从第三方库获取的值，你知道其类型
- 在联合类型中明确指定具体类型
- 与 DOM API 交互时（TypeScript 通常给出更宽泛的类型）

#### 主要区别

| 特性 |	value is string	 |  value as string|
| --------- | ----------- | ----------- |
| 用途|	类型谓词，用于类型保护函数	|类型断言，强制指定值的类型|
| 位置|	函数返回类型位置	    |   直接应用于值|
| 运行时行为|	执行实际的类型检查	|不执行任何运行时检查|
| 安全性	|安全，基于实际检查	|可能不安全，仅是编译时假设|
| 类型收窄|	用于 if 语句中收窄类型	|直接当作指定类型使用|

### type和interface区别

type和interface都可以用于ts中定义类型

- 声明合并
    - type如果重复定义，那么后面会覆盖前面的
    - interface如果重复定义，那么会合并
    ``` ts
    interface User {
        name: string;
    }
    interface User {
        age: number;
    }
    // 合并后：{ name: string; age: number }
   ```
- 扩展方式
   - `interface` 使用 `extends` 关键字进行扩展。
   - `type` 使用交叉类型（`&`）进行扩展。
   ``` ts
   // interface 扩展
   interface A { a: string }
   interface B extends A { b: string }
   // type 扩展
   type C = { c: string };
   type D = C & { d: string };
   ```
   注意：interface也可以扩展type，type也可以使用交叉类型扩展interface。

- implements
    implements可以使用interface和type作为class类的继承
    - interface可以继承联合类型
    - type不可以使用联合类型
    ``` ts
    // 联合类型定义
    type PointUnion = 
    | { x: number; y: number } 
    | { lat: number; lng: number };

    class Point3 implements PointUnion { 
    // ❌ 错误：无法实现联合类型
    }
    ```
- 类型范围
    - `interface` 只能用于定义对象类型（包括函数、索引签名等），不能用于定义非对象类型，如原始类型、联合类型、元组等。
    - `type` 更灵活，可以定义任意类型，包括原始类型、联合类型、元组、映射类型、条件类型等。
- 工具提示
    - interface：鼠标悬停时，不显示具体的定义内容
    - type: 鼠标悬停时，会显示具体的定义内容
- 性能
    - interface：编译更快
    - type：复杂类型时编译稍慢

