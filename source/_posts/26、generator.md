---
title: 26、generator
date: 2021-03-29 16:34:45
tags:
---

### ES6 如何让遍历停下来

```js
而不是像break、contiune那样退出
什么时候停下来，什么时候继续
```

### generator

```js
function* loop() {
  for (let i = 0; i < 5; i++) {
    yield console.log(i);
  }
}

const l = loop();
l.next();  // 0
l.next();
l.next();
l.next();
l.next();

l.next(); // 执行完了啥都不执行

generator本质就是一个函数，比普通函数多了"*",内部多了yield关键字，只能在generator内部使用，哪里要暂停加yield
```

### yield 关键字没有返回值

```js
function* gen() {
  let val;
  val = yield 1;
  console.log(val);
}

const l = gen();
l.next(); // 没输出
l.next(); // undefined

generator返回一个对象，有个next方法（next就是向下执行的过程），用来找yield关键字或函数结尾（找到其中一个就会暂停或结束）
调用一次next，执行yield关键字后面的语句，并在执行完之后停下来，等待下一个next的执行，才会继续执行yield下面的语句
（上面例子打印val就是在第二次才打印，第一次yield就是1，没有任何输出，所以第一次执行next无任何反应
第二次next的时候才开始做赋值的操作，继续往下执行找yield，执行完log还没找到yield，此时函数已经结束了，所以这时候过程才真正结束了）

第一个例子因为yield后面是打印语句，所以第一次next有打印内容
```

### yield 的第二种语法

```js
function* gen() {
  let val;
  val = yield* [1, 2, 3];
  console.log(val);
}

const l = gen();
console.log(l.next()); // {value:1,done:false}
console.log(l.next()); // {value:2,done:false}

next返回值是一个对象：value当前遍历的对象的值是什么，第二个循环是否已经结束
yield加*号后，后面表示的是遍历的对象（可迭代的对象），还可以是generator实例（可以嵌套generator对象）

// yield后不加：*
function* gen() {
  let val;
  val = yield [1, 2, 3];
  console.log(val);
}

const l = gen();
console.log(l.next()); // {value:Array(3),done:false}
// undefined
console.log(l.next()); // {value:undefined,done:true} // 注意每一次调用next时的返回值对应的value值是不一样的
```

### 总结

```js
1、generator是一个函数，这个函数定义时比普通函数多了一个*号
2、在generator内部想要让函数停下来，得加yield关键字
3、generator可以嵌套，可以在yield后面再加*号
4、每次执行是用next恢复执行，同时返回当前执行的状态（done）和数据（value）
```

### 高级语法：next 传参（把参数当做 yield 的返回值，不传 undefined）

```js
function* gen() {
  let val;
  val = yield [1, 2, 3];
  console.log(val);
}

const l = gen();
// 执行过程
第一次没有赋值，第二次才有赋值的操作（执行完才打印返回值）

console.log(l.next(10)); // {value:Array(3),done:false}
// 20
console.log(l.next(20)); // {value:undefined,done:true} // 注意每一次调用next时的返回值对应的value值是不一样的

// 分析：next就是把当次调用传进去的参数当做这次yield的返回值，来改变每次执行的结果
执行next就是找yield表达式
```

### 终止 next 执行 1

```js
function* gen() {
  let val;
  val = yield [1, 2, 3];
  console.log(val);
}

const l = gen();
// 执行过程
第一次没有赋值，第二次才有赋值的操作（执行完才打印返回值）

console.log(l.next(10)); //
l.return()
console.log(l.next(20)); //

// 打印结果
{value:Array(3),done:false}
{value:undefined,done:true}

return函数也有返回值
console.log(l.next(10)); //
console.log(l.return(100))
console.log(l.next(20)); //

// return 函数打印结果（不传100变undefined）
{value:Array(3),done:false}
{value:100,done:true}
{value:undefined,done:true}
```

### 终止 next 执行 2

```js
function* gen() {
  while (true) {
    try {
      yield 1;
    } catch (e) {
      console.log(e.message);
    }
  }
}
const g = gen();
console.log(g.next());
console.log(g.next());
console.log(g.next());
g.throw(new Error("ss")); // 这里像for循环的contiune一样，直接绕过去了
console.log(g.next());
```

### 总结

```js
1.next是可以传参数的，修改内部运行的数据
2.可以提前终止generator函数内部运行（靠return 函数）
3.可以从外部向内部抛出异常（throw函数），内部通过try...catch捕获异常，程序运行不会受干扰，不会中断
```

### 应用

```js
// ES5抽奖
function draw(first = 1, second = 3, third = 5) {
  let firstPrize = [
    "1A",
    "2A",
    "3A",
    "4A",
    "5A",
    "6A",
    "7A",
    "8A",
    "9A",
    "10A",
  ];
  let secondPrize = [
    "1B",
    "2B",
    "3B",
    "4B",
    "5B",
    "6B",
    "7B",
    "8B",
    "9B",
    "10B",
    "11B",
    "12B",
    "13B",
    "14B",
    "15B",
    "16B",
  ];
  let thridPrize = [
    "1C",
    "2C",
    "3C",
    "4C",
    "5C",
    "6C",
    "7C",
    "8C",
    "9C",
    "10C",
    "11C",
    "12C",
    "13C",
    "14C",
    "15C",
    "16C",
  ];

  let result = [];
  let random;

  for (let i = 0; i < first; i++) {
    random = Math.floor(Math.random() * firstPrize.length);
    result = result.concat(firstPrize.splice(random, 1));
  }
  for (let i = 0; i < second; i++) {
    random = Math.floor(Math.random() * secondPrize.length);
    result = result.concat(secondPrize.splice(random, 1));
  }
  for (let i = 0; i < third; i++) {
    random = Math.floor(Math.random() * thirdPrize.length);
    result = result.concat(thirdPrize.splice(random, 1));
  }

  return result;
}
let t = draw();
for (let value of t) {
  console.log(value);
}

// ES6抽奖
let count = 0
let random

while(1){
    if(count < first){
        random = Math.floor(Math.random() * firstPrize.length)
        yield firstPrize[random]
        count++
        firstPrize.splice(random,1)
    }else if(count < first+second){
        random = Math.floor(Math.random() * firstPrize.length)
        yield secondPrize[random]
        count++
        secondPrize.splice(random,1)
    }else if(count < first + second + third){
        random = Math.floor(Math.random() * thirdPrize.length)
        yield thirdPrize[random]
        count++
        thirdPrize.splice(random,1)
    }else{
        return false
    }
}

let d = draw()

console.log(d.next().value) // 一等奖
console.log(d.next().value)
console.log(d.next().value)
console.log(d.next().value) // 二等奖
console.log(d.next().value)
console.log(d.next().value)
console.log(d.next().value)
console.log(d.next().value)
console.log(d.next().value) // 三等奖
console.log(d.next().value)
```

```js
// 小游戏：一直加，三的倍数输出，无上限
function* count(x = 1) {
  while (1) {
    if (x % 3 === 0) {
      yield x;
    }
    x++;
  }
}

const num = count();
console.log(num.next().value); // 3
console.log(num.next().value); // 6
console.log(num.next().value); // 9
console.log(num.next().value); // 12
console.log(num.next().value); // 15
```

```js
练习：
1.用genarator给自定义数据结构写一个遍历器
2.用generator实现一个斐波那切数列
```
