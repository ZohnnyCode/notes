---
title:
  "[object Object]": null
date:
  "[object Object]": null
tags:
  - null
categories:
  - null
---

#### 第一节：create-react-app 脚手架安装、git 仓库建立

```js
起项目，暴露webpack
```

#### 第二节：路由模式（react-router-dom）、node-sass 配置

```js
{
  loader: require.resolve("file-loader"),
  // Exclude `js` files to keep "css" loader working as it injects
  // its runtime that would otherwise be processed through "file" loader.
  // Also exclude `html` and `json` extensions so they get processed
  // by webpacks internal loaders.
  exclude: [/\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/],
  options: {
    name: "static/media/[name].[hash:8].[ext]",
  },
},
下面增加：
{
  test: /\.scss$/,
  loaders: ["style-loader", "css-loader", "sass-loader"],
},
```

#### 第三节：全局配置 Sass 变量、重置浏览器样式、antd 按需加载

```js
项目中所有的scss文件均可使用变量和方法，无需再次单独引入
安装依赖
npm i sass-resources-loader -D

```
