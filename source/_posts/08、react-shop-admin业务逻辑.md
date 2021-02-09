---
title: 业务逻辑
date: 2021-01-25 23:26:00
tags: 业务逻辑
---

#### 表单验证通过之后：

```js
if (!err) {
  // 调用api接口
} else {
  message.error("请输入正确的内容");
}
```

#### 某个元素居中对齐

```js
.login-form {
  width:400px;
  position:relative;
  left:50%;
  top:50%;
  tramsform:translate3d(-50%,-50%,0)
}
```

#### 登录逻辑

```js
当用户点击登录之后，肯定是要调服务端接口的，判断是否登录成功，成功则把对应的token存到本地，后面根据token值判断用户是否登录
utils.js
  auth.js
    // 获取token
    export function getToken(){
      return localStorage.getItem('token')
    }
    // 设置token
    export function setToken(token){
      localStorage.setItem('token',token)
    }
    // 退出登录清楚token
    export function clearToken(){
      localStorage.removeItem('token')
    }
    // 判断是否登录
    export function isLogin(){
      if(localStorage.getItem('token')){
        return true
      }
      return false
    }



在所有访问'/admin'的时候，先判断登录与否
isLogin()?<Frame>
            <Switch>{adminRoutes.map(route=>{
              return <Route
              key={route.path}
              path={route.path}
              exact={route.exact}
              render={routeProps=>{
                return <route.component {...routeProps}/>
              }}
            >
            })}
            <Redirect to={adminRoutes[0].path} from='/admin' />
            <Redirect to='/404'>
            </Switch>
          </Frame>:<Redirect to='/login'>

点击登录简单判断
if(!err){
  setToken(values.username)
  props.history.push('/admin') // 登录成功跳转到管理后台首页
}
实操：
if(!err){
  loginApi({
    userName:value.username,
    password:value.password
  }).then(res=>{
    if(res.code === 'success'){
      message.success('登录成功');
      setToken(res.token);
      props.history.push('/admin')
    }else{
      message.info(res.message)
    }
  }).catch(err=>{
    message.error('用户不存在')
  })
}
```

#### 管理后台头像处理

```js
const popMenu = {
  <Menu onClick={p=>{
    if(p.key==='logOut'){
      clearToken();
      props.history.push('/login')
    }else{
      message.info(p.key)
    }
  }}>
    <Menu.Item key='notice'>通知中心</Menu.Item>
    <Menu.Item key='setting'>设置</Menu.Item>
    <Menu.Item key='logOut'>退出</Menu.Item>
  </Menu>
}

DOM:
<div>
  <Dropdown overlay={popMenu}>
    <Avatar>U</Avatar>
    <span style={{}}>超级管理员</span>
    <Icon type='down'>
  </Dropdown>
</div>
```

#### 管理后台所有接口调用都要传递一个 token 值，在请求头里面

```js
token在用户第一次登录的时候会返回给客户端，验证用户是否有权限访问我们接口数据
```

#### 封装请求

```js
request.js;
// 请求拦截器拦截请求增加请求头
config.headers["authorization"] = "Bearer " + getToken();

export function get(url, params) {
  return axios.get(url, {
    params,
  });
}
export function post(url, data) {
  return axios.post(url, data);
}
export function put(url, data) {
  return axios.put(url, data);
}
export function del(url) {
  return axios.delete(url);
}

service.js;
// 登录接口
export function loginApi(user) {
  return post("/路径", user);
}

// 商品接口
export function listApi(page = 1) {
  return get("/", { page, per: 2 }); // 每页显示两条
}
export function createApi(data) {
  return post("/", data);
}
export function modifyOne(id, data) {
  return put(`/${id}`, data);
}
export function deleteOne(id) {
  return del(`/${id}`);
}
```

#### 表格数据处理

```js
{
  title:'序号',
  key:'_id',// 表格返回的数据的唯一值
  wigth:80,
  align:'center',
  render:(txt,record,index)=>index+1
}

rowKey='_id'

const loadData = page=>{

}

<Table
  pagination={{
    total,
    defaultPageSize:2 // 默认10条(total跟pageSize结合就可以实现分页效果了)
    onChange:loadData // 点击分页
  }}
/>
```

#### 一般表单校验并提交逻辑

```js
// 接口在提交成功后，可以做路由跳转
// 新增
createApi(values).then((res) => {
  console.log(res);
  props.history.push("/admin/products");
});

// 编辑
export function getOneById(id) {
  return get(`/${id}`);
}

// 列表页
<Button
  onClick={() => {
    props.history.push(`/admin/products/edit/${record._id}`);
  }}
>
  修改
</Button>;

// 编辑页
props.match.params.id拿路由参数;存在的话表示编辑，否则表示新增

useEffect(()=>{
  if(props.match.params.id){
    getOneById(props.match.params.id).then(res=>{
      setCurrentData(res)
    })
  }
},[])
拿到值（currentData）之后设置表单的默认值

// 表单验证成功之后，判断是新增还是修改然后调用不同的接口
values为表单验证通过之后的对象数据

if(props.match.params.id){
  modifyOne(props.match.params.id,values).then(res=>{
    props.history.push('/admin/products')
  }).catch(err=>{
    console.log(err)
  })
}else{
  createApi(values).then((res) => {
    console.log(res);
    props.history.push("/admin/products");
  }).catch(err=>{
    console.log(err)
  })
}
```

#### 删除列表页的某一行

```js
<Popconfirm
  title="确定删除？"
  onCancel={() => console.log("取消删除")}
  onConfirm={() => {
    delOne(record._id).then((res) => {
      // 删除成功之后重新加载数据
      loadData(1);
    });
  }}
>
  <Button type="danger">删除</Button>
</Popconfirm>
<Buuton
  onClick={()=>{
    // 修改后台数据的在售状态
    modifyOne(record._id,{onSale:!record.onSale}).thne(res=>{
      loadData(1)
    })
  }}
>{record.onSale?'下架':'上架'}</Buuton>
```

#### 表格一行的 className

```js
<Table rowClassName={(record) => (record.onSale ? "" : "row-bg")} />
```

#### 解决在点击 ---第二页页码---页码显示第二页 但是数据是第一页的 bug

```js
const [currentPage,setCurrentPage] = useState(1) // 保存页的状态

点击页码加载数据的时候，保存一下当前的页码（setCurrentPage）
在哪一页删除，重新加载数据的时候加载currentPage页的数据
在哪一页上下架，加载哪一页currentPage页的数据
```

#### 各文件都要用到的变量

```js
utils > cnfig.js;
export const serverUrl = "http://localhost:3009";
```

#### 图片上传功能(最后显示)

```js
const [imageUrl, setImageUrl] = useState("");
const [loading, setLoading] = useState(false);

const uploadButton = (
  <div>
    {loading ? <LoadingOutlined /> : <PlusOutlined />}
    <div style={{ marginTop: 8 }}>Upload</div>
  </div>
);

handleChange = info => {
  if (info.file.status === 'uploading') {
    // 上传中
    setLoading(true)
    return;
  }
  if (info.file.status === 'done') {
    // 上传成功
    setLoading(false)
    setImageUrl(info.file.response.info) // '/uploads/20200219/13612261897.jpg'
  }
};

<Upload
  name="file" // 后台给的字段
  listType="picture-card"
  className="avatar-uploader"
  showUploadList={false}
  action={serverUrl + '/api/v1/common/file_upload'} // 上传目标的服务器地址
  onChange={(info) => handleChange(info)} // info 上传过程的信息
>
  {imageUrl ? <img src={serverUrl+imageUrl} alt="picture" style={{width:100%}}/> : uploadButton}
                        服务器路劲加上传成功之后图片的路径
</Upload>;
```

#### 图片数据保存起来

```js
// 点击新建的时候传给后台
createApi({ ...values, coverImg: imageUrl })
  .then((res) => {
    console.log(res);
    props.history.push("/admin/products");
  })
  .catch((err) => {
    console.log(err);
  });

// 在列表页面新增一列显示图片
{
  title:'主图',
  dataIndex:'coverImg',
  render:(text,record)=>{
    return record.coverImg?<img src={serverUrl + record.coverImg} alt={record.name} style={{width:'120px'}}/>:'暂未上传'
  }
}
```

#### 点击编辑的时候图片显示出来

```js
在编辑页加载数据的时候，保存一下图片

useEffect(()=>{
  if(props.match.params.id){
    getOneById(props.match.params.id).then(res=>{
      setCurrentData(res)
      setImageUrl(res.coverImg) // 编辑进来设置imageUrl
    })
  }
},[])

在编辑---保存的时候重新提交更改的图片数据
modifyOne(props.match.params.id,{...values,coverImg:imageUrl).then(res=>{ // 新增imageUrl
    props.history.push('/admin/products')
  }).catch(err=>{
    console.log(err)
  })
```

#### 新增通知中心

```js
// 配置路由文件
{
  path:'/admin/notice',
  component:Notice,
  isShow:false
}

const popMenu = {
  <Menu onClick={p=>{
    if(p.key==='logOut'){
      clearToken();
      props.history.push('/login')
    }else{
      // message.info(p.key)
      if(p.key==='notice'){
        props.history.push('/admin/notice') // +++点击跳转
      }
    }
  }}>
    <Menu.Item key='notice'>通知中心</Menu.Item>
    <Menu.Item key='setting'>设置</Menu.Item>
    <Menu.Item key='logOut'>退出</Menu.Item>
  </Menu>
}

<Card title="通知中心" extra={<Button>全部已读</Button>}>
  <List
    bordered
    dataSource={data}
    renderItem={item=>(
      <List.Item>
        {item}
        <Button>已读</Button>
      </List.Item>
    )}
  ></>
</Card>

// 有未读信息，在昵称后面显示小红点
<Dropdown overlay={popMenu}>
  <Avatar>U</Avatar>
  <Badge dot>
    <span style={{color:'#fff'}}>超级管理员</span>
  </Badge>
  <Icon type='down'>
</Dropdown>
```

#### 跨组件传参

```js
redux, react - redux, redux - thunk;

store.js;
import { createStore } from "redux";
const noticeReducer = function (state = { isAllRead: false }, action) {
  switch (action.type) {
    case "ALL_READ":
      return { ...state, isAllRead: true };
    default:
      return state;
  }
};
const store = createStore(noticeReducer);
export default store;

index.js;
import { Provider } from "react-redux"; // 给整个应用提供仓库
<Provider store={store}></Provider>;

// 改变状态组件
import { connect } from "react-redux";
<Button onClick={() => props.dispatch({ type: "ALL_READ" })}>全部已读</Button>;
export default connect((state) => state)(Notice);

// 使用状态组件
import { connect } from "react-redux";
dot={!props.isAllRead} // 小红点的显示与否判断
const mapStateToProps = (state) => state;
export default connect(mapStateToProps)(withRouter(Frame));
```
