---
title: 【css】滚动贴合
time: 2023-07-15 10:36:58
categories: css
---
# 【css】滚动贴合

## scroll-snap-type

### 作用

可以设置滚动贴合的方向和方式。

### 属性

- `none`

    没有贴合点的效果

- `x`/`y` 或者 `inline`/`block`

    - `x`: 
        
        可以设置水平轴上的捕捉位置
    
    - `y`: 
        
        可以设置垂直轴上的捕捉位置
    
    - `block`
        
        可以设置块状元素排列一个滚动方向的捕捉位置，默认文档流指的是垂直方向

    - `inline`
        
        可以设置内联状元素排列一个滚动方向的捕捉位置，默认文档流指的是水平方向

    在默认文档流的情况下，这`x`和`inline`, `y`和`block`的表现都是一样的。但其实际上还是有区别的

    > `x`和`y`是物理轴，x对应于水平滚动，y对应于垂直滚动。

    > `block`和`inline`是逻辑轴，block对应于垂直滚动，inline对应于水平滚动。

    在改变文档流(`writing-mode`)之后，那么`x`和`inline`的表现就会不一致了。

    <font color=red>普通</font>文档流的情况下，设置的`x`属性，贴合属性的<font color=red>生效</font>
    [![pC5wZWV.png](https://s1.ax1x.com/2023/07/15/pC5wZWV.png)](https://imgse.com/i/pC5wZWV)

    
    <font color=red>改变</font>文档流的情况下，设置的`x`属性，贴合属性的<font color=red>不生效</font>
    [![pC5wnQU.png](https://s1.ax1x.com/2023/07/15/pC5wnQU.png)](https://imgse.com/i/pC5wnQU)
    
    <font color=red>改变</font>文档流的情况下，设置的`inline`属性，贴合属性的<font color=red>生效</font>
    [![pC5wuyF.png](https://s1.ax1x.com/2023/07/15/pC5wuyF.png)](https://imgse.com/i/pC5wuyF)


- `both`

   滚动容器会独立捕捉到其两个轴上的位置（可能会捕捉到每个轴上的不同元素）

   不管是正常文档流和改变文档流后，都会被捕获

- `mandatory`和`proximity`

    - `mandatory`    

        滚动操作结束后，滚动容器必须对齐到最近的一个滚动捕捉点。无论用户滚动的距离有多远，都会强制对齐到最近的捕捉点。这种模式下，用户不能停在两个捕捉点之间。

    - `proximity`
        当滚动操作结束后，只有当最近的滚动捕捉点在一定范围内时，滚动容器才会对齐到该捕捉点。如果最近的捕捉点距离太远，用户可以停在两个捕捉点之间。

    所以，`mandatory`比`proximity`更严格，它会强制滚动容器对齐到最近的滚动捕捉点，而`proximity`则允许用户在两个滚动捕捉点之间停下。

## scroll-snap-align

### 作用

用于子元素的属性，设置其捕获点

### 属性

- `start`

    设置捕获点为元素的起始位置

    [![pC5woYq.png](https://s1.ax1x.com/2023/07/15/pC5woYq.png)](https://imgse.com/i/pC5woYq)

- `center`

    设置捕获点是元素居中

    [![pC5wHpV.png](https://s1.ax1x.com/2023/07/15/pC5wHpV.png)](https://imgse.com/i/pC5wHpV)

- `start`

    设置捕获点为元素的结束位置

    [![pC5wblT.png](https://s1.ax1x.com/2023/07/15/pC5wblT.png)](https://imgse.com/i/pC5wblT)    

## scroll-margin / scroll-padding

### 作用

可以设置贴合间距

[![pC5rEBF.png](https://s1.ax1x.com/2023/07/15/pC5rEBF.png)](https://imgse.com/i/pC5rEBF)

`scroll-margin`是作用于子元素的，而`scroll-margin`是父元素才生效的。这两个属性和`margin`/`padding`一样，可以设置例如`scroll-margin-top`这些属性用于不同方向的样式

## scroll-behavior

### 作用

可以设置滚动时是否需要显示动画

### 属性

- `auto`

    默认值，表示滚动行为立即跳转到目标位置，没有过渡效果

- `smooth`

    滚动行为会平滑地过渡到目标位置，产生一种动画效果。

## `overscroll-behavior`

### 作用

用于控制页面在滚动到底部或顶部时的行为。即如果子元素滚动到最底部后继续滚动，是否需要连带父元素滚动

### 属性

- `auto`

    允许滚动行为传播到父元素
    
- `contain`

    防止滚动行为传播到父元素，但允许页面的弹性滚动效果（比如在移动设备上，滚动到底部时页面会稍微弹一下）
    
- `none`

    不允许滚动行为传播到父元素

## 兼容性

1. `scroll-snap-type` 和 `scroll-snap-align`：这两个属性在所有主流浏览器中都得到了支持，包括 Firefox、Chrome、Edge、Opera 和 Safari。

2. `scroll-margin`：这个属性在 Firefox 和 Chrome（包括基于 Chromium 的浏览器如 Edge 和 Opera）中得到了支持，但在 Safari 中的支持情况不太理想。

3. `overscroll-behavior`和`scroll-behavior`：这个属性在 Firefox 和 Chrome（包括基于 Chromium 的浏览器如 Edge 和 Opera）中得到了很好的支持，但在 Safari 中尚未被支持。