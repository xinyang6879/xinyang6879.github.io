---
title: PerformanceObserver
categories: 浏览器
tag: 浏览器
---

### 介绍

PerformanceObserver 和 Performance 作用都是用于获取网站性能数据的

Performance 可以获取当前页面从打开到加载完成的各个时间（基于浏览器打开的时间，而不是依据系统的时间，以 us 为单位），属于同步函数，调用时就会给出具体的数据

[PerformanceObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceObserver)监测性能度量事件，属于异步函数，调用时可能不会立马执行，而是到对应的时间时才会调用其回调函数

而 MutationObserver 可以监听 DOM 节点的属性变化，比如添加、删除、修改属性。

### 使用

- 新建一个 PerformanceObserver 实例，并传入一个回调函数用于处理数据

  ```js
  // 1. 创建观察者实例
  const observer = new PerformanceObserver((list) => {
    list.getEntries().forEach((entry) => {
      // 处理每个性能条目
      console.log("-------------------------------");
      console.log(`Entry Type: ${entry.entryType}`, entry);
      console.log("*******************************");
    });
  });
  ```

- 指定需要观察的内容

  - 多项观察

  ```js
  // buffered标识是否获取历史的数据
  observer.observe({
    entryTypes: [
      "element",
      "event",
      "first-input",
      "largest-contentful-paint",
      "layout-shift",
      "long-animation-frame",
      "longtask",
      "mark",
      "measure",
      "navigation",
      "paint",
      // "resource",
      "visibility-state",
    ], // 可同时监听多种类型
    buffered: true,
  });
  ```

  - 单项观察

  ```js
  observer.observe({ type: "event", buffered: true });
  ```

注：实测下来，`event`类型不支持在`entryTypes`中，只能通过`type`去处理

- 关闭

```js
observer.disconnect();
```

### 监听类型

[entryType](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry/entryType)包含了多种类型

[PerformanceElementTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceElementTiming)

[LargestContentfulPaint](https://developer.mozilla.org/en-US/docs/Web/API/LargestContentfulPaint)

- `element`：可以获取元素的加载时间，需要搭配`elementtiming`属性使用

  ```html
      <img elementtiming="test-ele" src="xxx"></p>
      observer.observe({ type: 'element', buffered: true });
  ```

  ```js
   // 返PerformanceElementTiming类型
   {
     duration: 0,
     element: img,// 监听的元素
     entryType: 'element',
     identifier: 'test-ele',// 元素的标识符
     name: 'img-paint',
     loadTime: 0,//图片加载时间，文本的话是0
     renderTime： x,
     startTime: x,
     url: "src",//图片的话是图片地址，文本为空
     intersectionRect: DOMRectReadOnly,
     naturalWidth: "图片原本宽度，文本是0",
     naturalHeight: "图片原本高度，文本是0",
   }
  ```

- `event`: 可以监听页面的点击事件，但这个属性只能单独使用，在 entryTypes 中会不生效
  时间间隔可以通过 startTime-processingEnd 获取

  ```js
  observer.observe({
    type: "event",
    // buffered: true,
    durationThreshold: 16, // 只记录持续时间超过16ms的事件
  });
  ```

  ```JS
    // 返PerformanceElementTiming类型
  {
      cancelable: Boolean，
      duration： number,
      entryType: "event",
      name: "click" | "pointerleave" | "pointerout" | "pointerup" | "pointerover" | "pointerenter" | "mouseout" | "mouseleave" | "mouseover" | "mouseup",
      processingEnd: number,
      processingStart: number,//事件调度开始的时间
      startTime: number,//创建事件的时间，可以被视为用户交互发生时间的代理。
      target: HTMLElement //触发的元素
  }
  ```

- `first-input`：首次用户交互延迟，当用户第一次点击时触发，可以作为 FID 的上报数据

  返回和`event`事件返回一致，时间间隔=processingStart-startTime

- `largest-contentful-paint`：LCP，最大内容绘制时间，可能会触发多次，存在可能上报一个当前最大的，但后续如果又有一个内容宽高大于之前的，那么还会触发一次

  ```js
  // 返回LargestContentfulPaint类型
  {
      duration： number,
      entryType: "largest-contentful-paint",
      element: HTMLElement,
      startTime: number, // 如果此元素的renderTime值不为0，则返回其值，否则返回此条目的loadTime值。
      loadTime: number,//元素加载时间
      renderTime: number,//元素呈现到屏幕上的时间。如果元素是在没有Timing Allow origin标头的情况下加载的跨原点图像，则可能不可用。
      size：number, //元素的固有大小返回为面积（宽*高）
      url： string,// 如果元素是图片，那么这个就是图片的地址
  }
  ```

- `layout-shift`: CLS，页面布局偏移

  返回[LayoutShift](https://developer.mozilla.org/en-US/docs/Web/API/LayoutShift)类型

  ```js
  {
      hadRecentInput: boolean, // 500ms内是否有输入
      lastInputTime: number，// 上次输入的时间
      sources：LayoutShiftAttribution[]， //返回有哪些元素进行了偏移
      startTime： number，// 布局转换开始的时间
      value: number，// 布局偏移分数，影响分数（被偏移的视口分数）乘以距离分数（作为视口分数移动的距离）计算得出的。
  }
  ```

- `long-animation-frame`: LoAF，长动画帧，延迟超过 50ms 的渲染更新

  返回[PerformanceLongAnimationFrameTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongAnimationFrameTiming)

  ```js
  {
      blockingDuration: number,// 计算LoAF中持续时间超过50ms的所有长任务，从每个任务中减去50ms，将渲染时间加到最长的任务时间上，并将结果相加来计算的。指示主线程被阻止响应高优先级任务（如用户输入）的总时间
      duration: number,// 处理整个LoAF所用的时间
      firstUIEventTimestamp: number,// 在当前动画帧期间排队的第一个UI事件（如鼠标或键盘事件）的时间。
      renderStart: number, //渲染周期的开始时间，其中包括Window.requestAnimationFrame（）回调、样式和布局计算、ResizeObserver回调和IntersectionObserver回调。
      script: PerformanceScriptTiming[],
      startTime: number,
      styleAndLayoutStart: number, //当前动画帧的样式和布局计算所花费的时间段的开始。
  }
  ```

- `longtask`: 监听长任务，执行时间＞ 50ms 的任务，可以用于检测卡顿

  返回[PerformanceLongTaskTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongTaskTiming)

  ```js
  {
      attribution： TaskAttributionTiming[],
      duration: number, // 任务开始和结束之间经过的时间，粒度为1ms。
      entryType: 'longtask',
      name: cross-origin-ancestor" | "cross-origin-descendant" | "cross- origin-unreachable" | "multiple-contexts" | "same-origin-ancestor" | "same-origin-descendant" | "same-origin" | "self" | "unknown",
      startTime: number, // 任务开始时间
  }
  ```

- `paint`: 用于检测 FCP（首次内容绘制）/FP（首次绘制）时间

  返回[PerformancePaintTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformancePaintTiming)

  ```js
  {
      name: "first-paint" | "first-contentful-paint",
      startTime: number, // 任务开始时间
      duration: 0
  }
  ```

- `resource`: 用于监听资源加载时间

  返回[PerformanceResourceTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceResourceTiming)，其中包含资源各个的加载、链接开始和结束时间等详细信息

- `visibility-state`: 检测当前网页是否在屏幕中显示

  ```js
  {
      name: "hidden" | "visible",
      startTime: number, // 任务开始时间
      duration: 0
  }
  ```

- 自定义：`measure`和`mark`，调用是通过 performance 调用的，而不是创建的 PerformanceObserver 实例

  ```js
  performance.mark("mark-start");
  const { data } = await apiPost(params);
  performance.mark("mark-end");
  performance.measure("mark-finish", "mark-start", "mark-end");
  ```

  measure 输出的`duration`是 start 和 end 的时间间隔，通过两个 mark 的 startTime 相减得到的

### 使用实例

#### 测量 FCP/FP

```js
// 1. 创建观察者实例
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntriesByName("first-contentful-paint")) {
    console.log("FCP:", entry.startTime);
  }
  for (const entry of list.getEntriesByName("first-paint")) {
    console.log("FP:", entry.startTime);
  }
});
observer.observe({
  entryTypes: ["paint"],
  buffered: true,
});
```

#### 测量 LCP

```js
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log("-------------------------------");
    console.log(`LCP: ${entry.size}`, entry);
    console.log("*******************************");
  });
});
observer.observe({
  type: "largest-contentful-paint",
  buffered: true,
});
```

#### 测量 FID

```js
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`FID: ${entry.processingStart - entry.startTime}`, entry);
  });
});
observer.observe({
  type: "first-input",
  buffered: true,
});
```

#### 测量 CLS

```js
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log("-------------------------------");
    console.log(`CLS: ${entry.value}`, entry);
    console.log("*******************************");
  });
});
observer.observe({
  type: "layout-shift",
  buffered: true,
});
```

#### 测试代码块执行时长

```js
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}执行时间: ${entry.duration}ms`, entry);
  });
});
observer.observe({
  type: "measure",
  buffered: true,
});
// 需要测量的代码块
performance.mark("test-mark");
await getList();
performance.mark("test-mark-end");
performance.measure("test-measure", "test-mark", "test-mark-end");
```

#### 检测长任务

``` js
const observer = new PerformanceObserver(list => {
  list.getEntries().forEach(entry => {
    console.log("-------------------------------");
    console.log(`长任务耗时: ${entry.duration}ms`, entry);
    console.log("*******************************");
  });
});
observer.observe({
  type: "longtask",
  buffered: true
});
```

#### 用户交互时长

``` js
const observer = new PerformanceObserver(list => {
  list.getEntries().forEach(entry => {
    console.log("-------------------------------");
    console.log(
      `用户${entry.name}事件: ${entry.processingStart - entry.startTime}ms`,
      entry
    );
    console.log("*******************************");
  });
});
observer.observe({
  type: "event",
  buffered: true,
  durationThreshold: 16
});
```

#### 长动画帧

``` js
const observer = new PerformanceObserver(list => {
  list.getEntries().forEach(entry => {
    if (entry.duration > 50) {
      console.log("----------------------------------------");
      console.log(`总耗时：${entry.duration}`, entry);
      entry.scripts.forEach(script => {
        console.log(
          `${script.sourceURL} 在 ${script.invoker} 耗时 ${script.duration} ms`
        );
        console.log(
          {
            sourceURL: script.sourceURL,
            duration: script.duration,
            functionName: script.sourceFunctionName,
            invoker: script.invoker
          },
          script
        );
      });
      console.log("*****************************************");
    }
  });
});
observer.observe({
  type: "long-animation-frame",
  buffered: true
});
```

### 参链

- [前端PerformanceObserver使用](https://juejin.cn/post/7516353515142348863)
- [Performance在性能方面的完全运用](https://juejin.cn/post/7370963861645557769#heading-4)
- [前端如何监控用户访问界面时发生了卡顿](https://juejin.cn/post/7407084026739294258?from=search-suggest)