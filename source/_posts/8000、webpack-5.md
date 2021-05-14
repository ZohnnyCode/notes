---
title: 8000、webpack(5)
date: 2021-04-27 17:09:20
tags:
---

#### 原生打包命令

```js
全局webpack打包
webpack

局部webpack打包
01、npx webpack
02、./node_modules/.bin/webpack
03、也可以通过package.json文件
scripts:{
    "build":"webpack"
}

npm run build // 局部如果没有，会自动找全局的
```

#### 浏览器的兼容性

```js
> 1%
last 2 versions
not dead

一个个条件--->告诉我们使用的工具（post-css，babel），我的项目到底要适配哪些浏览器
```

## 第二节课、webpack 配置和 css 处理

```js
指定配置文件
scripts:{
    "build":"webpack --config wk.config.js"
}

运行打包会生成一个关系依赖图，遍历图结构，打包一个个模块（根据文件的不同使用不同的loader来解析）

mudule.rules允许我们配置多个loader，rules是一个数组，里面一个个对象
对象有test属性和use属性
```

#### loader 处理样式的三种写法

```js
module:{
    rules:[
        {
            test://
            // loader:'css-loader' // 写法一
            // use:['css-loader']  // 写法二
            use:[
                {
                    loader:'css-loader'
                }
            ]
        }
    ]
}
```

#### browserslist 工具

```js
>1% : css要兼容市场占有率大于1%的浏览器，如果我们是通过工具来达到这种兼容性的，如后面讲到的postcss-prest-env、babel、autoprefixer等
n 如何可以让他们共享我们的配置呢？就是用Browserslist
Browserslist是一个在不同的前端工具之间，共享目标浏览器和Node.js版本的配置

如何配置browserslist
方案一：在package.json中配置
    "browserslist":[
        "last 2 version",
        "not dead",
        "> 0.2%"
    ]
方案二：单独的一个配置文件.browserslistrc文件
    "last 2 version",
    "not dead",
    "> 0.2%"
```

#### 认识 postcss 工具

```js
postcss是通过javascript来转换样式的工具
这个工具可以帮助我们进行一些CSS的转换和适配，比如自动添加浏览器前缀、css样式的重置
但是实现这些功能，我们需要借助于PostCSS对应的插件；

如何使用postcss
第一步：安装postcss-loader
第二步：安装需要的postcss相关的插件（现在一般用postcss-preset-env（依赖autoprefixer），不用autoprefixer）

因为postcss需要有对应的插件才会起效果，所以我们需要配置它的plugin
{
    loader:'postcss-loader',
    options:{
        postcssOptions:{
            plugins:[
                // require('autoprefixer')
                require('postcss-preset-env')
            ]
        }
    }
}

单独的postcss配置文件：postcss.config.js
module.exports = {
    plugins:[
        // require('autoprefixer')
        "postcss-preset-env"
    ]
}

postcss-preset-env也是一个插件，功能更加强大，将一些现代的CSS特性，转成大多数浏览器认识的CSS，并且会根据目标浏览器或者运行时环
境添加所需的polyfill

举个例子：
我们这里在使用十六进制的颜色时设置了8位；
但是某些浏览器可能不认识这种语法，我们最好可以转成RGBA的形式；
但是autoprefixer是不会帮助我们转换的；
而postcss-preset-env就可以完成这样的功能
.content {
    color:'12345678'
}
```

#### 对于一个 css 文件中@import 另一个 css 文件，不会被 postcss-loader 再次处理的问题

```js
一个css文件、@import另一个css文件被loader处理的流程
首先在js文件里面加载css的时候，可以被postcss-loader处理、然后来到css-loader
css-loader可以处理@import语句和里面的css
但是@import里面的css不会再次回退到让postcss-loader执行
所以@import里面的样式不会转化为大部分浏览器都识别的样式，不会加前缀

postcss-loader一般用postcss-preset-env处理浏览器的兼容问题

解决方案：利用css-loader的一个属性importLoaders回退到前面的loader
use:[
    "style-loader",
    {
        loader:"css-loader",
        options:{
            importLoaders:1 //(下面还有多少个loader就可以写几)
        }
    },
    "postcss-loader"
]
```

## 第三节课、webpack 加载处理其他资源

#### 注意 file-loader 处理图片资源引入问题

```js
4.x对于require引入方式进来的图片直接是资源路径
5.x对于require的引入方式把图片资源放在了返回对象的default属性上

直接使用import的方式就不会有这种问题
import xxx from "./xxx.png"

webpack5的话可以不用file-loader了
```

#### url-loader 的默认

```js
默认情况下url - loader会将所有的图片文件转成base64编码

开发中我们往往是小的图片需要转换（直接嵌入到boundle.js文件中，减少了http请求），但是大的图片直接使用图片(弊端是增加了http请求)
{
    test: /\.(png|jpe?g|gif|svg)$/,
    use: [
        {
        loader: 'url-loader',
        options: {
            name: "img/[name].[hash:6].[ext]",
            limit: 100 * 1024, // 单位是byte（大于100kb直接使用图片，小于转化成64格式）
            // outputPath:"/img"
        }
        }
    ]
}
// base64编码可以和页面一起请求下来
// 直接打包成图片的话。就是单独一个http请求
```

#### webpack 可以不需要使用 file/url-loader 处理这些图片资源了，用 type

```js
使用资源模块类型（asset module type）来代替loader
type有四种类型：
type:asset/resource 类似 file-loader
    asset/inline 类似 url-loader
    asset/source 类似 row-loader(在加载txt、json文件的时候使用)
    asset

两种方式改变资源打包路径
方式一：
output: {
    filename: "bundle.js",
    // 必须是一个绝对路径
    path: path.resolve(__dirname, "./build"),
    assetModuleFilename: "img/[name].[hash:6][ext]"
},

方式二：加generator属性
{
    test: /\.(png|jpe?g|gif|svg)$/,
    type: "asset/resource", // file-loader的效果
    generator: { // 注意当type为asset/inline的时候，因为不生成文件。需要把generator注释掉 ***************************************************************************************
        filename: "img/[name].[hash:6][ext]"
    },
}

最终版本：
{
    test: /\.(png|jpe?g|gif|svg)$/,
    // type: "asset/resource", file-loader的效果
    // type: "asset/inline", url-loader
    type: "asset", // asset 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用 url-loader，并且配置资源体积限制实现
    generator: {
        filename: "img/[name].[hash:6][ext]"
    },
    parser: {
        dataUrlCondition: {
            maxSize: 100 * 1024
        }
    }
},
```

#### 处理字体文件

```js
字体文件，音频文件都可以用file-loader对应的type来处理

{
    test: /\.ttf|eot|woff2?$/i,
    type: "asset/resource",
    generator: {
        filename: "font/[name].[hash:6][ext]"
    }
}

使用字体图标：
在font文件夹中引入字体文件和iconfont.css文件
给i标签加类名
```

#### loader 和 plugin 的区别

```js
loader是用来加载特定类型的模块的;
Plugin可以用于执行更加广泛的任务，比如打包优化、资源管理、环境变量注入等
```

#### node 的模块化

```js
module.exports跟exports导出的对象是指向同一个的;

导出一个属性;
module.exports.abc = exports.abc;
```
