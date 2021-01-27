---
title:
  "[object Object]": null
date:
  "[object Object]": null
tags:
  - null
categories:
  - null
---

```js
import React, { useState } from "react";
// 引入编辑器组件
import BraftEditor from "braft-editor";
// 引入编辑器样式
import "braft-editor/dist/index.css";
const [editorState, setEditorState] = useState(
  BraftEditor.createEditorState(null)
);

const handleEditorChange = (editorState) => {
  setEditorState(editorState); // 编辑框改变获取更新改变值
};

// 在提交到后台的时候要把富文本转化成html代码给后台
const handleFinish = () => {
  // console.log(editorState.toHTML())
  modifyOne(props.match.params.id,{...values,coverImg:imageUrl,content:editorState.toHTML()).then(res=>{
    // 新增imageUrl,富文本内容---新增的时候也一样操作
    props.history.push('/admin/products')
  }).catch(err=>{
    console.log(err)
  })
};

<BraftEditor value={editorState} onChange={(e) => handleEditorChange(e)} />;


// 进来初始化读取数据的时候
useEffect(()=>{
  if(props.match.params.id){
    getOneById(props.match.params.id).then(res=>{
      setCurrentData(res)
      setImageUrl(res.coverImg) // 编辑进来设置imageUrl
      setEditorState(BraftEditor.createEditorState(res.content)) // ++++++++服务端返回的数据给富文本赋值
    })
  }
},[])

```
