---
title: 41、Vue
date: 2021-03-17 22:35:13
tags:
---

```js
引入的组件要注册
```

# 创建 Vue 项目

```js
安装脚手架：npm install @vue/cli -g
创建模板：vue create 名字
```

### 组件传值

```js
父向子：通过属性
// 父组件
<child :msg="message"/>

// 子组件
export default {
  props:["msg"] // msg现在在子组件中就可以使用了
}


// 子向父：自定义事件
// 父组件
// 先定义一个自定义事件
<child @myEvent="changeData" />
// 还没传过来，先定义一个状态，等待接收
data(){
  return {
    childData:''
  }
}

methods:{
  changeData(childData){
    this.childData = childData
  }
}

// 子组件
// 子组件触发这个自定义事件就可以传值了
<button &click="sendData"></button>

data(){
  return {
    childData:"I'm child"
  }
}
methods:{
  sendData(){
    // this.$emit可以触发父级的自定义事件。第一个参数自定义事件，第二个参数要传的数据
    this.$emit('myEvent',this.childData)
  }
}

// 购物车案例
// 一个cart组件引入counter组件，cart组件定义商品数据
// cart引入counter组件，并把数量传递过去
// counter组件点击加减分发cart组件的自定义事件改变cart的商品数量，注意区分是哪个商品的数量

// 兄弟间共享数据
// 定义一个store.js文件
export default {
  state:{
    message:"共享的数据"
  },
  setStateMessage(str){
    this.state.message = str;
  }
}

// 在各组件间引用
import store form "/store"
export default {
  data(){
    return {
      state:store.state
    }
  }
}

<p>{{state.message}}</p>
<button @click="changeData"></button> // 在某一兄弟组件中改变共享的数据

methods:{
  changeData(){
    store.setStateMessage("兄弟共享的数据")
  }
}
```

### 侦听器

```js
侦听器：一个值变化，影响多个值（为了观察一个值）******检测路由变化，只能用侦听器
计算属性：多个值改变为了得到一个结果，性能较好

大多数情况都可以用计算属性解决问题
```

### Vue 实例

```js
new Vue()创建一个Vue实例

所有的组件其实都是Vue实例

每个Vue实例在被创建时都要进过一系列的初始化过程

create(){
  this.getData()
}
methods:{
  getData(){
    setTimeout(()=>{
      this.fruitList = ["西红柿","番茄蛋","橙子"]
    },2000)
  }
}

// 显示加载中
data(){
  return {
    loading:true
  }
}
<p v-if="loading">loading...</p>
<ul v-if="!loading"></ul>
```
