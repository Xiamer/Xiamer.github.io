---
title: js中call apply bind的简单实现
date: 2019-04-28 15:37:34
categories: 
- web前端
tags:
- js
---

## call的实现

### call的简单用法
```js
function a(x, y) {
  this.x = x;
  this.y = y;
};
let obj = {};
a.call(obj, 1, 2);

console.log(obj); // {x: 1,y: 2};

```

### 概念
call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

### 语法
```js
function.call(thisArg, arg1, arg2, ...)
```

#### 参数
* thisArg：可选的。在 function 函数运行时使用的 `this` 值。请注意，this可能不是该方法看到的实际值：如果这个函数处于**非严格模式**下，则指定为 null 或 undefined 时会自动替换为指向**全局对象**，原始值会被包装。
  
* arg1, arg2, ...
指定的参数列表。

### 思考：怎么模拟实现call的用法？
call主要做了什么？
1. 改变`this`指向, 指向了thisArg。
2. 执行了该函数。
   
如何实现？
1. 绑定function到thisArg的属性上。即：thisArg.fn = function
2. 执行该属性。即 thisArg.fn()
3. 删除该属性。即 delete thisArg.fn
   
### 代码实现

```js
// ES3写法
Function.prototype.myCall = function (context) {
  context = context || window;  // 当context为null，指为window
  context.fn = this;
  // 获取参数
  var args = [];
  for (var i = 1; i < arguments.length; i++) {
    args.push('arguments[' + i + ']')
  };
  // args 会自动调用 Array.toString() 这个方法
  var result = eval('context.fn(' + args + ')');
  delete context.fn;
  return result;
}

// es6写法
Function.prototype.myCall2 = function (context, ...args) {
  context = context || window;  // 当context为null，指为window
  context.fn = this;
  let result =  context.fn(...args);
  delete context.fn;
  return result;
}

```
### 测试代码正确性
```js
var obj = {
    value: 1
}

function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}

console.log(bar.call(obj, 'kevin', 18));     // {value: 1,name: 'kevin', 18}
console.log(bar.myCall(obj, 'kevin', 18));   // {value: 1,name: 'kevin', 18}
console.log(bar.myCall2(obj, 'kevin', 18));  // {value: 1,name: 'kevin', 18}

```

## apply的实现

### apply的简单用法
```js
function a(x, y) {
  this.x = x;
  this.y = y;
};
let obj = {};
a.apply(obj,[1,2]);

console.log(obj); // {x: 1,y: 2};

const numbers = [5, 6, 2, 3, 7];

const max = Math.max.apply(null, numbers);

console.log(max); // 7

```
### 概念
apply() 方法调用一个具有给定this值的函数，以及作为一个**数组**（或**类似数组对象**）提供的参数。

注意：call()方法的作用和 apply() 方法类似，区别就是call()方法接受的是参数列表，而**apply()方法接受的是一个参数数组**。

### 语法
```js
function.apply(thisArg, [argsArray])
```
### 参数
* thisArg必选的。在 function 函数运行时使用的 `this` 值。请注意，this可能不是该方法看到的实际值：如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装。
* argsArray
可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 function 函数。如果该参数的值为 null 或  undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象。

### 代码实现

```js
Function.prototype.myApply = function (context, arr) {
  context = context || window;  // 当context为null，指为window
  context.fn = this;
  let result;
  if (!arr) {
    result = context.fn();
  } else {
    let args = [];
    for (let i = 0; i < arr.length; i++) {
      args.push('arr[' + i + ']');
    };
    result = eval('context.fn(' + args + ')');
  }
  delete context.fn;
  return result;
}

Function.prototype.myApply2 = function (context, arr) {
  context = context || window;  // 当context为null，指为window
  context.fn = this;
  let result;
  if (!arr) {
    result = context.fn();
  } else {
    result = context.fn(...arr);
  }
  delete context.fn;
  return result;
}
```

### 测试代码正确性

```js
var obj = {
    value: 1
}

function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}

Function.prototype.myApply = function (context, arr) {
  context = context || window;  // 当context为null，指为window
  context.fn = this;
  let result;
  if (!arr) {
    result = context.fn();
  } else {
    let args = [];
    for (let i = 0; i < arr.length; i++) {
      args.push('arr[' + i + ']');
    };
    result = eval('context.fn(' + args + ')');
  }
  delete context.fn;
  return result;
}

Function.prototype.myApply2 = function (context, arr) {
  context = context || window;  // 当context为null，指为window
  context.fn = this;
  let result;
  if (!arr) {
    result = context.fn();
  } else {
    result = context.fn(...arr);
  }
  delete context.fn;
  return result;
}

console.log(bar.apply(obj, ['kevin', 18]));     // {value: 1,name: 'kevin', 18}
console.log(bar.myApply(obj, ['kevin', 18]));   // {value: 1,name: 'kevin', 18}
console.log(bar.myApply2(obj, ['kevin', 18]));    // {value: 1,name: 'kevin', 18}
```


## bind的实现

### 概念 
bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。

### 语法

```js
function.bind(thisArg[, arg1[, arg2[, ...]]])
```

* thisArg 调用绑定函数时作为 this 参数传递给目标函数的值。 **如果使用new运算符构造绑定函数，则忽略该值。**当使用 bind 在 setTimeout 中创建一个函数（作为回调提供）时，作为 thisArg 传递的任何原始值都将转换为 object。如果 bind 函数的参数列表为空，执行作用域的 this 将被视为新函数的 thisArg。

* arg1, arg2, ...
当目标函数被调用时，被预置入绑定函数的参数列表中的参数。

### 用法与实现

#### bind传参数->执行时再传参
##### 常见用法
```js
var foo = {
    value: 1
};

function bar(name, age) {
  console.log(this.value); // 1
  console.log(name);       // daisy
  console.log(age);        // 18

}
var bindFoo = bar.bind(foo, 'daisy');
bindFoo('18');
```

##### 模拟实现
```js
Function.prototype.mybind = function (context) {
  const self = this;
  // 获取bind时传入参数
  const args = Array.prototype.slice.call(arguments, 1);
  return function() {
    // 获取执行时传入参数
    const contentArgs =  Array.prototype.slice.call(arguments);
    return self.apply(context, args.concat(contentArgs));
  }
}
```

#### bind传参 -> new bind返回函数

##### 常见用法
```js
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}

bar.prototype.friend = 'kevin';

var bindFoo = bar.bind(foo, 'daisy');

var obj = new bindFoo('18');
// undefined 当作为new 时，函数中的this指向实例化后的对象
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// kevin
```

##### 模拟实现

```js
Function.prototype.mybind2 = function (context) {
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
  }
  const self = this;
  const args = Array.prototype.slice.call(arguments, 1);
  let fNop = function () {};
  
  const fBound = function() {
    cons contentArgs =  Array.prototype.slice.call(arguments);
    // this instanceof fNop 为true时，代表new fn()；为false时，代表直接执行fn()，此时this是window应指向context
    return self.apply(this instanceof fNop ? this: context, args.concat(contentArgs));
  };
  // 维护原型关系
  // 若改了fBound(即：bind())的prototype 会影响原始函数的prototype，所以我们应该找个中介替代
  // fBound.prototype = this.prototype;
  // bind 方法所返回的函数并不包含 prototype 属性，并且将这些绑定的函数用作构造函数所创建的对象从原始的未绑定的构造函数中继承prototype 这就意味着如果你打印构造函数所创建的对象的 constructor 属性，应该指向未绑定的构造函数
  fNop.prototype = this.prototype;
  fBound.prototype = new fNop();
  return fBound;
}
```

### 参考链接
* [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
* [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
* [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
* [JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)
* [JavaScript深入之call和apply的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
