---
title: 06、webpack
date: 2021-02-07 13:14:03
tags:
---

> 版本：webpack：……4.44.2 webpack-cli：……3.3.12

```js
mode: "none/development/production"; // webpack自带一些插件，模式分三种
        nono:默认不开启插件
        development：开启一些插件，关闭一些插件
        production：又开启一些插件（js压缩），关闭另一些插件

loader:模块转换器 模块处理器
        webpack默认只支持js和json文件
        最重要的是要知道：什么时候使用什么loader
    module:{
        rules:[
            {},
            {}
        ]
    }

plugins:webpack功能扩展
        生成html文件的插件：html-webpack-plugin -D
        清楚文件的插件：clean-webpack-plugin -D

bundle:输出的资源文件 就叫bundle文件（dist目录里面的文件）

module：模块

chunk：代码片段（被webpack处理过后的js模块（处理import语句和reruire语句），eval函数里面的字符串内容）

```

```js
bundle、chunks、chunk、module的关系
一个bundle对应一个chunks
一个chunks对应一个或多个chunk（模块）
一个chunk就是一个module

多入口的每个文件对应的key叫：chunk name

注意一个入口可以拆分成多个bundle（代码分割）

动态引入：import './index.css'
静态引入：import css from "./index.css"
```

### webpack 工程化实战

```js
- PC 端还是移动端
    移动端 spa（ssr）
    pc 端 mpa
    兼容性：需要兼容的浏览器和版本

- 多人？单人？

- 技术栈
  1.vue
  2.react
  3.样式
    less
    sass
    postcss(功能类似babel，babel作用于js领域，而postcss作用于css领域)
  4.TS babel->ES6+
  5.模板引擎
    ejs
    pug
  6.是否支持第三方字体

- 工具类
    1.安装依赖包，切换国内源（npm config）
    2..npmrc

- Eslint + Prettier

- 提交规范
```

### 实战开始

```js
01、指定项目所使用的源
    1、项目根目录下面新建".npmrc"文件
        # 指定本项目使用国内淘宝源安装依赖
        registry=https://registry.npm.taobao.org

02、样式处理（支持less）
    postcss-loader是webpack和postcss沟通的工具，打电话的工具
    less-loader是webpack和less打电话的工具

    postcss中autoprefixer（自动加前缀）的使用：
        1、安装依赖：postcss-loader，postcss，autoprefixer  ****** -D ******
        2、项目根目录下新建配置文件：postcss.config.js
            module.exports = {
                plugins: [
                    require("autoprefixer")({
                    // 兼容浏览器最近的两个版本
                    // 兼容市场上占有率大于1%的浏览器（世界的）
                    //   overrideBrowserslist：package.json里面的Browserslist的配置优先级比较高，使用此字段覆盖package.json文件，使用自己的配置
                    overrideBrowserslist: ["last 2 versions", ">1%"],
                    }),
                ],
            };
        3、把postcss-loader引入到webpack（postcss-loader一定要放到css-loader后面）
            {
                test: /\.less$/,
                use: ["style-loader", "css-loader", "postcss-loader", "less-loader"],
            },

03、打包单独的样式文件（mini-css-extract-plugin -D）
    01、new MiniCssExtrePlugin({
            filename: "main.css",
        })
    02、{
            test: /\.less$/,
            use: [
            MiniCssExtrePlugin.loader, // 把css抽离出来，不使用style-loader动态生成style标签插入样式
            "css-loader",
            "postcss-loader",
            "less-loader",
            ],
        }
    03、引入html-webpack-plugin自动打包生成html文件
            new HtmlWebpackPlugin({
                template:"./src/index.html",
                filename: "main.html",
            }),
        引入clean-webpack-plugin打包自动删除dist目录再生成(解构引入)
            const { CleanWebpackPlugin } = require("clean-webpack-plugin");
            new CleanWebpackPlugin(),

```
