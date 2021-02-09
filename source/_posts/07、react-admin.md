---
title: 手把手react后台管理开发
date: 2021.1.30
tags: react项目
categories: react项目逻辑
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

 {
              test: sassRegex,
              exclude: sassModuleRegex,
              use: getStyleLoaders(
                {
                  importLoaders: 3,
                  sourceMap: isEnvProduction && shouldUseSourceMap,
                },
                'sass-loader'
              ).concat({
                loader: 'sass-resources-loader',
                options: {
                    resources: [
                        // 这里按照你的文件路径填写
                        path.resolve(__dirname, './../src/styles/main.scss')
                    ]
                }
              }),
              sideEffects: true,
            },

引入normal.scss,并在main.scss引入

Antd按需加载
npm install babel-plugin-import --save-dev

plugins: [
  [
    require.resolve("babel-plugin-named-asset-import"),
    {
      loaderMap: {
        svg: {
          ReactComponent:
            "@svgr/webpack?-svgo,+titleProp,+ref![path]",
        },
      },
    },
  ],
  ["import", { libraryName: "antd", style: "css" }], // antd按需加载
  isEnvDevelopment &&
    shouldUseReactRefresh &&
    require.resolve("react-refresh/babel"),
]
```

#### 第四节：登录界面制作，组件化切换登录和注册页面、antd 表单使用(自定义验证)

```js
antd表单自定义验证：
<Form.Item
  name="password"
  rules={
    [
      {require:true,message:''},
      {min:6,message:''},
      {max:20,message:""},
      {pattern:/^[0-9]*$/,message:''}
      {pattern:reg_password,message:''}

      // 自定义
      ({ getFieldValue }) => ({
          validator(role, value){
              if(value !== getFieldValue("password")){
                  return Promise.reject("两次密码不一致")
              }
              return Promise.resolve();
          }
      })

      // 验证码
      { required: true, message: "请输入长度为6位的字符", len: 6 },

    ]
  }
>

工程化：
// 正则
export const reg_password = /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{6,20}$/;
```

#### 第五节:axios 拦截器、项目环境、环境变量

```js
// 跨域配置
npm i http-proxy-middleware

// 2、新建文件
// src/目录新建 setupProxy.js
const { createProxyMiddleware } = require("http-proxy-middleware");

module.exports = function(app) {
    app.use(createProxyMiddleware("/manage", {
        target: "http://admintest.happymmall.com" , //配置你要请求的服务器地址
        changeOrigin: true,
        pathRewrite: {
        "^/devApi": "",
        },
    }))

    * 1、匹配到devApi，开始做代理  http://www.web-jshtml.cn/api/react
    * 2、/devApi/login/ => /login/
    * 3、替换之后的地址：http://www.web-jshtml.cn/api/react/login/

    app.use(proxy("/manage/api", {
        target: "http://admintest.happymmall.com:7000" ,
        changeOrigin: true,
    }))
};

// axios全局配置：utils--->request.js
创建实例》》》请求拦截》》》响应拦截
baseUrl可以是：'/devApi'

// api管理
api--->export function name(data){
  return service.request({
    url:'',
    method:'',
    data, //post请求
    // params:data // get请求
  })
}

// 环境变量：
create-react-app 创建的项目有内置的环境变量 NODE_ENV, 可通过 process.env.NODE_ENV 读取变量。

NODE_ENV：默认的三个值
开发：development - npm start
测试：test - npm run test
生产：production - npm run build

在src目录下新建三个文件：
.env.development
  REACT_APP_API = "/devApi"   开发环境读这个(npm start的时候)
  REACT_APP_BASE_URL = "http://www.web-jshtml.cn/api/react"
.env.production
  REACT_APP_API = ""   生产环境读取这个
.env.test
  REACT_APP_API = ""
三个文件里面的变量命名必须是："REACT_APP_"开头

const service = axios.create({
  baseUrl:process.env.REACT_APP_API,
  timeout:5000
})

区分production（生产与测试）：
项目打包配置环境变量

依赖： npm install -g dotenv-cli
"build:dev": "dotenv -e  .env.development  react-app-rewired build",
"build:pro": "dotenv -e .env.production react-app-rewired build",
"build:test": "dotenv -e .env.test react-app-rewired build"

// 使用环境变量的跨域
app.use(createProxyMiddleware([process.env.REACT_APP_API], {
    target: process.env.REACT_APP_BASE_URL, //配置你要请求的服务器地址
    changeOrigin: true,
    pathRewrite: {
        [`^${process.env.REACT_APP_API}`] : ""
    },
}))
```
