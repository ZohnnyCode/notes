---
title: 40、Vue
date: 2021-03-16 11:42:57
tags:
---

```js
安装脚手架2：npm i vue-cli -g
安装Vue脚手架3：npm i -g @vue/cli

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
}
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
const Home = () => import("../views/home/Home")
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
]

// App.vue组件
;<template>
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
@imoport "assets/css/iconfont.css"

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

### Vue3 使用插槽

```js
// 自定义顶部导航条
// 可以在多个项目中使用：components/common/navbar/NavBar.vue
// 首先定义插槽
// template
<template>
    <div class="nav-bar">
        <div class="left">
            <slot name="left">
                <img src="~assets/images.left.png"/>
            </slot> // 具名插槽
        </div>
        <div class="center">
            <slot>默认值</slot>
        </div>
        <div class="footer">
            <slot name="footer"></slot>
        </div>
    </div>
</template>
.nav-bar {
    background-color:var(--color-tint);
    color:#FFF;

    position:fixed;
    left:0;
    right:0;
    top:0;
    z-index:9

    height:45px;
    line-height:45px;
    text-align:center;
    box-shadow:0 2px 0px rgba(100,100,100,0.1)

    display:flex; // 做居中

    .left,.right {
        width:60px;
    }

    // 修改箭头图片的样式
    .left img {
        width:45px;
        padding:12px; // 缩放缩小一点
    }

    .center {
        // background-color:pink // 测试颜色
        flex:1
    }
}

// 在首页中使用
使用的地方引入：import NavBar from "./"
注册组件：component:{
    NavBar
}

使用
<template>
    <div>
        <nav-bar>
            <template v-slot:left> @lt; </template>
            <template v-slot:default>SHOP</template> // vue3得使用模板
            <template></template>
        </nav-bar>
    </div>
</template>
插槽：传了值，使用该值，不传，就是没写，使用默认值
```

```js
<div class="left" @click="goback">
  <slot name="left">
    <img src="~assets/images.left.png" />
  </slot>
  // 具名插槽
</div>
// 点击后退，使用组合式api
import {useRouter} from "vue-router"
export default {
    name:"NavBar",
    setup(){
        const router = useRouter() // 创建路由器对象
        const goback = () => {
            router.go(-1) // 回到上一次路由
        }
        return {
            goback // 组合式api必须把方法返回出去
        }
    }
}

// 现在去每个页面使用定义好的插槽，有默认值的可以不传
```

```js
// 改变每个页面的标题：使用导航守卫
// 去router.js文件
{
    path:"/home",
    name:"Home",
    component:Home,
    meta:{
        title:"图书兄弟-首页"
    }
}
router.beforeEach((to,from,next)=>{ // 这是全局导航守卫，也可以到每个页面里面做局部导航守卫
    // 判断登录与否，在这里条login
    next();
    document.title = to.meta.title
})
```

### 做首页 banner 图

```js
<template>
    <div>
        <nav-bar>
            <template v-slot:default>图书兄弟</template>
        </nav-bar>

        <div class="banner">
            <img src="~assets/images/1.png"/>
        </div>
    </div>
</template>

.banner img {
    width:100%;
    height:auto;
    mergin-top:45px; // 顶部导航条的高度 //使用bscroll的时候要去掉
}
```

### 推荐

```js
// 只是首页用到：home/childComponent/RecommendView.vue
<template>
  <div class="recommend">
    <div class="recommend-item">
      <a>
        <img src="~assets/images/1.png" />
        <div>细说php</div>
      </a>
    </div>
  </div>
</template>

.recommend {
    display:flex;
    width:100%;

    text-align:center;
    padding:15px 0 30px;
    border-bottom:8px solid #EEE

    font-size:12px

    .recommend-item {
        flex:1
        img {
            width:70px
            height:70px
            margin-bottom:10px
        }
    }
}

这里用到的数据从首页通过属性传递过来
```

```js
import {getHomeAllData} from "network/home"
import {ref,reactive,onMounted} from "vue"

// <recommend-view :recommends="recommemds"></recommend-view>

setup(){
    onMounted(()=>{
        const recommends = ref([])
        getHomeAllData().then(res=>{
            recommends.value = res.goods.data
        })

        return {
            recommends
        }
    })
}

// 接收
props:{
    recommemds:{
        type:Array,
        default(){
            return []
        }
    }
}

<div class="recommend-item" v-for="item in recommemds" :key="item.id">
    <a @click.prevent="goD(item.id)">
        <img :src="item.cover_url" alt=""/>
        <div>{{item.title}}</div>
    </a>
</div>

import {useRouter} from "vue-router"
setup(){
    const router = useRouter()
    const goD = () => {
        router.push({psth:"/detail",query:{id}})
    }
    return {
        goD
    }
}
```

```js
详情页组件接收传递的id
import {useRoute} from "vue-router"
import { ref } from "vue"
setup(){
    const route = useRoute()
    let id = ref(0)

    id.value = route.query.id
    // 动态路由：route.params.id
    return {
        id
    }
}
```

### 首页导航设置

```js
// components/content/tabControl/TabControl.vue
<template>
    <div class="tab-control">
        <div class="tab-control-item active">
            <span>畅销</span>
        </div>
    </div>

    // 遍历
    <div class="tab-control">
        <div v-for="(item,index) in title"
            :key="index"
            class="tab-control-item"
            @ click="itemClick(index)"
            : class={active:index === currentIndex}
            >
            <span>{{ item  }}</span>
        </div>
    </div>
</template>

props:{
    title:{
        type:Array,
        default(){
            return []
        }
    }
}
// 点击切换tab
import {ref} from "vue"
setup(){
    let currentIndex = ref(0)
    const itemClick = (index) => {
        currentIndex.value = index

        // 通知父组件
        emit("tabClick",index)
    }
    return {
        currentIndex,
        itemClick
    }
}

// 现在点击选项卡，下面的内容要跟着改变（知道点击的是哪个索引，让数据变成响应式的（计算属性？））
// setup()有两个参数：第一个props，第二个上下文context对象，可以结构emit实现父子级通信
// setup(props,{emit})
// 父组件
<tab-control @ tabClick="tabClick" : title="['畅销书'，'新书','精选']"></tab-control>
setup(){
    let temid = ref(0)
    const tabClick = (index) => {
        temid.value = index
    }
    return {
        temid, // 拿到数据就可以请求响应的数据了
        tabClick
    }
}

.tab-control {
    display:flex
    height:40px
    line-height:40px
    text-align:center
    font-size:14px
    background-color:yellow

    width:100%

    // 内容很长
    position:sticky // 但是动态加载数据会失效
    top:44px

    z-index:10

    .tab-control-item {
        flex:1
        span {
            padding:6px
        }
    }
    .active {
        color:var(--color-tint);
        span{
            border-bottom:3px solid var(--color-tint)
        }
    }
}

// 用的地方传递属性、用插槽不太好办，假如有四个或两个选项卡
// 直接用属性传进来，也可以不考虑响应式，直接传
<tab-control : title="['畅销书'，'新书','精选']"></tab-control>
```

### 商品列表

```js
components/content/goods/GoodsList.vue // 商品列表
components/content/goods/GoodsListItem.vue // 商品列表项

// 商品列表
<div class="goods">
    <goods-list-item></goods-list-item>
</div>
.goods {
    display:flex
    flex-wrap:wrap
    justify-content:space-around
    padding:5px
}

// 列表项
<div class="goods-item">
    <img src="~assets/images/11.png" alt=""/>
    <div class="goods-info">
        <p>标题</p>
        <span class="price"><small>￥</small>100</span>
        <span class="collect">3</span>
    </div>
</div>

.goods-item {
    width:46%
    padding-bottom:40px

    position:relative

    img {
        width:100%
        border-radius:2px
    }
    .goods-info {
        font-size:12px
        position:absolute
        bottom:5px
        left:0 //居中
        right:0 // 居中

        overflow:hidden
        p {
            overflow:hidden
            text-overflow:ellipsis
            white-space:nowrap
            margin-bottom:3px
        }
        .price {
            color:red
            margin-right:20px
        }
        .collect {
            position:relative
        }
        .collect::before {
            content:''
            position:absolute
            left:-15px
            top:-1px

            width:14px
            height:14px

            background:url('~assets/images/collect.png') 0 0/14px 14px
                                                         位置/大小
        }
    }
}
```

### 调用后台接口获取商品列表数据

```js
在首页获取数据，因为选项卡封装的是一个子组件，点击的时候把点击的是谁传到父级，父级通过计算属性知道点击的是哪个选项卡
进而通过属性把相应的数据传递过去
（通常选项卡处理数据，都是通过计算属性来处理）
export function getHomeGoods(type="sales",page=1){
    return request({
        url:'/api/index?'+ type + '=1&page=' +page
    })
}

import { ref,reactive,onMounted,computed } from "vue"
setup(){
    // 商品列表数据模型
    let goods = reactive({
        sales:{page:0,list:[]}
        recommend:{page:0,list:[]}
        news:{page:0,list:[]}
    })

    let currentType = ref('sales')

    const showGoods = computed(()=>{
        return goods[currentType.value].list
    })

    onMounted(()=>{
        getHomeGoods('sales').then(res => {
            goods.sales.list = res.goods.data
        })
        getHomeGoods('recommemds').then(res => {
            goods.recommemd.list = res.goods.data
        })
        getHomeGoods('news').then(res => {
            goods.news.list = res.goods.data
        })
    })


    console.log(goods) // 去target查看

    return {
        goods,
        showGoods
    }
}

<goods-list : goods="showGoods"></goods-list> // 这里只传进去一个类型的数据就行

// 去商品列表组件
props:{
    goods:Array,
    default(){
        return []
    }
}
<goods-list-item v-for="item in goods" : product="item" : key="item"></goods-list-item>
// 去商品项组件
props:{
    product:Object,
    default(){
        return {}
    }
}
<div class="goods-item">
    <img : src="product.cover_url" alt=""/>
    <div class="goods-info">
        <p>{{ product.title }}</p>
        <span class="price"><small>￥</small>{{ product.price }}</span>
        <span class="collect">{{ product.collects_count }}</span>
    </div>
</div>

// 现在怎么通过点击tab切换相应数据
// 子组件把点击的tab的索引传递到父组件
// 父组件来操作
setup(){
    let currentType = ref('sales')

    const tabClick(自定义事件) = (index) => {
        let types = ["sales","recommends","news"]

        currentType.value = types[index]
    }
}

改变下标、列表数据变化、数据变化，页面就发生变化（数据是响应式的）
```

### 上拉加载更多数据

```js
安装better-scroll插件：npm i better-scroll --save
给要滚动的内容包两个壳
<div class="wrapper">
    <div class="content"></div>
</div>

#home {
    height:100vh
    position:relative
}
.wrapper {
    position:absolute
    top:45px
    bottom:50px
    left:0
    right:0
    overflow:hidden
}
import BScroll from "better-scroll"
import { watchEffect,nextTick } from "vue"

let bscroll = reactive({})
onMounted(){
    // 创建better-scroll对象
    bscroll = new BScroll(document.querySelector(".wrapper"),{
        probeType:3, // 0,1,2,3 // 3只要在滑动就会触发scroll事件
        click:true, // 是否允许点击
        pullUpLoad:true // 上拉加载更多
    })
    // 监听滚动事件
    bscroll.on('scroll',(position)=>{
        console.log(position.y)
    })
    // 上拉加载更多
    bscroll.on('pullingUp',()=>{
        console.log("上拉加载更多。。。")
        console.log(document.querySelector('.content').clientHeight)

        const page = goods[currentType.value].page + 1

        getHoomGoods(currentType.value,page).then(res => {
            goods[currentType.value].list.push(...res.goods.data)
            goods[currentType.value].page += 1
        })

        // 完成上拉，等数据请求完成，要将数据展示出来
        bscroll.finishPullUp();

        bscroll.refresh() // 拉的时候也要重新计算高度

        console.log('contentHeight:' + document.qurySelector('.content').clientHeight)
        console.log("当前类型："+currentType.value+",当前页："+page)
    })
}
// 监听任何一个变量有变化
// 不能数据一变化，就刷新，要等到数据变化，页面加载完成之后才刷新,重新计算高度，最后滚动事件才好使
watchEffect(()=>{
    nextTick(()=>{
        // 延迟执行，重新计算高度
        bacroll && bscroll.refresh()
    })
})


const tabClick(自定义事件) = (index) => {
    let types = ["sales","recommends","news"]

    currentType.value = types[index]

// 切换选项卡的时候,重新计算高度
    nextTick(()=>{
        // 延迟执行，重新计算高度
        bacroll && bscroll.refresh()
    })
}
```

# 首页 Home.vue

```js
<template>
  <div id="home">
    <nav-bar>
      <template v-slot:default>图书兄弟</template>
    </nav-bar>

    <div class="wrapper">
        <div class="content">
            <div class="banner">
                <img src="~assets/image/1.png" alt=""/>
            </div>

            <recommend-view : recommends="recommends"></recommend-view>

            <tab-control></tab-control>

            <goods-list></goods-list> // 这里只传进去一个类型的数据
        </div>
    </div>
  </div>
</template>
```

### 组件划分

```js
common:其他项目页可以使用的组件
content：本项目公用组件
```
