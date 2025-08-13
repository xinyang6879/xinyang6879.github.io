---
title: 【css】记录小点
categories: css
---

## border：0与border：none区别

### 相同点

两者都可以将元素的边框设为不可见

### 不同点

- 性能差异

- border：0

其表示为将元素边框设置为0像素，虽然在页面上无法看见，但是浏览器依旧会对边框进行渲染。因此渲染的是一个像素为0的b order。

即border：0依旧会占用内存

- border：none

其表示的是将元素边框设置无，因此浏览器在解析时，并不会对其进行渲染

即其不会占用浏览器内存

- 兼容

在ie6/7的button元素中，border：none并不会生效

## 垂直居中

### 图片

``` js
display: table-cell;
// 子元素
vertical-align: center;
```

### 文字

- line-height

``` css
height: 20px;
line-height: 20px;
```

## line-height

- line-height指的是两行文字基线与基线之间的高度

- 单位为%时，计算规则是相对于当前元素的font-size计算的，即0.x*fontsize

- 无单位时，是相对于当前元素的font-size计算的，即x*fontsize


## vertical-align


- 元素没有设置时，继承的是父元素line-height的像素值，即如果父元素的line-height是%单位，那么浏览器计算出来的实际line-height值才会被子元素继承。

## 元素选择器

| 选择器 | 说明 |
|---|---|
| # | id选择器 |
| . | class选择器 |
| M N | 后代选择器，选择M元素内部后代的所有N元素 |
| M>N | 子代选择器，选择M元素内部后代的第一个子级N元素 |
| M～N | 兄弟选择器，选择M元素后所有的同级N元素 |
| M+N | 相邻选择器，选择M元素相邻的下一个N元素，M和N是同一级 |

属性选择器:
- A[attr]: 下面其带有指定属性的元素,可以有多个,例如.name[title][id]:选择name的class下同时带有title和id属性的元素
- A[attribute~=value]: 选取属性值中包含指定词汇的元素.例如p[class~=name],只要是个p元素且class包含的name的元素就符合
- A[attribute|=value]: 选取属性值中全等于指定词汇的元素,例如p[class|=name],只要是p元素且class属性是name的元素就符合,如果class有其他的,那就不符合
- A[attribute^=value]: 选取属性值以value开头的元素
- A[attribute$=value]: 选取属性值以value结尾的元素
- A[attribute*=value]: 选取属性值包含value的元素
- A[attribute|=value]: 选取属性值等于value或者以value开头的元素

## 包含块

### 作用

与css盒子模型类似。作用是为这个矩形内部的后代元素提供一个参考，一个元素的大小和定位往往是由该元素所在的包含块决定的。

### 类型

- 根元素

html元素，它没有父元素，是页面中最顶端的元素。根元素存在的包含块，被称为初始包含块。

- 固定定位元素

若元素的position=fixed，那么它的包含块是当前可视窗口，即当前浏览器的窗口

注意：**小程序**中position=fixed会**受到box-shadow**的影响

    box-shadow 会创建一个新的层级上下文，这可能会影响 fixed 元素的层级表现，导致它不如预期地固定在视口上。

- 静态定位和相对定位元素

若元素的position=relative/absolute，那么其包含块是由离他最近的块级祖先元素创建的。祖先元素必须是block、inline-block或table-cell

- 绝对定位元素

若元素的position=absolute，那么其包含块是最近的position！=static的元素。祖先元素可以是块元素，也可以是行内元素

## em

em是相对于当前元素的父元素计算的

但是当用于fontsize时，如果多有个嵌套的元素，那么从父层到子层，其元素字号会越来越小

## background-size: auto 渲染规则

- 如果图像没有内在尺寸和内在比例

按背景定位区域的大小进行渲染，等同于设置属性100%

- 水平和垂直方向同时具有内在尺寸

按图像原始大小进行渲染

- 没有内在尺寸，但有内在比例

渲染效果等同于contain

- 只有一个方向有内在尺寸，但具有内在比例

图像会拉伸到该内在尺寸的大小，同时宽高比符合内在比例

- 只有一个方向有内在尺寸，没有内在比例

图像有内在尺寸的一侧会拉伸到该内在尺寸大小，没有设置内在尺寸的一侧会拉伸到背景定位区域大小

##background-size：一个为auto，一个为非auto

- 有内在比例

会拉伸到指定的尺寸，宽高依然保持原有的比例

- 没有内在比例

图像会拉伸到指定尺寸。

如果图像有内在尺寸，则auto到计算尺寸就是图像的尺寸

如果图像没有内在尺寸，那么auto的计算尺寸就是背景定位区域的尺寸

## background-position

- 只有一个值

例如：20px == 20px center

如果只有一个值，那么无论是具体的数值或者百分比或者是关键字，另一个值一定是center

- 两个值

- 两个都是关键属性值

left、right表示水平，top、bottom表示垂直。

不能包含对立的方位，即top bottom是无效的

- 一个是关键属性值，一个是数值或者百分比

如果第一个值是百分比或者数值，那么表示水平方向，另一个关键属性值表示垂直方向

如果第一个值是关键属性值，那么表示水平方向，另一个百分比或者数值表示垂直方向

- 两个值都是数值或者百分比

第一个表示水平方向，第二个表示垂直方向

- 3个值或者4个值

数值和百分比表示偏移量，第一个值一定要是关键属性值，这个关键属性值表示偏移方向

## opacity

opacity不等于1的元素会创建一个层叠上下文，层叠顺序会变高

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