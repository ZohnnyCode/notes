---
title: 25、proxy
date: 2021-03-14 14:53:09
tags:
---

### proxy:代理

```js
ES6强大的代理功能怎么使用？
代理的核心价值：屏蔽原始信息，保障原始信息的安全（类似房东跟中介，房东不想自己的信息满天飞）

js中原始信息可以是对象数组函数都可以，然后你想对这些信息做什么，有个操作（卖房子，租房子，代理跟这很想）

Proxy是一个类，要new一下来做代理
参数：第一个（要代理谁），第二个（操作（代理之后能干什么（劫持构造函数，劫持get、set）））

实例：中介模型
let o(房东) = {
  name:"大帅比",
  price:190
}

let d(中介) = new Proxy(o,{
  get(target,key){
    if(key==='price'){
      return target[key] + 20
    }else{
      return target[key]
    }
  }
})

console.log(d.price,d.name) // 210 大帅比
// 这个代理是改变信息
```

```js
01、服务端拿到数据备份，不想对它进行赋值操作（不允许set，把它变只读的，留一份存底，保证永远没有被修改过）
把数据拷贝一份再做各种操作
想还原的时候再拿这份原始数据

需求：后端拿回的数据要排序，但有个小按钮，点击要还原原来的数据

set拦截，不让它赋值，直接return false，利用这个原则保证o不被修改，暴露中介d（让用户访问d），不暴露o
Proxy第二个参数对象叫handle
let o = {
  name:'大帅比',
  price:100
}

let d = new Proxy(o,{
  get(target,key){
    return target[key]
  },
  set(target,key,value){
    return false
  }
})

这个代理是从我这里过读可以，但不允许更改
代理没有把o锁死，中介自己还是可以操作的

// ES5
for(let [key] of Object.entries(o)){
  Object.defineProperty(o,key,{
    writable:false
  })
}
o.price = 300
console.log(o.name,o.price) // 大帅比 190 // 只读，不可写

ES5完全把o锁死了，就连中介自己也不可以操作
```

```js
Object.entries(o); // 让一个对象以键值对的形式变成可遍历的
```

```js
02、对赋值进行校验
// 01、数据结构不能够被改变
// 02、对赋的值进行限制（price大于300，不能赋值）

let o = {
  name:'大帅比',
  price:100
}

let d = new Proxy(o,{
  get(target,key){
    return target[key] || "" // 访问没有的属性，兼容一下返回空字符串
  },
  set(target,key,value){
    if(Reflect.has(target,key)){
      if(key==="price"){ // 限制price范围
        if(value>300){
          return false
        }else{
          target[key] = value
        }
      }else{
        target[key] = value
      }
    }else{
      return false // 结构不可改变
    }
  }
})

d.age=400 // 写不进去
d.age // "" 读则undefined

Proxy轻松做到了数据结构不被破坏，对你的写操作进行校验，做到能写就写，否则拒绝

// 解耦
怎么做到校验跟业务逻辑不要耦合？上面逻辑还是耦合到proxy里面了
解耦：写多个validator
把不同的验证函数放到不同的模块下面（日历、邮箱）

const validator = (target,key,value) => {
  if(Reflect.has(target,key)){
      if(key==="price"){ // 限制price范围
        if(value>300){
          return false
        }else{
          target[key] = value
        }
      }else{
        target[key] = value
      }
    }else{
      return false // 结构不可改变
    }
}

let d = new Proxy(o,{
  get(target,key){
    return target[key] || "" // 访问没有的属性，兼容一下返回空字符串
  },
  set:validator
})

d.price = 301
d.name = "大锤子"
d.age = 18
console.log(d.price,d.name,d.age) // 结果： 190 大锤子 ""
```

```js
03、监控，做上报处理（收集是谁做的）
判断的时候，做一个错误的触发，违规了就要做触发报错
报错跟上报是两个逻辑（错就是错了）

怎么上报？
上报逻辑要在监听错误的逻辑里面

// 监听错误
window.addEventListener('error',(e)=>{
  console.log(e.message)
  report() // 上报逻辑
},true) // 要捕获而不是冒泡

// 校验规则
const validator = (target,key,value) => {
  if(Reflect.has(target,key)){
      if(key==="price"){ // 限制price范围
        if(value>300){
          throw new TypeError("price is more then 300") // 不满足规则就要触发错误，某一天不想上报，但还是要触发错误，用户就不能执行违规的操作，也可以感知，捕获到错误之后就可以上报了
        }else{
          target[key] = value
        }
      }else{
        target[key] = value
      }
    }else{
      return false // 结构不可改变
    }
}

d.price = 301 // 触发报错，下面语句不执行,下面代码不执行
              // 相当于用错误中断了用户的违规操作，并且能上报
              // 这里利用proxy结合加上错误处理，让代码更加解耦，完成上报机制
d.name = "大锤子"
d.age = 18
console.log(d.price,d.name,d.age) // 结果： 190 大锤子 ""
```

```js
场景3：我有多个组件，每个组件有唯一的id，让我知道报错的时候是谁报错
生成组件的时候都会随机生成一个唯一的id，生成的时候就只读了，不可更改

随机生成一个随机数，转化成字符串，截取后8位
Math.random().toString(36进制).splice(-8)

// 十次都一样的
class Component {
  constructor(){
    this.id = Math.random().toString().slice(-8)
  }
}

现在实例化的时候生成的id都是随机且唯一的，且每次读的时候都是一样的
    但是id不是只读的可以被修改
let com = new Component()
let com2 = new Component()
for(let i=0;i<=10;i++){
  console.log(com.id) // 十次都一样
}
com.id = "abc"
console.log(com.id,com2.id) // abc

// 定义类的只读属性，但是现在10次每次返回的都是随机的数值
class Component {
  get id(){
    return Math.random().toString().slice(-8)
  }
}

com.id = "abc"
console.log(com.id,com2.id) // 结果：随机值 随机值

// 现在放到构造函数里面不行，只读的属性里面也不行

03、使用代理
// 每次生成都是唯一的（不同实例之间唯一的）、只读的
class Component {
  constructor(){
    this.proxy = new Proxy({
      id:Math.random().toString().slice(-8)
    },{}) // 第二个参数空，代表无操作
  }

  get id(){
    return this.proxy.id
  }
}
for(let i=0;i<=10;i++){
  console.log(com.id) // 十次都一样
}
com.id = "abc"
console.log(com.id,com2.id) // 结果：随机值 随机值(真正实现)
```

### 想到中介，要想到 proxy

```js
把我们的信息放到proxy做代理操作;
```

### 撤销代理操作

```js
不能用：new Proxy
得用：Proxy.revocable(),后面操作跟new一样

let o = {
  name:'xiaoming',
  price:190
}

let d = Proxy.revocable(o,{
  get(target,key){
    if(key === "price"){
      return target[key] + 20
    }else{
      return target[key]
    }
  }
})

console.log(d.proxy.price,d)
setTimeout(()=>{
  d.revoke()
  setTimeout(()=>{
    console.log(d.proxy.price) // 报错，撤销后的代理数据无法再读取了
  },100)
},1000)

此时的d不是简单的代理数据了，它现在包含两个内容，一个是代理的数据（d.proxy.price/name），一个是撤销的操作两个东西
```
