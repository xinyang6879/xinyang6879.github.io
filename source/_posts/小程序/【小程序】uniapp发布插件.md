---
title: 【小程序】uniapp发布插件
time: 2023-07-01 14:14:05
categories: 小程序
---
# 【小程序】uniapp发布插件

[uni-modules-插件](https://uniapp.dcloud.net.cn/plugin/uni_modules.html#%E4%BD%BF%E7%94%A8-uni-modules-%E6%8F%92%E4%BB%B6)

## 新建发布uni_modules插件

- 在`uni_modules`右键，选择新建一个`uni_modules插件`

  此时会生成一个插件模板代码

- 写好对应的文件后，在`插件目录右键`，选择`发布到插件市场`

  [![pCBR6uq.png](https://s1.ax1x.com/2023/07/01/pCBR6uq.png)](https://imgse.com/i/pCBR6uq)

  填写好对应的信息后，直接进行保存即可

- 发布好后，即可在uniapp插件市场中进行下载

## 更新插件

- 更新好具体的内容后，右键选择`发布到插件市场`

  [![pCBWUMR.png](https://s1.ax1x.com/2023/07/01/pCBWUMR.png)](https://imgse.com/i/pCBWUMR)

更新即可

## 注意点

- 发布和导入是同一个项目

  项目里面如果又导入了这个插件，那么在uni_modules下导入的插件会被覆盖

- 发布的包必须包含的文件

  - `components/插件名/插件名.vue`
    
    如果没有这个文件，那么在发布时会报错

  - `package.json`
    
    用于配置发布插件的一些信息