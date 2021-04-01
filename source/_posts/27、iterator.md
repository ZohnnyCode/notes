---
title: 27、iterator
date: 2021-03-30 10:00:00
tags:
---

# Iterator:可遍历的接口

# ES6 如何让不支持遍历的数据结构“可遍历”

### 遍历

```js
let authors = {
  allAuthors: {
    fiction: ["Asgh", "Yhuc", "Ehjd"],
    scienceFiction: ["Negj", "Whvj", "Ucdi", "Qghu"],
    fantasy: ["Bhus", "Puod", "Rtyu", "Bjij"],
  },
  Adsres: [],
};

ES5遍历：
let r = []
for(let [k,v] of Object.entries(authors.allAuthors)){
    r.concat(v)
}
console.log(r)

// 我们希望这样子遍历
let r = []
for(let v of authors){
    console.log(v)
    r.push(v)
}
```

### 如果想让对象可遍历一定是部署了可遍历的接口

```js
怎么部署接口？写自定义遍历的接口
1.在对象上加Symbol.iterator的key，赋值一个方法，方法的输入是this，输出是一个对象，有next方法返回一个对象有两个属性done和value
authors[Symbol.iterator] = function(){
    // 输入this
    // 输出
    return {
        next(){
            return {
                done:false,
                value:1
            }
        }
    }
}

```

### 实现自定义遍历器

```js
authors[Symbol.iterator] = function () {
  let allAuthors = this.allAuthors;
  let keys = Reflect.ownKeys(allAuthors);
  let values = [];

  return {
    next() {
      if (!values.length) {
        if (keys.length) {
          values = allAuthors[keys[0]];
          keys.shift();
        }
      }
      return {
        done: !values.length,
        value: values.shift(),
      };
    },
  };
};

// 现在数据结构跟业务逻辑解耦了
let r = [];
for (let v of authors) {
  console.log(v);
  r.push(v);
}

可迭代协议：对象上有Symbol.iterator这个key对应值是function，那对象就是可迭代的

迭代器协议：迭代过程就叫迭代器，返回next，无参数


```

### generator 也遵循迭代器协议

```js
authors[Symbol.iterator] = function* () {
  let allAuthors = this.allAuthors;
  let keys = Reflect.ownKeys(allAuthors);
  let values = [];

  while (1) {
    if (!values.length) {
      if (keys.length) {
        values = allAuthors[keys[0]];
        keys.shift();
        yield values.shift();
      } else {
        return false;
      }
    } else {
      yield values.shift();
    }
  }
};

let r = [];
for (let v of authors) {
  console.log(v);
  r.push(v);
}
```
