---
title: 39、koa
date: 2021-03-22 11:50:58
tags:
---

### 为什么使用 nunjucks 模板开发

```js
还是使用html开发，但可以使用后台动态数据
```

### 使用模板引擎

```js
npm i koa-views nunjucks
koa-views负责指定使用那个模板引擎

const views = require('koa-views')
// views两个参数：第一个指定模板存放目录，第二个，使用那个模板引擎
app.use(views(__dirname + "/views",{
    map:{
        html:'unnjucks'
    }
}))

app.use(async ctx => {
    await ctx.render("index") // 异步方法，得加await
})

动态传数据给模板，使用render的第二个参数以对象的方式传递：{title:"第一个模板"}
模板中使用：{{ title }}
```

### 模拟登录

```js
<form action="/login" method="post">
    <input type="submit" value="登录">
</form>
```

### 写个接口

```js
// 获取get请求的参数：ctx.query.xxx
// 获取post请求的参数
npm i koa-parse

const parser = require('koa-parser')
app.use(parser())

现在就可以解析post请求的参数了

ctx.request.body.xxx
```

### nunjucks 模板语法

```js
// 循环语句
{% for student in studentList %}
    <li>{{ student }}</li>
{% endfor %}

// 条件语句
{% if isLogin %}
    <p>欢迎：{{ username }}</p>
{% else %}
    <p>请登录</p>
{% endif %}

// 模板继承
// 定义模板
<div>
    <a href="/">首页</a>
    <a href="/images">图片</a>
</div>
{% block content %} {% endblock %}
// 使用模板、
{% extends "./views/layout.html" %}
{% block content %}
    <h1>图片页面</h1>
{% endblock %}

// 某些独有内容(公共问价单独用一个文件来写，哪个地方用，include引进)
// footer.html
<div>
    powerBy 大锤子
</div>
// 首页index.html引入
{% include "./views/footer.html" %}
```

### cookie 和 session 登录和保持登录状态效果

```js
设置cookie
cookie是服务器写在客户端的凭证:ctx.cookies.set("user","admin")

第一次请求首页，服务器写给客户端，第二次客户端会自动带上cookie来请求
以后访问其他页面，也会携带cookie，这样就能记住用户信息了

获取cookie
ctx.cookies.get("user")

小练习：记住客户的访问次数
router.get("/",async ctx => {
    let count = ctx.cookies.get("count")
    if(count>0){
        ++count;
        ctx.cookies.set("count",count)
    }else{
        count = 1;
        ctx.cookies.set("count",count)
    }
    ctx.body = count
})

// 设置cookie过期时间
ctx.cookies.set("count",count,{
    maxAge:2000 // 两秒过期
})
```

### session

```js
npm i koa-session

const session = require("koa-session")

session.keys = ["123456"] // 加密秘钥
app.use(session({
    maxAge:3000
},app))

// 设置session
router.get("/set_session",async ctx => {
    ctx.session.user = "admin";
    ctx.body = "set Session"
})

router.get("/get_session",async ctx => {
    let user = ctx.session.user;
    ctx.body = user
})

// 小例子
router.get("/",async ctx => {
    if(ctx.session.count > 0){
        ++ctx.session.count
    }else{
        ctx.session.count = 1
    }
    ctx.body = ctx.session.count
})
```

### 小练习

```js
// session用法
const session = require("koa-session");
const app = new Koa();
app.keys = ["123456"];
app.use(
  session(
    {
      maxAge: 30 * 1000,
    },
    app
  )
);

// parser用法
const parser = require("koa-parser");
app.use(parser())

// 模板引擎用法
const views = require("koa-views");
const nunjucks = require("nunjucks");
app.use(
  views(__dirname + "/views", {
    map: {
      html: "nunjucks",
    },
  })
);

// 路由用法
const router = require("koa-router")();
app.use(router.routes)


实现登录之后访问特定的权限，记住用户名，记住用户的登录状态
// 谁都可以访问
router.get("/", async (ctx) => {
  await ctx.render("home.html");
});
// 登录用户可访问
router.get("/list", async (ctx) => {
  if (ctx.session.user) {
    await ctx.render("list.html");
  } else {
    ctx.redirect("/");
  }
});
router.get("/login", async (ctx) => {
  await ctx.render("login.html");
});

router.post("/login", async (ctx) => {
  let username = ctx.request.body.username;
  let password = ctx.request.body.password;
  if (username === "admin" && password === "123456") {
    ctx.session.user = "admin";
    ctx.redirect("/list");
  } else {
    ctx.redirect("/");
  }
});

// 注销
router.get("/logout", async (ctx) => {
  ctx.session.user = "";
  ctx.redirect("/");
});
```
