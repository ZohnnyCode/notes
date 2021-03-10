---
title: 16、react-router
date: 2021-02-20 09:39:40
tags:
---

### 01、对 props 进行限制(props 是只读的)

```js
每写一个组件标签，react都会new一个组件实例

import PropTypes from 'prop-types'

// 必传
组件标签.propTypes = {
    标签属性:PropTypes.string.isRequired,
    speak:PropTypes.func
}

// 默认值
Person.defaultProps = {
    sex:'不男不女'
}

这就是对标签属性进行类型，必要性，默认值的限制

限制简写方式：直接写在类里面
static propTypes = {}
static defaultProps = {}

构造器是否接收props，是否传递给super，取决于：是否希望在构造器中通过this访问props（构造器开发时尽量不写）
```

### 前端路由原理

```js
前端路由的基石：BOM身上的history对象（它在操作浏览器的历史记录（是个栈结构，push推，replace替换））
history有两种工作模式
BrowserHistory:直接使用H5推出的history身上的API
HashHistory：hash值（类似锚点、会形成历史记录、兼容性极佳） // <a href="#demo1">跳转到demo1</a>

点击导航区域，路径发生变化，浏览器（被前端路由器所）监测到，匹配并展示对应组件（路由就是一种映射关系，每一个 路径对应一个组件（key(path):value(组件component或者Function)））

后端路由：（key(path):value(Function)）
工作过程：当node接收到一个请求时，根据请求路径（path）找到匹配的路由，调用路由函数处理请求，返回响应数据

前端路由：浏览器的path匹配
```

```js
对路由的理解：
一个路由器管理多个路由
react-router-dom（印记中文）：react前端路由的插件库、专门用来实现单页面（单页面、多组件）应用（注意区分导航区（路由链接）和展示区）

原生html中，靠a标签跳转不同的页面
在react中靠路由链接（Link）实现切换组件
靠Route（注册路由）匹配路径对应的组件
在最外层包裹路由器对象
```
