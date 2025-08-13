---
title:【vueTest】测试方法
categories: VueTest
---

[api 文档](https://v1.test-utils.vuejs.org/zh/api/wrapper/)

## 基本例子

```js
import WlButton from "@packages/components/WlButton/src/index.vue";
import { mount, shallowMount } from "@vue/test-utils";

describe("test-WlButton.vue", () => {
  test("test1", () => {
    // 渲染组件
    const wrapper = shallowMount(WlButton, {
      // 传递给组件的props参数
      propsData: {
        text: "测试1",
      },
    });
    // 进行断言，判断页面上是否包含测试1的文字
    expect(wrapper.text()).toContain("测试1");
  });
});
```

- mount 和 shallowMount 区别

  mount 方法会渲染整个组件，而 shallowMount 只会渲染一层组件树。

  例如当前测试的组件 A，使用了子组件 B，子组件又包含了孙子组件 C，使用 mount 方法会把 ABC 都挂载出来，而 shallowMount 只会挂载 AB

  shallowMount 在挂载组件之前会对所有子组件进行存根，可以确保对一个组件进行独立的测试，有助于避免测试中因子组件的渲染输出而混乱结果

## 判断条件

### 包含

```js
expect(wrapper.text()).toContain("测试1");
expect(wrapper.text()).toBe("测试1");
```

`text()`方法会返回测试组件渲染的所有文本

`toBe()`：检查判断组件所有文本是否与预期值完全相符合

`toContain()`: 检查是否包含在所检查的字符串中的某个位置.可以判断 string 和 array

## DOM 属性

### attributes

```js
const attrs = wrapper.attributes();
// { class: 'btn-type-primary btn' }
```

`attributes()`: 返回组件属性对象

### classes

```js
const attrs = wrapper.classes();
// ['btn-type-primary', 'btn' ]
```

### 样式

```js
const width = wrapper.element.style.width;
```

其实就是访问元素的 style,如果元素上面 style 上面没有 width,那么实际上是获取不到的

## 查找组件

### `find`

`find()`: 返回渲染树中第一个匹配的节点，类似于`document.querySelector`

```js
// 返回el节点下第一个span元素的文本
el.find("span").text();
```

### `findAll`

`findAll()`: 类似`document.querySelectorAll`，在渲染输出中搜索与选择器匹配的节点，并返回一个包含匹配节点的包装器的类数组对象

```html
<div>
  <div>
    <span>11</span>
    <span>11</span>
    <span>11</span>
  </div>
  <span>11</span>
  <span>11</span>
  <span>11</span>
</div>
```

```js
const list = el.findAll("span");
list.length; //6
expect(list).toHavaLength(6);
```

## 参数

### props()

```js
const wrapper = shallowMount(WlButton, {
  // 传递给组件的props参数
  propsData: {
    text: "测试1",
    a: 1,
  },
});
wrap.props(); // {text: "测试1", disabled: false}
// 返回一个object，除了传入的text，没有传入,但是组件定义的props都能获取到
// 如果传入了组件内部未定义的prop,那么获取时是获取不到的
```

### minix

- 局部
  和 props 类似,在挂载节点时,定义 minix 对象传入.

  挂载组件后会自动调用回调的

  ```js
  const MixinExp = {
    created() {
      console.log("created--mixins");
    },
  };
  test("mixin", () => {
    const comp = mount(WlButton, {
      mixins: [MixinExp],
    });
  });
  ```

- 全局

  ```js
  import Vue from "vue";
  Vue.config.productonTip = false;
  Vue.mixins(MixinExp);
  ```

  在配置项中设置[setupFiles](https://jestjs.io/zh-Hans/docs/configuration#setupfiles-array)选项

  ``` json
  {
    ...,
    "setupFiles": ["filepath"]
  }
  ```

## 定时器

### 假定时器

使用假定时器来替换真实的定时器,调用 jest 的`useFakeTimers`方法时,jest 假定时器会替换真实全局的定时器.定时器被替换后,使用 runTimerToTime 推进假时间

```js
jest.useFakeTimers();
setTimeout(() => {
  console.log("setTimeout");
}, 100);
console.log("setTimeout-after");
jest.runTimersToTime(100);
console.log("setTimeout-after2");
// 输出顺序: setTimeout-after setTimeout setTimeout-after2
```

### 判断定时器是否被清除:spy

使用`jest.spyOn(window, functionName)`,然后再用`expect().toHaveBeenCalledWith()`去判断这个方法是否被调用 就

```js
jest.useFakeTimers();
jest.spyOn(window, "clearTimeout");
setTimeout.mockReturnValue(111);
const timer = setTimeout(() => {
  console.log("setTimeout");
}, 100);
console.log(timer); ///111
jest.runTimersToTime(100);
clearTimeout(timer); //如果没有调用,那么就会fail
expect(window.clearTimeout).toHaveBeenCalledWith(111); //success
```

## mock

vue3 中使用`root.$options.mocks`进行访问,vue2 直接用 this 可以访问

```js
const $aaa = {
  start: () => {
    console.log("start");
  },
  end: jest.fn(),
};
const wrapper = shallowMount(WlButton, {
  mocks: { $aaa },
});
```

```js
// 组件中
if (root.$options.mocks) {
  root.$options.mocks.$aaa.end();
}
```

- 测试 router 方法就是将数据挂载到 mock 里面

## 异步

- 使用 await 等待请求结束

```js
const fetchData = jest.fn(() => Promise.resolve(""));
```

```js
test("http", asycn () => {
  const data = await fetchData()
  expect(data).toBe("")
})
```

- 使用`flushPromises`(@vue/test-utils 内置),会在 promise 回调运行结束后再进行下面的行为

```js
Promise.resolve().then(() => {
  console.log(1);
});
console.log(2);
await flushPromises();
console.log(3);
// 输出顺序: 2 1 3
```

## 事件

### trigger 方法

需要在组件内部手动触发传入的事件

```js
test("event", () => {
  const onClick = jest.fn();
  const wrapper = shallowMount(WlButton, {
    propsData: {
      onClick,
    },
  });
  wrapper.find(".btn").trigger("click");
  expect(onClick).toHaveBeenCalled();
});
```

```js
// 组件文件
const onButtonClick = () => {
  (root.$attrs as any).onClick()
};
```

### 自定义事件

使用`emitted`可以监测到组件 emit 的事件

```js
expect(wrapper.emitted("onClick")).toHaveLength(1);
```

### 测试自定义事件

使用`el.vm`可以访问到具体的元素,调用`$emit`就可以模拟组件的事件

## 表单

### 修改输入框内容

```js
// 方式1
el.find("input").value = "测试输入内容";
el.find("input").trigger("change");

// 方式2
el.find("input").setValue("测试输入内容");
```

### 单选框

```js
// 方式1
el.checked = true;
// 方式2
el.setChecked();
```
