---
title: 从地址栏到页面展示的流程
categories: 浏览器
tag: 浏览器
---

## 浏览器进程

<h1></h1>

> 1、<font color=red>UI 线程</font>会判断输入的内容是搜索关键词还是 URL

- 如果是搜索关键词，跳转至默认搜索引擎对应都搜索 URL，
- 如果输入的内容是 URL，则开始请求 URL。

> 2、UI 线程将关键词搜索对应的 URL 或输入的 URL 交给<font color=red>网络线程</font>

- UI 线程使 Tab 前的图标展示为加载中状态


> 3、网络线程发出请求，获取请求返回内容

- 先查询是否有url是ip还是域名，如果是域名，先去进行查看是否有对应的ip解析缓存，没有就进行DNS查询
- 建立三次握手（AKC，seq这些）
- 发送请求
- 服务器返回请求
- 如果收到服务器的 301 重定向响应，它就会告知 UI 线程进行重定向然后它会再次发起一个新的网络请求。
- 根据响应头中的 Content-Type 字段来确定响应主体的媒体类型
- 如果媒体类型是一个<font color=red>HTML</font>文件，则将响应数据交给<font color=red>渲染进程</font>
- 如果是<font color=red>zip </font>文件或者<font color=red>其它</font>文件，会把相关数据传输给<font color=red>存储线程</font>，下载管理器。
- 浏览器会进行 <font color=red>Safe Browsing </font>安全检查，如果域名或者请求内容匹配到已知的恶意站点，网络线程会展示一个<font color=red>警告页</font>。除此之外，网络线程还会做<font color=red> CORB</font>（Cross Origin Read Blocking）检查来确定那些敏感的跨站数据不会被发送至渲染进程
- 四次挥手

> 4、<font color=red>网络线程</font>确信浏览器可以导航到请求网页，网络线程会通知<font color=red> UI 线程</font>数据已经准备好，UI 线程会查找到一个 <font color=red>渲染进程</font>进行网页的渲染。

- 浏览器为了对查找渲染进程这一步骤进行优化，考虑到网络请求获取响应需要时间，所以在第二步开始，浏览器已经预先查找和启动了一个渲染进程，如果中间步骤一切顺利，当 network thread 接收到数据时，渲染进程已经准备好了，但是如果遇到重定向，这个准备好的渲染进程也许就不可用了，这个时候会重新启动一个渲染进程。

> 5、浏览器进程 会向 渲染进程 发送 IPC 消息(进程通信的一种方式)来确认导航

- 浏览器进程将准备好的数据发送给渲染进程，渲染进程接收到数据之后，又发送 IPC 消息给浏览器进程，告诉浏览器进程导航已经提交了，页面开始加载。

> 6、当导航提交完成后，渲染进程开始加载资源及渲染页面，当页面渲染完成后（页面及内部的 iframe 都触发了 onload 事件），会向浏览器进程发送 IPC 消息，告知浏览器进程，这个时候 UI thread 会停止展示 tab 中的加载中图标。

## 渲染进程-关键渲染路径(CRP)

<h1></h1>

> 当渲染进程接受到导航的确认信息后，开始接受来自浏览器进程的数据，<font color=red>主线程</font>会解析数据转化为 DOM 对象。

- 解析到图片、CSS、JavaScript 脚本等<font color=red>资源</font>，这些资源是需要从网络或者缓存中获取的，主线程在构建 DOM 过程中如果遇到了这些资源，逐一发起<font color=red>请求</font>去获取，而为了提升效率，浏览器也会运行<font color=red>预加载扫描</font>（preload scanner）程序。“预加载扫描器”是<font color=red>并发</font>运行的，如果如果 HTML 中存在 img、link 等标签，预加载扫描程序会把这些请求传递给<font color=red>浏览器进程的网络线程</font>进行资源下载。
- 找到一个<font color=red>`< script >`标签</font>时，它会<font color=red>暂停</font> HTML 文档的解析，并且必须加载、解析和执行 JavaScript 代码。因为<font color=red> JS 可以使用`document.write()`改变整个 DOM 结构之类的东西来改变文档的形状</font>（ HTML 规范中的解析模型概述有一个很好的图表）。这就是 HTML 解析器必须等待 JavaScript 运行才能继续解析 HTML 文档的原因。[V8 关于 JS 执行中的事情](https://mathiasbynens.be/notes/shapes-ics)
- 因此如果 js 没有`document.write()`，可以添加 async 或 defer 属性到`< script >`标签。然后浏览器异步加载和运行 JavaScript 代码，这样不会阻止解析 DOM

> <font color=red>主线程</font>依据 Css 选择器以及浏览器默认样式来计算每个元素应该具备的具体样式

> <font color=red>主线程</font>会遍历 DOM 及相关元素的计算样式，构建出包含每个元素的页面坐标信息及盒子模型大小的布局树（Render Tree），遍历过程中，会<font color=red>跳过</font>隐藏的元素（display: none），另外，<font color=red>伪元素</font>虽然在 DOM 树上不可见，但是在<font color=red>布局树</font>上是可见的。

> 遍历布局树（layout tree），生成一系列的绘画记录（paint records）。绘画记录可以看做是记录各元素绘制先后顺序的笔记-绘画顺序表。

> <font color=red>主线程</font>需要遍历渲染树来创建一棵<font color=red>层次树</font>（Layer Tree），对于添加了<font color=red> will-change</font> CSS 属性的元素，会被看做<font color=red>单独</font>的一层，没有 will-change CSS 属性的元素，浏览器会根据情况决定是否要把该元素放在单独的层。

- 当页面的层超过一定的数量后，层的合成操作要比在每个帧中光栅化页面的一小部分还要慢，因此衡量你应用的渲染性能是十分重要的一件事情。

> 主线程会把这些信息通知给<font color=red>合成器</font>线程，合成器线程开始对层次数的每一层进行光栅化。 <br/>为了优化显示体验，合成线程可以给不同的光栅线程赋予不同的优先级，将那些在视口中的或者视口附近的层先被光栅化。

- 有的层的可以达到整个页面的大小，合成器需要将它们切分为一块又一块的小图块（tiles）

- 合成器将这些小图块分别进行发送给一系列<font color=red>光栅</font>线程（raster threads）进行光栅化

- 结束后光栅线程会将每个图块的光栅结果存在 GPU Process 的内存中

> 合成线程会收集图块上面叫做绘画四边形（draw quads）的信息来构建一个合成帧（compositor frame）。

- 绘画四边形：包含图块在内存的位置以及图层合成后图块在页面的位置之类的信息。

- 合成帧：代表页面一个帧的内容的绘制四边形集合。

> <font color=red>合成线程</font>就会通过 IPC 向<font color=red>浏览器进程</font>（browser process）提交（commit）一个<font color=red>渲染帧</font>。

- 这个时候可能有另外一个合成帧被浏览器进程的 UI 线程（UI thread）提交以改变浏览器的 UI。这些合成帧都会被发送给 GPU 从而展示在屏幕上。

- 如果合成线程收到页面滚动的事件，合成线程会构建另外一个组合帧发送给 GPU 来更新页面。

## 事件处理

> 当点击或者输入的时候，首先接受到事件的是<font color=red>浏览器进程</font>

> 浏览器进程不处理，将事件丢给<font color=red>渲染进程</font>

> 渲染进程依据事件发生的<font color=red>坐标</font>，找到<font color=red>目标对象</font>，运行附加的事件侦听器来适当地处理事件。

> <font color=red>合成器线程</font>会标记页面中绑定有事件处理器的区域为<font color=red>非快速滚动区域</font>(non-fast scrollable region)

- 当合成器线程向主线程发送输入事件时，首先要运行的是命中测试以找到事件目标。命中测试使用渲染过程中生成的绘制记录数据来找出发生事件的点坐标下方的内容。

- 如果事件发生在这些存在标注的区域，合成器线程会把事件信息发送给主线程，等待主线程进行事件处理

- 如果事件不是发生在这些区域，合成器线程则会直接合成新的帧而不用等到主线程的响应。

- 所以，在进行事件监听的时候，尤其是事件捕获或者对整个文档进行事件监听的时候需要考虑一下，因为整个页面都被标记为非快速滚动区域。这意味着即使不关心来自页面某些部分的输入，合成器线程也必须与主线程通信并在每次输入事件进入时等待它。因此，合成器的平滑滚动能力被打败了。

```js
// 浏览器在主线程中侦听事件，但合成器也可以继续合成新帧。
document.body.addEventListener(enent, func, { passive: true });
```


## 参考链接

[合成器线程详解&#x2705;](https://blog.csdn.net/qq_41499782/article/details/120039980)

[一文搞懂浏览器的工作原理&#x270B;](https://blog.csdn.net/qq_35546787/article/details/107788179?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.topblog&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

 [chrome 渲染器进程的内部工作原理&#x270B;](https://blog.csdn.net/qq_41499782/article/details/120035602)