---
title: 我的常用js函数
date: 2021.1.30
tags: js库
categories: js函数
---

#### 一、后台没做分页，前端分批显示

```js
list: [],    //存放数据容器
page: 0,    //第一条数据
number: 5    //第五条数据

// 页面默认加载前五条数据
axios.get('xxx')
.then((res) => {
  console.log(res);
  if(res.status == "200"){
    console.log(res.data.data);
    this.list = res.data.data.slice(this.page, this.number);
  }
})

// 点击按钮加载更多
onLoad(){
  //累加5条数据
  this.page = this.page + 5;
  this.number = this.number + 5;
  // 截取后返回的是一个数组对象
  axios.get('xxx')
  .then((res) => {
    console.log(res);
    if(res.status == "200"){
      let data = res.data.data.slice(this.page, this.number);
      console.log(data);
      data.forEach((item, index) => {
        //因此只需要遍历里面的对象 因为数组push不进数组 push对象到数组里即可
        console.log(item);
        this.list.push(item);
      })
      //咳咳 用push方法有点麻烦 竟然返回数组直接用concat()拼接就可以了 emmm... 而且在小程序里上拉加载push数据不会叠加数据~
    }
})

// 优化：可以一次性把请求回来的数据存起来，点击加载更多按钮的时候就不用重新请求了
```
