---
title: Chrome的Performance
categories: 浏览器
tag: 
  浏览器
  Performance
  Chrome
---

# Chrome 的 performance

## 开始记录

可以通过调整这两个参数模拟低网低 cpu 情况 [![bRXxL4.png](https://s1.ax1x.com/2022/03/09/bRXxL4.png)](https://imgtu.com/i/bRXxL4)

点击按钮，刷新页面或者只需要旁边的刷新按钮开始进行记录分析 [![bRjuTA.png](https://s1.ax1x.com/2022/03/09/bRjuTA.png)](https://imgtu.com/i/bRjuTA)

## 操作设置栏 controls

- 可以通过这个下拉框看到之前的分析数据

[![bRjI1K.png](https://s1.ax1x.com/2022/03/09/bRjI1K.png)](https://imgtu.com/i/bRjI1K)

- 其他配置

[![bRvDUA.png](https://s1.ax1x.com/2022/03/09/bRvDUA.png)](https://imgtu.com/i/bRvDUA)

## 页面性能的高级汇总 overview

[![bRzmmF.png](https://s1.ax1x.com/2022/03/09/bRzmmF.png)](https://imgtu.com/i/bRzmmF) [![bWF68H.png](https://s1.ax1x.com/2022/03/09/bWF68H.png)](https://imgtu.com/i/bWF68H)

- 颜色表示
> <font color=#87CEFA>HTML</font>

> <font color=yellow>脚本</font>

> <font color=#9370DB>样式</font>

> <font color=green>媒体资源</font>

> <font color=gray>其他资源</font>

| 名称 | 描述 |
| --- | --- |
| FPS，帧数 | <font color=green>绿色</font>竖线越高，FPS 越高。 FPS 图表上的<font color=red>红色</font>块表示长时间帧，很可能会出现卡顿 |
| CPU，CPU 资源 | 指示消耗 CPU 资源的事件类型 |
| NET，网络请求 | 每条彩色横杠表示一种资源。横杠越长，检索资源所需的时间越长。 每个横杠的浅色部分表示等待时间（从请求资源到第一个字节下载完成的时间），可以在屏幕快照下面查看具体的网络请求数据 |

[![bWCE4J.png](https://s1.ax1x.com/2022/03/09/bWCE4J.png)](https://imgtu.com/i/bWCE4J)

## 火焰图： CPU 堆叠可视化

[![bWEmgH.png](https://s1.ax1x.com/2022/03/09/bWEmgH.png)](https://imgtu.com/i/bWEmgH)


- Timing

> FCP: First Contentful Paint，白屏时间，第一个元素出现的时间

> [LCP](https://zhuanlan.zhihu.com/p/174837488): Largest Contentful Paint，视窗最大可见图片或者文本块的渲染时间

> [FMP](https://blog.csdn.net/qiwoo_weekly/article/details/98818202): First Meaningful Paint，首次有效绘制时间，页面的“主要内容”开始出现在屏幕上的时间点

> DCL: DOMContentLoaded Event，dom 加载完毕时间

> L: Onload Event，完全加载完毕时间

| 名称 | 描述 |
| --- | --- |
| Network | 资源加载顺序及时长 |
| Main | 渲染进程中主线程的执行记录，点击 main 可以看到某个任务执行的具体情况[![bWV5Os.png](https://s1.ax1x.com/2022/03/09/bWV5Os.png)](https://imgtu.com/i/bWV5Os) |
| Timings | 用户交互操作，比如点击鼠标、输入文字、动画等 |
| Compositor | r 合成线程的执行记录，用来记录 html 绘制阶段 (Paint)结束后的图层合成操 |
| Raster | 光栅化线程池，用来让 GPU 执行光栅化的任务 |
| GPU | GPU 进程主线程的执行过程记录，如 可以直观看到何时启动 GPU 加速 |
| Frame | ifream 框架加载详情 |
| Memory | 不同的时间段的执行情况。页面中的内存使用的情况[![bWZwNV.png](https://s1.ax1x.com/2022/03/09/bWZwNV.png)](https://imgtu.com/i/bWZwNV) |


* 在火焰图上看到一到三条垂直的虚线。蓝线代表 DOMContentLoaded 事件。 绿线代表首次绘制的时间。 红线代表 load 事件

* 如果是耗时长的 Task，其右上角会标红，这个时候，我们可以选中标红的 Task，然后放大，看其具体的耗时点。放大后，这里可以看到都在做哪些操作，哪些函数耗时了多少,这里代码有压缩，看到的是压缩后的函数名。然后我们点击一下某个函数，在面板最下面，就会出现代码的信息，是哪个函数，耗时多少，在哪个文件上的第几行等。这样我们就很方便地定位到耗时函数了。
[![bWu7JP.png](https://s1.ax1x.com/2022/03/09/bWu7JP.png)](https://imgtu.com/i/bWu7JP)

## Summary性能摘要

[![bWmDOJ.png](https://s1.ax1x.com/2022/03/09/bWmDOJ.png)](https://imgtu.com/i/bWmDOJ)
- 颜色表示
> <font color=#87CEFA>Loading</font>：网络通信和HTML解析

> <font color=yellow>Scripting</font>：JavaScript执行

> <font color=#9370DB>Rendering</font>：样式计算和布局，即重排

> <font color=green>Painting</font>：重绘

> <font color=gray>other</font>：其它事件花费的时间

> <font color=white>Idle</font>：空闲时间

[事件包含](https://www.cnblogs.com/zjjing/p/9106111.html)


### 参考

[Chrome Performance 使用栗子](https://zhuanlan.zhihu.com/p/29879682)

[Chrome Performance 页面性能分析指南](https://zhuanlan.zhihu.com/p/163474573)

[饼状图分析](https://www.jianshu.com/p/b6f87bac5381)

[performance](https://www.cnblogs.com/xiaohuochai/p/9182710.html)