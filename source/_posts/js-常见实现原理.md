---
title: js 常见实现原理
date: 2020-06-27 17:45:03
tags:
---

## new

```js
function objectFactory(fn, ...arg) {
  var obj = new Object();
  obj.__proto__ = fn.prototype
  let res = fn.apply(obj, arg)
  return typeof res === 'object' ? ret : obj;
}
```

## call apply

```js
/**
 * 
 * 手写 call 参数为一个个， apply参数为数组
 * 
 * function a () {}
 * a.call(obj, x1, x2)
 */
Function.prototype.call = function (context, ...args) {
  let context = context || window;
  const fn = new Symbol()
  context[fn] = this

  var result = context.fn(...arg)

  delete context.fn
  return result;
}

```

## bind

```js
Function.prototype.bind = function (context, ...arg) {
  var context = context || window;
  var self = this;

  var fNOP = function () { };

  var fBound = function (...arg2) {
    // 当作为构造函数(new Obj)时，this 指向实例，此时结果为 true
    // 当作为普通函数时，this 指向 context
    return self.apply(this instanceof fNOP ? this : context, arg.concat(arg2));
  }

  // 我们直接修改 fBound.prototype 的时候，也会直接修改绑定函数的 prototype
  // fBound.prototype = this.prototype;

  // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
  fNOP.prototype = this.prototype;
  fBound.prototype = new fNOP();

  return fBound
}
```

## instanceof 

```js
function instance_of(L, R) {
  let l = L.__proto__
  let r = R.prototype
  while(true) {
    if (l === r) return true
    if (l === null) return false
    l = l.__proto__
  }
}
```

## Object.Create

Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。

```js
function objCreate(obj) {
  let fn = function () { };
  fn.prototype = obj;
  return new fn();
}
```

## event emit 发布订阅

```js
class EventEmit {
  constructor() {
    this._eventpool = {};
  }
  on(type, callback) {
    if (this._eventpool[type]) this._eventpool[type].push(callback)
    else this._eventpool[type] = [callback]
  }
  emit(type, ...arg) {
    this._eventpool[type] && this._eventpool[type].forEach(callback => {
      callback.apply(this, arg)
    });
  }
  off(type, callback) {
    let cbList = this._eventpool[type]
    if (!cbList) return false
    if (callback) {
      const i = cbList.indexOf(callback)
      if (i >= 0) this._eventpool[type].splice(i, 1)
    } else {
      delete this._eventpool[type]
    }
  }
  once(type, callback) {
    const self = this
    function _fn(...arg) {
      callback(...arg)
      self.off(type, _fn)
    }

    this.on(type, _fn)
  }
}
```

## 节流throttle 防抖debounce

节流 throttle 当持续触发事件时，保证一定时间段内只调用一次事件处理函数。
防抖 debounce 当持续触发事件时，一定时间段内没有再触发事件，事件处理函数才会执行一次.

```js
const throttle = (fn, delay = 300) => {
  let flag = true
  return (...args) => {
    if (!flag) return
    flag = false
    setTimeout(() => {
      flag = true
      fn.apply(this, arg)
    }, delay);
  }
};

const debounce = (fn, delay = 300) => {
  let timer = null;
  return (...args) => {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
};

```

## 寄生组合式继承 (call+寄生式封装)

[参考](https://juejin.cn/post/6844904013096288269#heading-22)

```js
function Parent(name) {
  this.name = name
}
Parent.prototype.say = function() {
  console.log('parent say', this.name)
}

function Sub() {
  Parent.call(this, 'li')
}

function inhertPro(son, father){
    // 原型式继承
    var fatherPrototype = Object.create(father.prototype);
    // 设置Son.prototype的原型是Father.prototype
    son.prototype = fatherPrototype;
    // 修正constructor 指向
    // constructor的作用：返回创建实例对象的Object构造函数的引用。
    // 在这里保持constructor指向的一致性
    son.prototype.constructor = son; 
}

inhertPro(Sub, Parent);
Sub.prototype.write = function() {}
var sonInstance = new Sub();
```

## map 实现

```js
Array.prototype.map = function (fn, context) {
  let self = this;
  let result = [];
  for (let i = 0; i < self.length; i++) {
    result.push(fn.call(context, self[i], i, self));
  }
  return result
};
```

## reduce 实现
```js
Array.prototype.reduce = function (fn, initialVal) {
  let previous = initialVal; k = 0;
  if (initialVal === undefined) {
    previous = this[0];
    k = 1;
  }
  for (; k < this.length; k++) {
    previous = fn(previous, this[k], k, this);
  }
  return previous;
}
```

## reduce 实现map

```js
Array.prototype._map = function(fn, callbackThis) {
  let res = [];
  let CBThis = callbackThis || null;
  this.reduce((brfore, after, idx, arr) => {
      res.push(fn.call(CBThis, after, idx, arr));
  }, null);
  return res;
};
```

## 数组扁平化

```js
 Array.prototype.flat = function (deep = 1) {
  let self = this
  return self.reduce((pre, next) => {
    return pre.concat(Array.isArray(next) && deep > 1 ? next.flat(deep - 1) : next);
  }, [])
}

 Array.prototype.flat = function (deep = 1) {
  let self = this
  return [].concat(...self.map(v => {
    return Array.isArray(v) && deep > 1 ? v.flat(deep - 1) : v
  }))
}
```


## async await （Generator自治性）

```js
/**
 * async await
 * gennarator 语法糖
 * 
  async function fn(args) {
    // ...
  }
  function fn(args) {
    return spawn(function* () {
      // ...
    });
  }

  spawn函数就是自动执行器
 * 
 */
 function spawn(genF) {
  return new Promise(function (resolve, reject) {
    const gen = genF();
    function step(nextF) {
      let next;
      try {
        next = nextF();
      } catch (e) {
        return reject(e);
      }
      if (next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function (v) {
        step(function () { return gen.next(v); });
      }, function (e) {
        step(function () { return gen.throw(e); });
      });
    }
    step(function () { return gen.next(undefined) });
  });
}
```


## koa compose 洋葱模型
```js
const middleware = []
let mw1 = async function (ctx, next) {
  console.log('next 前，第1个中间件')
  await next()
  console.log('next 后，第1个中间件')
}
let mw2 = async function (ctx, next) {
  console.log('next 前，第2个中间件')
  await next()
  console.log('next 后，第2个中间件')
}
let mw3 = async function (ctx, next) {
  console.log('第三个，没有next了')
}

function use(mw) {
  middleware.push(mw)
}
use(mw1)
use(mw2)
use(mw3)
let i = 0;
function fn(ctx) {
  let cuurrentMW = middleware[i]
  // mid
  return cuurrentMW(ctx, middleware[++i])
}
fn()

function fn(ctx) {
  function dispatch(i) {
    let currentMW = middleware[i]
    if (!currentMW) return
    return currentMW(ctx, dispatch.bind(null, i + 1))
  }
  dispatch(0)
}

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

## deepclone (WeakMap)

```js
function deepClone(obj, hash = new WeakMap()) {
  if (hash.has(obj)) return obj;
  var cobj;
  // null
  if (obj === null) { return obj }
  let t = typeof obj;

  // 基本类型
  switch (t) {
    case 'string':
    case 'number':
    case 'boolean':
    case 'undefined':
      return obj;
  }

  // 数组
  if (Array.isArray(obj)) {
    cobj = [];
    obj.forEach((c, i) => { cobj.push(deepClone(obj[i])) });
  } else {
    cobj = {};
    // object // symbol
    if (Object.prototype.toString.call(obj) === "[object Object]") {
      Object.getOwnPropertyNames(obj).concat(Object.getOwnPropertySymbols(obj)).forEach(c => {
        hash.set(obj, obj);
        cobj[c] = deepClone(obj[c], hash);
      });
    } else {
      //内置Object
      cobj = obj;
    }
  }
  return cobj;
}

```

## req limit

```js
// https://github.com/fedono/fe-questions/issues/20
function asyncLimit(requests, limit) {
  let requestLimits = requests.splice(0, limit).map((request, i) => {
    // 在 map 里面，每个请求里面的 then 都要返回一个å i
    // 用来在 race 执行完成之后，添加过来的知道如何替换掉已经执行过的请求
    return request().then(() => i);
  });

  let p = Promise.race(requestLimits);
  for (let i = 0; i < requests.length; i++) {
    // 主要是没有明白这里的 p = p.then 是什么意思？
    // 每次都需要更新一次 p，要不然每次都是第一次的 Promise.race(requestLimits) ？
    // 如果这里只是 p.then 而不是 p = p.then 那么 requestLimits中第一次会输出一次结果，然后 requests 中的结果会一次性全输出，总共输出两次
    // 每次都要 p = p.then 应该是每次都要更新p，因为每次在 race 中都有一个函数执行完成了，所以需要更新一下
    // 0 1 2 , 0 1 2 , 0, 1 2 ->
    // 3 1 2, 3 1 2, 3 1 2 ->
    // 3 1 4, 3 1 4, 3 1 4 ->
    p = p.then(index => {
      // p.then(index => {
      // 一定要在这里加上 .then(() => index) 这样才能把 index 传给下一个函数
      requestLimits[index] = requests[i]().then(() => index);
      // 每次 race 中的函数执行完成后，都是返回 index，所以这里要 return Promise.race，来让下一个 then 中接受的参数是 index
      return Promise.race(requestLimits);
    });
  }
}
```


## Promise 简单实现

[https://zhuanlan.zhihu.com/p/21834559](史上最易读懂的 Promise/A+ 完全实现)

```js

function Promise(executor) {
  var self = this
  self.status = 'pending' // Promise当前的状态
  self.data = undefined  // Promise的值
  self.onResolvedCallback = [] // Promise resolve时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面
  self.onRejectedCallback = [] // Promise reject时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面

  function resolve(value) {
    if (self.status === 'pending') {
      setTimeout(()=>{
        self.status = 'resolved'
        self.data = value
        for(var i = 0; i < self.onResolvedCallback.length; i++) {
          self.onResolvedCallback[i](value)
        }
      })
    }
  }

  function reject(reason) {
    if (self.status === 'pending') {
      setTimeout(()=>{
        self.status = 'rejected'
        self.data = reason
        for(var i = 0; i < self.onRejectedCallback.length; i++) {
          self.onRejectedCallback[i](reason)
        }
      })
    }
  }

  try { // 考虑到执行executor的过程中有可能出错，所以我们用try/catch块给包起来，并且在出错后以catch到的值reject掉这个Promise
    executor(resolve, reject) // 执行executor
  } catch(e) {
    reject(e)
  }
}

Promise.prototype.then = function(onResolved, onRejected) {
  var self = this
  var promise2

  // 根据标准，如果then的参数不是function，则我们需要忽略它，此处以如下方式处理
  onResolved = typeof onResolved === 'function' ? onResolved : function(value) {return value}
  onRejected = typeof onRejected === 'function' ? onRejected : function(reason) {throw reason}

  if (self.status === 'resolved') {
    // 如果promise1(此处即为this/self)的状态已经确定并且是resolved，我们调用onResolved
    // 因为考虑到有可能throw，所以我们将其包在try/catch块里
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onResolved(self.data)
        if (x instanceof Promise) { // 如果onResolved的返回值是一个Promise对象，直接取它的结果做为promise2的结果
          x.then(resolve, reject)
        }
        resolve(x) // 否则，以它的返回值做为promise2的结果
      } catch (e) {
        reject(e) // 如果出错，以捕获到的错误做为promise2的结果
      }
    })
  }

  // 此处与前一个if块的逻辑几乎相同，区别在于所调用的是onRejected函数，就不再做过多解释
  if (self.status === 'rejected') {
    return promise2 = new Promise(function(resolve, reject) {
      try {
        var x = onRejected(self.data)
        if (x instanceof Promise) {
          x.then(resolve, reject)
        }
      } catch (e) {
        reject(e)
      }
    })
  }

  if (self.status === 'pending') {
  // 如果当前的Promise还处于pending状态，我们并不能确定调用onResolved还是onRejected，
  // 只能等到Promise的状态确定后，才能确实如何处理。
  // 所以我们需要把我们的**两种情况**的处理逻辑做为callback放入promise1(此处即this/self)的回调数组里
  // 逻辑本身跟第一个if块内的几乎一致，此处不做过多解释
    return promise2 = new Promise(function(resolve, reject) {
      self.onResolvedCallback.push(function(value) {
        try {
          var x = onResolved(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })

      self.onRejectedCallback.push(function(reason) {
        try {
          var x = onRejected(self.data)
          if (x instanceof Promise) {
            x.then(resolve, reject)
          }
        } catch (e) {
          reject(e)
        }
      })
    })
  }
}


Promise.prototype.all = function (arr) {
  return new Promise(function (resolve, reject) {
    let result = [];
    let count = 0;
    for (const i = 0; i < arr.length; i++) {
      Promise.resolve(arr[i]).then(res => {
        count++;
        result[i] = res;
        if (arr.length === count) {
          resolve(result);
        }
      }).catch(err => {
        reject(err);
      })

    }
  })
}

Promise.prototype.resolve = function (value) {
  return new Promise(function (resolve, reject) {
    resolve(value);
  })
}

Promise.prototype.reject = function (reason) {
  return new Promise(function (resolve, reject) {
    reject(reason);
  })
}
Promise.prototype.any = function(iterators) {
  const promises = Array.from(iterators);
  const num = promises.length;
  const rejectedList = new Array(num);
  let rejectedNum = 0;

  return new Promise((resolve, reject) => {
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => resolve(value))
        .catch(error => {
          rejectedList[index] = error;
          if (++rejectedNum === num) {
            reject(rejectedList);
          }
        });
    });
  });
};



const formatSettledResult = (success, value) =>
  success
    ? { status: "fulfilled", value }
    : { status: "rejected", reason: value };

Promise.prototype.allSettled = function(iterators) {
  const promises = Array.from(iterators);
  const num = promises.length;
  const settledList = new Array(num);
  let settledNum = 0;

  return new Promise(resolve => {
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          settledList[index] = formatSettledResult(true, value);
          if (++settledNum === num) {
            resolve(settledList);
          }
        })
        .catch(error => {
          settledList[index] = formatSettledResult(false, error);
          if (++settledNum === num) {
            resolve(settledList);
          }
        });
    });
  });
};

Promise.prototype.finally = function(cb) {
  return this.then(
    value => Promise.resolve(cb()).then(() => value),
    error =>
      Promise.resolve(cb()).then(() => {
        throw error;
      })
  );
};

```