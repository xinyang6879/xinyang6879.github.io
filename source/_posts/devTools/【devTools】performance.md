---
title: 【devTools】performance
time: 2023-06-11 10:59:41
categories: 浏览器
---
# Performance面板

## 开始

### 前提

- 保持环境整洁，例如使用隐私模式、清除缓存等

- 确定目标，在执行过程中，尽可能缩短持续时间，避免额外的操作等

### 开始记录

- 打开devTools，切换到Performance面板

- 点击左上角的圆形按钮开始记录

[![pCVcIH0.png](https://s1.ax1x.com/2023/06/11/pCVcIH0.png)](https://imgse.com/i/pCVcIH0)

- 记录过程种可以做一些交互

- 点击stop停止监测

### 生成内容

[![pCVvNc9.png](https://s1.ax1x.com/2023/06/11/pCVvNc9.png)](https://imgse.com/i/pCVvNc9)

- 工具栏：与整体面板有关的操作选项和设置

- overview图表：可视化呈现完整时间轴的基本信息

- Activities：将性能信息以方块式的Activity为单位显示在不同种类的列表中

## 工具栏

[![pCVvshD.png](https://s1.ax1x.com/2023/06/11/pCVvshD.png)](https://imgse.com/i/pCVvshD)

### Disabled Javascript Call Stack

Main列表不会显示js的Call stack信息

### Enable advanced paint instrumentation

记录绘制性能的详细信息，并显示在

- Frames：Frame activity的Layers分页

- Main：Paint activity的Paint Profiler分页

## Overview图表

[![pCVv79g.png](https://s1.ax1x.com/2023/06/11/pCVv79g.png)](https://imgse.com/i/pCVv79g)

| 名称 | 描述 |
| --- | --- |
| FPS，帧数 | <font color=green>绿色</font>竖线越高，FPS 越高。 FPS 图表上的<font color=red>红色</font>块表示长时间帧，很可能会出现卡顿 |
| CPU，CPU 资源 | 指示消耗 CPU 资源的事件类型 |
| NET，网络请求 | 每条彩色横杠表示一种资源。横杠越长，检索资源所需的时间越长。 每个横杠的浅色部分表示等待时间（从请求资源到第一个字节下载完成的时间），可以在屏幕快照下面查看具体的网络请求数据 |

### FPS

绿色方块：每秒帧数的变化，红色、粉色横条为低帧数警告，即可能会让用户感受到卡顿的部分

### CPU

- 灰色：浏览器内部的工作

- 蓝色：HTML请求、文件解析

- 黄色：事件、js

- 绿色：图像处理、画面绘制

- 紫色：样式计算

### NET

- 蓝色：有请求正在执行

- 深色：优先权较高的请求

## Activities

### Main

[![pCVxZE6.png](https://s1.ax1x.com/2023/06/11/pCVxZE6.png)](https://imgse.com/i/pCVxZE6)

- 作用

  显示主线程所有的任务，持续事件超过50ms（长任务）的任务会以红色虚线和右上角的三角形标识

任务底下的Activities依据类型有不同颜色，黄色的js Activity底下以随机颜色显示Call Stack Activities

### Network

[![pCVx5rR.png](https://s1.ax1x.com/2023/06/11/pCVx5rR.png)](https://imgse.com/i/pCVx5rR)

- 左侧的细线：连接至发送请求前

- 浅色区域：等待服务器响应

- 深色区域：下载资源

- 右侧的细线：解析资源

- 左上角的小方块：请求优先级，深色表示高，浅色表示浅

### Frames

显示每一帧画面的详细信息

### Timeings

显示网页使用的重要时间点

- DCL：HTML已经加载且解析完毕

- FP：绘制出默认背景颜色之外的任何内容

- FCP：绘制出任何文字、图片、有颜色的canvas时

- LCP：绘制出页面最大的内容时

- L：解析HTML期间请求的资源都载入完成时

### Experience

显示所有元素位移并计算分数，越低表示页面稳定性越高

### GPU

显示GPU的使用事件

### Raster

定义：浏览器渲染流程中Paint阶段的一环

作用：显示产生Raster时各个线程的信息

## 信息面板：

- summary：显示activity的持续时间，并将期间发生的其他activities分类显示

- botton-up：将同一种activity的运行时间加总

- call tree：以触发关系自上而下显示activities，最上方的称为root activity，是下面各个activities的起点

- event log：以时间顺序显示activities

> 注：activity占用主线程超过50ms会被加上红色三角形，成为long task

### Summary

显示该Activity的持续事件，并将期间发生的其他Activists分类显示

### call tree

[![pCV2ztK.png](https://s1.ax1x.com/2023/06/11/pCV2ztK.png)](https://imgse.com/i/pCV2ztK)

call tree会显示任务由哪些activities组成，若activity的类型为程序代码，则层层展开可以看到函数的call stack

- self time
  
  函数本身的运行时间，并不包含函数执行其他函数的时间

- total time

  函数本身和其下所有函数的运行时间的总和

### bottom-up

会将同一种activity的运行时间加总，因此分页中self time较长的函数通常是性能瓶颈的来源

注：总运行时间长也可能是因为执行次数多

### Event Log

以触发事件顺序显示Activities

## performance monitor

### 打开

1、在devtool按esc打开drawer，在左上角三个点打开

2、在devtools右上角的三个点，打开more tools打开

### 作用

实时监测性能信息，用于检查特定功能是否存在内存泄漏的问题，实时反应内存用量的趋势。一般会把重点放在js heap size（js内存使用占有量）和dom tools

[![pCVf0T1.png](https://s1.ax1x.com/2023/06/11/pCVf0T1.png)](https://imgse.com/i/pCVf0T1)

## Web Vitals

### LCP：前端性能指标，用于表示加载速度

可以在performance的timeing中可以看到

[![pCV4QP0.png](https://s1.ax1x.com/2023/06/11/pCV4QP0.png)](https://imgse.com/i/pCV4QP0)

### FID：表示首次输入延迟

通过rendering分页的Core Web Vitals来判断

[![pCV45z8.png](https://s1.ax1x.com/2023/06/11/pCV45z8.png)](https://imgse.com/i/pCV45z8)

### CLS：表示累计布局偏移

通过performance的Experience的layout shift标签