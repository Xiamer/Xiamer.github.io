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
  while(l) {
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
function Father(...arr) {
}
Father.prototype.someFn = function() {
}
function Son() {
    Father.call(this, 'xxxx');
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

inhertPro(Son, Father);
var sonInstance = new Son();
```

## map 实现

```js
Function.prototype.map = function (fn, context) {
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
Function.prototype.reduce = function (fn, initialVal) {
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