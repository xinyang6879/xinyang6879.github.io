---
title: 【js】属性描述符
categories: js
tag: js
---


# object属性与方法

- 通过使用`Object.getOwnPropertyDescriptor(object, name)`来配置属性

## wirteable

- 设置后无法修改属性值，严格模式下会报错

- 类似于设置了getter和setter，但是setter是个空操作的。
> 如果想用setter实现wirteable，那么需要在严格模式下进行报错

## configurable

- 表明属性是否可以再进行配置以及是否禁止删除这个属性

``` is
Object.getOwnPropertyDescriptor(object, name,{configurable: false})
//报错
//Object.getOwnPropertyDescriptor(object, name,{configurable: true})
```

- ```configureable:false```时可以将```wirteable```从true改为false，但是不能从false改为true

## enumerable

- 是否出现在对象的枚举属性中，设置为```false```后，在```for..in```中就不会再出现这个属性

- ```obj.propertyIsEnumerabel(key)```检查该键是否存在对象(不检查原型链)中且enumerable为true，判断该键是否可枚举。

## Object.preventExtensions(obj)

- 禁止一个对象添加新属性并且保留已有属性

## Object.seal(obj)

- 不能添加新属性，也无法重新配置或者删除任何属性

- 实际上实际上是在一个现有对象上调用 ```Object.preventExtensions(obj) ```，并且把所有现有属性标记为 ```configurable: false```


## Object.freeze(obj)

- 禁止对对象本身及其任意属性的修改

- 实际上是在现有对象上调用 ```Object.seal```，并且把所有属性标注为 ```. wirtable: true```

