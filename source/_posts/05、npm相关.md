---
title: 05、npm相关
date: 2021-02-02 09:50:11
tags:
---

> 开发依赖：-S === --save
> 生产依赖：-D === --save-dev
> 全局安装：-g
> 删除安装版本的描述符(只安装单一版本)：-S -E
> 查看当前版本的相关信息：npm info package
> 查看历史发布的所有版本：npm info package versions

> package 描述符解读

```js
^: 指定安装大版本(第一个数字)
~:指定安装小版本(第二个数字) --->这个得自己手动加

```

### 记录安装 node-sass

```js
01、没有python装python
02、连接超时挂代理
    set https_proxy=http://127.0.0.1:12639
    set http_proxy=http://127.0.0.1:12639

03、Visual C++
    npm install --global --production windows-build-tools
```
