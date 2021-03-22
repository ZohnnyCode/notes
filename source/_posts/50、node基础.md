---
title: 50、node基础
date: 2021-03-20 00:28:56
tags:
---

```js
查看目录：dir
```

```js
node.js模块化：
引入一个模块：require("./xxx")
导出模块接口：module.exports = 变量名/函数名

node的模块分类：
核心模块、自定义模块、第三方模块（npm下载的）

核心模块之：
fs
语法：fs.readFile("相对或绝对路径","异步回调函数")
fs.readFile("text.txt",(err,data) => {
  if(err){
    console.log(err)
  }
  console.log(data) // 现在是buffer类型，也就是二进制文件，得把它变成字符串
  console.log(data.toString())
})

path
语法：
let result = path.join(damain,url,id) // 将多个参数合并成一个路径

http
// 几个概念：
请求：浏览器向服务器要数据
响应：服务器给浏览器发送数据
地址：通过域名或ip可以访问到一个网站、域名或者ip就是网站的地址
端口：一台服务器可以对外服务多个网站，他们的端口是不同的，因此访问网站除了ip跟域名，还需要端口
      平时不输入，是因为使用了默认的80端口

创建一个服务器：
const http = require("http")

const server = http.createServer((req,res) => {
  res.end("hello node")
})

server.listen(3000,()=>{
  console.log("server is runing")
})
```

### koa 服务器开发框架

```js
npm install koa --save

const Koa = require("koa") // 为什么大写
因为Koa是一个构造函数，需要用着构造函数创建koa应用

const app = new Koa() // 创建应用

// 处理一些响应内容、引入中间件
中间件就是一个（async）函数，***在请求和响应之间调用***，有一个参数ctx（请求响应的上下文信息，存储了请求的相关信息，也可以设置响应的相关信息）
响应信息：ctx.body = "hello koa"

app.use(async (ctx) => {
  ctx.body = "hello koa!" // 现在输入任何地址都是“hello koa ”
})

// 监听端口
app.listen(3000,()=>{
  console.log("server is runing")
})
```

### 引入路由

```js
npm i koa-router --save

// koa-router是一个函数
const router = require("koa-router")() // 引入并执行

router.get("/",async ctx => {
  ctx.body = "home page"
})

router.get("/video",async ctx => {
  ctx.body = "video page"
})

// 现在router和koa应用是各自独立的模块
app.use(router.routes()) // 在koa应用中引入router
```

### 在服务器创建静态文件目录

```js
获取当前项目的绝对路径：__dirname

npm i koa-static --save

const static = require("koa-static")
app.use(static(__dirname + "/public")) // 在服务器指定静态资源文件夹

public // 静态资源文件夹
  images
  css
  js
这样在浏览器输入地址可直接访问图片（不用写public）：127.0.0.1:3000/images/logo.jpg

如何设置静态文件目录并引入图片
ctx.body = `<img src="/images/logo.jpg" />`

static是一个函数，调用它，可以将参数的目录设置为一个静态目录
```

```js
每一个koa的第三方模块都要通过use方法引入到koa应用里面才能被使用
```

### 使用模板引擎返回数据给客户端

```js
npm i koa-views --save
npm i nunjucks --save

const views = require("koa-views")  // views第一个参数模板引擎（就是html文件）的位置，第二个参数使用拿个模板引擎
app.use(views(__dirname + "/views",{
  map:{
    html:"nunjucks"
  }
}))

// 访问页面
app.use(async ctx => {
  // 用render，不用body，是异步函数，得用await
  // 第一个参数模板的名字，就是views目录下的文件名，不带后缀.html
  // 第二个参数，是给模板的动态数据对象
  await ctx.render("index",{
    title:"hello nunjucks"
  })
})

// 模板中使用动态数据：用双花括号，直接写属性
<h1>{{ title }}</h1>

// 使用路由
const router = require("koa-router")()

router.get('/',async ctx => {
  await ctx.render("index",{title:"首页"})
})
router.get('/video',async ctx => {
  await ctx.render("index",{title:"视频"})
})
app.use(router.routes())

// 使用模板主要是响应的时候用render方法（以上是路由跟模板一起使用）
```

### 做登录

```js
// action是把数据提交到服务器的哪里
// name属性是传递给后台的字段
<form action="/login">
  <input type="text" name="username" />
  <input type="password" name="password" />
  <input type="submit" value="登录" />
</form>

// 后台接一下
```
