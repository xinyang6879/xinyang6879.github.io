---
title:【vueTest】vitetest测试Store踩坑记录
categories: VueTest
tag: VueTest
---

## 环境

- node: 20.18.0
- pnpm: 9.14.2
- pinia: 2.3.0
- vue: 3.5.13
- vite: 6.0.5
- vitest: 2.1.8
- jsdom: 25.0.1
- @vue/test-utils: 2.4.6
- @vitest/coverage-v8: 2.1.8
- @vitest/eslint-plugin: 1.1.20
- @vitejs/plugin-vue: 5.2.1

## 被测试的函数用到内容

- npm 包函数
- pinia 数据
- 项目的函数

## 报错记录

### npm 包报错

报错内容： `Error: Failed to resolve entry for package "xxx". The package may have incorrect main/module/exports specified in its package.json.`

解决办法：

在`vite.config.ts`的 alias 添加对应包的 module 路径的配置

比如这个项目中用到的是`@wltech/utils`，然后在 node_modules 中找到这个包的`package.json`，找到`module`字段，那么这个包的路径映射就是`包名/包module路径`

### 未模拟 pinia

在被单测函数中，在对应的 store 取值时会报错`TypeError: Cannot read properties of undefined (reading 'xxx')`

### 直接使用 mock 模拟路径函数

#### default 导出

```js
vi.doMock("xxx", () => ({
  functionName: vi.fn().mockReturnValue(""),
}));
```

如果需要 mock 的文件，里面导出是`export default`，那么在 mock 时需要在数据最外层加上`default`，否则会报错
`Error: [vitest] No "default" export is defined on the "@/hooks/useJudgePcOrAppOrWx" mock. Did you forget to return it from "vi.mock"?
If you need to partially mock a module, you can use "importOriginal" helper inside:`

解决方案

```js
vi.doMock("xxx", () => ({
  default: vi.fn().mockReturnValue(""),
}));
```

所以 pinia 在模拟时，需要加上`default`

#### 模拟数据用导入的数据

```js
import { mockData } from "./mock";
vi.mock("@/store", () => ({
  default: {
    piniaData: mockData,
  },
}));
```

报错：`Error: [vitest] There was an error when mocking a module. If you are using "vi.mock" factory, make sure there are no top level variables inside, since this call is hoisted to top of the file. Read more: https://vitest.dev/api/vi.html#vi-mock`

原因：在`vitest`中，`vi.mock`的 mock 函数是`hoisted`到文件顶部的，其优先级高于 import，导致在`import`之前就执行了`vi.mock`，导致`import`时，pinia 的 store 没有被 mock 到

#### 同一个 describe 中，mock 同一个路径，所有获取的是 mock 最后一次的数据

```js
vi.mock("@/store", () => ({
  default: {
    piniaData: 1,
  },
}));
console.log(Store.piniaData);
// 2
vi.mock("@/store", () => ({
  default: {
    piniaData: 2,
  },
}));
console.log(Store.piniaData);
// 2
```

和上个问题原因一样，mock 是所有一次性执行的，试过在加上下面这些清除数据，但是依旧无效

```js
beforeEach(() => {
  vi.clearAllMocks();
  vi.resetModules();
  vi.stubEnv("VITE_APP_OPEN_COLLECT", "true");
});
afterEach(() => {
  // 清理 mock
  vi.clearAllMocks();
  vi.resetModules();
});
```

## 解决方案：doMock 替代 mock

好处：

- mock 的数据可以通过外部导入
- 每次 mock 然后获取，都是拿到的最新数据

注意点：
- doMock使用时，需要和动态引入被单测的文件
    ``` js
    const { fuName } = await import('需要单测的函数路径');
    const result = fuName();
    expect(result).toEqual("预期结果");
    ```

### 示例

```js
import { describe, test, expect, vi, beforeEach, afterEach } from "vitest";

const mockRunEnv = (
  isMiniApp = true,
  getQueryVariable = "",
  isWxWorkBrowser = false,
  isWxBrowser = true,
  isIOS = false,
  isAndroid = true,
  isPC = false
) => {
  vi.doMock("xxx", () => ({
    isMiniApp: vi.fn().mockReturnValue(isMiniApp),
    getQueryVariable: vi.fn().mockReturnValue(getQueryVariable),
    isWxWorkBrowser: vi.fn().mockReturnValue(isWxWorkBrowser),
    isWxBrowser: vi.fn().mockReturnValue(isWxBrowser),
    isIOS: vi.fn().mockReturnValue(isIOS),
    isAndroid: vi.fn().mockReturnValue(isAndroid),
  }));
  vi.doMock("xxx", () => ({
    isInMobileWechat: vi.fn().mockReturnValue(isMiniApp),
  }));
  vi.doMock("xxx", () => ({
    useJudgePcOrAppOrWx: vi.fn().mockReturnValue(isPC),
  }));
};
describe("函数单测", () => {
  beforeEach(() => {
    vi.clearAllMocks();
    vi.resetModules();
    vi.stubEnv("VITE_APP_OPEN_COLLECT", "true");
  });
  afterEach(() => {
    // 清理 mock
    vi.clearAllMocks();
    vi.resetModules();
  });
  test("在微信浏览器环境中", async () => {
    mockRunEnv(false, "", false, true);
    const { fuName } = await import('需要单测的函数路径');
    const result = fuName();
    expect(result).toEqual("在微信浏览器环境中的预期结果");
  });
  test("在PC环境中", async () => {
    mockRunEnv(false, "", false, false, false, false, true);
    const { fuName } = await import('需要单测的函数路径');
    const result = fuName();
    expect(result).toEqual("在PC环境中的预期结果");
  });
});
```

## 运行
- `pnpm test:unit`
    运行单测
- `pnpm test:coverage`
    查看所有测试覆盖率