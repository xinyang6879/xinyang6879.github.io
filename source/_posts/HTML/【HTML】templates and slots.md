---
title: 【HTML】templates and slots
time: 2023-07-14 14:03:50
categories: HTML
---

# 【HTML】templates and slots

## <h1>概况</h1>

可以用来灵活填充 Web 组件的 shadow DOM 的模板。可以复用相同的标记结构。

## 使用

- 编写一个 template

  直接在页面中编写一个 template 不会直接在页面中显示出来

  ```html
  <template id="test">
    <style>
      p {
        color: white;
        background-color: #666;
        padding: 5px;
      }
    </style>
    <p>测试template</p>
  </template>
  ```

- 注册自定义模板

  ```js
  customElements.define(
    "test-template",
    class extends HTMLElement {
      constructor() {
        super();
        let template = document.getElementById("test");
        let templateContent = template.content;

        const shadowRoot = this.attachShadow({ mode: "open" });
        shadowRoot.appendChild(templateContent.cloneNode(true));
      }
    }
  );
  ```

- 组件显示

  ```html
    <test-template />
  ```

- 页面显示

  [![pC4aUiV.png](https://s1.ax1x.com/2023/07/14/pC4aUiV.png)](https://imgse.com/i/pC4aUiV)

  template 的元素的样式也是独立的，并不会因为父级定义的同一个样式而影响内部。

  虽然 template 和使用在同一个页面，但是如果直接在 Elements 面板里面修改 template 的内容，**不会**影响到渲染的地方。

  [![pC4aWRO.png](https://s1.ax1x.com/2023/07/14/pC4aWRO.png)](https://imgse.com/i/pC4aWRO)

- 添加 slot

  添加的slot的样式只收到父级的影响，并不会受到template中定义的样式影响

  如果是template中并没有定义对于的slot，那么传入的slot并不会被渲染

  ```html
    <template id="test">
      <style>
        p {
          color: white;
          background-color: #666;
          padding: 5px;
        }
      </style>
      <slot></slot>
      <slot name="text1"></slot>
      <p>测试template</p>
    </template>
    <test-template>
        <p>默认slot的内容</p>
        <p slot="text1">具名插槽的内容</p>
    </test-template>
  ```

  [![pC4azLj.png](https://s1.ax1x.com/2023/07/14/pC4azLj.png)](https://imgse.com/i/pC4azLj)

  - 获取slot名

  ``` js
    let el = document.querySelectorAll('test-template p')
      el.forEach(ele=>{
          console.log(ele.slot)
      })
    //  输出："", "text1", "text2"
  ```

## 参链

[使用 templates and slots](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components/Using_templates_and_slots#%E4%BD%BF%E7%94%A8%E6%A7%BD_slots_%E6%B7%BB%E5%8A%A0%E7%81%B5%E6%B4%BB%E5%BA%A6)

[shadow dom解析](https://cloud.tencent.com/developer/article/1009633?areaSource=106001.5)