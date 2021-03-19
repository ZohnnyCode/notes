---
title: 40、Vue
date: 2021-03-16 11:42:57
tags:
---

```js
安装脚手架：npm i vue-cli -g
查看版本：vue -V
初始化项目：vue init webpack project-name

.editorconfig
该文件定义项目的编码规范，编译器的行为会与.editorconfig文件中定义的一致，并且其优先级比编译器自身的设置要高

```

### 自定义组件

```js
写组件肯定得先有个模板
// html
<template id="tep">
    <div>
        <p></p>
        <p></p>
        <p></p>
    </div>
</template>
// js
let dashuaibi = {
    template:'#tep'
}
let app = new Vue({
    data(){
        return {
            dashuaibiData:{
                blogUrl:'Dachuizi.github.io',
                netName:'大帅比',
                skill:'大前端'
            }
        }
    },
    components:{
        "dashuaibi"(组件名称):dashuaibi(模板名称)
    }
})

// 页面thml使用自定义组件并传值：用slot属性
<dashuaibi>
    <span slot="blogUrl">{{dashuaibiData.blogUrl}}</span>
    <span slot="netName">{{dashuaibiData.netName}}</span>
    <span slot="skill">{{dashuaibiData.skill}}</span>
</dashuaibi>

// 模板接收值:用slot标签，并写上name属性
<template id="tep">
    <div>
        <p>博客地址：<slot name="blogUrl"></slot></p>
        <p>网名：<slot name="netName"></slot></p>
        <p>技能类型：<slot name="skill"></slot></p>
    </div>
</template>

vue2.0、template中必须要有一个大容器div包裹其他html标签
```

# Vue3

```js
01、
vue create project-name
vue、babel、router、vuex、css（Vue3、history、sass（dart）、package.json）

02、项目目录下新建vue.config.js文件
03、换网站图标
04、assets目录下新建css、images文件夹
05、components-->common-->content

每个页面的组件在每个页面下创建
```

### 项目初始化

```js
目录结构
目录别名
    wepack：默认@符代表src
    注意起别名后：如果不是变量，别名前面要加"~"号，不管是html还是css*************************************
    <img src="~assets/images/2.png"/> // 加了~号才认为是别名
    background:url('~assets/images/3.png')
公共样式
    下载normalize.css
    base.css
    @import "./normalize.css"
        :root {
            --color-text:#666;
            --color-high-text:#42bBaa;
            --color-tint:#42b983;
            --color-background:#FFFFFF;
            --font-size:14px;
            --line-height:1.5
        }
        *,*::before,*::after {
            margin:0;
            paddig:0;
            box-sizing:border-box;
        }
        body {
            user-select:none;
            background:var(--color-background);
            color:var(--color-text);
            width:100vw; // 整个设备的宽度
            // height:100vh;
        }
        a {
            color:var(--color-text);
            text-decoration:none
        }
        .left {
            float:left
        }
        .right {
            float:right
        }
        这是全局的css，让所有组件都使用:在App.vue中引入就行，App的style不加scoped，其他组件就可以使用
        加了就成局部的了
请求封装
    import axios from "axios"
    export function request(config){
        const instance = axios.create({
            baseURL:'',
            timeout:5000
        })

        // 请求拦截
        instance.interceptors.request.use(config=>{

            return config
        },err=>{

        })

        // 响应拦截
        instance.interceptors.response.use(res=>{
            return res.data?res.data:res
        },err=>{

        })

        return instance(config)
    }

    import {request} from "./request"
    export function getHomeAllData(params(如果有)){
        return request({
            url:'/api/index',
            // method:'get',
            // params:{

            // }
        })
    }
```

### 骚操作

```js
// html
<img :src="imgSrc"/>
// js
data(){
    return {
        imgSrc:require('../assets/images/1.png')
    }
}
```

### 配置别名

```js
// 配置后直接使用别名
module.exports = {
  configureWebpack: {
    resolve: {
      alias: {
        assets: "@/assets", // @代表src
      },
    },
  },
  publicPath: "./",
};
```

### Vue3 语法

```js
import { onMounted, ref } from "vue";
// 把数据放到页面上：ref(响应式的)
export default {
  name: "Home",
  setup() {
    const banner = ref([]); // 初始值
    onMounted(() => {
      // 组件挂载完毕
      // 调用异步方法
      ().then(res=>{
          banner.value = res.list
      })
    });
  },
};
```

### 配置路由

```js
// 懒加载导入路由
const Home = () => import("../views/home/Home");
const routers = [
  {
    path: "/", // 默认路由
    name: "DefaultHome",
    component: Home,
  },
  {
    path: "/home",
    name: "Home",
    component: Home,
  },
];

<template>
    <router-view></router-view>
    <div id="nav">
        <router-link to="/">首页</router-link>
        <router-link to="/category">分类</router-link>
        <router-link to="/shopcart">购物车</router-link>
        <router-link to="/profile">我的</router-link>
    </div>
</template>
```
### 引入字体图标
```js
去iconfont下载代码引入到css文件夹中，然后在App.vue中引入iconfont.css文件
<router-link class="tab-bar-item" to="/">
    <div class="icon"><i class="iconfont icon-shouye"></i></div>
    <div>首页</div>
</router-link>

#nav {
    background-color:#f6f6f6
    display:flex;
    // 定在最下面
    position:fixed;
    left:0;
    right:0;
    bottom:0;
    box-shadow:0(上下) -3px(左右) 1px(阴影) rgba(100,100,100,0.1)

    a {
        color:var(--color-text);
    }
    &.router-link-exact-active {
        color:var(--color-high-text);
    }

    .tab-bar-item {
        flex:1;
        text-align:center;
        height:50px; // 移动端导航默认高度
        font-size:var(--font-size);
        .icon {
            width:24px;
            height:24px;
            margin-top:5px;
            vertical-align:middle;
            display:inline-block;
        }
    }
}
```
### Vue3使用插槽
```js
首先定义插槽
template

使用的地方引入：import NavBar from "./"
注册组件：component:{
    NavBar
}

使用
<nav-bar>
    <template v-slot:left></template>
    <template v-slot:default></template> // 
    <template></template>
</nav-bar>
```