
---
title: babel-基础知识(一)
date: 2021-01-04 15:37:34
categories: 
- web前端
tags:
- babel
---
## babel 是什么？

Babel 是一个 JavaScript 编译器。

简单来说把 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，让低端运行环境(如浏览器和 node )能够认识并执行。
```js
// es6
[1,2,3].map(n => n + 1);
// 转化为es5
[1,2,3].map(function(n) {
  return n + 1;
});

```

简单来说 Babel 的工作就是：

* 语法转换
* 通过 [Polyfill](https://babeljs.io/docs/en/babel-polyfill) 方式在目标环境中添加缺失的特性
* 源码转换 (codemods)


## 使用方法

1. 使用单体文件 (standalone script)
2. 命令行 (cli)
3. 构建工具的插件 (webpack 的 babel-loader, rollup 的 rollup-plugin-babel)。

这三种方式只有入口不同而已，调用的 babel 内核，处理方式都是一样的，更详细可查看[官方配置](https://babeljs.io/docs/en/configuration)


## Babel 三个编译阶段

Babel 的编译过程和大多数其他语言的编译器相似，可以分为三个阶段：

* 解析（Parsing）：将代码字符串解析成抽象语法树。
* 转换（Transformation）：对抽象语法树进行转换操作。
* 生成（Code Generation）: 根据变换后的抽象语法树再生成代码字符串。

![](/images/babel1.jpg)


## 插件(plugins)

插件总共分为两种：
1. **语法插件(syntax plugin)**：语法插件作用于 @babel/parser，负责将代码解析为抽象语法树（AST）（官方的语法插件以 babel-plugin-syntax 开头）。语法插件虽名为插件，但其本身并不具有功能性。语法插件所对应的语法功能其实都已在 @babel/parser 里实现，插件的作用只是将对应语法的解析(内部使用的解析类库为 **babylon**)功能打开。
2. **转换插件（transform plugin）**：转换插件作用于 @babel/core，负责转换 AST 的形态（官方的转换插件以 babel-plugin-transform（正式）或 babel-plugin-proposal（提案）开头）。同一类语法可能同时存在语法插件版本和转译插件版本。

> **转换插件会自动启用语法插件，如果我们使用了转译插件，就不用再使用语法插件了。**

## 预设(presets)

preset 作为 Babel 插件的组合，开发者就不需要一个个插件添加添加并安装，preset甚至可以作为可以共享的 [options](https://babeljs.io/docs/en/options) 配置。

preset 分为以下几种：

1. 官方 Preset 
  * [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
  * [@babel/preset-flow](https://babeljs.io/docs/en/babel-preset-flow)
  * [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
  * [@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)

  更多可查看[npm](https://www.npmjs.com/search?q=babel-preset)

2. Stage-X 

  这里面还细分为
  * Stage 0 - 稻草人: 只是一个想法，经过 TC39 成员提出即可。
  * Stage 1 - 提案: 初步尝试。
  * Stage 2 - 初稿: 完成初步规范。
  * Stage 3 - 候选: 完成规范和浏览器初步实现。
  * Stage 4 - 完成: 将被添加到下一年度发布。

  例如 `syntax-dynamic-import` 就是 stage-2 的内容，`transform-object-rest-spread` 就是 stage-3 的内容。
  此外，低一级的 stage 会包含所有高级 stage 的内容，例如 stage-1 会包含 stage-2, stage-3 的所有内容。
  stage-4 在下一年更新会直接放到 env 中，所以没有单独的 stage-4 可供使用。

3. 创建 Preset

* 如需创建一个自己的 preset，只需导出一份配置即可。

> 可以是返回一个插件数组...

```js
module.exports = function() {
  return {
    plugins: [
      "pluginA",
      "pluginB",
      "pluginC",
    ]
  };
}
```

  > preset 可以包含其他的 preset，以及带有参数的插件。

```js
    module.exports = () => ({
      presets: [
        require("@babel/preset-env"),
      ],
      plugins: [
        [require("@babel/plugin-proposal-class-properties"), { loose: true }],
        require("@babel/plugin-proposal-object-rest-spread"),
      ],
    });
```

4. Preset 的路径

  你还可以指定指向 preset 的绝对或相对路径。
```json
{
  "presets": ["./myProject/myPreset"]
}
```

##  执行顺序

* Plugin 会运行在 Preset 之前。
* Plugin 会从前到后顺序执行。
* Preset 的顺序则 **颠倒的**(从后向前)。

  preset 的逆向顺序主要是为了保证向后兼容，因为大多数用户的编写顺序是 ['es2015', 'stage-0']。这样必须先执行 stage-0 才能确保 babel 不报错。因此我们编排 preset 的时候，也要注意顺序，其实只要按照规范的时间顺序列出即可。

```json
{
  "plugins": ["transform-decorators-legacy", "transform-class-properties"],
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```
先执行 transform-decorators-legacy ，在执行 transform-class-properties。
先执行 @babel/preset-react，然后是 @babel/preset-env


## 插件和 preset 的配置项

简略情况下，插件和 preset 只要列出字符串格式的名字即可。但如果某个 preset 或者插件需要一些参数，就需要把自己先变成数组。第一个元素依然是字符串，表示自己的名字；第二个元素是一个对象，即配置对象。
```js
{
  "plugins": [
    ["@babel/plugin-proposal-class-properties", { "loose": true }]
  ],
  presets: [
    // 带了配置项，自己变成数组
    [
        // 第一个元素依然是名字
        env,
        // 第二个元素是对象，列出参数
        {
          module: false
        }
    ],
    // 不带配置项，直接列出名字
    'stage-2'
  ]
}

```


## @babel/polyfill

### 简介

babel 对一些新的 API 是无法转换，比如 Generator、Set、Proxy、Promise 等全局对象，以及新增的一些方法：includes、Array.form 等。所以这个时候就需要一些工具来为浏览器做这个兼容。

> 官网的定义：babel-polyfill 是为了模拟一个完整的 ES6+ 环境，旨在用于应用程序而不是库/工具。

### 缺点

* 使用 babel-polyfill 会导致打出来的包非常大，很多其实没有用到，对资源来说是一种浪费。
* babel-polyfill 可能会污染全局变量，给很多类的原型链上都作了修改，这就有不可控的因素存在。

### 现阶段优化

因上面两个问题，Babel7 中增加了 babel-preset-env，我们设置 `"useBuiltIns":"usage"` 这个参数值就可以实现`按需加载` babel-polyfill。

**V7.4.0** 版本开始`@babel/polyfill`已经被废弃, 需单独安装 `core-js` 和 `regenerator-runtime` 模块。

### demo地址
  * [@babel/polyfill demo](https://github.com/Xiamer/babel-example/blob/main/rollup.config.polyfill.js)



## @babel/preset-env

### 简介

`@babel/preset-env` 主要作用是对我们所使用的并且目标浏览器中缺失的功能进行代码转换和加载 `polyfill`，在不进行任何配置的情况下，`@babel/preset-env` 所包含的插件将支持所有最新的 `JS` 特性(ES2015,ES2016 等，不包含 stage 阶段)，将其转换成 ES5 代码。例如，如果你的代码中使用了可选链(目前，仍在 stage 阶段)，那么只配置 @babel/preset-env，转换时会抛出错误，需要另外安装相应的插件。

需要说明的是，`@babel/preset-env` 会根据你配置的目标环境，生成插件列表来编译。对于基于浏览器或 Electron 的项目，官方推荐使用 .browserslistrc 文件来指定目标环境。默认情况下，如果你没有在 Babel 配置文件中(如 .babelrc)设置 targets 或 ignoreBrowserslistConfig，@babel/preset-env 会使用 browserslist 配置源。

### 属性

* targets：描述为项目支持的浏览器或者node环境，若未指定会将所有ES2015-ES2020代码转化为ES5。配置查考[browserslist-coompatible](https://github.com/browserslist/browserslist)
* useBuiltIns：处理如何加载 `polyfill`。当设置 `useBuiltIns: usage` 时， 会根据支持的环境 `按需加载` babel-polyfill。

更多属性可[参考](https://babeljs.io/docs/en/babel-preset-env)

### e.g.

index.js
```js

export const oProm = new Promise((resolve, reject) => {
    resolve(100);
});

class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  };
  getX() {
    return this.x;
  }
}

export let oPoint = new Point(1, 2);
export const isHas = [1,2,3].includes(2);

```

babel.config.js
```js
  plugins: [
    babel({
      presets: [[
        '@babel/preset-env',
        {
          modules: false,
          targets: {
            browsers: '> 1%, IE 11, not op_mini all, not dead',
            node: 8
          },
          useBuiltIns: 'usage'
        }
      ]],
    }),
  ]
```

index.build.js
```js
'use strict';

Object.defineProperty(exports, '__esModule', { value: true });

require('core-js/modules/es.array.includes.js');
require('core-js/modules/es.object.to-string.js');
require('core-js/modules/es.promise.js');

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}

function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  return Constructor;
}

var oProm = new Promise(function (resolve, reject) {
  resolve(100);
});

var Point = /*#__PURE__*/function () {
  function Point(x, y) {
    _classCallCheck(this, Point);

    this.x = x;
    this.y = y;
  }

  _createClass(Point, [{
    key: "getX",
    value: function getX() {
      return this.x;
    }
  }]);

  return Point;
}();

var oPoint = new Point(1, 2);
var isHas = [1, 2, 3].includes(2);

exports.isHas = isHas;
exports.oPoint = oPoint;
exports.oProm = oProm;
```

demo地址 [@babel/preset demo](https://github.com/Xiamer/babel-example/blob/main/rollup.config.preset.env.js)
我们通过配置 `useBuiltIns: 'usage'` 实现了 `按需加载` polyfill。但编译后的代码中，`_classCallCheck` 就是一个辅助功能实现的工具函数。如果多个文件中都用到了`class`，每一个文件编译后都生成一个工具函数，最后就会产生大量重复代码，增加文件体积，并且引入新特性被暴露在全局或直接修改了原型的方法，而 `plugin-transform-runtime` 就是为了解决这个问题。


## @babel/plugin-transform-runtime

### 简介

`@babel/plugin-transform-runtime` 是一个可以重复使用 `Babel` 注入的帮助程序，以节省代码大小的插件。

`@babel/plugin-transform-runtime` 需要和 `@babel/runtime` 配合使用。


首先安装依赖，`@babel/plugin-transform-runtime` 通常仅在开发时使用，但是运行时最终代码需要依赖 `@babel/runtime`，所以 `@babel/runtime` 必须要作为生产依赖被安装:

```shell
npm install --save-dev @babel/plugin-transform-runtime
npm install --save @babel/runtime
```

`@babel/plugin-transform-runtime` 可以减少编译后代码的体积外，我们使用它还有一个好处，它可以为代码创建一个沙盒环境，如果使用 `@babel/polyfill` 及其提供的内置程序（例如 `Promise` ，`Set` 和 `Map` ），则它们将污染全局范围。虽然这对于应用程序或命令行工具可能是可以的，但是如果你的代码是要发布供他人使用的库，或者无法完全控制代码运行的环境，则将成为一个问题。


`@babel/plugin-transform-runtime` 会将这些内置别名作为 `core-js` 的别名，因此您可以无缝使用它们，而无需 `polyfill`。


### e.g.
index.js
```js
export const oProm = new Promise((resolve, reject) => {
    resolve(100);
});

class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  };
  getX() {
    return this.x;
  }
}

export let oPoint = new Point(1, 2);
```

babel.config.js
```js
{
  plugins: [
    babel({
      babelHelpers: 'runtime',
      presets: [[
        '@babel/preset-env',
        {
          modules: false,
          targets: {
            browsers: '> 1%, IE 11, not op_mini all, not dead',
            node: 8
          },
          useBuiltIns: 'usage'
        }
      ]],
      plugins: [
        ['@babel/plugin-transform-runtime',
          {
            useESModules: true,
            corejs: {
              version: 3,
              proposals: true
            }
          }
        ]
      ]
    }),
  ]
}
```

bundle.js
```js
'use strict';

Object.defineProperty(exports, '__esModule', { value: true });

var _includesInstanceProperty = require('@babel/runtime-corejs3/core-js/instance/includes');
var _classCallCheck = require('@babel/runtime-corejs3/helpers/esm/classCallCheck');
var _createClass = require('@babel/runtime-corejs3/helpers/esm/createClass');
var _Promise = require('@babel/runtime-corejs3/core-js/promise');

function _interopDefaultLegacy (e) { return e && typeof e === 'object' && 'default' in e ? e : { 'default': e }; }

var _includesInstanceProperty__default = /*#__PURE__*/_interopDefaultLegacy(_includesInstanceProperty);
var _classCallCheck__default = /*#__PURE__*/_interopDefaultLegacy(_classCallCheck);
var _createClass__default = /*#__PURE__*/_interopDefaultLegacy(_createClass);
var _Promise__default = /*#__PURE__*/_interopDefaultLegacy(_Promise);

var _context;

var oProm = new _Promise__default['default'](function (resolve, reject) {
  resolve(100);
});

var Point = /*#__PURE__*/function () {
  function Point(x, y) {
    _classCallCheck__default['default'](this, Point);

    this.x = x;
    this.y = y;
  }

  _createClass__default['default'](Point, [{
    key: "getX",
    value: function getX() {
      return this.x;
    }
  }]);

  return Point;
}();

var oPoint = new Point(1, 2);
var isHas = _includesInstanceProperty__default['default'](_context = [1, 2, 3]).call(_context, 2);

exports.isHas = isHas;
exports.oPoint = oPoint;
exports.oProm = oProm;

```

可以看出，帮助函数现在不是直接被 `inject` 到代码中，而是从 `@babel/runtime-corejs` 中引入。 其他特性没有直接去修改 `Array.prototype`，或者是新增 `Promise` 方法，避免了全局污染。


## 参考

* [babel 官方文档](https://babeljs.io/docs/en/)
* [babel 了解](https://juejin.cn/post/6844903743121522701)
* [babel7 相关知识](https://juejin.cn/post/6844904008679686152)