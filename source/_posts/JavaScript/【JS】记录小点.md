---
title: 【JS】记录小点
time: 2025/08/04
categories: JS
tag: js
---

### Object.is、==和===

判断两个值是否相等，满足其中一项条件则返回true

- == 

    - 相同类型的数据

        - 数组/对象： 判断引用地址是否相同
        - 字符串：判断相同的字符且顺序相同
        - 数字：+0和-0相等；任何一个NaN不等（两个NaN不等）；值相同相等
        - Boolean： 相同时相等
        - BigInt：值相同相等
        - Simple：引用相同符号相等
        
    - 不相同的数据类型
        - undefined/null：undefined和null相等，其他任意类型都不相等
        - 对象：非引用类型/symbol比较时，会先把对象通过`valueOf()`转换一下，然后再进行比较
        - boolean：如果另一个是数字，那么把布尔值转为数字，true->1，false->0
        - number和string：number转为string，转换失败导致NaN，而导致结果为false
        - number与BigInt：按数值进行比较，如果数字是+∞或者NaN，返回false
        - string与BigInt：使用与BigInt()构造函数相同的算法将string转为BigInt，然后进行比较

    - document.all
        在比较中被视为undefined，因此`document.all == null`/`document.all == undefined`为true

- ===

    不会进行数据格式转换
    - 数据类型不同：返回false
    - 都是对象时：比较引用地址是否相同
    - null !== undefined
    - NaN与任意一个都不相等，包括NaN，即NaN !== NaN为true
    - 数字：+0与-0相等

- Object.is

    - 都是undefined
    - 都是null
    - 都是true或者false
    - 都是长度相同、字符相同、顺序相同的字符串
    - 数据/对象引用的是同一个地址
    - BigInt：有相同的数值
    - Symbol：引用相同的symbol值
    - 都是数字且
        - 都是+0
        - 都是-0
        - 都是NaN
        - 有相同的值且非零且不是NaN

Object.is()与===区别：

 - +0和-0：Object.is()把+0和-0判断为相等，===把-0和+0判断为不相等
 - NaN： Object.is()把NaN判断为相等，===把NaN判断为不相等

Object.is()与==区别：

不会进行数据类型转换

