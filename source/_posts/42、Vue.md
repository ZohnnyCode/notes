---
title: 42、Vue
date: 2021-03-18 09:36:17
tags:
---

### 插槽：创建更加灵活，更易扩展的组件

```js
MyButton.vue
<template>
    <div>
        <button>
            <slot></slot> // 这就是一个插槽，这个位置可以通过插槽自定义
        </button>
    </div>
</template>

// 父级如何往插槽填充内容呢,就往*******组件标签********填充内容就好了，内容会替换slot
App.vue
<template>
    <div id="app">
        <my-utton>提交</my-utton>
        <my-utton>注册</my-utton>
    </div>
</template>
<script>
import MyButton from "./MyButton"
export default {
    components:{
        MyButton
    }
}
</script>

// 它是一个组件，我希望父级可以往里面插入一些东西：插槽
// 具名插槽：定义名字、可定义多个插槽
Layout.vue
<template>
    <div>
        <div class="header">
            <slot name="header"></slot>
        </div>
        <div class="content">
            <slot name="content"></slot>
        </div>
        <div class="footer">
            <slot name="footer"></slot>
        </div>
    </div>
</template>

使用：template+v-slot
<layout>
    <template v-slot:header>
        <h1>header</h1> // 把想添加到插槽里的内容就可以添加到这里
    </template>
    <template v-slot:content>
        <h1>content</h1>
    </template>
    <template v-slot:footer>
        <h1>footer</h1>
    </template>
</layout>
```

### DOM 操作

```js
// 原生js
<div id="box">Hello World</div>
mounted(){
    let box = document.querySelector("#box")
    let style = window.getComputedStyle(box)
    log(style.width)
}

// Vue的方式
<div ref="box">Hello World</div>
mounted(){
    let box = this.$refs.box
    let style = window.getComputedStyle(box)
    log(style.width)
}
```

### 计算属性和侦听器

```js
// 计算属性：data属性和computed属性定义的值都可以直接绑定在表达式中
<p>{{message}}</p> // 类似data
computed:{
    message(){
        return "hello computed"
    }
}

data(){
    return {
        firstname:'张',
        lastname:'帅'
    }
}
computed(){
    name(){
        return this.firstname + this.lastname
    }
}

// 例子
总价 = 单价*数量*折扣
data(){
    return {
        price:99,
        num:1,
        discount:0.5
    }
}
<button @click="add">+</button>
    {{num}}
<button @click="sub">-</button>
add(){
    this.num++
}
sub(){
    if(this.num>0){
        this.num--
    }
}
computed:{
    totalPrice(){
        return this.price * this.num * this.discount
    }
}

// 侦听器:监听一个属性的变化
// 是一个函数，里面的形参是属性变化之后的结果
data(){
    return {
        price:99,
        num:0,
        discount:0.5,
        totalPrice:0
    }
}
watch:{
    num(val){
        // log(val) // 变化后的结果
        return this.price * val * this.discount
    }
}
```

### 通过固定算法重新组织数据

```js
// 定义过滤器
filters:{
    mySplit(value){
        // value就是 | 前面的变量
        return value.split("").join();
    },
    dateFormat(value){
        let date = new Date(value)
        let year = date.getFullYear();
        let month = date.getMonth() + 1;
        let d = date.getDate();
        return `${year}年${month}月${date}日`
    }
}
// 使用过滤器
<h1>{{ message | mySplit }}</h1>

例子：日期格式化（2021-3-18-->2021年3月18日）
date:"2021-3-18"
<h1>{{ date | dateFormat }}</h1>
```

### 表单处理:v-model

```js
formData:{
    username:'',
    password:'',
    hobby:'',
    sex:'',
    skill:[]
}

<form @submit.prevent="postData">
    <button>提交表单</button> // 点击button默认会触发form的submit事件
</form>

postData(){
    console.log(this.formData)
}

// 文本框
<input type="text" v-model="formData.username" />

// 密码框
<input type="password" v-model="formData.password" />

// 下拉框
<label for="">爱好</label>
<select v-model="formData.hobby">
// 传递后台的内容给到value里面、显示的内容写在标签里面
    <option value="basketball">篮球</option>
    <option value="football">足球</option>
    <option>足球</option>
</select>

// 单选框
<label>性别：</label>
<label for="">男</label>
<input type="radio" value="男" v-model="formData.sex"/>
<label for="">女</label>
<input type="radio" value="女" v-model="formData.sex"/>

// 复选框
<label for="">技能：</label>
<label for="">前端</label>
<input type="checkbox" value="前端" v-model="formData.skill" />
<label for="">java</label>
<input type="checkbox" value="java" v-model="formData.skill" />
```

### 路由

```js
router-link必须写明to属性才可以显示标签体内容

// 路由例子
首页 | 博客 | 视频 || 欢迎：小明  注销(按钮)

还有一个登录页，点击登录，存数据，跳转到首页
此时要初始化欢迎信息：使用监视，"$route.path"，路由切换，主动获取localStorage里面的值
点击注销，调到登录页，用户名清空
在router/index.js文件里面：做路由守卫

<span v-if="showLogout">欢迎：{{username}} <button>注销</button></span> // 没登录的时候不显示

// App.vue
data(){
    return {
        username:localStorage.getItem('usr')
        showLogout:localStorage.getItem('usr')
    }
},
watch:{
    '$route.path':function(){
        this.username = localStorage.getItem('usr')
        this.showLogout = localStorage.getItem('usr')
    }
},
methods:{
    logout(){
        localStorage.clear()
        this.$router.push('/login')
    }
}

// Login.vue
doLogin(){
    localStorage.setItem('usr',this.username) // username是v-model双向绑定到表单里面的
    this.$router.push('/')
}

// router/index.js
router.beforeEach((to,from,next)=>{
    if(to.path !== "/login"){
        if(localStorage.getItem('usr')){
            next()
        }else{
            next("/login") // 没登录，重定向
        }
    }else{
        next()
    }
})
```

### data 里面的数据变成响应式

```js
// 第一种方法
this.$set(item, "bring_img", "2");

// 第二种方法
this.rowData = {
  ...item,
  bring_img: "2",
  fruit: ["西瓜"],
};

// 这样不管在哪里，新增的属性就是响应式的了，只要改变，页面就会重新渲染
```

### 复选框操作

```js
<CheckboxGroup v-model="checkAllGroup" @on-change="checkAllGroupChange">
    <Checkbox label="香蕉"></Checkbox>
    <Checkbox label="苹果"></Checkbox>
    <Checkbox label="西瓜"></Checkbox>
</CheckboxGroup>

checkAllGroup这个数组中的元素值有label中的啥，啥就被选中
// 判断全选全不选很简单，定义一个变量 \
isAllChecked:false
checkAllGroupChange(checkedArr){
    if(checkedArr.length === 3){
        isAllChecked = true
    }
}
if(isAllChecked){
    checkAllGroup = ["香蕉","苹果","西瓜"]
}else{
    checkAllGroup = []
}

// 复选框标签里面还可以使用图片img标签
```

### 前后台数据交互

```js
// axios获取数据
getFruitList(){
    axios.get("http://127.0.0.1:3000/fruits").then(res => {
        this.fruitList = res.data;
    })
}

// 删除根据索引删除
<button @click="del(index)">删除</button>

del(index){
    axios.delete(`http://127.0.0.1:3000/fruits/${index}`)
        .then(res=>{
            this.getFruitList();
        })
}

// 获取数据：getFruitList()
// 添加数据：postData()
// 删除数据：del()

// 以后封装axios
```

# 注意点

```js
遍历的时候注意加key;
```

# 项目部署

```js
// 配置文件
.env.development
.env.production
注意变量名必须以VUE_APP_XXXX开头

怎么在组件中使用环境变量呢
data(){
    return {
        title:process.env.VUE_APP_TITLE
    }
}

VUE_APP_BASE_URL = "http://localhost:3000" // 开发环境
VUE_APP_BASE_URL = "" // 生产环境
```

### 异步数据对搜索引擎不友好（Vue 单页面应用）

### 服务端渲染更适合制作网站（nunjucks Vue 也可以制作服务端渲染的应用）

### 项目发布

```js
1.发布到公网
2.域名直接访问
```
