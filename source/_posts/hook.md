---
title: hook
date: 2019-06-15 16:29:58
categories: 
- web前端
tags:
- js
  
---
本文主要介绍下`React Hook` 和 `Vue Function-based API RFC`，通过一些简单案列进行对比。

## React Hook

### 常见问题
1. 组件状态复用性逻辑困难（render props 、 HOC）
2. 无状态Function组件
3. class中的this指向

### 定义

在不编写 class 的情况下使用 state 以及其他的 React 特性。
主要解决的问题是状态共享，是继 `render-props` 和 `HOC` 之后的第三种状态共享方案，不会产生 `JSX` 嵌套地狱问题。

### useState

`useState`可以让函数组件也具备类组件的state功能

先来看一个简单计数的例子，Component写法
```js
class Example extends React.Component {
 constructor(props) {
   super(props);
   this.state = {
     count: 0
   };
 }
 render() {
   return (
     <div>
       <p>You clicked {this.state.count} times</p>
       <button onClick={() => this.setState({ count: this.state.count + 1 })}>
         Click me
       </button>
     </div>
   );
 }
}

```
useState写法
```js
import React, { useState } from 'react';
function Example() {
 const [count, setCount] = useState(0);
 return (
   <div>
     <p>You clicked {count} times</p>
     <button onClick={() => setCount(count + 1)}>Click me</button>
   </div>
 );
}
```
调用了react的useState钩子函数，用数组结构赋值 `count` `setConut`代替了 `this.state.conut` `this.setState({count: xx})`。

多个state是怎么定义呢？
```js
function ExampleWithManyStates() {
 const [age, setAge] = useState(42);
 const [fruit, setFruit] = useState('banana');
 const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
}
```
执行过程又是怎样的？
```js
 //第一次渲染
 useState(42);  //将age初始化为42
 useState('banana');  //将fruit初始化为banana
 useState([{ text: 'Learn Hooks' }]); //...

 //第二次渲染
 useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）
 useState('banana');  //读取状态变量fruit的值（这时候传的参数banana直接被忽略）
 useState([{ text: 'Learn Hooks' }]); //...
 
 ```
useState的使用
1. 声明state const [count, setCount] = useState(0); （参数、返回值）
2. 读取state count
3. 更新state setCount(count + 1)
4. 多个state 多次调用useState
5. 声明顺序来确保useState 相互独立，底层链表实现。

### Effect

定义: 你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。
可看做： componentDidMount，componentDidUpdate 和 componentWillUnmount的集合。

来看一个移动鼠标在页面显示坐标的例子

```js
import React, { Component } from 'react';

export default class MyClassApp extends Component {
 constructor(props) {
   super(props);
   this.state = {
     x: 0,
     y: 0
   };
   this.handleUpdate = this.handleUpdate.bind(this);
 }
 componentDidMount() {
   document.addEventListener('mousemove', this.handleUpdate);
 }
 componentDidUpdate() {
   const { x, y } = this.state;
   document.title = `(${x},${y})`;
 }
 componentWillUnmount() {
   window.removeEventListener('mousemove',this.handleUpdate);
 }
 
handleUpdate(e) {
   this.setState({
     x: e.clientX,
     y: e.clientY
   });
 }
 render() {
   return (
     <div>
       current position x:{this.state.x}, y:{this.state.y}
     </div>
   );
 }
}
```
useEffect 自定义hook高级写法：
```js
import React, { useState, useEffect } from 'react';

// 自定义hook useMousePostion
const useMousePostion = () => {
 // 使用hookuseState初始化一个state
  const [postion, setPostion]=useState({ x: 0, y: 0 });
  function handleMove(e) {
    setPostion({ x: e.clientX, y: e.clientY });
  }
 // 使用useEffect处理class中生命周期可以做到的事情
  useEffect(() => {
 // 同时可以处理 componentDidMount 以及 componentDidUpdate 中的事情
   window.addEventListener('mousemove', handleMove);
   document.title = `(${postion.x},${postion.y})`;
   return () => {
     // return的function 可以相当于在组件被卸载的时候执行 类似于 componentWillUnmount
     window.removeEventListener('mousemove', handleMove);
   };
   // [] 是参数，代表deps，也就是说react触发这个hook的时机会和传入的deps有关，内部利用object.is实现
   // 默认不给参数会在每次render的时候调用，给空数组会导致每次比较是一样的，只执行一次。
 }, [postion]);
 // postion 可以被直接return，这样达到了逻辑的复用~，哪里需要哪里调用就可以了。
 return postion;
};
export default function App() {
 const { x, y } = useMousePostion(); // 内部维护自己的postion相关的逻辑
 return (
   <div>
     current position x: {x}, y: {y}
   </div>
 );
}

```

### useContext

数据传递
```js
const { Provider, Consumer } = React.createContext(null);
function Bar() {
 return <Consumer>{color => <div>{color}</div>}</Consumer>;
}
function Foo() {
 return <Bar />;
}
function App() {
 return (
   <Provider value={"grey"}>
     <Foo />
   </Provider>
 );
}
```
用useContext hook 写法：
```js
import React, { useState, useContext } from 'react';
const colorContext = React.createContext("gray");
function Bar() {
 const color = useContext(colorContext);
 return <div>{color}</div>;
}
function Foo() {
 return <Bar />;
}
function App() {
 return (
   <colorContext.Provider value={"red"}>
     <Foo />
   </colorContext.Provider>
 );
}
```
传递给 useContext 的是 context 而不是 consumer，返回值即是想要透传的数据了，减少了组件层级。

### useReducer

`useState` 实现登陆：
```js
import React, { useState, useReducer } from 'react';
function LoginPage() {
 const [name, setName] = useState(''); // 用户名
 const [pwd, setPwd] = useState(''); // 密码
 const [isLoading, setIsLoading] = useState(false); // 是否展示loading，发送请求中
 const [error, setError] = useState(''); // 错误信息
 const [isLoggedIn, setIsLoggedIn] = useState(false); // 是否登录

 const login = (event) => {
   event.preventDefault();
   setError('');
   setIsLoading(true);
   login({ name, pwd })
     .then(() => {
       setIsLoggedIn(true);
       setIsLoading(false);
     })
     .catch((error) => {
       // 登录失败: 显示错误信息、清空输入框用户名、密码、清除loading标识
       setError(error.message);
       setName('');
       setPwd('');
       setIsLoading(false);
     });
 }
 return (
     //  返回页面JSX Element
 )
}

```

`useReducer` 实现登陆

```js
    const initState = {
        name: '',
        pwd: '',
        isLoading: false,
        error: '',
        isLoggedIn: false,
    }
    function loginReducer(state, action) {
        switch(action.type) {
            case 'login':
                return {
                    ...state,
                    isLoading: true,
                    error: '',
                }
            case 'success':
                return {
                    ...state,
                    isLoggedIn: true,
                    isLoading: false,
                }
            case 'error':
                return {
                    ...state,
                    error: action.payload.error,
                    name: '',
                    pwd: '',
                    isLoading: false,
                }
            default: 
                return state;
        }
    }
    function LoginPage() {
        const [state, dispatch] = useReducer(loginReducer, initState);
        const { name, pwd, isLoading, error, isLoggedIn } = state;
        const login = (event) => {
            event.preventDefault();
            dispatch({ type: 'login' });
            login({ name, pwd })
                .then(() => {
                    dispatch({ type: 'success' });
                })
                .catch((error) => {
                    dispatch({
                        type: 'error'
                        payload: { error: error.message }
                    });
                });
        }
        return ( 
            //  返回页面JSX Element
        )
    }
```
虽然用`useReducer`写法代码比较长，但是可读性会变好很多。把**做什么**和**怎么做**分开，组件中只需要考虑**做什么**，**怎么做**逻辑全部都抽到`Reducer`中。

### React other hook

* useClassback 记忆函数
* useMemo 记忆组件
* useRef 保存引用值
* useImperativeHandle 透传 Ref
* useLayoutEffect 同步执行副作用

具体详情建议去官网链接 [React Hook](https://react.docschina.org/docs/hooks-intro.html)


### Hook 使用规则
* 只能在函数最顶层调用 Hook。不要在循环、条件判断或者子函数中调用。
* 只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。（除了自定义Hook）


## Vue （Function base on RFC）

### 常见问题
* mixins (多个逻辑变得复杂，命名冲突)
* HOC （props）
* slot 多层嵌套

### value
我们来直接看一个例子

```js
import { value }  from Vue;
function useMouse() {
 const x = value(0)
 const y = value(0)
 const update = e => {
   x.value = e.pageX
   y.value = e.pageY
 }
 onMounted(() => {
   window.addEventListener('mousemove', update)
 })
 onUnmounted(() => {
   window.removeEventListener('mousemove', update)
 })
 return { x, y }
}

// 在组件中使用该函数
const Component = {
 setup() {
   const { x, y } = useMouse()
   // 与其它函数配合使用
   const { z } = useOtherLogic()
   return { x, y, z }
 },
 template: `<div>{{ x }} {{ y }} {{ z }}</div>`
}
```

把逻辑抽到一个函数中，组件引用时使用数据来源清晰,参数重命名。
value()
* 返回是一个包装对象 value wrapper ()
* 只有一个属性 value

**为什么需要包装对象？**

我们知道在 JavaScript 中，原始值类型如 string 和 number 是只有值，没有引用的。如果在一个函数中返回一个字符串变量，接收到这个字符串的代码只会获得一个值，是无法追踪原始变量后续的变化的。

用法

* 声明 let msg = value('look')
* 读取 msg.value
* 修改 msg.value = 'here'
* 模板 unwrap {{msg}}


### Computed

```js
import { value, computed } from 'vue'

const count = value(0)
const countPlusOne = computed(() => count.value + 1)

console.log(countPlusOne.value) // 1

count.value++
console.log(countPlusOne.value) // 2
const count = value(0)
const writableComputed = computed(
 // read
 () => count.value + 1,
 // write
 val => {
   count.value = val - 1
 }
)
```
computed() 返回的是一个只读的包装对象，它可以和普通的包装对象一样在 setup() 中被返回 ，也一样会在渲染上下文中被自动展开。默认情况下，如果用户试图去修改一个只读包装对象，会触发警告。双向计算值可以通过传给 computed 第二个参数作为 setter 来创建。


### Watchers

watch() API 提供了基于观察状态的变化来执行副作用的能力。
参数
1. 数据源 （返回任意值的函数、包装对象、数组）
2. Callback
3. watchOption
```ts
interface WatchOptions {
 lazy?: boolean
 deep?: boolean
 flush?: 'pre' | 'post' | 'sync'
 onTrack?: (e: DebuggerEvent) => void
 onTrigger?: (e: DebuggerEvent) => void
}
 ```
 创建时会执行一次 和 v2.x immediate: true类似
 
```js
const MyComponent = {
 props: {
   id: Number
 },
 setup(props) {
   const data = value(null)
   watch(() => props.id, async (id) => {
     data.value = await fetchData(id)
   })
   return {
     data
   }
 }
}
```
初始化时和接收到的id发生变化时 执行watch


Watchers 清理副作用
```js
watch(idValue, (id, oldId, onCleanup) => {
 const token = performAsyncOperation(id)
 onCleanup(() => {
   // id 发生了变化，或是 watcher 即将被停止. （分页）
   // 取消还未完成的异步操作。
   token.cancel()
 })
})
```

### 声明周期函数

所有现有的生命周期钩子都会有对应的 onXXX 函数（只能在 setup() 中使用）：
```js
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
 setup() {
   onMounted(() => {
     console.log('mounted!')
   })
   onUpdated(() => {
     console.log('updated!')
   })
   // destroyed 调整为 unmounted
   onUnmounted(() => {
     console.log('unmounted!')
   })
 }
}
```

## same point 

* 解决问题是一样  逻辑复用过乱，代码体积体积过大 ，this问题（Function很少和this打交道）。
* 使用方式类似 复用一些单独的逻辑抽离到函数，返回组件所用的数据，内部维护数据更新，从而触发视图更新。

## diff point

* react（链表）每一个hook的next指向下一个hook
* vue hook 只会在setup调用时执行一次
* react 数据更改时导致render执行，会重新注册hook，需要借助useCallBack、uesMomo等处理

### 尤大大总结

* 整体更符合js的直觉
* 不受调用顺序的限制，可有条件地调用
* 会在后续更新时不断产生大量的内联函数而影响引擎优化或是导致 GC 压力；
* 不需要总是使用 useCallback 来缓存传给子组件的回调以防止过度更新；
* 不需要担心传了错误的依赖数组给 useEffect/useMemo/useCallback 从而导致回调中使用了过期的值 —— Vue 的依赖追踪是全自动的。

## 参考链接
* [React Hook](https://react.docschina.org/docs/hooks-intro.html)
* [Vue Function-based API RFC](https://zhuanlan.zhihu.com/p/68477600)
* [一篇看懂 React Hooks](https://zhuanlan.zhihu.com/p/50597236)


