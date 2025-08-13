---
title: 【pwa】mainifest
categories: PWA
---

# mainifest.json

## 用途

通过 manifest.json 可以实现自定义启动画面、打开 url、设置界面颜色、设置桌面图标等

## 常用字段

```json
{
  "short_name": "pwa1",
  "name": "pwa-测试用例1",
  "icons": [
    {
      "src": "qr-code-fill-144.png",
      "sizes": "144x144",
      "type": "image/png"
    }
  ],
  "start_url": "/test1",
  "display": "standalone",
  "theme_color": "blue",
  "background_color": "black"
}
```

- `name: string`

描述应用的名称，会显示在桌面图标的标题位置和启动画面中

- `short_name: string`

  描述应用的短名称。当应用名字过长，在桌面图标无法全部显示时，会显示 shortname

- `scope：string`

  设置 manifest 对于网站的作用范围。

- `start_url: string`

  描述用户从设备主屏幕点击图标进入时的第一个地址，start_url 必须在 scope 的作用范围内

  - 如果为空，则以 manifest.json 作为 url

  - 如果 url 打开失败，则和正常显示的网页打开错误的样式一样

  - 如果设置的 url 和当前的项目不在一个域下，无法正常显示

  - 如果 starturl 为相对地址，那么根路径基于 manifest 的路径

  - 如果 starturl 为绝对路径，那么根路径为将

- `icon：TIcon`

  设置 webapp 图标集合。

  ```ts
  type TIcon = {
    src: string; //图标地址
    type: string; //图标mime类型，只能为image/png
    sizes: string; //图标大小，用来表示width x height，单位为px，如果图标要适配多个尺寸，则多个尺寸用空格隔开。与真实图片大小要一致
  };
  ```

  适配规则：

  - 将 webapp 添加到桌面时，浏览器会适配最合适尺寸的图标。浏览器会首先去找与显示密度想匹配且尺寸调整为 48dp 屏幕密度的图标。例如在 2 倍像素的设备上使用 96px，3 倍像素的设备上使用 144px 的

  - 如果没有找到合适的图标，那么会查找与设备特性匹配度最高的图标

  - 如果图标路径错误，那么将显示浏览器的默认图标

- `background_color: string`

  启动画面的背景颜色。rgbs、hsl、hsla 等写法浏览器不支持。未设置时，默认白色

- `theme_color: string`

  显示 web app 的主题色，显示在 banner 位置

- `display: 'fullscreen'|'standalone'|'minimal-ui'|'browser'`

  webapp 被启动时显示的类型

- `orientation`: `'landscape-primary'|'landscape-secondary'|'landscape'|'portrait-primary'|'portrait-secondary'|'portrait'|'natural'|'any'`

  webapp 在屏幕上的显示方向

- `dir: 'ltr'|'rtl'|'auto'`

  文字的显示方向

- `related_applications: 'platform'|'id'`

  用于定义对应的原生应用，类似应用安装横幅提示的形式去推广、引流原生应用

- `prefer_related_applications: Boolean`

  设置是否只允许用户安装原生应用

## 生效条件

- 必须是 https 或者 localhost

- 必须注册运行 service worker，且有 fetch 事件监听

- manifest 必须要有 icons,且必须要至少有尺寸为144x144的

- diaplay 设置为 standalone 或者 fullscreen

- 必须有 name 或者 short_name，start_url

- prefer_related_applications 未设置或者为 false

## 引导安装

```js
window.addEventListener("beforeinstallprompt", (e) => {
  console.log("beforeinstallprompt");
  e.preventDefault();
  e.prompt();//显示安装弹窗
});
navigator.serviceWorker
  .register("pwa1.js")
  .then((res) => {
    console.log("service-pwa1注册成功", res);
  })
  .catch((err) => {
    console.log("service-pwa1注册失败", err);
  });
```
