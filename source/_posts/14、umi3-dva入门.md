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

### 搭建基础页面

```js
01、写样式、引入antd的Table
02、第一步：用户访问users，通过订阅者调用reducer返回仓库数据给页面（页面的state包含有model的namespace）
    ({users}) => ({users})

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

### 走订阅和 reducer

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

export default UserModel
```

### TS 加持

```js
umi对reducers，effets，subscribtions有默认的类型声明
固定值就直接可使用固定值
```

### 走订阅+effect+reducer+connect

### 异步处理完整流程数据流 service--->effects--->reducer

```js
页面监听，订阅以后分发effects，effects然后call（services）再put给reducers，reducer再最后返回
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
    // 这里的action没用到
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

### ****************************************************************走异步函数（加async）*******************************
// 第二步
// 现在模拟请求接口
services文件：
export const getRemoteList = async params => {
    const data = []
    return data
}
model文件中引入：
import {getRemoteList} from "./services"

reducers: {
    getList(state, { payload }) {   *******************解构一个action*********************************
      return payload;
    },
  },

effects: {
    *getRemote(action, { put, call }) {
      const data = yield call(getRemoteList)    ************************注意加yield************************************

      yield put({
        type: "getList",
        payload: {
            data // 要这样写，再去用到数据的地方改写
        }
      });
    },
  },

<Table dataSource={users.data} />

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
        payload:data // 这个data不是简单的数组对象结构
      });
    },
  },

```

### Dav:用来 model 自动启用

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

### 引入 Modal 子组件，通过属性的方式控制显示影藏（在父组件定义一个状态）

```js
    点击改变状态，传给子组件

    实现modal框的确定和取消按钮点击逻辑，因为状态在父组件那里，所以取消的回调定义在父组件里头

```

### 接下来的流程

```js
01、拿到数据给Table赋值
02、修改弹窗显示内容结构（表单）
03、给表单搞默认值(三步)

  ###  ***********************父组件点击编辑把单条数据rowData通过属性传递给子组件**************************************
  const [form] = Form.useForm();

  useEffect(()=>{ // 在渲染的时候不要赋值，在页面挂载之后才赋值
    form.setFieldsValue(rowData); // const [rowData,setRowData] = useState({}) 点击编辑的时候保存的一行值 // 给表单赋值初始值、可更新
    // initialValues:只在初始化的时候有效，不能更新

    // rowData是一个对象，属性为Form.Item对应的name属性
  },[visible])

  <Form form={form}></Form>

注意：
01、Modal中用Form报警告问题：forceRender
```

### \***\*\*\*\*\***\*\*\*\*\***\*\*\*\*\***点击 Modal 的 ok 来触发表单的提交\***\*\*\*\*\***\*\*\*\*\***\*\*\*\*\***

```js
点击提交onOk，触发表单的onFinish，修改仓库数据（dispatch(从state中解构出来)，注意action的type写法），然后call（接口，注意传参）再和后端接口交流
onFinish的形参默认接收表单的name属性对应的值

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
  return request(`http://public-api-v1.aspirantzhang.com/users/${id}`,{
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

### 处理完修改后，应该调用**\*\*\*\***\*\*\***\*\*\*\***异步的 effect 请求列表数据\***\*\*\*\*\*\*\***\*\***\*\*\*\*\*\*\***

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

        // 顺便清空表单
        setRowData(undefined)
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
    if(rowData === undefined){ // 点击添加按钮的时候，设置的undefined。setRowData(undefined)
        form.resetFields();
    }else{
        form.setFieldsValue(rowData)
    } // 如果是undefined就清空，否则赋值
},[visible])

注意：给表单赋值form.setFieldsValue(rowData)，如果rowData为无效值（undefined），则表单赋值不生效
```

### http 状态错误处理及提示优化（umi-request）

```js
import request, { extend } from "umi-request"

const errorHandler = function (error) {
  const codeMap = {
    "021": "An error has occurred",
    "022": "It’s a big mistake,",
  }
  if (error.response) {
    console.log(error.response.status) // 状态码
    console.log(error.response.headers) // 头部
    console.log(error.data) // 主体内容
    console.log(error.request) // 请求相关的信息（例如网址）
    console.log(codeMap[error.data.status]) // 相应状态码的提示信息

    // 处理逻辑
    if(error.response.status > 400){
      message.error(error.data.message?error.data.message:error.data)
    }
  } else {
    // 请求发送了，但是没有收到回复
    // console.log(error.message)
    message.error("Network Error")
  }
  // throw error // If throw. The error will continue to be thrown. // 上面做了错误处理，再抛出错误，会再抛，友好提示已经做了，就无须再抛了
}
// 1. Unified processing
const extendRequest = extend({ errorHandler })

********** 把下面的request换成extendRequest ******************
修改显示逻辑
export const getRemoteList = async params => {
  return extendRequest('http://public-api-v1.aspiranzhang.com/users',{
    method:'get',
  }).then(res=>{
    return res // 有结果的返回结果
  }).catch(err=>{
    return false // 否则返回布尔值
  })
}

增加、修改、删除无实际返回结果，改为
.then(res=>{
    return true // 操作成功
  }).catch(err=>{
    return false // 操作失败
  })

****************** 现在service返回布尔值给model *****************
所以如果返回的是值，布尔判断也为true
umi-request先做了一层拦截，service返回数据或者布尔值给到model，model中给出友好提示
effets:{
  *getRemote(action,{put,call}){
    const data = yield call(getRemoteList);
    if(data){ // 这样就保证了reducers的返回值不是undefined
      yield put({
        type:"getList",
        payload:data
      })
    }
  }，
  *edit({payload:{id,values}},{put,call}){
    const data = yield call(editRecord,{id,values});
    if(data){
      message.success('Edit,success.')
      yield put({
        type:'getRemote'
      })
    }else{
      message.error('Edit failed.')
    }
  }
}
```

```js
service与后端打交道，所以错误处理在request这里，
```

### 列表显示加载中(loading)

```js
loading是在state里面的
const mapStateToProps = ({ users, loading }) => {
  console.log(loading)
  return {
    users,
    userListLoading: loading.models.users,
  }
}

使用：
<Table loading={userListLoading} />
```

### 使用 ts 进行加持

```js
interface UserModelType {
  state: UserState;
  reducers: {
    getList: Reducer<UserState>,
  };
}

interface UserState {
  data: SingleUserType[];
  meta: {
    total: number,
    per_page: number,
    page: number,
  };
}

interface SingleUserType {
  id: number;
  name: string;
  email: string;
  create_time: string;
  update_time: string;
  status: number;
}

const UserModel: UserModelType = {
  state: {
    data: [],
    meta: {
      total: 0,
      per_page: 5,
      page: 1,
    },
  },
};
```

```js
01、
import { Dispatch,Loading,UserState } from "umi"
// umi里面可以导出在model里面定义暴露的接口
interface UserListPage {
  // users:[];
  users:UserState;
  dispatch:Dispatch;
  userListLoading:boolean
}
const mapStateToProps = ({ users, loading }:{users:*** UserState ***,loading:Loading}) => {
  console.log(loading)
  return {
    users,
    userListLoading: loading.models.users,
  }
}

02、
import { FC } from "react"
import { SingleUserType } from "data.d"

interface FormValues {
  [name: string]: any;
}

interface UserModalProps {
  visible: boolean;
  record: SingleUserType | undefined; // 因为在setRowData的时候赋值了undefined
  closeHandler: () => void; // 无参无返回值
  onFinish: (values: FormValues) => void;
}

****** 对于TS对props的写法 ******
// 01、泛型---约束props的写法
const UserModal: FC<UserModalProps> = (props) => {}

// 02、使用两个对象的形式写法
({}:{})
对于对象的写法（写两个对象）
{id,values}:{id:number,values:FormValues}

03、useState中定义ts（使用泛型）
const [record,setRecord] = useState<SingleUserType | undefined>(undefined)
```

### antd 的 Table 的封装 Pro-Table，request 获取数据，暂时先注释仓库里面走订阅获取数据的 dispatch

```js
01、引入，然后直接替换掉Table，写一个request属性获取数据
02、改造service接口，传入分页数据
export const getRemoteList = async ({page,per_pege})=>{
  return extendRequest(`http://public-api-v1.aspirantzhang.com/users?page=${page}&per_page=${per_page}`,{
    method:'get'
  })
}
在使用到getRemoteList的地方传入对应的参数就行
页面request获取数据写法
const requestHander = async ({pageSize,current})=>{
  const users = await getRemoteList({
    page:current,
    per_page:pageSize
  })
  if(users){
    return { // pro-table的表格数据
        data:users.data,
        success:true,
        total:users.meta.total
    }
  }else{
    return {
        data:[],
        success:true,
    }
  }
}

03、修改删除逻辑
在model怎么拿到仓库的数据呢？
select方法就可
*delete({payload:{id}},{put,call,select}){
  const data = yield call(deleteOneList,{id})

  // 成功之后更新页面
  if(data){
    const {page,per_page} = ***yield*** select(state=>state.***users***.meta)
    yield put({
        type:"getRemote",
        // 因为调用getRemote要用到page和per_page,所以多传递一个payload字段
        payload:{
          page,
          per_page
        }
    })
  }else{
    message.error("Delete Failed")
  }
}

*getRemote({payload:{page,per_page}},{put,call}){
  const data = yield call(getRemoteList,{page,per_page})
  if(data){
    yield put({ // 传递数据给页面
      type:"getList",
      payload:data
    })
  }
}
```

### 复用 pro-table 的 reload 方法

```js
01、引入interface
02、定义ref
03、赋值table：actionRef={ref}
04、点击reload的时候
const reloadHandler = ()=>{
  ref.current.reload()
}
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
05、******* reducers不允许返回undefined ******************
06、model中定义的接口interface，导出后，可以很神奇的在umi中导入进来
07、很多文件都要用的接口，可单独新建一个文件夹写接口data.d.ts,使用的时候导入,可以省略ts结尾的文件
08、对于太复杂的数据，判断类型直接使用any
09、表单提交失败的情况一般是校验不通过的时候触发onFinishFailed
10、pro-table的刷新是基于request方法来的，使用antd的Pagination的话，request和reload就失效了
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

### umi-request 请求流程：

```js
本地访问远程会发送两次请求
第一次options请求询问远程支持什么方式的请求
第二次才是真正的前后端交互

umi的请求主体直接是在error.data里面
umi-request则在json对象里面：error.data.message
```

### 本 demo 页面加载流程-思路逻辑

```js
约定式路由在进入某一个页面之后，订阅者会监听路径，触发相应的effects和reducers，得到数据之后返回给页面
在使用pro-table的时候，首先页面监听，然后触发pro-table的request属性，请求数据，但是reducers还在等异步结果
所以表格没数据，但是使用pro-table刷新一下就有了（又运行了一下request）

解决：pro-table直接使用异步接口返回数据（绕过仓库，直接使用接口）（仓库和页面各自走各自的）
```

```js
01、
隐藏查询：api
search属性置为false
02、
pro-table的分页是定死的，如果后端出错，分页显示调到对应页了，但数据是原来页的
如果改用antd的分页，则pro-table的request和reload也不能用了
pagination={false} 取消pro-table的默认分页，引入antd的Pagination，model处重新订阅getRemote获取数据
是第一次，给page和per_page默认值
if(pathname === "/users"){
  dispatch({
    type:"getRemote",
    payload:{
      page:1,
      per_page:5
    }
  })
}

<Pagination
  total={users.meta.total}
  onChange={paginationHandler}
  onShowSizeChange={paginationHandler}
  current={users.meta.page} // 通过后端来控制显示当前页（解决切换页码数据失败的问题）
  pageSize={users.meta.per_page} // 通过后端来显示默认显示多少条（解决第一页显示五条，第二页显示10条的问题）
/>

03、分页回调
const paginationHandler = (page: number, pageSize?: number) => {
  dispatch({
    type: 'users/getRemote',
    payload: {
      page,
      per_page: pageSize ? pageSize : users.meta.per_page,
    },
  });
};

const pageSizeHandler = (current: number, size: number) => {
  dispatch({
    type: 'users/getRemote',
    payload: {
      page: current,
      per_page: size,
    },
  });
};
```

### 大 Bug，错误处理，必须 throw error，后端接口才会捕捉 catch，然后 return 我们写的 false

```js
01、
优化：
编辑后台失败的时候，不要关闭弹窗
最好的效果是：点击提交后显示loading，成功后关闭，失败后不关闭弹框

const onFinish = async (values:FormValues) => {
  let id = 0;
  if(record){
    id = record.id
  }
  if(id){
    const result = await editRecord({id,values});
    if(result){
      // 编辑成功
      setIsShowModel(false)
    }else{
      message.error("Edit Failed")
    }
  }else{
    const result = await addRecord({values});
    if(result){
      setIsShowModel(false)
    }else{
      message.error("Add failed")
    }
  }
}

02、
// 简化代码
let serviceFun;
if(id){
  serviceFun = editRecord
}else{
  serviceFun = addRecord
}
const result = await serviceFun({id,values});
if(result){
  setIsShowModel(false)
  message.success(`${id===0?"Add":"Edit"} Successfully.`)

  dispatch({
    type:"getRemote",
    payload:{
      page:users.meta.page,
      per_page:users.meta.per_page
    }
  })
}else{
  message.error(`${id===0?"Add":"Edit"} Failed.`)
}

03、
// 点击提交的时候加一个loading状态
<Modal
  confirmLoading={confirmLoading}
/>
// 在父组件加一个state状态，并传递给子组件
const [confirmLoading,setConfirmLoading] = useState(false)
// 在点击提交的时候改成true，在成功或者失败的时候都要改成false，给使用者重新提交
const onFinish = async (values:FormValues) => {
  setConfirmLoading(true) // 加载中
  let id = 0;

  let serviceFun;
  if(id){
    serviceFun = editRecord
  }else{
    serviceFun = addRecord
  }
  const result = await serviceFun({id,values});
  if(result){
    setIsShowModel(false)
    message.success(`${id===0?"Add":"Edit"} Successfully.`)
    dispatch({
      type:"getRemote",
      payload:{
        page:users.meta.page,
        per_page:users.meta.per_page
      }
    })
    // 成功取消加载
    setConfirmLoading(false) // 加载中取消
  }else{
    // 失败取消加载
    setConfirmLoading(false) // 加载中取消
    message.error(`${id===0?"Add":"Edit"} Failed.`)
  }
}
```

### 重写实现 reload

```js
<ProTable
  options={{
    density: true,
    fullScreen: true,
    reload: () => {
      resetHandler();
    },
    setting: true,
  }}
  // 添加title
  headerTitle="User List"
  toolBarRender={() => [
    <Button type="primary" onClick={addHandler}>
      Add
    </Button>,
    <Button onClick={resetHandler}>Reload</Button>,
  ]}
/>;
const resetHandler = () => {
  刷新页面就是请求下仓库数据;
  dispatch({
    type: "users/getRemote",
    payload: {
      page: users.meta.page,
      per_page: users.meta.per_page,
    },
  });
};

现在可以删除以前写的通过ref实现的刷新功能;
```

### Form.Item 里面给 DatePicker 赋初始值，应该在给调单赋初始值的时候

```js
<DatePicker showTime />;

form.setFieldsValue({
  ...record,
  create_time: moment(record.create_time),
});
```

### Input、DatePicker 给 value，Switch 给 checked

### 优化

```js
调整表单显示布局;
const layout = {
  labelCol: { span: 4 },
  wrapperCol: { span: 20 },
};
<Form {...layout} />;
```

### Modal 的 title 动态显示编辑与添加

```js
title={ record ? "Edit ID:" + record.id : "Add" }
```

### 分页的 pageSize 是可选的、注意给默认值

```js
pageSize?:number

per_page: pageSize ? pageSize : users.meta.per_page
```

### 给表格列添加对应的字段类型（valueType）

```js
{
  title:"ID",
  dataIndex:'id',
  valueType:"digit",
  key:'id'
}
```

### 给表格的列 columns 添加 ts 类型声明

```js
const columns: ProColumns<SingleUserType>[] = [{}, {}];
```

### 复用代码

```js
添加或者编辑成功之后，要刷新列表
直接复用刷新表格的函数：
resetHandle()

```

### 点击添加的时候---给表单项的开关赋初始值

```js
<Form
  initialValues={{
    status: true,
  }}
/>
```

### 编辑的时候开关的显示开与否，转后端给的值为布尔值

```js
form.setFieldsValue({
  ...record,
  create_time: moment(record.create_time),
  status: Boolean(record.status),
  status: record.status === 1 ? true : false,
});
```

### 捋思路

```js
添加逻辑：
        点击添加，显示弹框，清空表单
        表单编辑好后、点击提交，根据全局的rowData判断有无id
        有，是编辑
        无，是添加，dispatch一个add的effect，带上type和payload
        编辑成功，关闭弹窗

        model逻辑：去写effect，去写service

        此时实现添加，但是有个bug（点编辑后取消编辑，去添加会遗留编辑的数据），在添加的时候，要清空表单（表单内容是从父级传递过来的）
        useEffect(()=>{
            if(rowData === undefined){ // 点击添加按钮的时候，设置的undefined。setRowData(undefined)
                form.resetFields();
            }else{
                form.setFieldsValue(rowData)  ******  如果rowData为无效值，表单赋值不生效 ******
            } // 如果是undefined就清空，否则赋值
        },[visible])

错误提示优化：
            service层有值的返回值，无值的返回布尔值给model
```

### useEffect 第二个参数如果是常量，则代表不会变

```js
useEffect(() => {}, [1]);
```

### 真正组件化

```js
01、每一部分都拆开成一个函数组件，然后调用:{name()}    ***************************************************************

02、{ variable && Object.keys(variable).length? (ReactNode) : null } // 这里面可以判断要不要渲染某个组件

03、partial<T> : 则T里面的属性代表可选，partial(可以直接引用)

04、分步骤是通过一个变量来判断是第几步，返回对应的reactNode

05、表单校验

// 文本域
<FormItem
  name="desc",
  label="规则描述",
  rules={[
    {
      requied:true,
      message:'请输入至少五个字符的规则描述！',
      min:5
    }
  ]}
>
  <TextArea rows={4} placeholder="请输入至少五个字符" />
</FormItem>

// 时间范围选择
<FormItem
  name="time",
  label="开始时间",
  rules={[
    {
      requied:true,
      message:'请选择开始时间！',
    }
  ]}
>
  <DatePicker style={{ width : '100%' }}
    showTime
    format="YYYY-MM-DD HH:mm:ss"
    placeholder="选择开始时间"
  />
</FormItem>

// 下拉框
<FormItem
  name="frequency",
  label="调度周期",
>
  <Select
    style={{width:'100%'}}
  >
    <Option value="month">月</Option>
    <Option value="week">周</Option>
  </Select>
</FormItem>
```

### 编辑和删除

```js
const editAndDelete = (key, currentItem) => {
  if (key === "edit") showEditModal(currentItem);
  else if (key === "delete") {
    Modal.confirm({
      title: "删除任务",
      content: "确定删除改任务吗？",
      okText: "确认",
      cancelText: "取消",
      onOk: () => deleteItem(currentItem.id),
    });
  }
};
```

### 更多 hover 显示菜单

```js
const MoreBtn = ({ item }) => (
  <Dropdown
    overlay={
      <Menu onClick={({ key }) => editAndDelete(key, item)}>
        <Menu.Item key="edit">编辑</Menu.Item>
        <Menu.Item key="delete">删除</Menu.Item>
      </Menu>
    }
  >
    <a>
      更多 <DownOutLined />
    </a>
  </Dropdown>
);
```

### 通知 reducers

```js
yield put({
  type:'queryList',
  payload:Array.isArrray(result)?result:[]
})
```

### 判断添加成功与否

```js
const handleAdd = async (fields) => {
  const hide = message.loading("正在添加");

  try {
    await addRule({
      desc: fields.desc,
    });
    hide();
    message.success("添加成功");
    return true;
  } catch (error) {
    hide();
    message.error("添加失败请重试！");
    return false;
  }
};
```

### @umijs/plugin-model 的使用

```js
src/models/zhang.js，里面的每个文件暴露一个***需要默认导出***一个函数

export default () => {
  let myName = "zhang"
  return { myName }
}

文件名则对应最终 model 的 name，你可以通过插件提供的 API 来消费 model 中的数据
useModel 是一个 Hook，提供消费 Model 的能力，使用示例如下
import { useModel } from 'umi';

const { myName } = useModel('zhang') //model的name就是文件名
其他地方即可使用***myName***
```

### 为啥要使用泛型

```js
做到对类型的灵活控制;
```

### 表单初始值，umi 路由跳转，插件

```js
01、
reducer一调用就代表仓库给页面返回值了;

02、
约定式路由（无路由配置）
（有订阅者先走订阅者（初始化仓库），protable的request不走）

不走订阅，走request，则仓库初始化不了
正常来说走仓库（例如删除成功之后可以直接通过select方法获取仓库数据）

03、刷新是基于request的，有request才有protable自带的刷新
分页也是基于request的，有request才有分页
```

### Form 的默认值只给表单的赋值是 value 的

```js
如果是checked，需要在Form.Item上声明valuePropName="checked",然后给表单赋初始值才会生效,

给表单在初始化的时候赋值：initialValues={{...:...}}
```
