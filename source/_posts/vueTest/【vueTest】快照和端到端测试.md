---
title:【vueTest】快照和端到端测试
categories: VueTest
---

# 快照测试

## 作用

快照测试是获取代码的快照，如果之前对应的文件有快照的文件，那么会与之前保存的快照文件进行比较，如果不匹配那么测试失败

jest 在比对快早文件时，是比对的序列化值。

## 测试

### 基本测试

```js
test("snapshot-test1", () => {
  const wrap = shallowMount(Button);
  expect(wrap.element).toMatchSnapshot();
});
```

运行之后，如果该文件之前没有快照文件，那么会在测试文件同目录下的`__snapshots__`文件夹新增一个`snap`文件，后续比对就是依据这个文件进行比对的

可以在命令后加`-- -u`(`npm run test -- -u`)表示每次都更新 snap 比对失败文件。也可以加上`-- --watch`(`npm run test -- --watch`)会在有失败快照时，在命令行按`i`可以浏览失败的快照测试，按`u`更新快照文件

# 端到端测试

## 作用

自动运行浏览器和测试文件检查文件行为是否正确。

## 环境

- 需要配置 java 环境

[java 文件下载](https://www.oracle.com/java/technologies/downloads/#jdk19-windows)

- 安装包

  ```js
  npm i nightwatch selenium-server chromedriver
  ```

- nightwatch.config.js

  ```js
  const seleniumServer = require("selenium-server");
  module.exports = {
    src_folders: ["test/e2e"],
    output_folder: "test/e2e/reports",
    selenium: {
      start_process: true,
      server_path: seleniumServer.path,
      host: "127.0.0.1",
      port: 4444,
      cli_args: {
        "WebDriver.chrome.driver": require("chromedriver").path,
      },
    },
    test_settings: {
      chrome: {
        desiredCapabilities: {
          browserName: "chrome",
        },
      },
    },
  };
  ```

- 命令配置

  ```js
   "test:e2e": "nightwatch --config nightwatch.config.js --env chrome"
  ```

## 测试

```js
module.exports = {
  test1: function (browser) {
    browser
      .url("http://localhost:3000/#/")
      // 测试页面元素
      .waitForElementVisible(".button", 2000)
      // 测试路由
      .assert.urlContains("WlButton")
      .end();
  },
};
```

运行时，需要先将项目跑起来，然后再执行e2e的执行命令