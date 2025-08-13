---
title: 【css】层叠上下文
categories: css
---
# 【css】层叠上下文

## 层叠水平

层叠水平，stacking level，其决定了同一个层叠上下文中元素在z轴上的显示顺序

所有元素都有层叠水平，包括层叠上下文和普通元素。但对于普通元素的层叠水平只局限于当前的层叠上下文中。

zindex在某些情况下可以影响层叠水平，但仅限于定位元素以及flex盒子的子元素，而层叠水平是所有元素都存在。

### flex与zindex

flex布局下zindex生效是父元素设置了flex，子元素设置了zindex，子元素的zindex生效，flex当前元素是不生效的
- 非flex布局下：
  遵循后来者居上原则，son3覆盖了son2
  ``` html
    .father{
      width: 600px;
      height: 150px;
      z-index: 3;
      background-color: aquamarine;
    }
    .son2{
      z-index: 2;
      width: 150px;
      height: 150px;
      display: inline-block;
      background-color: white;
    }
    .son3{
      z-index: 1;
      width: 150px;
      height: 150px;
      display: inline-block;
      margin-left: -20px;
      background-color: chocolate;
    }
    <div class="father">
      son1
      <div class="son2">son2</div>
      <div class="son3">son3</div>
    </div>
  ```

  [![pEkTk1U.png](https://s21.ax1x.com/2025/01/20/pEkTk1U.png)](https://imgse.com/i/pEkTk1U)

- flex布局下

  遵循zindex谁大谁上的原则，son2覆盖了son3
  ``` diff
    .father{
      width: 600px;
      height: 150px;
      z-index: 3;
  + display: flex;
      background-color: aquamarine;
    }
    .son2{
      z-index: 2;
      width: 150px;
      height: 150px;
      display: inline-block;
      background-color: white;
    }
    .son3{
      z-index: 1;
      width: 150px;
      height: 150px;
      display: inline-block;
      margin-left: -20px;
      background-color: chocolate;
    }
    <div class="father">
      son1
      <div class="son2">son2</div>
      <div class="son3">son3</div>
    </div>
  ```

  [![pEkTZnJ.png](https://s21.ax1x.com/2025/01/20/pEkTZnJ.png)](https://imgse.com/i/pEkTZnJ)

## 层叠顺序

stacking order，其表示元素层叠时有特定的垂直显示顺序，其是规则而非概念

### 层叠顺序规则

从下到上的顺序为：

层叠上下文 background/border -> 负zindex -> block块状水平盒子 -> float浮动盒子 -> inline水平盒子 -> zindex=auto/0 -> 正zindex

注：

- 位于最下面的background/border指层叠上下文元素的边框和背景色，每个层叠顺序规则仅适用于当前层叠上下文元素的小世界

- inline水平盒子指的是 inline/inline-block/inline- table

- background/border作为装饰属性，float/block一般用于布局，inline都是内容，而内容的重要性是相对优先的

## 层叠准则

当元素发生层叠的时候，其覆盖关系遵循下面两天准则

- 谁大谁上：当有明显的层叠水平标识时，如生效的zindex，在同一个层叠上下文领域，层叠水平值大的一个覆盖小的一个

- 后来居上：当元素的层叠水平一致、层叠顺序相同时，在dom流中处于后面的元素覆盖前面的元素

## 层叠上下文

### 定义

层叠上下文（Stacking Context）是一个HTML元素的三维概念。在CSS2.1规范中，每个盒模型的位置是三维的，分别是平面画布上的x轴、y轴和表示层叠的z轴。

它是页面中的一部分，决定了元素在**Z轴**上的堆叠顺序。在层叠上下文中，元素可以在三维空间中相对于其他元素前后排列，类似于一叠卡片。

层叠上下文的创建是可以嵌套的，也就是说一个层叠上下文内部可以包含另一个层叠上下文.在这种情况下，内部的层叠上下文会被整体地放在外部层叠上下文的某个层叠等级上，而内部层叠上下文的z-index值不会影响到外部层叠上下文的堆叠顺序。

> 作用

这个概念主要用于解决覆盖和重叠元素的问题，例如当元素的部分透明或定位不同导致元素重叠时，层叠上下文就会决定哪个元素在上面，哪个在下面。

### 特性

- 层叠上下文的层叠水平高于普通元素

- 层叠上下文可以阻断元素的混合模式

- 层叠上下文可以嵌套，内部层叠上下文及其所有子元素均受制于外部的层叠上下文

- 每个层叠上下文和兄弟元素独立，即当进行层叠变化或者渲染时，只需要考虑后代元素

- 每个层叠上下文是自成体系的，当元素发生层叠时，整个元素被认为是在父层叠上下文的层叠顺序中

### 层叠等级

<h1></h1>

层叠等级（stacking level），又称层叠级别、层叠水平。在**同一个**层叠上下文中有作用。

> 作用

- 决定该层叠上下文中的层叠上下文元素在z轴上的上下顺序

- 在普通元素中，它决定这些普通元素在z轴上的上下顺序

### 层叠顺序

<h1></h1>

层叠顺序（stacking order）表示元素发生层叠时有着特定的垂直显示顺序。

注: 层叠上下文和层叠水平是概念，层叠顺序是规则

[![堆叠顺序规则](https://s1.ax1x.com/2023/07/28/pCzHpgf.png)](https://imgse.com/i/pCzHpgf)


### 层叠上下文与层叠顺序

- 如果层叠上下文元素不依赖于zindex，其层叠顺序是zindex：auto，可看成zindex=0

- 如果层叠上下文依赖于zindex数值，则层叠顺序由zindex决定

从下到上的顺序为：

层叠上下文 background/border -> 负zindex -> block块状水平盒子 -> float浮动盒子 -> inline水平盒子 -> zindex=auto/0 ，不依赖zindex的层叠上下文-> 正zindex

因此，元素一旦成为定位元素，其zindex会自动生效，此时其zindex就是默认的auto，即0级别，根据上面的层叠顺序表，其会覆盖float/block/inline元素


## 触发条件

### 创建

#### 根层叠上下文

指页面根元素，可以看成是html元素.  HTML中的根元素<html></html>本身就具有层叠上下文，称为“根层叠上下文”。这也时绝对定位元素在没有其他定位元素限制时，会相对于浏览器窗口定位的原因。

页面中所有元素一定处于至少一个层叠结界中

#### 定位元素与传统层叠上下文

- position=relative/absolute

- position=fixed（Firefox/ie，不包括chrome

chrome在此情景下，会将此元素作为天然层叠上下文元素

当满足上面其一条件时，zindex！=auto时，会创建层叠上下文

#### css3

- flex元素，同时zindex！= auto

- opacity ！= 1

- mix-blend-mode ！= normal

- filter ！= none

- isolation = isolate

- will-change为上面2～6的任意一个

- -webkit- overflow-scrolling = touch



## 比较规则

- 同一个层叠上下文中，

  - 元素层级不同
  
    比较“内部元素层叠级别”，层叠等级大的元素显示在上，层叠等级小的显示在下

  - 两个元素的层叠等级相同
  
    后面元素堆叠到前面元素的上面，即“后来者居上”

- 在不同层叠上下文中

  比较“父级元素层叠等级”，元素显示顺序以“父级”的层叠级别来决定元素的先后顺序，与自身的层叠顺序无关

- 当页面中两个元素发生堆叠时，其或其祖先元素必处于同一层叠上下文（最差情况下同处根层叠上下文）

## 参链

- [层叠上下文、层叠等级、层叠顺序](https://blog.csdn.net/m0_56229413/article/details/115458436)
