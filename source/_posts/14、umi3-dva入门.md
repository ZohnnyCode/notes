---
title: 14、umi3+dva
date: 2021-02-25 11:16:48
tags:
---

### 基础

```js
01、箭头函数

02、析构赋值

const a = {
    name:"大帅比"
}
const {name} = a

const b = {
    name:"大帅比",
    age:18
}

const {name,...rest} = b
console.log(rest) // {age:18}

03、json转换
JSON.stringify() // 先json再变成字符串
JSON.parse()

04、异步相关
Promise.resolve()

function *function_name(){
    yeild 异步任务 // 等异步任务执行之后，才往下
}

async function function_name(params){
    await 异步动作
}
```

### DVA 介绍

```js
DVA = (redux、redux-saga、react-router)

UMI = DVA + 插件

UMI主要是路由的简化

```

```js
DVA：公共数据******model******（以前redux里面的store）
    同步数据（reducer）
    异步数据（effect）
        页面（dispatch分发）---> reducer（connect） --->页面
        页面（dispatch分发）---> effect（put(给)） --->页面

    订阅（subscription）（dva快捷方法）
        subscription（dispatch(分派)）--->effect

私有数据：state

页面 <---> effect(call(召唤)) <---> service
```

### 小例子文件结构

```js
users (tsx参与界面展示，有jsx语法)
    index.tsx
    model.ts
    service.ts // 里面都是异步函数
    components/UserModal.tsx

umi配置：.umirc.ts(默认有路由配置，所以不是约定式路由)

全局样式
Umi 中约定 src/global.css 为全局样式，如果存在此文件，会被自动引入到入口文件最前面。(写样式后需要重启)

```

### umi 跟 dva 结合

```js
01、
把静态的data放到dav的同步数据reducer里面的save里面并返回
组件使用的时候跟redux的写法一样，只不过connect是直接从umi里面导出

const mapStateToProps = ({users})=> {
    return {
        users
    }
}

table的dataSource可以通过props拿到users

此处用户访问uses的时候，subscription监听到路由，dispatch一个reducer，返回数据的名称就是namespace对应的字段，就可以拿到数据了
所以我们的页面通过connect方式绑定到了仓库，这就实现了umi和dva的结合
```

```js
02、dva本质写法
const UserModel = {
    namespace:'users',
    state:{},
    reducers:{
        getList(state,action){
            return newState;
        }
    },
    effects:{
        *function_name(action,effects){
            yeild put()
        }
    },
    subscriptions:{
        function_name({dispatch ,history},done){}
    }
}

effect无需返回值

dispatch(action)
action = {
    type:"getList",
    payLoad:{}
}
```

```js
03、手写model  +  ts

import { Reducer,Effect,Subscription } from "umi"

interface UserModelType {
    namespace:'users';
    state:{};
    reducers:{
        getList:Reducer
    };
    effects:{};
    subscriptions:{
        setup:Subscription
    }
}

const UserModel:UserModelType = {
    namespace:'users',
    state:{},
    reducers:{
        getList(state,action){
            const data = []
            return data
        }
    },
    effects:{},
    subscriptions:{
        setup({dispatch,history}){
          return history.listen(({pathname})=>{
            if(pathname === '/users'){
              dispatch({
                type:'getList'
              })
            }
          })
        }
    }
}

exports default UserModel
```

### TS 加持

```js
umi对reducers，effets，subscribtions有默认的类型声明
固定值就直接可使用固定值
```

### 异步处理完整流程数据流 service--->effects--->reducer

```js
页面监听，订阅以后分发effects，effects再put给reducers，reducer再最后返回
// 第一步
interface UserModelType {
  effects: {
    getRemote: Effect,
  };
}

const UserModel: UserModelType = {
  namespace: "users",
  state: {},
  reducers: {
    getList(state, { payload }) {
      return payload;
    },
  },
  effects: {
    *getRemote(action, { put, call }) { // effects找reducers用put，找service用call
      const data = [];

      yield put({
        type: "getList",
        payload: data,
      });
    },
  },
  subscriptions:{
        setup({dispatch,history}){
          return history.listen(({pathname})=>{
            if(pathname === '/users'){
              dispatch({
                // type:'getList' // 不是同步的了
                type:'getRemote' // 异步
              })
            }
          })
        }
    }
};

现在谁来触发effects呢，把订阅subscriptions中的dispatch中的type改成effects中的函数名即可

// 第二步
// 现在模拟请求接口
services文件：
export const getRemoteList = async params => {
    const data = []
    return data
}
model文件中引入：
import {getRemoteList} from "./services"

effects: {
    *getRemote(action, { put, call }) {
      const data = yield call(getRemoteList)

      yield put({
        type: "getList",
        payload: {
            data // 要这样写，再去用到数据的地方改写
        }
      });
    },
  },

// 第三步：真正调用后端接口
import { request } from "umi"

export const getRemoteList = async params =>{
    // return 记住return
    return request("http://public-api-v1.aspirantzhang.com/users",{
        methods:'get'
    }).then(function(response){
        return response
    }).catch(function(error){
        console.log(error)
    })
}

effects: {
    *getRemote(action, { put, call }) {
      const data = yield call(getRemoteList)

      yield put({
        type: "getList",
        payload:data
      });
    },
  },

```

### 代理,只在开发的时候好使

```js
.umirc.ts
proxy: {
    '/api': {
      'target': 'http://jsonplaceholder.typicode.com/',
      'changeOrigin': true,
      'pathRewrite': { '^/api' : '' },
    },

return request("/api/users",{
        methods:'get'
    }).then(function(response){
        return response
    }).catch(function(error){
        console.log(error)
    })
```

### umi 的一些配置

```js
01、使用dva默认配置开启，一旦用来model则就开启dva插件了
02、request请求插件也是默认开启的
```

### 奇怪的点

```js
01、
return res.data

02、
*getRemoteList(action,{put,call}){
    const data = yield call(getRemoteListData)
    console.log(data,"返回结果");

    yield put({
        type:'getList',
        payLoad:{data} // 得这样写
    })
}

03、
reducers:{
    getList(state,{payLoad}){
        console.log(payLoad,"555");

        return payLoad // 此处的payLoad返回一个对象
    }
},
```

### 接下来的流程

```js
01、拿到数据给Table赋值
02、修改弹窗显示内容结构（表单）
03、给表单搞默认值(三步)
  const [form] = Form.useForm();

  useEffect(()=>{
    form.setFieldsValue(rowData); // const [rowData,setRowData] = useState({}) 点击编辑的时候保存的一行值 // 给表单赋值初始值
  },[visible])

  <Form form={form}></Form>

注意：
01、Modal中用Form报警告问题：forceRender
```

### 处理修改逻辑

```js
01、点击编辑：modal弹框，点击modal的ok按钮，要触发表单的提交（onFinish方法需要放在父级页面中调用，通过属性传递给子组件，因为子组件不能和仓库进行沟通）
    const onOk = () => {
        form.submit();
    };
该方法一调用就会触发form表单的onFinish方法
const onFinish = (values: any) => {
    console.log('Finish:', values);

    const id = rowData.id
    // 这里把修改数据提交到仓库，仓库再调用service中的接口改变仓库数据，最后通过reducer把新数据给到页面
    // 需要用到dispatch（从props里面来）
    dispatch({
        type:'users/edit', // 页面写法一定要这样子写
        payload:{
            values,
            id // 把id传到model中
        }
    })

    // 修改成功之后关闭弹窗
    setIsShowModal(false);
};

02、去写service接口
export const edit = async ( {id,values} ) =>{
  return request(`http://public-api-v1.aspirantzhang.com/users${id}`,{
    method:'put',
    data:values // ******************* 注意valus本来就是对象了 *********************
  }).then(res=>{
    console.log("ok"); // 通过状态码来判断有无添加成功
  }).catch(err=>{
    console.log(err);
  })
}

03、effects:{
    // { payload: { id,values } } 双重解构，action和payload
    *edit({payload:{id,values}},{put,call}){
        // console.log('edit here')
        const data = yield call(edit,{id,values}) // ************* 注意call的传参方法 *************
    }
}
```

### 处理删除逻辑

```js
01、
    // 确认删除
  const confirm = (id) => {
    props.dispatch({
      type:"users/delete",
      payload:{id}
    })
  }
02、
    // 删除一条数据
      *delete({payload:{id}},{put,call}){
        const data = yield call(deleteOneList,{id})

        // 成功之后更新页面
        yield put({
            type:"getRemote"
        })

      }
03、
export const deleteOneList = async ({id}) =>{
  return request(`http://public-api-v1.aspirantzhang.com/users/${id}`,{
    method:'delete',
  }).then(res=>{
    console.log("ok");

    // 删除成功之后给提示
    message.success("Delete successfully")

  }).catch(err=>{
    console.log(err);

    message.error("Delete failed")
  })
}

```

### 处理添加逻辑

```js
01、添加一个按钮，点击打开弹窗
    <Button type="primary" onClick={()=>{
        setIsShowModal(true)
    }}>Add</Button>

02、修改和添加都是用的同一个回调函数（onOk触发表单的submit），修改有id，而添加没有
    const onFinish = (values: any) => {

        let id = 0
        if(rowData){
            id = rowData.id // rowData的setRowData的默认值为undefined
        }

        if(id){
            console.log("edit)
            dispatch({
                type:'users/edit', // 页面写法一定要这样子写
                payload:{
                    values,
                    id // 把id传到model中
                }
            })
        }else{
            console.log("add")
            dispatch({
                type:"users/add",
                payload:{
                    values
                }
            })
        }
        // 修改成功之后关闭弹窗
        setIsShowModal(false);
    };

03、
effects:{
    *add({payload:{values}},{put,call}){
        const data = yield call(addOneList,{values})
        yield put({
            type:'getRemote'
        })
    }
}

04、
export const addOneList = async ({values}) =>{
  return request(`http://public-api-v1.aspirantzhang.com/users`,{
    method:'post',
    data:values
  }).then(res=>{
    console.log("ok");
    // 添加成功之后给提示
    message.success("Add successfully")

  }).catch(err=>{
    console.log(err);
    message.error("Add failed")
  })
}

注意：现在有一个bug：点击编辑，取消后，点添加，表单的数据会遗留下来
当初给表单赋值的时候是在，弹框显示与否的副作用里面
当点击编辑的时候，保存这一行的值，并传到子组件里面，在弹框显示与否中给表单付初始值

useEffect(()=>{
    if(rowData === undefined){
        form.resetFields();
    }else{
        form.setFieldsValue(rowData)
    }
},[visible])

```

### 小技巧

```js
查看文件夹结构;
tree / f;
网站：ctrl+home回到顶部
```

### antd 的 Table 默认会找 dataSource 里面的 key 属性值当做自己的 rowKey

```js
01、Table必须要有唯一的rowKey值
02、Modal中的Form控制台报错，强制预渲染Modal
03、react生命周期render是不能有setState更新
04、在model同一个中的subscribtion订阅中dispatch的时候，type可以直接写effects或者reducers中的函数名字，但在页面和仓库沟通时候的dispatch的type要加上namespace："users/edit"
```

### useState 这个 hook 在 set 之后的**_下一步_**怎么获取到最新状态

```js
01、直接不使用set更新状态
02、利用useRef
03、结合useEffect

具体写法有待研究
Table的某一行可以直接拿到这一样的数据啊

你都已经可以set了，说明可以直接用了啊，就可以不参与状态了

修改成功之后关闭弹窗，刷新的话就是重新调用一下列表，删除也同理
```
