---
title: react的redux
date: 2021.1.30
tags: react
categories: react
---

```js
store
  一、index.js
    import {createStore,combineReducers,compose,applyMiddleware} from 'redux'
    import noticeReducer from './reducers/noticeReducer'
    import productsReducer from './reducers/productsReducer'
    const rootReducer = combineReducers({
      noticeReducer,
      productsReducer
    })
    export default createStore(rootReducer,compose(applyMiddleware(...[thunk])))

  二、reducers
    noticeReducer.js
    export default const noticeReducer = function (state = { isAllRead: false }, action) {
      switch (action.type) {
        case "ALL_READ":
          return { ...state, isAllRead: true };
        default:
          return state;
      }
    };
    productsReducer.js
    export default const productsReducer = function(state={list:[],page:1,total:0},action){
      switch(action.type){
        case 'PRODUCT_LOADED':
          return {
            ...state,
            list:action.data.products,
            page:action.data.page,  // 新增一个page属性
            total:action.data.totalCount}
      }
    }

  三、actions
    products.js
      export const loadProduct = payload=> async dispatch=>{  // payload是异步action的参数
        const res = await listApi( payload.page ) // 这个接口默认需要页码
        // 当异步操作完成之后通过dispatch触发reducer改变数据
        dispatch({
          type:'PRODUCT_LOADED',
          data:{
            ...res,
            page:payload.page
          }
        })
      }

// 在使用仓库数据的地方要注意mapStateToProps返回的值
const mapStateToProps = state=>state.noticeReducer
export default connect(mapStateToProps)(Frame)
```

#### 在列表页

```js
import { connect } from "react-redux";
// 导入异步action
import { loadProduct } from "路径";

useEffect(() => {
  props.dispatch(
    loadProduct({
      page: 1, // 第一次默认为1
    })
  );
}, []);

// 只映射productsReducer里面的数据
export default connect((state) => state.productsReducer)(List);

// 数据加载完之后从仓库中取数据
const { list, page, total } = props;

// 把以前的listApi().then().catch()代码删掉
// 点击分页的loadData也删掉
const loadData = () => {
  props.dispatch(
    loadProduct({
      page: page, // 点击删除或者上下架的时候传递仓库里面的页码------------具体位置直接吊loadData就好了，不需要在传递参数了
    })
  );

  ---listApi().then((res) => {
  ---  setDataSource(res.products);
  ---  setTotal(res.totalCount);
  ---  setCurrentPage(page);
  ---});
};

<Table
  rowKey='_id'
  pagination={{total,defaultPageSize:2,onChange:(p)=>{
    props.dispatch(
      loadProduct({
        page: p, // 点击页码更新仓库里面的页码
      })
    );
  }}}
  columns={columns}
  bordered
  dataSource={list}
/>
```
