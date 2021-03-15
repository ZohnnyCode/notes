---
title: 24、反射Reflect
date: 2021-03-14 01:08:53
tags:
---

### 01、理解反射

```js
在静态扫描的时候，你不知道是哪个方法在使用apply，使用Reflect的apply，没有绑定在哪个方法上
当你真正在执行这个apply的时候，要执行apply的方法是作为参数传进来的
在静态扫描的时候，这个方法并没有执行，这就是js反射

console.log(Math.floor.apply(null,[3.78])) // ES5
console.log(Reflect.apply(Math.floor,null,[4.73])) // ES6 向下取整

这种反射有什么用？
类似场景：如果价格大于100，向下取整，否则向上取整
let price = 101.5
// ES5（必须先指定谁使用apply）
if(price>100){
  peice = Math.floor.apply(null,[price])
}else{
  peice = Math.ceil.apply(null,[price])
}

// ES6（反射，直接使用apply，根据条件决定那个方法使用apply）
price = Reflect.apply(price>100?Math.floor:Math.ceil,null,[price])
// 用反射不需要指明谁调用apply，根据条件执行的时候判断执行哪个方法

// 非反射是先指定方法，反射是执行过程中根据条件判断使用啥方法

// 这就是反射的意义

// 这里的apply没有取作用域、主要是突出反射的作用，没有直接使用Math.ceil()来做
// 如果要改变作用域，必须使用apply
```

### 其他可以使用反射的 api

```js
01、动态构造不同的实例
Reflect.construct():调用不同的类动态的实例化一个对象,而不用通过new去做
if...else // 分情况实现不同的实例对象
switch...case

学习反射的效果：1.什么事反射机制？2.什么情况下使用反射（比直接使用new，apply），更加简单更加动态化
是什么？有什么用？

反射实现生成的实例对象跟new出来的没有区别，d instanceof Date

// 以上是construct用反射机制去动态的实例化一个类

// es5
let d = new Date()
console.log(d.getTime())

// es6
let d = Reflect.construct(Date,[]) // 参数必须是一个空数组
console.log(d.getTime(),d instancof Date)

02、给对象新增一个属性
Reflect.defineProperty() // 给一个对象新增一个属性
// 参数：给谁添加、属性名字、属性的特性
// 返回值：属性定义成功与否boolean

和Object.defineProperty()唯一的不同是返回值不同
// 返回值：被定义的对象

// es5
let student = {}
let res = Object.defineProperty(student,"name",{value:"大帅比"})
console.log(res) // {name:"大帅比"}

// es6
let student = {}
let res = Reflect.defineProperty(student,'name',{value:"大帅比"})
console.log(res) // true

03、删除对象上的一个属性
Reflect.deleteProperty()
// 删除对象的属性
// 返回值：boolean代表删除成功与否
let s= {
      x:1,
      v:2
    }
let res = Reflect.deleteProperty(s,"x")
console.log(res) // true

// Object.deleteProperty() // 此方法被废除？

04、获取对象或数组的值
Reflect.get()
// 动态读取对象(属性)或数组（下标对应）的值
// 读对象
const obj = {x:1,y:2}
Reflect.get(obj,"x") // log一下就是：1

// 读数组
Reflect.get([5,8],1) // log一下就是：8（某种场景动态化的读取数据）

05、获取对象某个属性的属性描述符
Reflect.getOwnPropertyDescriptor(obj,"属性")
Object.getOwnPropertyDescriptor(obj,"属性")
// 二者返回结果一样

06、查看一个实例是由谁生成的（构造函数是谁）、有哪些原型对象上的方法
调用别人的类的实例的时候，不知道它的原型对象是什么、调用这个方法，快速知道原型上部署了哪些方法
Reflect.getPrototypeOf("实例")

let d = new Date()
console.log(Reflect.getPrototypeOf(d)) // constructor

07、判断一个对象是否包含一个属性或方法
Reflect.has("对象","属性")
// 返回值boolean

const obj = {x:1,y:2}
Reflect.has(obj,x) // log一下就是true

08、判断一个对象是否是可扩展的,是否被冻结了
Reflect.isExtensible("对象")
// 返回值boolean

const obj = {x:1,y:2}
Reflect.isExtensible(obj) // log一下就是true

// 把obj冻结
Object.freeze(obj)
obj.z = 3
Reflect.isExtensible(obj) // log一下就是false
console.log(obj) // {x:1,y:2}

09、获取对象或数组自身的属性
Reflect.ownKeys("对象或数组")
// 返回值：由对象或数组自身属性组成的数组

const obj = {x:1,y:2}
Reflect.ownKeys(obj) // ["x","y"]
Reflect.ownKeys([1,2]) // ["0","1","length"]

10、Reflect自己冻结对象,让对象不可扩展，跟Object.freeze()起相同的作用
Reflect.preventExtensions(obj)

11、给一个对象或数组设置属性
Reflect.set("对象","属性名","属性值")

const obj = {x:1,y:2}
Reflect.set(obj,"z",4)

const arr = ["like","like","like"]
Reflect.set(arr,2,"love")

12、给一个对象修改它的原型对象,修改它的原型链
Reflect.setProtypeOf("改谁","改到哪里")
sort方法是部署在arr实例对象的原型对象上的

const arr = ["like","like","like"]
Reflect.setPrototypeOf(arr,String.prototype)
arr.sort() // 报错：sort is not a function

console.log(Reflect.getPrototypeOf(arr)) // String.prototype
```

```js
原来部署在Object上的方法，在Reflect对象上都有
Reflect上有的，Object上面可能没有
Reflect是新的标准，让操作更加标准化
```
