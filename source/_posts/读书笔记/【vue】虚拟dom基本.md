---
title: 【vue】虚拟dom基本
categories: vue
tag: vue
---

## 原由

在项目运行时，页面状态会不断变化，每当状态变化时，都需要重新渲染。如果每次都直接将页面全部替换更新，那么由于访问 dom 是相对昂贵的，这样会导致非常多的性能浪费。
因此当某个状态变化时，只更新与这个状态相关联的 dom 节点。虚拟 dom 通过状态声称一个虚拟节点树，然后使用虚拟节点树进行渲染。在更新的时候，只需要比较上一次的节点树与当前节点树有何不同，更新不同的地方即可。

## vnode

### 类型

#### 注释节点

- 有效属性
  `text`和`isComment`
- 元素：

```html
<！—注释—>
```

- vnode：

```js
{
text：”注释”，
isComment：true
}
```

#### 文本节点

- 有效属性
  `text`

- 创建过程

```js
function createTextVnode(val) {
  return new VNode(undefined, undefined, undefined, String(val));
}
```

#### 克隆属性

- 特殊属性
  `isCloned`

- 创建过程

```js
function createCloneVnode(vnode, deep) {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  );
  cloned.ns = vnode.ns;
  cloned.isStatic = vnode.isStatic;
  cloned.key = vnode.key;
  cloned.isComment = vnode.isComment;
  cloned.isCloned = true;
  if (deep && vnode.children) {
    cloned.children = createCloneVnode(vnode.children);
  }
}
```

#### 元素节点

- 有效属性
  `tag`: 节点名称，例如 div、li 等
  `data`: 节点上的数据，例如 attrs、class 等
  `children`: 当前节点的子节点列表
  `context`: 当前组件的 vuejs 实例

- 元素：

```html
<div><span>123</span></div>
```

- vnode

```js
{
children: [VNode, VNode]，
context: {…},
data：{…}，
tag: “div”
}
```

#### 组件节点

- 特有属性：
  `componentOptions`: 组件节点的选项参数，包括 propsData、tag、children 等
  `componentInstance`: 组件实例

#### 函数式组件

- 特有属性：
  `functionContext`
  `functionOptions`
