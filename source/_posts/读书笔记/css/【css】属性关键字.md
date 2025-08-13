---
title: 【css】属性关键字
categories: css
tag: css
---

## inherit

继承，从IE8开始支持

## initial

初始值关键字，把当前css属性的计算值还原为css语法中规定的初始值

注：并不是将其还原为浏览器默认的初始值

## unset

如果当前使用的css属性是具有继承特性的，例如color，那么就和inherit表现一致

如果当时使用的css属性没有继承特性，那么就表现和initial一致

## revert

当前元素还原为浏览器内置的样式

## all

all属性可以重置除unicode-bidi、direction以及css自定义属性外的所有css属性。

``` css
all：initial ｜inherit ｜unset｜revert
```

## fit-content

用子元素的宽高来确定当前元素的数据。

## min-content

最小内容宽度或首选最小宽度

### 替换元素

min-content手当前元素内容自身的宽度。

### CJK文字

如果是没有标点的中文，那么min-content是单个汉字的宽度

如果是包含避头标点（，。？、！等）或者避尾标点（“（等），且line-break不是anywhere的中文，那么min-content是包含标点字符的宽度

### 非GJK文字

min-content是由字符单元的宽度决定的，所有连续的英文字母、数字和标点都被认为是一个字符单元，直到遇到中断字符

### 最终的首选最小宽度

是所有内部子元素中最大的那个首选最小宽度值

## max-content

最大内容宽度

## stretch、available、fill-available

都是让元素尺寸自动填满可用空间

- stretch

弹性拉伸，替换之前的fill- available和available

- available

可用空间，firefox的关键字，需加上-moz-前缀

- fill-avaiable

填充可用空间，需配合-webkit-私有前缀
