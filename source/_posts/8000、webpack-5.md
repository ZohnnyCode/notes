---
title: 8000、webpack(5)
date: 2021-04-27 17:09:20
tags:
---

#### 原生打包命令

```js
全局webpack打包
webpack

局部webpack打包
01、npx webpack
02、./node_modules/.bin/webpack
03、也可以通过package.json文件
scripts:{
    "build":"webpack"
}

npm run build // 局部如果没有，会自动找全局的
```
