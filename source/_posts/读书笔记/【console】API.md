---
title: consoleAPI
categories: js
tags:
- console
- chrome 
- js
---

## assert

### 参数

```
console.assert(boolean, …args)
```

### 输出

但第一个参数为false时进行输出

## count

### 参数

```
console.count(args)

console.countReset(args) // 归零累计值
```
### 输出
累计特定标签出现的次数

## group

### 使用

```
console.group(“start”)
console.log(“info”)
console.groupCollapsed(“child”)
console.log(“data”)
console.groupEnd()
console.log(“data2”)
console.groupEnd()
```

### 作用

让输出信息更加显眼聚合，易于查看

## table

### 使用

```
console.table(rows,[key1,key2]
```

### 作用

以表格打印对象内容，一次性显示更多信息

## time

### 使用

```
console.time(args)
console.timeEnd(args)
```

### 作用

测量时间

## trace

### 使用

```
console.trace()
```
### 作用

打印当前的call stack

## monitor

### 使用

```
function fn(…args){
….
}
monitor(fn)
fn(“data”, “data2”)
```

## 作用

执行该函数时，输出参数。但是无法执打印箭头函数的参数

## monitorEvents

### 使用

```
monitorEvents(window, “click”)
```

### 作用

监听并打印元素触发的事件

## getEventListeners

### 使用

```
getEventListeners(el)
```
### 作用

打印所有注册在元素上的事件监听器

## queryObjects

### 使用

```
queryObjects(obj)
```
### 作用

打印所有原型链中包含该原型的对象
