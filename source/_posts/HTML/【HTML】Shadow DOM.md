---
title: 【HTML】Shadow DOM
time: 2023-07-12 14:27:06
categories: HTML
---

# 【HTML】Shadow DOM

## <h1>概况</h1>

### 定义

能够为 Web 组件中的 DOM 和 CSS 提供了封装，实际上是在浏览器渲染文档的时候会给指定的 DOM 结构插入编写好的 DOM 元素，但是插入的 Shadow DOM 会与主文档的 DOM 保持**分离**，也就是说 Shadow DOM**不存在**于主 DOM 树上。shadow root 节点为起始根节点，在这个根节点的下方，可以是任意元素，和普通的 DOM 元素一样。

并且 Shadow DOM 封装出来的 DOM 元素是**独立**的，外部的配置不会影响到内部，内部的配置也不会影响外部。

[![pCfsSyD.png](https://s1.ax1x.com/2023/07/12/pCfsSyD.png)](https://imgse.com/i/pCfsSyD)

### 查看

控制台-》设置按钮-》Preference -》 Elements -》点击 show user anent shadow dom 的 checkbox

[![pCfrOF1.png](https://s1.ax1x.com/2023/07/12/pCfrOF1.png)](https://imgse.com/i/pCfrOF1)

打开之后可以看到一些元素的真实布局及内容组成。

### 结构

![image.png](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM/shadowdom.svg)

- `Shadow host`：一个常规 DOM 节点，Shadow DOM 会被附加到这个节点上。

- `Shadow tree`：Shadow DOM 内部的 DOM 树。

- `Shadow boundary`：Shadow DOM 结束的地方，也是常规 DOM 开始的地方。

- `Shadow root`: Shadow tree 的根节点。


## <h1>custom element(自定义标签)</h1>

### 概况

可以注册一个自定义标签，`CustomElementRegistry`提供注册自定义元素和查询已注册元素的方法，使用`customElements`可以直接获取其实例。

### 方法

| 方法名            | 作用                                 | 参数          | 返回值 |
| ----------------- | ------------------------------------ | --------------- | --------------- |
| `customElements.define` | 定义了一个自定义元素 | `name`: 自定义元素名; `constructor`: 自定义元素构造器; `options`: 控制元素如何定义(目前只支持extends) | - |
| `customElements.get` | 返回以前定义自定义元素的构造函数 | `name`: 返回引用的构造函数的自定义元素的名字 | 指定名字的自定义元素的构造函数，如果没有使用该名称的自定义元素定义，则为undefined。|
| `customElements.upgrade` | 将更新节点子树中所有包含阴影的自定义元素，甚至在它们连接到主文档之前也是如此 | `root`: 待升级的包含阴影的派生元素节点 | - |
| `customElements.whenDefined` | 当一个自定义节点被定义时走入then，如果这个元素名没有被定义，那么返回的是一直pending状态 | `name`: 自定义元素名 | Promise |

### 使用

``` js
  customElements.define("test-shadow-dom", TestShadowCls)
```

## <h1>使用</h1>

### 挂载 shadow dom

可以调用`Element.attachShadow`将 Shadow Dom 选择挂载或者卸载。此方法返回的是一个类 dom，可以像操作普通 dom 一样对其进行操作

| 参数名            | 作用                                 | 可选值          |
| ----------------- | ------------------------------------ | --------------- |
| `mode`            | `指定 Shadow DOM 树封装模式的字符串` | `open`/`closed` |
| `delegatesFocus ` | `焦点委托`                           | `boolean`       |

- 挂载

  ```js
  const shadow = el.attachShadow({
    mode: "open",
  });
  console.dir(shadow);
  ```

  [![pCfgDbD.png](https://s1.ax1x.com/2023/07/12/pCfgDbD.png)](https://imgse.com/i/pCfgDbD)

- 操作 shadow dom

  ```js
  const html = `<p>测试1</p>`;
  shadow.innerHTML = html;
  ```

  [![pCfRCm8.png](https://s1.ax1x.com/2023/07/12/pCfRCm8.png)](https://imgse.com/i/pCfRCm8)


### 自定义元素

Shadow DOM 可以渲染自定义的元素，类似于 vue3 中的组件，但里面的样式并不互相干扰。

- 新建类

  ```js
  class TestShadowCls extends HTMLElement {
    constructor() {
      super(); //必要的，因为属于子类
    }
  }
  ```

- 创建一个 Shadow DOM 并为其加上需要的数据

  ```js
    addShadow(){
      this.shadow = this.attachShadow({
          mode: "open"
      })
    }

    createChild(){
      const el = document.createElement("div");
      const pel = document.createElement("p");
      pel.textContent = "测试文案";
      pel.setAttribute("class", "test")
      const imgae = document.createElement("img");
      imgae.setAttribute("src", "https://psstatic.cdn.bcebos.com/video/wiseindex/aa6eef91f8b5b1a33b454c401_1660835115000.png")
      imgae.src="https://psstatic.cdn.bcebos.com/video/wiseindex/aa6eef91f8b5b1a33b454c401_1660835115000.png";
      el.appendChild(pel);
      el.appendChild(imgae)
      this.shadow.appendChild(el)
    }
  ```

- 定义自定义的元素名

  ``` js
  customElements.define("test-shadow-dom", TestShadowCls)
  ```

- 使用

  ``` html
  <style>
    p{
      color: red;
    }
  </style>
  <test-shadow-dom />
  ```

  渲染效果如下，里面的文案并没有因为`style`加上的元素样式而生效

  [![pCfqRu4.png](https://s1.ax1x.com/2023/07/12/pCfqRu4.png)](https://imgse.com/i/pCfqRu4)

- 添加样式

  ``` js
    createClass(){
      const style = document.createElement("style");
      style.textContent = `
          .test{
              color: blue;
              font-weight: 600
          }
      `
      this.shadow.appendChild(style)
    }
  ```
  加上了之后，样式生效

  [![pCfL98f.png](https://s1.ax1x.com/2023/07/12/pCfL98f.png)](https://imgse.com/i/pCfL98f)

## 参链

[使用 shadow DOM](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components/Using_shadow_DOM)

[究竟什么是 Shadow DOM？](https://zhuanlan.zhihu.com/p/559759502)
