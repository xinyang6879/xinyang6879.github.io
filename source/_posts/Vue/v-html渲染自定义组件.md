---
title: v-html渲染自定义组件
categories: vue
---

## v-html渲染自定义组件

### 前景
- 需求场景

  后端返回的特定的元素，前端用穿梭框显示，拿到对应的元素后，将其替换为vant组件进行渲染。

- 坑点：

  `v-html`是vue3用于渲染html的指令，但是由于是直接渲染html的，自定义组件vue不会默认重新转化渲染。因此需要进行处理

### 方案

使用vue自带的`compile`函数进行进行渲染转换，再用`h`函数进行显示

- 将`v-html`所用到的组件设置为全局组件

    ``` js
        import { Lazyload, SwipeItem, Swipe } from "vant";
        const app = createApp(App);
        app.component('Swipe', Swipe)
        app.component('swipeitem', SwipeItem)
    ```

- 新建一个用于渲染的组件

  ``` js
    <script lang="ts">
        import { h,} from 'vue';
        // 如果直接从vue引入会报warning，因此引入路径需要修改
        import {compile} from "vue/dist/vue.esm-bundler.js"

        export default {
        props: {
            html: { type: String, required: true }
        },
        setup(props){
            return () => h(compile(props.html))
        }
        }
</script>
  ```

### 优缺点

- 优点

  自定义组件也可以进行渲染

- 缺点

  - 需要将用到的组件设置为全局组件，那么这样会使初始包变大

  - 无法响应事件。组件标签上的事件无法执行，会有warning；事件只能在内部实现，无法暴露出去

### 参考link

[记录一下vue3 渲染带组件html字符串的方法](https://juejin.cn/post/7153814550414884871)