---
title: 【HTML】记录
categories: HTML
tag: HTML
---

### input type=file的cancel事件

mdn并未写明当前的`input`标签支持cancel事件

#### 触发条件

在pc端中，点击上传的输入框，然后在弹出的文件选择器中点击取消按钮，即可触发当前这个事件

#### 表现

``` html
<!-- 子组件 -->
<input type="file" id="ipt1" @cancel="cancle2" placeholder="上传123"></input>

<!-- 父组件 -->
<childComp @cancel="cancle1"></childComp>
```

在vue3项目中，会触发子组件的`cancle2`事件

若在子组件中没有`cancle2`事件，那么这个事件会冒泡到父组件中，执行`cancle1`事件。因此若父组件有监听子组件的cancel事件，那么在这个情况下会直接执行，可能会导致触发其他问题

浏览器：谷歌、火狐都支持

#### 监听

> vue
- 组件直接监听
- 子组件未监听，父组件监听
- window.addEventListener('cancel'): 会执行两次，触发元素是window
- document.getElementById('').addEventListener('cancel'): 不执行

> 普通html
- window.addEventListener('cancel'): 只执行一次，触发元素是window
- document.getElementById('').addEventListener('cancel'): 只执行一次，触发元素是对应的元素

技术栈：vue3+ts+pinia+vite+vantui
项目介绍：PC+H5，支持多种资料类型上传，可配置资料留资，悬浮码等功能，增加内容分发等功能，企业微信侧边栏可快速发送

● 负责核心复杂功能模块的设计与实现
1、资料上传：支持pdf、图片、公众号等5种资料类型上传，可配置资料留资，悬浮码等功能，增加内容分发等功能，企业微信侧边栏可快速发送