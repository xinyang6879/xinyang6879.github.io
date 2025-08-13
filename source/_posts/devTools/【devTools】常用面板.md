---
title: 【devTools】常用面板
categories: 浏览器
---

# 【devTools】常用面板

## Elements面板

### 将元素存入变量

- 右击

- 选择store as global variable

- 通过$0、$1...获取对应的元素

[![pCVIrHH.png](https://s1.ax1x.com/2023/06/11/pCVIrHH.png)](https://imgse.com/i/pCVIrHH)

## console面板

### 自动合并相似信息

Group similar message in console，浏览器默认开启的

[![pCVoeqe.png](https://s1.ax1x.com/2023/06/11/pCVoeqe.png)](https://imgse.com/i/pCVoeqe)

### Hide network

隐藏网络的错误提示信息

### preserve log

页面跳转时保留console信息

### show timestamps

打印时，会默认输入每一行信息的时间

打开：devtools右上角设置按钮，perferces下show timestamps

[![pCVoGM8.png](https://s1.ax1x.com/2023/06/11/pCVoGM8.png)](https://imgse.com/i/pCVoGM8)

### $_

作用：获取最近一次的执行结果

[![pCVosMT.png](https://s1.ax1x.com/2023/06/11/pCVosMT.png)](https://imgse.com/i/pCVosMT)

### $和$$

`$`：document.querySelector

`$$`: document.querySelectorAll

### $x

可以使用xpath选择元素。

eg：$x("/html/body/div")

[![pCVofiR.png](https://s1.ax1x.com/2023/06/11/pCVofiR.png)](https://imgse.com/i/pCVofiR)

### debug

执行到该函数时就会触发断点

``` js
const fn = () => {
    return 1;
}
debug(fn)
fn()
```

[![pCVo4Rx.png](https://s1.ax1x.com/2023/06/11/pCVo4Rx.png)](https://imgse.com/i/pCVo4Rx)

[![pCVooQK.png](https://s1.ax1x.com/2023/06/11/pCVooQK.png)](https://imgse.com/i/pCVooQK)

### monitor

函数执行时打印参数值，但无法打印箭头函数的参数

执行unmonitor删除效果

``` js
function fn3(a, b){
    return a+b
}
monitor(fn3)
fn3(1,2)
//输出：VM3370:1 function fn3 called with arguments: 1, 2
```

### monitroEvents

监听并打印元素触发的事件，可以用数组一次性监听多个事件。

执行unmonitorEvents取消监听


### getEventListeners

获取注册在元素上的所有事件监听器

### queryObjects

获取所有原型链中包含该原型的对象

## source面板

### FileSystem

可以直接与本地的文件连接，在devtools修改文件之后，会将本地的文件内容也进行修改

[![pCVXADP.png](https://s1.ax1x.com/2023/06/11/pCVXADP.png)](https://imgse.com/i/pCVXADP)

### overrides

可以以本地的文件取代页面中载入的资源

### 断点

#### 条件断点

在source代码块需要设置时，在对应的行号右键，选择Add Condition Break Point，写上具体的条件

[![pCVXrb6.png](https://s1.ax1x.com/2023/06/11/pCVXrb6.png)](https://imgse.com/i/pCVXrb6)

#### 断点打印信息，Logpoint

在执行时经过该程序代码时打印信息

[![pCVXR8H.png](https://s1.ax1x.com/2023/06/11/pCVXR8H.png)](https://imgse.com/i/pCVXR8H)

#### dom断点

在element tab，右键元素，展开break on，有三种断点形式：

- subtree modifications ：元素内发生变化时暂停，如添加、删除、修改子节点

- attribute modifications ：添加、删除、修改元素本身的属性时暂停

- node removal：元素被删除时暂停，同时删除dom断点

#### 请求断点

在debugger时，点击在XHR/fetch Breapoints列表右上角的+按钮，输入data，回车保存。

#### 事件监听器断点

在debugger时，在Event Listener BreakPoint列表的Control下勾选对应的事件，然后手动触发对应的事件

#### 忽略进入文件

在对应的文件代码内容区域，右键选择Add script to ignore list，添加之后，这个文件就不会在调试时进入了

## NetWork

### 设置区域
[![pCVjISJ.png](https://s1.ax1x.com/2023/06/11/pCVjISJ.png)](https://imgse.com/i/pCVjISJ)

- Use large request rows

  使用宽版的流量记录列表来显示

- Group by frame

  将来自相同iframe的请求聚焦在一起

- show overview

  是否显示时间轴
[![pCVj7O1.png](https://s1.ax1x.com/2023/06/11/pCVj7O1.png)](https://imgse.com/i/pCVj7O1)

- Capture screenslots

  是否截图