---
title: 【css】border-raduis
categories: css
---

## border-raduis

### 语法

- 只有一个值

表示圆角属性作用在全部四个角上

- 有两个值

第一个作用于左上角和右下角，第二个作用于右上角和左下角

- 三个值

第一个作用于左上角，第二个作用于右上角和左下角，第三个作用于右下角

- 四个值

按照顺时针的方向，左上、右上、右下、左下

### 水平半径和垂直半径

``` css
border-left-top-radius: 10px 20px
border-raduis：10px / 20px
```
表示圆角是水平半径为10px，垂直半径为20px的椭圆产生的

### 重叠曲线

f=min(Lh/Sh, Lv/Sv)

S为半径之和，L为元素宽高，h和v表示方向，f为计算值。

Lh：元素宽

Sh：垂直方向的半径和

Lv：元素高度

Sv：水平方向的半径和

如果f计算值小于1，那么所有圆角半径都乘以f

eg：

``` css
border-top-left-raduis：30px 100%；
border-bottom-left-raduis：30px 100%；
width：150px；
height：100px；
```
左上角和左下角的垂直半径是100%（元素高），水平半径是30px
f=min（150/60，100/200）=0.5

所以渲染结果为

``` css
border-top-left-raduis：15px 50%；
border-bottom-left-raduis：15px 50%；
```

## 渲染细节

如果元素设置了border边框，则圆角半径会被分为内半径和外半径

- padding边缘的圆角大小为设置的border- radius - 边框厚度

- 如果相邻两侧边框厚度不同，则圆角大小将在较厚和较薄边界之间显示平滑过度

- 圆角边框的连接线和直角边框连接线位置一致，但是角度有所不同

- border-raduis不支持负值

- 圆角以外的区域不可点击，无法响应click事件

- borser-radius没有继承性

- 支持transition过渡效果，支持animation动画效果
