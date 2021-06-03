---
title: CommonJS模块加载机制
date: 2020-07-02 16:51:10
categories: 
- node
tags:
- node
---

## Common.js 规范


模块必须通过 <font>module.exports</font> 导出对外的变量或接口，通过 <font>require()</font> 来导入其他模块的输出到当前模块作用域中。

## Common.js 模块特点

1. 所有代码运行在当前模块作用域中，不会污染全局作用域。
2. 模块同步加载，根据代码中出现的顺序依次加载。
3. 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。


## exports 和 module.exports 的区别

* module.exports 默认值为{}
* exports 是 module.exports 的引用
* exports 默认指向 module.exports 的内存空间
* require() 返回的是 module.exports 而不是 exports
* 若对 exports 重新赋值，则断开了 exports 对 module.exports 的指向。


## module对象

[Module 源码地址](https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L168)

```javascript
function Module(id = '', parent) {
  // module.id 模块的识别符，通常是带有绝对路径的模块文件名。
module.filename 模块的文件名，带有绝对路径。
module.loaded 返回一个布尔值，表示模块是否已经完成加载。
module.parent 返回一个对象，表示调用该模块的模块。
module.children 返回一个数组，表示该模块要用到的其他模块。
module.exports 表示模块对外输出的值。
  // 模块的识别符，通常是带有绝对路径的模块文件名。
  this.id = id;
  // 文件名的目录
  this.path = path.dirname(id);
  // 表示模块对外输出的值
  this.exports = {};
  // moduleParentCache.set(this, parent);
  // updateChildren(parent, this, false);
  // 模块的文件名，带有绝对路径。
  this.filename = null;
  // 返回一个布尔值，表示模块是否已经完成加载。
  this.loaded = false;
  // 返回一个数组，表示该模块要用到的其他模块。
  this.children = [];
}
```

## require机制

当调用require方法引入一个模块的时候，就涉及到Commonjs模块的引入规则了。

```javascript
// test.js
let a = function() {
}

module.exports = {
  b: 2,
  a
}

console.log('module', module)


// 2.js
var {a, b} = require('./1.js')

```

执行 `node 2.js`打印

```javascript
Module {
  id: '/Users/xiamer/Desktop/test/node/1.js',
  path: '/Users/xiamer/Desktop/test/node',
  exports: { b: 2, a: [Function: a] },
  filename: '/Users/xiamer/Desktop/test/node/1.js',
  loaded: false,
  children: [],
  paths: [
    '/Users/xiamer/Desktop/test/node/node_modules',
    '/Users/xiamer/Desktop/test/node_modules',
    '/Users/xiamer/Desktop/node_modules',
    '/Users/xiamer/node_modules',
    '/Users/node_modules',
    // path 属性是一个数组，包含了模块可能的位置
    '/node_modules'
  ]
}
```

## 模块加载规则

require命令是CommonJS规范之中，用来加载其他模块的命令。它其实不是一个全局命令，而是每个模块提供的一个内部方法，也就是说，只有在模块内部才能使用 require(Mo) 命令。另外，require 其实内部调用 Module._load 方法。

[Module._load 源码](https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js#L168)

```js
Module.prototype.require = function(path){
  return Module._load(path, this)  
}

Module._load = function(request, parent, isMain) {
  //  计算绝对路径
  var filename = Module._resolveFilename(request, parent);

  //  第一步：如果有缓存，取出缓存
  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;

  // 第二步：是否为内置模块
  if (NativeModule.exists(filename)) {
    return NativeModule.require(filename);
  }

  // 第三步：生成模块实例，存入缓存
  var module = new Module(filename, parent);
  Module._cache[filename] = module;

  // 第四步：加载模块
  try {
    module.load(filename);
    hadException = false;
  } finally {
    if (hadException) {
      delete Module._cache[filename];
    }
  }

  // 第五步：输出模块的exports属性
  return module.exports;
};
```


第四步会采用module._compile()执行指定模块的脚本

1.对文件内容进行处理，注入exports, require, module, __filename, __dirname

```js
"(function (exports, require, module, __filename, __dirname) { 
" + scriptContent +"
});"
```

2. 将字符串形式的代码转换成可执行的代码块。

```js
var compiledWrapper = vm.runInThisContext(wrapper, {
    filename: filename,
    lineOffset: 0,
    displayErrors: true
  });
```

3. 执行模块代码，真正注入当前的this.exports, require, this, filename, dirname，最后 this.exports 挂载执行后的结果

```js
compiledWrapper.call(this.exports, this.exports, require, this, filename, dirname);
```

4. 模块执行完之后将module.loaded设置为true.

this.loaded = true;


## Common.js 文件查找规则

![fehelper](/images/module/file-sort.png)


## 参考

* [node loader](https://github.com/nodejs/node/blob/master/lib/internal/modules/cjs/loader.js)
* [require 源码解读](http://www.ruanyifeng.com/blog/2015/05/require.html)