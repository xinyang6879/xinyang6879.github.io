---
title: 【css】contain
time: 2024-1-19 19:44
categories: css
tag: css
---

# 【css】contain

标示了元素及其内容尽可能独立于文档树的其余部分。可以防止元素内部在其包围盒外产生副作用。局限使 DOM 的一部分得以被隔离，且通过将布局、样式、绘制、尺寸或其任意组合的计算限制于 DOM 子树而非整个页面使性能受益。局限也可用于限制 CSS 计数器和引号的作用域。

局限的主要**益处**在于浏览器无需经常重渲 DOM 或页面布局，由此在静态页面的渲染中带来小幅性能收益，在更动态的应用中带来更多的性能收益。

允许我们指定特定的 DOM 元素和它的子元素，让它们能够独立于整个 DOM 树结构之外。目的是能够让浏览器有能力只对部分元素进行重绘、重排，而不必每次针对整个页面。著作权归作者所有。

## 属性值

此属性为五个标准值的子集或两个简写值之一构成的以空格分隔的列表

```css
/* 关键词值 */
contain: none;
contain: strict;
contain: content;
contain: size;
contain: inline-size;
contain: layout;
contain: style;
contain: paint;

/* 多个关键词 */
contain: size paint;
contain: size layout paint;
contain: inline-size layout;

/* 全局值 */
contain: inherit;
contain: initial;
contain: revert;
contain: revert-layer;
contain: unset;
```

> 注：`layout`、`paint`、`strict` 或 `content`将创建：
>
> > 新的包含区块（针对其 position 属性为 absolute 或 fixed 的后代元素）。
> >
> > 新的层叠上下文。
> >
> > 新的区块格式化上下文。

## `none`

元素照常渲染，不应用局限。

## `size`

在行向和块向上将尺寸局限应用于元素。元素尺寸可无视子元素单独计算。

此值不可与 `inline-size` 结合使用。

### 示例

- 原始代码

```html
  <div class="wrap" id="wrap">
    <p></p>
  </div>
  <div class="parent"></div>
</body>
```

```css
.wrap {
  width: 300px;
  border: 5px solid black;
  margin-top: 100px;
  min-height: 200px;
}
.parent {
  width: 300px;
  height: 200px;
  border: 5px solid yellowgreen;
}
p {
  width: 200px;
  height: 90px;
  background-color: royalblue;
}
```

```js
function fn() {
  document.getElementById("wrap").addEventListener("click", () => {
    const child = document.createElement("p");
    child.textContent =
      "这是内容这是内容这是内容这是内容这是内容这是内容这是内容";
    document.getElementById("wrap").appendChild(child);
  });
}

fn();
```

[![pFpDM24.png](https://s11.ax1x.com/2024/01/09/pFpDM24.png)](https://imgse.com/i/pFpDM24)

- 修改后的

```css
.wrap {
  width: 300px;
+ contain: size;
}
```

[![pFpDJVx.png](https://s11.ax1x.com/2024/01/09/pFpDJVx.png)](https://imgse.com/i/pFpDJVx)

## `inline-size`

将行向尺寸局限应用于元素。元素的行向尺寸可无视子元素单独计算

此值不可与 `size` 结合使用。

## `layout`

从页面的其余部分中隔离出元素的内部布局。此值意味着元素外的任意内容和元素内部布局**互不影响**。

### 示例

- 原始样式

```html
<body>
  <div class="wrap" id="wrap">
    <p>这是内容这是内容这是内容这是内容这是内容这是内容这是内容</p>
    <p style="top: 140px;">
      这是内容这是内容这是内容这是内容这是内容这是内容这是内容
    </p>
    <p style="top: 460px;">
      这是内容这是内容这是内容这是内容这是内容这是内容这是内容
    </p>
    <p class="float">这是浮动元素这是浮动元素这是浮动元素</p>
  </div>
  <div class="parent">
    <div class="">
      兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点
    </div>
  </div>
</body>
```

```css
.wrap {
  width: 300px;
  height: 200px;
  border: 5px solid black;
  margin-top: 100px;
}
.parent {
  width: 300px;
  height: 200px;
  border: 5px solid yellowgreen;
}
p {
  width: 200px;
  height: 90px;
  background-color: royalblue;
  position: fixed;
  top: 10px;
  margin-top: 10px;
  color: white;
}
.float {
  float: left;
  margin-top: 150px;
  position: inherit;
}
```

[![pFp6U0A.png](https://s11.ax1x.com/2024/01/09/pFp6U0A.png)](https://imgse.com/i/pFp6U0A)

- 修改样式

```css
.wrap {
  width: 300px;
  height: 200px;
  border: 5px solid black;
+  contain: layout;
  margin-top: 100px;
}
```

[![pFp6wkt.png](https://s11.ax1x.com/2024/01/09/pFp6wkt.png)](https://imgse.com/i/pFp6wkt)

这里有**两个**变化：

1、子元素的定位从相对于窗口定位变为了**相对于父元素定位**

2、子元素层级**高于**父元素的兄弟元素

3、浮动元素**不会影响**其他节点

## `paint`

元素后代不在元素边界外显示。若包含盒在屏外，则浏览器无需绘制其被局限的元素——这些元素因为完全局限于此盒故必定也在屏外。若后代元素溢出包含元素的边界，则此后代元素将被裁剪至包含元素的边框盒。

类似于`overflow:hidden`，但`contain:paint`超出的范围不会再被绘制，所以其对于性能更加友好，但是`overflow`的兼容性更高

[![pFpqEUf.png](https://s11.ax1x.com/2024/01/10/pFpqEUf.png)](https://imgse.com/i/pFpqEUf)

[![pFpqn2Q.png](https://s11.ax1x.com/2024/01/10/pFpqn2Q.png)](https://imgse.com/i/pFpqn2Q)

### 示例

- 原始代码

```html
<body>
  <div class="wrap" id="wrap">
    <p>这是内容这是内容这是内容这是内容这是内容这是内容这是内容</p>
  </div>
  <div class="parent">
    <div class="">
      兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点兄弟结点
    </div>
  </div>
</body>
```

```css
.wrap {
  width: 300px;
  height: 200px;
  border: 5px solid black;
  margin-top: 100px;
}
.parent {
  width: 300px;
  height: 200px;
  border: 5px solid yellowgreen;
}
p {
  width: 400px;
  height: 60px;
  background-color: royalblue;
  margin-top: 10px;
  color: white;
}
```

[![pFpq8aV.png](https://s11.ax1x.com/2024/01/10/pFpq8aV.png)](https://imgse.com/i/pFpq8aV)

- 修改样式

```css
.wrap {
  width: 300px;
  height: 200px;
  border: 5px solid black;
+  contain: paint;
  margin-top: 100px;
}
```

[![pFpqQrn.png](https://s11.ax1x.com/2024/01/10/pFpqQrn.png)](https://imgse.com/i/pFpqQrn)

超出父元素的内容都被隐藏掉了

## `style`

对于可在元素及其后代外产生影响的属性，其影响将不会逃离包含元素。计数器和引号的作用域被限制为元素及其内容。

### 示例

- 原始数据

```html
<ul>
  <li>元素甲</li>
  <li>元素乙</li>
  <li>元素丙</li>
  <li>元素丁</li>
  <li>元素戊</li>
</ul>

<span class="open-quote">
  外
  <span>
    <span class="open-quote">内</span>
  </span>
</span>
<span class="close-quote">闭 </span>
```

```css
ul {
  counter-reset: list-items;
}
li::before {
  counter-increment: list-items;
  content: counter(list-items) "：";
}

body {
  quotes: "【" "】" "〈" "〉";
}
.open-quote:before {
  content: open-quote;
}

.close-quote:after {
  content: close-quote;
}
```

[![pFpjPW8.png](https://s11.ax1x.com/2024/01/10/pFpjPW8.png)](https://imgse.com/i/pFpjPW8)

li 元素按 1、2、3 排序，引号也是按顺序排列

- 修改后

```html
<span class="open-quote">
  外
+  <span style="contain: style">
    <span class="open-quote">内</span>
  </span>
</span>
<span class="close-quote">闭 </span>
```

```diff
+ li:nth-last-child(2n + 2) {
+  contain: style;
+ }
```
[![pFpjFSS.png](https://s11.ax1x.com/2024/01/10/pFpjFSS.png)](https://imgse.com/i/pFpjFSS)

变化点：
1、元素计数器会重新开始，且不影响之前的数据

2、括号没有计算到第二个，直接以【为关闭

## 参链

[MDN-contain](https://developer.mozilla.org/zh-CN/docs/Web/CSS/contain)

[CSS新特性contain的语法、作用及使用场景](https://www.cnblogs.com/xiaonian8/p/14932371.html)

[初探CSS的容器模块](https://www.w3cplus.com/css/deep-dive-into-css-contain.html)