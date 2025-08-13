---
title: 【wasb】环境搭建
time: 2023-09-09 13:41:58
categories: webAssembly
---
# 【webAsb】- Emscripten环境搭建

## 依赖环境

- python

- git

这两项环境是必要的，否则无法进行安装

## 安装

### 下载项目

``` js
git clone https://github.com/juj/emsdk.git
```
下载emscripten项目

### 安装依赖包等

``` js
cd emsdk // 进入项目目录中
emsdk update // 安装各种工具
emsdk install latest //下载各种包，时间比较长
emsdk activate latest //生成 ~/.emscripten 文件，激活配置
```

[![pP6oCWT.png](https://s1.ax1x.com/2023/09/09/pP6oCWT.png)](https://imgse.com/i/pP6oCWT)

### 配置环境变量

可以先执行`emsdk_env`脚本，这个脚本默认会写入环境变量，但是也会有不成功的情况。

在非emsdk目录下执行`emcc --version`，判断是否报错，如果报错，就证明环境并未配置成功；如果未报错，那么环境已经配置完成啦

#### 环境未配置成功

- 执行 `emcmdprompt.bat`命令

  [![pP6xUSA.png](https://s1.ax1x.com/2023/09/09/pP6xUSA.png)](https://imgse.com/i/pP6xUSA)

- 将带有`PATH +=`的路径写入环境变量的Path中

  [![pP6zKhQ.png](https://s1.ax1x.com/2023/09/09/pP6zKhQ.png)](https://imgse.com/i/pP6zKhQ)

- 将下面带有键值对的写入系统变量中

  记住不能有空格，否则会执行不成功

  [![pP6z1cn.png](https://s1.ax1x.com/2023/09/09/pP6z1cn.png)](https://imgse.com/i/pP6z1cn)

- 在非`emsdk`目录下执行`emcc --version`

  [![pP6zJBV.png](https://s1.ax1x.com/2023/09/09/pP6zJBV.png)](https://imgse.com/i/pP6zJBV)

  环境配置成功

## 配置c++环境

### 安装配置c/c++

- 安装

  安装c/c++编译器：https://sourceforge.net/projects/mingw-w64/

- 配置

  在环境变量的Path中，把解压的mingw的bin目录加入进去

### 测试

在cmd输入`gcc -v`，没有报错即可

[![pPIgD91.png](https://z1.ax1x.com/2023/09/21/pPIgD91.png)](https://imgse.com/i/pPIgD91)

[参链](https://blog.csdn.net/weixin_43180456/article/details/126374156)

## 测试

### 编写测试程序

建立一个cpp文件，写入c的代码

``` c++
#include <iostream>

using namespace std;

int main() {
    cout << "Hello, world!" << endl;
    return 0;
}
```

### 生成js代码

> `emcc test.cpp -o test.html`

会生成`html`，`js`和`wasm`文件，html默认引入js文件，js的作用是引入wasm文件

 - `-s` 表明编译到 Wasm，否则编译到 Asm.js（Wasm 的前身）， 最初 emscripten 是用于编译到 Asm.js 的。

 - `SIDE_MODULE` 表明编译为副模块。有副模块就有主模块，简单理解副模块会去除 C 标准库函数，因为副模块会在运行时被链接到一个主模块，而主模块有C标准库函数。SIDE_MODULE的值可选 1 或者 2，前者会自动导出代码里所有的函数，而后者需要手动声明。

 - `-o` xxxx 导出选项，导出的文件可选 .html、.js、.wasm，区别在于前面两者会帮你把胶水代码写好 ，而 .wasm 则需要在 JS 自己编写胶水代码了，但是前面两者代码冗余，比如编译为 JS 文件时，JS 文件会包含两千多行代码，不过这是学习 Wasm 的现成实例。

### 搭建本地服务器

在代码路径中，搭建本地服务。在浏览器访问本地的文件会报错

- `pnpm init`

  创建package.json文件
  
- `pnpm i http-server`

  安装`http-server`

- `http-server -o`

  起本地服务，在浏览器打开html文件
  
### 验收

用`http-server`起了本地服务后，在浏览器访问对应的域名加上生成的html文件，正常运行项目

[![pPIfE5Q.png](https://z1.ax1x.com/2023/09/21/pPIfE5Q.png)](https://imgse.com/i/pPIfE5Q)

## 参链

[Emscripten编译器安装教程，亲测成功编译出第一个WebAssembly](http://www.taodudu.cc/news/show-5750307.html?action=onClick)