---
title: koa 洋葱模型
date: 2020-06-01 23:52:24
categories: 
- node
tags:
- node
---


## 中间件使用

```js
const Koa = require('koa')
const app = new Koa()

// logTime
app.use(async function(ctx, next) {
  console.log("next前，打印时间戳:", new Date().getTime())
  await next()
  console.log("next后，打印时间戳:", new Date().getTime())
})

// logUrl
app.use(async function(ctx, next) {
  console.log("next前，打印url:", ctx.url)
  await next()
  console.log("next后，打印url:", ctx.url)
})

// response
app.use(async ctx => {
  ctx.body = 'Hello World'
})

app.listen(3000)
```

执行 `node index.js`，打印

```js
// next前，打印时间戳: 1590988200341
// next前，打印url: /
// next后，打印url: /
// next后，打印时间戳: 1590988200854
```

可从打印中得出，中间件以`next()`分界，`next()`之前是按这<font color=red>正序</font>执行，`next()`之后是按这<font color=red>倒序</font>执行。

为什么这样设计呢？

1. 若链式调用无法保证前面加载的中间件使用后面添加的中间件。
2. 若都为倒序，与正常思维相违背。

koa 中的中间件执行流程是：
1. 外层中间件进行前期处理
2. 调用next（指向下个中间件），将控制流交给下个中间件，并await其完成，直到后面没有中间件或者没有next函数执行为止。
3. 完成后一层层回溯执行各个中间件的后期处理（next 后的逻辑）


## 洋葱模型

<img src="/images/koa/1.png" width="80%">
<img src="/images/koa/2.png" width="80%">


## 探索 自己实现洋葱模型中间件

1. 参数： 要把上下文ctx对象和下一个中间件next传给当前的中间件
2. 必须要等待下一个中间件执行完，再执行当前中间件的后续逻辑

```js
const middleware = []
let mw1 = async function(ctx, next) {
  console.log('next 前，第1个中间件')
  await next()
  console.log('next 后，第1个中间件')
}
let mw2 = async function(ctx, next) {
  console.log('next 前，第2个中间件')
  await next()
  console.log('next 后，第2个中间件')
}
let mw3 = async function(ctx, next) {
  console.log('第三个，没有next了')
}

function use(mw) {
  middleware.push(mw)
}
use(mw1)
use(mw2)
use(mw3)
let i = 0

let fn = function(ctx) {
  let cuurrentMW = middleware[i]
  return cuurrentMW(ctx, middleware[++i])
}
fn()
```

打印
```js
// next 前，第1个中间件
// next 前，第2个中间件
// next is not a function
```
`next`只有第一次只像第二个`middleware`,后面没有进行依次传递。

思考：怎么包装`middleware`传递给下个参数呢？

看个例子：

```js
function sum(a, b) {
  return a + b;
}
sum(1, 2) // 3
// ------>
sumPlus = sum.bind(null, 1 ,2)
sumPlus() // 3
```

bind会将当时的参数保留下来，修改下实现的代码

```js
const middleware = []
let mw1 = async function (ctx, next) {
    console.log("next前，第1个中间件")
    await next()
    console.log("next后，第1个中间件")
}
let mw2 = async function (ctx, next) {
    console.log("next前，第2个中间件")
    await next()
    console.log("next后，第2个中间件")
}
let mw3 = async function (ctx, next) {
    console.log("第3个中间件，没有next了")
}

function use(mw) {
    middleware.push(mw)
}

use(mw1)
use(mw2)
use(mw3)

let fn = function (ctx) {
  function dispatch(i) {
      let currentMW = middleware[i]
      if(!currentMW) return
      return currentMW(ctx, dispatch.bind(null, i + 1))
  }
  return dispatch(0)
}

fn()
```

打印
```js
// next 前，第1个中间件
// next 前，第2个中间件
// 第3个中间件，没有next了
// next后，第2个中间件
// next后，第1个中间件


```
这样就实现了简易版的中间件，下面我们看下源码的实现，相差不多，加了`promise`包装和一些错误处理。


## koa-compose 源码相关

1. `app.listen(port)` [源码地址](https://github.com/koajs/koa/blob/master/lib/application.js#L78)

```js
listen(...args) {
  debug('listen');
  const server = http.createServer(this.callback());
  return server.listen(...args);
}
```
createServer 执行 `this.callback()`


2. `this.callback()` [源码地址](https://github.com/koajs/koa/blob/698ce0afbfac6480400625729a4b8fc4b4203fdc/lib/application.js#L143)

```js
callback() {
  const fn = compose(this.middleware);

  if (!this.listenerCount('error')) this.on('error', this.onerror);

  const handleRequest = (req, res) => {
    const ctx = this.createContext(req, res);
    return this.handleRequest(ctx, fn);
  };

  return handleRequest;
}

```
我们只看第一行 `const fn = compose(this.middleware);`, `compose` 执行中间件

3. 那我们中间件何时添加的呢？当然是发生在 `use` [源码地址](https://github.com/koajs/koa/blob/698ce0afbfac6480400625729a4b8fc4b4203fdc/lib/application.js#L122)

```js
 use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    if (isGeneratorFunction(fn)) {
      deprecate('Support for generators will be removed in v3. ' +
                'See the documentation for examples of how to convert old middleware ' +
                'https://github.com/koajs/koa/blob/master/docs/migration.md');
      fn = convert(fn);
    }
    debug('use %s', fn._name || fn.name || '-');
    this.middleware.push(fn);
    return this;
  }
```


4. `compose 函数` [源码地址](https://github.com/koajs/compose/blob/master/index.js)

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }
  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```


## 参考链接

[koa git 仓库](https://github.com/koajs/koa)