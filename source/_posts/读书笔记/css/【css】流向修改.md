---
title: 【css】流向修改
categories: css
tag: css
---

## direction

### 值

- ltr

默认值，从左到右

- rtl

从右到左

### 场景

- 文字省略号转向，且不影响文字的流向

- 表格呈现顺序

## unicode-bidi

### 属性值

- normal

默认值，元素正常排列。但若设置了direction：rtl，则图片、按钮和问号、加号之类的字符会从右往左显示，但中英文还是从左往右

- embed

其只能作用于内联元素。其会开启一个看不见的嵌入层，然后自己在里面重新排序，所以不会受到外部unicode-bidi影响

其和在元素前嵌入U+202B和后嵌入U+202C效果一致

- bidi-override

重写双向排序规则，通常样式表现为所有字符都按照统一的direction顺序排序

其和在元素前嵌入U+202E和后嵌入U+202C效果一致

## wirting-mode

### 属性

#### css3语法

- horizontal-tb

文本流为水平方向，元素从上往下堆叠

- vertical-rl

垂直方向，从右往左堆叠，和古诗词顺序一致

- vertical-lr

垂直方向，从左往右堆叠

- inherit

- initial

- unset

#### ie

- lr-tb

ie7以上支持，从左往右，从上往下，且下一行水平元素在上一行元素的下面

- rl-tb

ie7+，从右往左，从上往下，且下一行水平元素在上一行元素的下面

- tb-rl

ie7+，从下往上，从右往左，下一个垂直行定位于前一个垂直行的左边

- bt-rl

ie7+，从下往上，从右到左，下一个垂直行定位于前一个垂直行的左边

- tb-lr

ie8+，从上往下，从左往右垂直

- bt-lr

ie8+，从下往上，从左往右垂直

- lr-bt

ie8+，从下往上，从右往左水平

- rl-bt

ie8+，从下往上，右往左水平

- lr

ie9+，svg和html上使用，等同于lr-tb

- rl

ie9+，svg和html上使用，等同于rl-tb

- tb

ie9+，svg和html上使用，等同于tb-rl

### 改变内容

- margin

margin重叠可以在垂直和水平方向上都会进行

- margin：auto

可以实现垂直居中

- text-align：center

图片垂直居中

- text-indent

可以用这个属性实现文字往下一沉的效果

- iconfonts旋转
