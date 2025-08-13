---
title: 【webpack】配置项
categories: webpack
---

# entry

## context
在查找相对路径时，以context为根目录。
默认为webpack启动时的所在工作目录。
注：context必须是一个绝对路径的字符串

## entry

类型：string/array/object

|类型|例子|说明|
| ----- | -----|-----|
|string|index•js|入口模块的文件路径，可以是相对路径|
|array|【index。js，index2。js】|入口模块的文件路径，可以是相对路径|
|object|{a：index。js}|配置多个入口，每个入口生成一个chunk|

如果是array，搭配output。library时，只有数组的最后一个入口文件的模块会被导出

# output

## filename
- id：chunk的唯一标识，从0开始
- name：chunk的名称
- hash：chunk的唯一标识的hash值
- chunkhash：chunk内容的hash

# 其他
## target
代码运行的不同环境，默认为web
- web：针对浏览器，所有代码都集中到一个文件中
- node：针对nodejs，使用require加载chunk模块
- async-node：针对nodejs，异步加载chunk代码
- webworker：针对webworker
- electron-main：针对electon主线程
- electron-rendrer：针对electron渲染线程

# loader

- svg- inline- loader：和raw- loader类似，但是会分析svg的内容，去除其不必要的部分，减少svg的体积

# 优化

## 缩小文件搜索范围

- loader配置
> 可以通过include去告诉webpack只有那些文件会被这个loader处理，用exclude排除不需要的文件

- 优化resolve.modules配置
配置webpack去哪些目录下寻找第三方模块
> 指明依赖包的具体目录位置，减少查询时间。

- 优化reslove.main Fields配置
配置第三方模块使用哪个入口文件
> 指明采用字段作为入口文件的描述字段，减少搜索步骤

注：需要考虑是否所有依赖包的第三方入口文件的描述字段，一个模块出错，就会导致所有的代码无法正常运行

- 优化reslove.alias
> 配置库的具体引用路径，减少查询时间

- resolve.extensions
>频率最高的放在最前面。

> 列表不能太长

- 优化module.noParse


## happypack

将任务分解给多个子进程去并发执行，子进程处理完后再将结果发送给主进程

## ParallelUglifyPlugin

uglfyjs是webpack内置的，但是在构建线上代码时会卡住，即是在进行代码压缩。压缩代码需要将代码解析成用object抽象表示的ast语法树，再去应用各种规则分析和处理ast，因此导致这个过程计算量很大，非常耗时

ParallelUglifyPlugin会开启多个子线程，将对多个文件的压缩工作分配给多个子进程去完成，每个子进程还是用uglfyjs压缩。

## 自动刷新

- 文件监听原理：

> 定时获取文件的最后编辑时间，每次存下最新的最后编辑时间，如果发现当前的获取的时间和最后一次保存的编辑时间不一致，就认为该文件发生了变化

>>watchOptions.poll用于控制定时时长，每秒检查多少次

> 当某个文件被判定为发生了变化时，并不会立刻告诉监听者，而是会先缓存起来，搜集一段时间后，再一次性告诉监听者。
>>watchOptions.aggregateTimeout 用于配置这个等待时间

- 自动刷新原理

控制浏览器自动刷新方法：
1. 借助浏览器扩展去通过浏览器的提供的借口刷新，比如webstorm的liveedit

2. 向要开发的网页中注入代理客户端的代码，通过客户端的去刷新页面

3. 将要开发的网页装入一个iframe中，通过刷新iframe去看到最新效果

webpack通过devServer.inline控制是否向每个chunk中注入代理客户端，在开启inline时，devserver会向每个chunk注入代理客户端的代码，而项目中chunk有很多chunk，因此会导致构建缓慢。

启动项目时，可以通过执行命令webpack-dev-server。—inline false完成。那么就会采用第三种方式进行页面刷新。

## 模块热替换

- 原理

和自动刷新原理类似，都需要在开发的网页中注入一个代理客户端来连接devserver和网页。当子模块发生更新时，更新事件会一层层向上传递，直到有某层的文件接收了当前变化的模块，这时就会掉用callback函数去执行自定义的逻辑。如果事件一直往上抛，到最外层都没有事件接收他，那么会直接刷新页面。

- 优化

监听更少的文件，忽略nodemodules目录下的文件。

# cdn加速

- 缓存问题

cdn一般会为资源开启很长时间的缓存，可能会导致最新的发布不能用

> 针对html入口文件，放到自己的服务器中，关闭自己服务器上的缓存，自己的服务器只提供html和数据接口

> 针对静态的js，css，图片等文件，开启cdn和缓存，每个文件带自己内容的hash值

- 使用

> 在output.publicPath设置js的地址

> 在css-loader.publicPath设置css导入的资源地址

> 在webplugin.stylepublicpath设置css文件的地址

## 压缩代码

- js

> 使用 uglifyplugin和paralleugifyplugin（多线程）进行压缩

- 压缩es6

> 优点：对于一样的逻辑，es6代码量少于es5；js对es6的语法做了性能优化，例如针对const声明的变量有更快的读取速度。

> 使用uglifyes进行压缩

- 压缩css

> 使用cssnano进行压缩

## tree shaking去除无用的死代码

配置babel保留es6模块化语句，presets：modules：false

启动时带上—display-used- exports参数，指出哪些函数有用和无用。

启动时带上—optimize-minimize去除无用代码

配置reslove.mainfields，优先采用jsnext：main（第三方包打包出的es6模块）

export default无法做tree shaking，因为导出的是一整个对象

## 提取公共代码

> 使用commonchunkplugin

## 分割代码按需加载

> 原则

1: 将整个网站划分成一个个小功能，再按照每个功能的相关程度将它们分成几类。

2: 将每一类合并为一个chunk，按需加载对应的chunk

3: 不要按需加载用户首次打开网站时需要看到的画面所对应的功能，将其放入执行入口所在的chunk中，以减少用户能感知的网页加载时间

4: 对于不依赖大量代码的功能点，例如依赖echart画图，将其进行按需加载

> webpackChunkName：name

import（/* webpackChunkName：name */ path）：为动态生成的chunk赋予一个名称，方便追踪和调试代码，若不指定名称，那么默认为[id].js

以path路径下的文件为入口重新生成一个chunk，当代码执行到import所在的语句时，才会去加载chunk对应生成的文件。import会返回一个promise，当文件加载成功时可以在promise的then方法获取对应的文件内容。

## scope hoisting

> 实现原理

分析模块之间的依赖关系，尽可能将被打散的模块合并到一个函数中。但前提是不能造成代码冗余，因此只有被引用了一次的模块才能被合并。

## prepack

> 在保持运行结果一致的情况下，改变源码的运行逻辑，输出性能更好的js代码

工作流程：

1: 通过Babel将js源码解析成抽象语法树，以更细粒度地分析yuanma

2: prepack实现了一个js解释器，用于执行源码。借助这个解释器，prepack才能理解源码具体是如何执行的，并将执行过程中的结果返回到输出中。

使用prepack-webpack-plugin插件。


## 可视化打包结果

- webpack-bundle- analyzer

- http://wepack.github.io/analyse
