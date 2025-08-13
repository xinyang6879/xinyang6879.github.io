---
title: 【css】不常用属性总结
date: 2023-07-28 15:42:03
categories: css
---
# 【css】、不常用样式总结

## 关于文字换行

- `word-break`

  换行规则

- `white-space`

  保留换行和空格

- `word-wrap`(老) / `overflow-wrap`(新) 

- `line-break`

- `hyphens`

  用于英文字符

- <wbr />

  有机会就断开换行

- <br />

  直接换行

  
## 背景

- `background-size`

- `background-image`

  可以设置多背景

- `background-clip`

  控制背景显示区域，还可以设置文字的渐变色

- `background-origin`

  背景定位原点

- `background-repeat`

  背景图片重复设置

- `background-position`

## 叠加属性

即父组件和子组件 同时设置一个属性，其表现并非是子组件覆盖父组件或者父组件覆盖子组件，而是将其两个属性值进行叠加计算

- `opacity`

  用乘法计算出的结果

- `text-decoration`

  会两个都进行显示

