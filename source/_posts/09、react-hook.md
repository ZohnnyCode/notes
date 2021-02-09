---
title: React的新特性之：hooks
date: now

tags:
  - React
categories:
  - React
---

> 类组件写法

```js
state = {
  count:0
}
componentDidMount(){
  this.timer = setInterval(()=>{
    this.setState({
      count:this.state.count + 1
    },1000)
  })
}

componentWillMount(){
  if(this.timer){
    clearInterval(this.timer)
  }
}

render(){
  return <span>{this.state.count}</span>
}

```

> 函数组件写法

```js
function FunC() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <span>{count}</span>;
}

注意：setCount(1) // 不基于最新的状态，小心闭包陷阱
      setCount(c=>c+1)  // 是基于最新的状态
```

> useReducer 的使用

```js
定义一个reducer：
function countReducer(state,action){
  switch (action.type){
    case 'ADD':
      return state+1
      // return Object.assign({}) // state比较复杂的时候
    case 'MINUS':
      return state-1
    default:
      return state
  }
}

function FunC() {
  const [count, dispatchCount] = useReducer(countReducer,0);

  useEffect(() => {
    const timer = setInterval(() => {
      dispatchCount({type:"ADD"})
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <span>{count}</span>;
}
```

> Effect hook 的使用

```js
const [name,setNane] = useState('wiking')

useEffect(() => {
  console.log("effect invoked");
  return () => {
    console.log("effect deteched");
  };
});

return (
  <div>
    <input value={name} onChange={e=>setName(e.target.value)}  />
    <button onClick={()=> dispatchCount({type:'add'})}>{count}</button>
  </div>
)

注意：组件的执行顺序是，第一次执行先执行invoked，以后每次更新先执行卸载返回deteched的回调，再执行invoked回调
官方使用建议：invoked回调里面使用了外部依赖的时候，都建议传入第二个参数数组里面，数组为空则只执行一次

useEffect的兄弟(少用)：useLayoutEffect():用法和useEffect一样，区别在于，useLayoutEffect会在dom挂载之前执行，uesEffect则会在dom挂载完毕之后执行
```

> useContext 使用

```js
lib:my-context.js
import React from 'react
export default React.createContext('') // 参数可不传，在外部引入了两次

在App.js引入：
import MyContext from '/lib/my-context'

使用它来提供整个应用的context
<MyContext.Provider value='text'>
  <App />
</MyContext.Provider>

子组件使用
import React,{useContext} from 'react'
import MyContext from '/lib/my-context'

const context = useContext(MyContext) // context获取的直接就是value

return <p>{context}</p>

注意当传递的value更新后使用了useContext的组件才会自动更新，视图重新渲染
```

> useRef 的使用（获取 dom 对象）

```js
类组件：
<span ref='abc' ></spam>
this.refs.abc = span // 此ref17版本要删

constructor(){
  super()
  this.ref = React.createRef()
}

<span ref={this.ref}></span>
获取span节点对应的dom对象
this.ref.current

ref作用：可以获取dom节点，或者组件的实例

函数组件
import React,{useRef} from 'react'
const inputRef = useRef()
<input ref={inputRef}>

useEffect(()=>{
  console.log(inputRef) // 结果：{current:input}
},[])
```

### 函数组件性能优化

```js
import React, {
  memo, //类似类组件的shouldComponetUpdate对比传进的props有无改变，判断渲染与否，返回true，不需要更新
  useMemo,
  useCallback,
} from "react";

function FunC() {
  const [count, dispatchCount] = useReducer(countReducer, 0);
  const [name, setName] = useStaet("wiking");

  const config = {
    text: `count id${count}`,
    color: count > 3 ? "red" : "blue",
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <Child config={config} onButtonClick={() => dispatch({ type: "add" })} />
    </div>
  );
}

function Child({ config, onButtonClick }) {
  // 更新与否看config, onButtonClick
  consol.log("child render");
  return (
    <div>
      <button onClick={onButtonClick} style={{ color: config.color }}>
        {config.text}
      </button>
    </div>
  );
}

// 优化1：使用memo
const Child = memo(function Child({ config, onButtonClick }) {
  consol.log("child render");
  return (
    <div>
      <button onClick={onButtonClick} style={{ color: config.color }}>
        {config.text}
      </button>
    </div>
  );
});

// 优化2：setName执行导致FunC重新渲染，导致传入的config都是新的，所以child重新渲染，（每次方法调用，都会形成自己的闭包）
// 使用uesMemo：参数是回调函数，没有第二个参数，每次生成的也都是新的对象，第二个参数效果跟useEffect是一样的
const config = useMeno(
  () => ({
    text: `count id${count}`,
    color: count > 3 ? "red" : "blue",
  }),
  [count]
);
// 这里是否需要重新生成取决于count变量，则依赖它,现在count不变，config都是指向同一个地方的

// 优化3
// 此时setName执行，child还是渲染了：这是因为setName执行，FunC重新渲染，导致onButtonClick每次也是新的匿名函数
// 使用useCallback优化
const handleButtonClick = useCallback(() => dispatch({ type: "add" }), []); // 此方法不依赖

<Child config={config} onButtonClick={handleButtonClick} />;

// 注意useCallback是useMeno的简化写法来的
// 同样可以使用useMeno优化方法
const handleButtonClick = useMeno(() => () => dispatch({ type: "add" }), []); // 接收一个函数返回一个新函数

// useMeno和useCallback要配合meno使用才有效果;
```

### 闭包陷阱

```js
// useRef()在任一次渲染的时候，都是返回的同一对象（规避了闭包问题），它的形式：{current:''} // 赋值为什么就是什么
// 现在闭包的是countRef,对其属性current没有影响

const countRef = useRef();
countRef.current = count;

const handleAlertButtonClick = function () {
  setTimeout(() => {
    // alert(count)  // 当count为3的时候alert，其实外面count已经改到5了
    alert(countRef.current);
  }, 2000);
};
```
