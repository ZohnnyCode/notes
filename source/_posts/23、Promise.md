---
title: 23、Promise
date: 2021-03-13 14:13:05
tags:
---

### ES5 做法

```js
回调地狱问题：a执行完b执行，b执行完c执行

对于异步操作，由于js是单线程的，调用函数如果函数体是异步操作，不会等异步操作执行完就会继续往下执行
做不到异步函数执行完执行同步函数（执行完异步函数，而异步函数体没执行（js会把它当做执行完了），js会继续往下执行）
异步操作会放到异步队列里面，等单线程执行完之后，才会继续执行异步队列里面的内容，所以是先输出同步函数结果

异步操作不会立马执行，而是会放到异步队列里面去，优先同步操作，完了，才去异步操作

function loadScript(src){
  let script = document.createElement('script')
  script.src = src
  document.head.append(script)
}

function test(){
  console.log("test")
}

loadScript("./1.js") // 1.js的内容是：console.log(1)
test()

执行结果：
test
1
```

```js
加载脚本是异步的过程，而异步操作函数体代码执行是同步的过程
异步加载标签会有个onLoad事件，异步完成后会触发这个事件

function loadScript(src,callback){
  let script = document.createElement('script')
  script.src = src

  script.onload = () => {
    callback()
  }

  document.head.append(script)
}

function test(){
  console.log("test")
}

loadScript("./1.js",test)

// 传参
function loadScript(src,callback){
  let script = document.createElement('script')
  script.src = src

  script.onload = () => {
    callback(src)
  }

  document.head.append(script)
}

function test(name){
  console.log(name)
}

loadScript("./1.js",test)
```

### 加载完一，再加载二，最后再加载三

```js
function loadScript(src, callback) {
  let script = document.createElement("script")
  script.src = src

  // 异步加载成功
  script.onload = () => {
    callback(src)
  }
  // 异步加载失败
  script.onerror = (err) => {
    callback(err)
  }

  document.head.append(script)
}

loadScript("./1.js", (script) => {
  console.log(script)
  if(script.message){
    // 监控上报，出现错误接下来怎么处理
  }else{

  }
  loadScript("./2.js", (script) => {
    console.log(script)
    loadScript("./3.js", (script) => {
      console.log(script)
    })
  })
})

这里只是加载静态资源文件，可能加载失败（网络中断，CDN出现问题），会报错，怎么处理错误
```

### ES6 做法：Promise

```js
Promise接受一个函数参数，有两个参数resolve，reject
在Promise传进来的函数参数体里面做异步操作（加载资源文件）

function loadScript(src){
  return new Promise((resolve,reject)=>{
    let script = document.createElement("script")
    script.src = src

    script.onload = () => resolve(src)
    script.onerror = () => reject(src)

    document.head.append(script)
  })
}

loadScript('./1.js')
  .then(loadScript('./2.js'))
  .then(loadScript('./3.js'))

用同步代码的方式表示异步操作、不嵌套、平行写
```

### 01、工作原理

```js
promise工作原理：
在执行new Promise()对象的时候，Promise对象会有一个状态（status）一个结果（result），pending
resolve,reject这两个方法是用来改变Promise对象状态和结果的

Promise基本用法、能解决什么问题：
刚开始：pending---undefined
resolve把状态改成：fulfilled，result就是resolve的参数值
reject执行：状态：rejected，result：error

函数要返回promise对象，后面才可以.then.then

没有调用.then是啥状态，结果是啥
promise对象是怎么改变状态的（状态是不可逆的，只能改变一次）
promise状态改变，它的结果是怎么传递数据的
```

### 02、then 方法详解之精通使用

```js
.then:是promise对象原型上面的方法，只要是promise对象就可以调用这个方法
支持两个函数类型的参数，第一个是必选的（对应reslve），第二个（对应reject）是可选的
如果没有这两个参数会返回一个空的promise对象，保证调用then就返回一个新的promise对象

官方语法：
promise.then(onFulfilled,onRejected)
这两个参数都是方法，上面是函数调用返回了promise对象，很明显这不是方法，也正确执行了
如果这两个参数被遗漏，都没写，这时候会忽略掉这两个参数，为什么又执行了
这是因为在判断then方法参数的时候，会把它当做一个表达式来看，居然是表达式就会计算表达式的值
一旦被计算，那就执行 loadScript('./2.js')，所以就执行了loadScript函数体内容

所以then的正确姿势是要传两个参数进去
返回值是一个新的promise对象，所以then可以链式调用
没传两个参数的时候then返回的是一个空的promise对象（注意是then（因为没有resolve，reject））

知道then的正确语法，传的是非函数的时候是有瑕疵的，有的时候正常工作，有的时候不能

loadScript('./1.js') // then的正确姿势
  .then(()=>{
    loadScript('./2.js')
  },(err)=>{
    consle.log(err)
  })

// 写了两个参数，如果没有返回新的promise对象来控制下一步流转的then
// then默认是返回一个新的空的promise对象所以下一流程的then是走成功的
// 下面结果打印1,3
loadScript('./1.js')
  .then(()=>{
    loadScript('./4.js')
  },(err)=>{
    consle.log(err)
  })
  .then(()=>{
    loadScript("./3.js")
  },(err)=>{
    console.log(err)
  })

// 如果要控制下一步流转的then，则手动返回一个promise对象
.then(()=>{
    return loadScript('./4.js') // 影响下一步是如何执行的
  },(err)=>{
    consle.log(err)
  })
```

### 03、两个静态方法把返回值不是 promise 对象的返回 promise 对象

```js
promise有两个静态类型方法：显示操作做类型转换（数字转promise对象）

情景：某些逻辑有异步操作，有些没有，例如，本地（localStorage）有离线数据返回数字
没有我要通过走线上接口去查（走promise异步操作）
Promsie.resolve(42)
Promise.reject(new Error('ss'))

function test(bool){
  if(bool){
    return new Promise()
  }else{
    return Promise.resolve(42)
  }
}

test(0).then(v=>{
  consle.log(v)
},err=>{
  console.log(err)
})

// 报错
test(1).then(v=>{
  consle.log(v) // 30
})
// 应该这样子写
return new Promise((resolve,reject)=>{
  resolve(30)
})

这就是同步操作后，用then做处理

静态方法：直接对象调用

在什么样的场合，用resolve和reject快速生成promise实例
```

### 04、catch 统一优雅的处理所有的报错

```js
catch()是实例方法，不是静态方法,用来处理链式操作上reject抛出的异常

test(0)
  .then((v) => {
    consle.log(v)
  })
  .catch((err) => {
    console.log(err)
  })

注意catch捕获错误的时候一定要reject的方式
不能throw new Error()

catch是捕获promise状态是reject的时候捕获
```

### 05、异步操作无非就是并行和串行两种方式之并行

```js
Promise.all() // 这个静态方法
// 代表着所有异步操作完成之后要做什么，因为多个异步操作，所以参数是数组，数组元素是不同的promise实例
// 返回的还是一个promise对象，可以使用then
const p1 = Promise.resolve(1)
const p2 = Promise.resolve(2)
const p3 = Promise.resolve(3)

Promise.all([p1, p2, p3]).then((value) => {
  console.log(value) // [1,2,3]
})

// 利用这个函数轻松实现接口的并行实现，then拿到的是所有的数据
// 不需要关心哪个异步接口先返回数据，不用自己整合数据细节，数据自动整合了
```

### 06、两个异步操作模拟两个线路的响应速度不一样（先到先得）

```js
情景：项目图片很重要，把它放在两台服务器上，确保可以从其中一台请求到就行

const p1 = () => {
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      resolve(1)
    },1000)
  })
}

const p2 = () => {
  return new Promise((resolve,reject)=>{
    setTimeout(()=>{
      resolve(2)
    },0)
  })
}

使用Promise类上的静态方法：race
参数为promise对象为元素组成的数组，我上面两个是函数所以调用一下返回promise对象
返回值是promise对象，调用then获取到竞争第一名的那个异步操作的结果
Promise.race([p1(),p2()]).then(value=>{
  console.log(value) // 2 (p2比较快)
})

比较all，所有的都要回来（所有成功则成功，一个失败就失败）
race传进的promise对象集合中，有一个是完成状态，它就把那个值放到then方法中，代表完成
```

```js
总结：关于异步操作要想到用promise解决，不要再回调了
哪些是静态方法（起什么作用，适合什么场景），哪些是实例对象的方法

练习：请用promise实现一个接口
```
