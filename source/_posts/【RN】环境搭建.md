---
title: 【RN】环境搭建
categories: "React Native"
---

# 【React Native】环境搭建

## 环境

- Nodejs: `v16.16.0`

- react-native: `0.72.3`

- react-native-cli: `2.0.13`

- webStorm: `2023.1.2`

- Android Studio: `2022.2.1.20`

## 基本环境配置

### 安装nodejs

[Node.Js中文网](https://nodejs.p2hp.com/)

### 安装react-native及其脚手架

> npm i -g react-native react-native-cli

### 安装Android Studio

这个是配置安卓环境的，需要配置好后，才能运行成功。

[官网下载](https://developer.android.google.cn/studio/)

[安装流程](https://reactnative.cn/docs/environment-setup)

- 必须安装的SDK：

    - `Android SDK`
    - `Android SDK Platform`
    - `Android Virtual Device`

安装过程由于下载内容较多，因此安装过程较慢。

- 安装后，需要额外安装的SDK

    - `Android SDK Platform 33`
    
    - `Intel x86 Atom_64 System Image`
    
    - `Android SDK Build-Tools 33.0.0`

    从设置按钮-》settings-》Appearance & Behavior -》 System Settings -》 Android SDK
    [![pCbGec8.png](https://s1.ax1x.com/2023/07/21/pCbGec8.png)](https://imgse.com/i/pCbGec8)

    [![pCb8Xfx.png](https://s1.ax1x.com/2023/07/21/pCb8Xfx.png)](https://imgse.com/i/pCb8Xfx)

    [![pCbGJ3V.png](https://s1.ax1x.com/2023/07/21/pCbGJ3V.png)](https://imgse.com/i/pCbGJ3V)

- 配置路径

    在环境变量中设置以下路径：

    > `ANDROID_HOME`: `上个步骤中tab中的Android SDK Location`

    在`Path`的环境变量中，添加如下文案：

    - `%ANDROID_HOME%\platform-tools`
    - `%ANDROID_HOME%\emulator`
    - `%ANDROID_HOME%\tools`
    - `%ANDROID_HOME%\tools\bin`

    设置完成后，点击应用即可

### 安装webStorm

[安装教程](https://www.bilibili.com/read/cv24375178/)

注：如果破解的code不生效，可以重启电脑然后再填入对应的code

## 新建项目

- 在webStorm新建一个React Native项目

    [![pCbJkb4.png](https://s1.ax1x.com/2023/07/21/pCbJkb4.png)](https://imgse.com/i/pCbJkb4)

- 创建项目的文件目录

    [![pCbJ1qe.png](https://s1.ax1x.com/2023/07/21/pCbJ1qe.png)](https://imgse.com/i/pCbJ1qe)

- 右上角的命令的edit，进入命令的编辑页面

    [![pCbJ4LF.png](https://s1.ax1x.com/2023/07/21/pCbJ4LF.png)](https://imgse.com/i/pCbJ4LF)

    [![pCbJodJ.png](https://s1.ax1x.com/2023/07/21/pCbJodJ.png)](https://imgse.com/i/pCbJodJ)

- 在Before launch的标签栏，点击+，选择`Run External Tool`，选择+

    [![pCbwt0K.png](https://s1.ax1x.com/2023/07/21/pCbwt0K.png)](https://imgse.com/i/pCbwt0K)

    [![pCbwwfH.png](https://s1.ax1x.com/2023/07/21/pCbwwfH.png)](https://imgse.com/i/pCbwwfH)

- 填写相关信息

    Name为该按钮的名字

    Program为react Native的路径，win终端命令:where react-native 

    working directory：该输入框中,先点击右边的insert macro,选择ProjectFileDir.

    [![pCb0mjI.png](https://s1.ax1x.com/2023/07/21/pCb0mjI.png)](https://imgse.com/i/pCb0mjI)

    填写内容：

    [![pCb0KDP.png](https://s1.ax1x.com/2023/07/21/pCb0KDP.png)](https://imgse.com/i/pCb0KDP)

- 保存以上操作，然后执行刚刚配置好的命令

    [![pCb0D5F.png](https://s1.ax1x.com/2023/07/21/pCb0D5F.png)](https://imgse.com/i/pCb0D5F)

## 参链

- [搭建开发环境](https://reactnative.cn/docs/environment-setup)

- [webstorm破解激活2023最新永久教程「亲测有效」](https://www.bilibili.com/read/cv24375178/)

- [WebStorm里配置运行React Native](https://blog.csdn.net/sinat_36279113/article/details/100576426)
