---
title:【vueTest】前言
categories: VueTest
---

## 环境

- vue3

- node：v16.16.0

## 安装包

- jest：测试工具

- vue-jest：用于解析 vue 文件

- babel-jest：用于文件引入的加载解析

- ts-jest： ts 解析加载

- @vue/babel-preset-jsx @vue/babel-helper-vue-jsx-merge-props @babel/core @babel/preset-env @vue/babel-preset-app:用于文件加载

- [@vue/test-utils](https://v1.test-utils.vuejs.org/zh/): 可以实现组件的挂载、与组件交互以及断言组件输出

注：jest、vue-jest、babel-jest大版本号(大版本号.x.x)需要保持一致

## jest

### 命令配置

   - 测试运行

    ```json
    "test": "jest --no-cache"
    ```

    - 覆盖率测试

    ``` json
    "test:coverage": "pnpm run test --coverage"
    ```
### 文件范围

jest 在查找项目的测试文件是使用默认的 glob 匹配模式，对于 non-glob 模式而会自动运行下面三个情况的文件：

- xxx.spec.js

- __tests__文件夹下的 js 和 jsx 文件

- xxx.test.js

### 常用命令

- toBe：匹配器有种类似于object.is或者===
- toContain：是否包含
- toEqual：测试对象的内容是否相等，不比较对象的地址，只关心对象的内容是否一致，递归检查对象或数组的每个字段
- toBeNull：判断是否为null
- toBeUndefined：判断是否为undefined
- toBeDefined：与上相反
- toBeNaN：判断是否为NaN
- toBeTruthy：判断是否为true
- toBeFalsy：判断是否为false
- toHaveLength：数组用，检测数组长度
- toThrow：异常匹配

## 配置

- jest.config.json

  ```js
  const path = require('path')
  const { pathsToModuleNameMapper } = require('ts-jest/utils');
  module.exports = {
    rootDir: path.resolve(__dirname),
    clearMocks: true,
    coverageDirectory: 'coverage',
    coverageProvider: 'v8',
    moduleFileExtensions: ['vue', 'js', 'json', 'jsx', 'ts', 'tsx', 'node'],
    <!-- 文件路径简写定义，vue定义的不生效 -->
    moduleNameMapper: pathsToModuleNameMapper({
        "@packages/*":[ "./packages/*" ]
    }, { prefix: '<rootDir>/' } ),
    transform: {
        <!-- 解析js、ts文件 -->
    '^.+\\.(js|mjs|cjs|ts)$': 'babel-jest',
    <!-- 解析vue文件 -->
        '^.+\\.vue$': '<rootDir>/node_modules/vue-jest',
    }
  }
  ```

- babel.config.js

    ```js
    module.exports = {
    presets: [
        "@vue/babel-preset-jsx",
        ["@babel/preset-env", { targets: { node: "current" } }],
    ],
    };
    ```

## 调试

- 在js文件中需要打断点的地方写上debugger

- 在命令行中加入：`"test:debugger": "node --inspect-brk ./node_modules/jest/bin/jest.js --no-cache --runInBand"`

- 运行此命令后，在chrome浏览器打开：`chrome://inspect`

- 可以看到有一个Remote Target部分中有node_modules/jest/bin/jest.js

- 单击`Inspect`可以打开Chrome Debugger窗口

## 报错

- Cannot destructure property 'config'

  第一次安装版本是29的，后改为26

- babelJest.getCacheKey is not a function

  babel-jest版本需要和jest版本匹配，安装26的
