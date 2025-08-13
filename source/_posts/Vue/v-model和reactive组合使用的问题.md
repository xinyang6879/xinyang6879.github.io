---
title: v-model 和 reactive 组合使用的问题
categories: vue
---

## 问题描述

### 概述

父组件的数据是一个`reactive`的数据，传入子组件时用`v-model:xxx`的形式传入子组件，那么子组件在此情况下调用`update:xxx`时，并不会引起父组件的数据变化(包含页面和通过`watch`等方式去监听的回调)

### 演示

> 父组件代码

```html
<test v-model:data="testdata" />
```

```js
const testdata = reactive({
  name: "父级参数",
});
watch(
  () => testdata,
  (val) => {
    console.log("父级变化了11111", val);
  },
  { deep: true }
);
```

> 子组件代码

```html
<template>
  <div>
    <input :value="test.name" @input="onIpt"/>
  </div>
</template>

<script setup>
import {reactive, watchEffect} from "vue"
const props = defineProps(['data'])
const emit = defineEmits(['update:data'])
const test = reactive({
  name: "123"
})
const onIpt = e => {
  test.name = e.target.value
  console.log("子组件更新", test)
  emit("update:data", test)
}
watchEffect(() => {
  console.log("子组件接收到的父组件数据更新", test)
  test.name = props.data.name;
})
</script>
```

> 实际操作

在进行输入的时候，只触发了`onIpt`函数，父组件的`watch`和子组件的`watchEffect`都未触发

[![pPIiMwT.png](https://z1.ax1x.com/2023/09/20/pPIiMwT.png)](https://imgse.com/i/pPIiMwT)


## 原因

在[vue](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#limitations-of-reactive)的官方文档上面写明的`reactive`的局限性：

- *有限的值类型*

  它只能用于对象类型 (对象、数组和如 Map、Set 这样的集合类型)。它不能持有如 string、number 或 boolean 这样的原始类型。

- *不能替换整个对象*

  由于 Vue 的响应式跟踪是通过属性访问实现的，因此我们必须始终保持对响应式对象的相同引用。这意味着我们不能轻易地“替换”响应式对象，因为这样的话与第一个引用的响应性连接将丢失

- *对解构操作不友好*

  当我们将响应式对象的原始类型属性解构为本地变量时，或者将该属性传递给函数时，我们将丢失响应性连接

上面第二条说明了`reactive`的响应式是依据  **属性访问** 实现的，而`update:data`的时候是传递的整个对象，所以并不会引起`reactive`的响应式。

## 解决方案

### 父组件的数据改为`ref`

ref是针对整体的一个响应式，因此就不会产生这个问题

### 利用js引用对象的特性，修改prop的属性

``` js
const onIpt = e => {
  const data = props.data;
  data.name = e.target.value
  console.log("子组件更新", data)
  // 实际是否加上update的函数，父组件的watch都会触发
  // emit("update:data", test)
}
```

但是这样不符合单向数据流的特性，因此这个方法不合适

### 父组件监听事件

``` html
  <test v-model:data="testdata" @update:data="onUpdateData"/>
```

``` js
const onUpdateData = (data) => {
  console.log("父组件事件的回调", data)
}
```

这样在子组件调用`update:data`的方法时，也会触发父组件的`onUpdateData`方法