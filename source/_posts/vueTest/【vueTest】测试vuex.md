---
title:【vueTest】测试vuex
categories: VueTest
---

# 文件分开测试

需要将 getter、mutations 分成不同的文件。

在测试文件内，传递一个假数据给 vuex，然后拿到经过处理后的数据，去判断是否符合预期

## getter

```js
test("getter", () => {
  const name = "啦啦啦啦";
  const origin_name = {
    name,
  };
  const data = getter.getter_name(origin_name);
  expect(data).toBe(name);
});
```

## mutation 测试

mutation 内容

```js
mutation_name(state, val){
    state.name = val
}
```

测试文件内容

```js
test("mutation", () => {
  const new_name = "啦啦啦啦";
  //   创建一个假的state存储对象
  const origin_name = {
    name: "哇哇哇",
  };
  //   调用方法，修改内容
  mutations.mutation_name(origin_name, new_name);
  expect(origin_name.name).toBe(new_name);
});
```

origin_name 的 name 变为了 new_name 的内容

## action

```js
import { flushPromises } from "@vue/test-utils";
import { httpTest1 } from "./mock";
jest.mock("./mock.js");

test("action", async () => {
  expect.assertions(1);
  const name = "哈哈哈";
  // 创建一个假的上下文
  const ctx = {
    commit: jest.fn(),
  };
  httpTest1.mockImplemntationNoce((calledWith) => {
    return Promise.resolve();
  });
  //  调用actions
  action.action_state_name(ctx, name);
  await flushPromises();
  //  判断是否调用对应的mutation
  expect(ctx.commit).toHaveBeenCalledWith("mutation_name");
});
```

actions 测试流程比 mutation 和 getter 复杂,因为涉及到请求和调用 commit,所以需要去模拟一个 mock,以此来拦截数据.

通过创建一个假的上下文,传给 action,以此来判断是否调用了 mutation

# 放在一个 Store

在beforeEach中,创建一个store,那么在此之后的测试代码中,每次都会调用这个beforeEach的函数,以便每个测试都有一个全新的store

```js
let localStore, store;
test("store", () => {
  beforeEach(() => {
    localStore = {
      getters: {
        getter_age: jest.fn(),
      },
      mutations: {
        mutation_age: jest.fn(),
      },
    };
    store = new Vuex.Store(localStore);
  });
});

test("vue", () => {
  console.log("localStore", localStore);
//   组件可以通过$store拿到测试的vuex数据
  shallowMount(componentName, {
    $store: localStore,
  });
});
```

# 缺点

## 分开写

需要模拟许多vuex方法,可能使测试通过,但是实际的测试还是不正确,因为完全是由外部传入的state

## 一起写

测试不够具体,如果测试失败,那么排查问题比较困难