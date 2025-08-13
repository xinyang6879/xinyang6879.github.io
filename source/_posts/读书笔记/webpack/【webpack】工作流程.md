---
title: 【webpack】工作流程
categories: webpack
tag: webpack
---

http://wepack.wuhaolin.cn/5-2输出文件分析.zip

bundle.js能直接中浏览器运行是因为输出的文件中通过__webpack_require__定义了一个可以在浏览器执行的加载函数，模拟nodejs的require。

原本一个个独立的模块文件被合并到了一个单独的bundle.js的原因是浏览器不能像nodejs一样快速在本地加载一个个模块文件，而必须通过网络请求去加载还未得到的文件。如果模块数量很多，那么加载时间会很长，因此将所有模块都存放到了数组中，执行一次网络加载。

## 流程概括

### 初始化参数

- 从配置文件和shell语句中读取和合并参数，得出最终的参数

#### 发生事件

| event name | desc |
|---|---|
|初始化参数|从配置文件和shell语句中读取合并参数，得出最终的参数，在这个过程中还会执行配置文件中的插件实例化语句 new Plugin（）|
|实例化compiler|用上一步得到的参数初始化compiler实例，compiler负责文件监听和启动编译，在compiler实例中包含了完整的webpack配置，全局只有一个compiler实例|
|加载插件|依次调用插件的apply方法，让插件可以监听后续的所有事件节点。同时向插件传入compiler实例的引用，以方便插件通过compiler调用webpack提供的api|
|environment|开始应用nodejs风格的文件系统到complier对象，以方便后续的文件寻找和读取|
|entry-option|读取配置的entrys，为每个entry实例化一个对应的entrypath，为后面该entry的递归解析工作做准备|
|after-plugins |调用完所有的内置和配置的插件apply方法 |
|after-resolvers|根据配置初始化resolver，resolver负责在文件系统中寻找指定路径的文件|

### 编译阶段

#### 开始编译

- 用上一步得到的参数初始化compiler对象，加载所有配置的插件，通过执行对象的run方法开始执行编译

#### 确定入口

- 配置的entry找出所有的入口文件


#### 编译模块

从入口文件出发，调用所有的配置的loader对模块进行翻译，再找出模块依赖的模块，再递归本步骤直至所有的入口依赖的文件都经过本步骤的处理

#### 完成编译模块

使用loader翻译完所有的模块后，得到每个模块被翻译后的最终内容以及它们之间的依赖关系

#### 发生事件

| 事件| 解释 |
|---|---|
| run| 启动一次新的编译 |
| watch-run | 和run类似，但是是在监听模式下启动编译，可以获取是哪些文件发生变化导致重新启动编译 |
| compile | 告诉插件一次新的编译将要启动，同时会给插件带上compiler对象 |
| compilation | 当webpack以开发模式运行时，每当检测到文件的变化，便有一次新的Compilation被创建。一个Compilation对象包含了当前当模块资源，编译生成资源，变化的文件等，此对象也提供很多事件回调给插件进行扩展 |
| make | 一个新的Compilation创建完毕，即将从entry开始读取文件，根据文件当类型和配置的loader对文件进行编译，编译完成后再找出该文件依赖的文件，递归的编译解析 |
|after-compile | 一次Compilation执行完成 |
|invalid|当遇到文件不存在，文件编译错误等异常时会触发该事件，该事件不会导致webpack退出|


#### compilation阶段的事件
| 事件 | 说明 |
|---|---|
| build-module | 使用对应的loader去转换一个模块 |
| normal- module- loader | 在用loader转换完一个模块后，使用acorn解析转换后的内容，输出对应的ast，方便web pack对后面的代码进行分析 |
| program | 从配置的入口模块开始，分析其ast，当遇到require等导入其他模块的语句时，便将其加入依赖的模块列表中，同时对新找出的依赖模块递归分析，最终弄清所有模块的依赖关系 |
| seal | 所有模块及其依赖的模块都通过loader转换完成，依据依赖关系开始生成chunk |

### 输出阶段
#### 输出资源

根据入口和模块之间的依赖关系，组装成一个个包含多个模块的chunk，再将每个chunk转换成一个单独的文件加入输出列表中。

#### 输出完成

确定好输出内容后，根据配置确定输出的路径和文件名，将文件的内容写入文件系统中

#### 发生事件
| 事件 |描述 |
|---|---|
| should-emit | 所有需要输出的文件已经生成，询问插件哪些文件需要输出，哪些不需要输出 |
| emit | 确定好要输出哪些文件后，执行文件输出，可以在此回调中获取和修改输出的内容 |
| after-emit | 文件输出完毕 |
| done| 成功完成一次完整的编译输出流程 |
| failed| 如果在编译和输出的流程中遇到异常，导致webpack退出，就会直接跳转到本步骤，插件在本事件中可以获取具体的错误原因 |


## 事件流

webpack通过tapable来组织广播打包流程。webpack在运行时会广播事件，插件只需要监听它所关心的事件，就能加入这条线中，去改变打包的结果。

webpack的事件流机制保证了插件的有序性，使得整个系统的扩展性良好。事件流机制应用了观察者模式，和nodejs的eventemitter类似。compiler和compilation都继承至tapable，可以直接中在complier和compilation对象上广播和监听事件。


