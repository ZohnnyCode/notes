---
title: 15、项目经验积累
date: 2021-02-02 10:11:30
tags:
---

#### 第一个：concat 不改变原数组，打印数组仍为原数组

#### 第二：代码逻辑

```js
①：对于很多组件要用到的数据，不管是路由组件还是普通组件，把请求数据的操作放在顶层（例如：App.js）中，把请求的数据存入store里面
其他组件要用到的时候（例如要用到这异步请求返回的数据在组件渲染时其他异步请求的参数），如果不做判断，会报undefined的错误
此异步请求必须要在上一次异步请求完成之后才能触发，可在useEffect中把上一次请求的结果当做依赖，里面的副作用加if判断，有值的时候才执行

依赖变化（变量变化）：依赖它的副作用就会执行
```

#### 第三：打包忽略 console.log

```js
plugin: [
  isEnvProduction
    ? new UglifyJsPlugin({
        uglifyOptions: {
          compress: {
            warnings: false,
            drop_console: true, // 不打印console
            pure_funcs: ["console.log"], // 移除console
          },
        },
        // sourceMap: config.build.productionSourceMap,
        // sourceMap: isEnvProduction ? shouldUseSourceMap : isEnvDevelopment,
        // parallel: true,
      })
    : undefined,
];
```

#### 第四：打包后在 scripts 文件夹中的 build.js 中区分测试环境跟生产环境

```js
process.env.BABEL_ENV = "development/production";
process.env.NODE_ENV = "development/production";
```

#### 第二个：类似分页，点击导出分批次请求数据

```js
let arr = [1, 2, 3, 4, 5]; // arr=[]
let x = 0; // x=1
function ajax(str) {
  return new Promise((res, rej) => {
    setTimeout(() => {
      console.log(str);
      res();
    }, 1000);
  });
}

function loopArray(fun) {
  fun().then(() => {
    x++;
    if (x < arr.length) {
      loopArray(ajax, arr3[x]);
    }
  });
}

loopArray(ajax, arr3[0]);

实例：
const dataAsync = () => {
    return coachActiveDataApi({
    // searchValue:
    // searchKey:
    companyId: 3006,
    startDate: defaultDays[0].format("YYYY-MM-DD"),
    endDate: defaultDays[1].format("YYYY-MM-DD"),
    currentPage,
    pageSize,
    });
};

let total = coachAsPeopleActiveData.length;
let speed = Math.floor(total / 2);

let resData = [];
let x = 1;
const loopData = (fun) => {
    fun().then((res) => {
    console.log(res, 23333333);
    x++;
    setCurrentPage(x);
    setPageSize(2);
    resData = resData.concat(res.data);
    if (x < speed) {
        loopData(dataAsync);
    }
    console.log(resData);
    });
};

loopData(dataAsync);
```
