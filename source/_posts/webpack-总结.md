---
title: webpack 总结
date: 2021-04-28 15:11:10
categories: 
- 前端
tags:
- webpack
---


## module chunk bundle

* Module 每个文件都是一个module
```js
module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: "style-loader"
          }, {
            loader: "css-loader"
          }
        ]
      },
      ...
    ]
  },

```
* Chunk Webpack打包过程中，一堆module的集合。 (从一个入口文件开始，入口模块引用这其他模块，模块再引用模块。Webpack通过引用关系逐个打包模块，这些module就形成了一个Chunk。)
* Bundle就是我们最终输出的一个或多个打包文件。

> Chunk 是过程中的代码块，Bundle是结果的代码块。
> 一个Chunk是一些模块的封装单元。Chunk在构建完成就呈现为bundle。

一个或多个moudle对应一个chunk
一个chunk 对应一个或多个bundle

## 产生Chunk的三种途径 （entry, 异步加载模块，代码分割splitChunks）

### 多entry产生chunk 

```js
entry: {
    main: './src/js/main.js',
    other: './src/js/other.js'
},
output: {
    path: __dirname + "/public",//打包后的文件存放的地方
    filename: "[name].js", //打包后输出文件的文件名
}
```

### 异步加载模块产生Chunk

1. webpack 特定的 require.ensure
```js
{
    entry: {
        "index": "pages/index.jsx"
    },
    output: {
         filename: "[name].min.js",
        chunkFilename: "[name].min.js"
    }
}
const myModel = r => require.ensure([], () => r(require('./myVue.vue')), 'myModel')
```

2.  import() 语法 来实现动态导入
```js
const loader = (path, name) => () => import(/* webpackChunkName: `${name}` */  `${path}`)
// 或
import(/* webpackChunkName: "plugins/myModule" */
    './myModule.js')
    .then(myModule => {
        console.log(myModule.default);
    });
```

### splitChunks 产生chunk

```js
entry: {
  index: './src/index.js',
},
optimization: {
    minimize: env === 'production' ? true : false, // 开发环境不压缩
    // 包含chunks 映射关系的 list单独从 app.js里提取出来，
    // 因为每一个 chunk 的 id 基本都是基于内容 hash 出来的，所以每次改动都会影响它，
    // 如果不将它提取出来的话，等于app.js每次都会改变。缓存就失效了。
    // 设置runtimeChunk之后，webpack就会生成一个个runtime~xxx.js的文件。
    // 可将 runtimeChunk 插入到到script内部，减少http请求
    //   （使用）new ScriptExtHtmlWebpackPlugin({
    //           inline: /runtime~.+\.js$/  //正则匹配runtime文件名
    //          })
    // 将当前模块的记录其他模块的hash单独打包为一个文件 runtime
    runtimeChunk: "single",
    splitChunks: {
        chunks: "async", // 共有三个值可选：initial(初始模块)、async(按需加载模块)和all(全部模块)
        minSize: 30 * 1024, // 模块超过30k自动被抽离成公共模块
        minChunks: 1, // 模块被引用>=1次，便分割
        maxAsyncRequests: 5,  // 异步加载chunk的并发请求数量<=5
        maxInitialRequests: 3, // 一个入口并发加载的chunk数量<=3
        automaticNameDelimiter: '~', // 命名分隔符
        name: true, // 默认由模块名+hash命名，名称相同时多个模块将合并为1个，可以设置为function
        cacheGroups: { // 缓存组，会继承和覆盖splitChunks的配置，
            // node_modules 文件会打包到 vendoors组的chunk中。 --> vendors~xxx.js
            // 满足上面的公共规则 如：大小超过30k 至少引用一次
            vendors: {
                test: /[\\/]node_modules[\\/]/, // 表示默认拆分node_modules中的模块
                priority: -10
            }
        },
        commons: { // 模块缓存规则，设置为false，默认缓存组将禁用
            minChunks: 2, // 模块被引用>=1次，拆分至common公共模块
            priority: -20, // 优先级
            reuseExistingChunk: true, // 如果当前要打包的模块和只之前已被提取的模块是同一个，就会复用，而不是重新打包。（多页面 都引入 jquery时）
        },
    }
}
```
chunk 4个 index(入口)、runtime(runtimeChunk)、commons~index(cacheGroups commons)、vendor(cacheGroups vendors)

## filename和chunkFilename的区别

* output.filename 决定了每个入口(entry) 输出 bundle 的名称。
* output.chunkFilename 决定了非入口(non-entry) chunk 文件的名称。

## 常用loader

* file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件 (处理图片和字体)
* url-loader：与 file-loader 类似，区别是用户可以设置一个阈值，大于阈值会交给 file-loader 处理，小于阈值时返回文件 base64 形式编码 (处理图片和字体)

* sass-loader：将SCSS/SASS代码转换成CSS
* css-loader：会对 @import 和 url() 进行处理，就像 js 解析 import/require() 一样, 返回的是 js 串。
* style-loader：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS(变为style标签)
* postcss-loader：扩展 CSS 语法，使用下一代 CSS，可以配合 autoprefixer 插件自动补齐 CSS3 前缀

* source-map-loader：加载额外的 Source Map 文件，以方便断点调试
* image-loader：加载并且压缩图片文件
* babel-loader：把 ES6 转换成 ES5
* ts-loader: 将 TypeScript 转换成 JavaScript
* awesome-typescript-loader：将 TypeScript 转换成 JavaScript，性能优于 ts-loader
* vue-loader：加载 Vue.js 单文件组件

* eslint-loader：通过 ESLint 检查 JavaScript 代码
* tslint-loader：通过 TSLint检查 TypeScript 代码

* i18n-loader: 国际化
* cache-loader: 可以在一些性能开销较大的 Loader 之前添加，目的是将结果缓存到磁盘里

## loader normal && pitch 

[参考 loader原理](https://zhuanlan.zhihu.com/p/104205895)

> loader: [a, b, c]

```pre
|- a-loader `pitch`
  |- b-loader `pitch`
    |- c-loader `pitch`
      |- requested module is picked up as a dependency
    |- c-loader normal execution
  |- b-loader normal execution
|- a-loader normal execution
```

* 无pitch， loader 从右向左执行。
* 有pitch，loader 的执行则会分为两个阶段：`pitch` 阶段 和 `normal execution` 阶段。
  * webpack 会先从左到右执行 loader 链中的每个 loader 上的 pitch 方法（如果有），
  * 然后再从右到左执行 loader 链中的每个 loader 上的普通 loader 方法。

### loader为什么是自右向左执行的？

`pitch` -> `iteratePitchingLoaders()`, `normal` -> `iterateNormalLoaders()`

`iteratePitchingLoaders()`会递归执行，并记录loader的`pitch`状态与当前执行到的`loaderIndex（loaderIndex++）`。当达到最大的loader序号时，才会处理实际的module：

```js
if(loaderContext.loaderIndex >= loaderContext.loaders.length)
    return processResource(options, loaderContext, callback);
```

当`loaderContext.loaderIndex`值达到整体loader数组长度时，表明所有pitch都被执行完毕（执行到了最后的loader），这时会调用`processResource()`来处理模块资源。主要包括：添加该模块为依赖和读取模块内容。然后会递归执行`iterateNormalLoaders()`并进行`loaderIndex--`操作，因此loader会“反向”执行。

### style-loader 和 css-loader

实现style-loader

1. style-loader 最终需返回一个 `js` 脚本：在脚本中创建一个 `style` 标签，将 `css` 代码赋给 `style` 标签，再将这个 `style` 标签插入 `html` 的 `head` 中。
2. 难点是获取 `css` 代码，因为 css-loader 的返回值只能在<font color=red>运行</font>时的上下文中执行，而执行 loader 是在<font color=red>编译阶段</font>。换句话说，css-loader 的返回值在 style-loader 里派不上用场。
3. 曲线救国方案：使用获取 `css` 代码的表达式，在运行时再获取 css (类似 `require('css-loader!index.css')`）。
4. 在处理 css 的 loader 中又去调用 `inline loader` require `css` 文件，会产生循环执行 loader 的问题，所以我们需要利用 `pitch` 方法，让 style-loader 在 `pitch` 阶段返回脚本，跳过剩下的 loader，同时还需要内联前缀 `!!` 的加持。

> 注：pitch 方法有3个参数：

1. remainingRequest：loader链中排在自己后面的 loader 以及资源文件的绝对路径以`!`作为连接符组成的字符串。
2. precedingRequest：loader链中排在自己前面的 loader 的绝对路径以`!`作为连接符组成的字符串。
3. data：每个 loader 中存放在上下文中的固定字段，可用于 pitch 给 loader 传递数据。

```js
// loaders/simple-style-loader.js
const loaderUtils = require('loader-utils');
module.exports = function(source) {
  // do nothing
}

module.exports.pitch = function(remainingRequest) {
  console.log('simple-style-loader is working');
    // 在 pitch 阶段返回脚本
    return (
      `
      // 创建 style 标签
      let style = document.createElement('style');

      /**
      * 利用 remainingRequest 参数获取 loader 链的剩余部分
      * 利用 ‘!!’ 前缀跳过其他 loader 
      * 利用 loaderUtils 的 stringifyRequest 方法将模块的绝对路径转为相对路径
      * 将获取 css 的 require 表达式赋给 style 标签
      */
      style.innerHTML = require(${loaderUtils.stringifyRequest(this, '!!' + remainingRequest)});
      
      // 将 style 标签插入 head
      document.head.appendChild(style);
      `
    )
}
```

可以这么理解：style-loader 以 pitch 的方式返回了一段类似于 require('!!css-loader!xxx.css') 的标准 js module，当前轮次的 loader 链执行完毕。
Webpack 继续处理返回的 require('!!css-loader!xxx.css') 启动一次新的 run-loader，不同的是这次跳过 style-loader 直接执行 css-loader

## 常用Plugin

* define-plugin：定义环境变量
* html-webpack-plugin：简化 HTML 文件创建 (依赖于 html-loader)
* web-webpack-plugin：可方便地为单页应用输出 HTML，比 html-webpack-plugin 好用
* clean-webpack-plugin: 目录清理
* mini-css-extract-plugin: 分离样式文件，CSS 提取为独立文件，支持按需加载 (替代extract-text-webpack-plugin)

* uglifyjs-webpack-plugin：不支持 ES6 压缩 (Webpack4 以前)
* terser-webpack-plugin: 支持压缩 ES6 (Webpack4)
* webpack-parallel-uglify-plugin: 多进程执行代码压缩，提升构建速度

* ignore-plugin：忽略部分文件
* serviceworker-webpack-plugin：为网页应用增加离线缓存功能
* ModuleConcatenationPlugin: 开启 Scope Hoisting
* speed-measure-webpack-plugin: 可以看到每个 Loader 和 Plugin 执行耗时 (整个打包耗时、每个 Plugin 和 Loader 耗时)
* webpack-bundle-analyzer: 可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)

## Webpack构建流程

* <font color=red>初始化参数</font>：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数
* <font color=red>开始编译</font>：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译
* <font color=red>确定入口</font>：根据配置中的 entry 找出所有的入口文件
* <font color=red>编译模块</font>：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
* <font color=red>完成模块编译</font>：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
* <font color=red>输出资源</font>：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
* <font color=red>输出完成</font>：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

参数 -> init compliler&&run -> entry -> loader&&得到依赖关系 -> chunk -> bundle

## loader 和 plugin 区别

Loader: 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。

Plugin：插件，基于事件流框架 Tapable，插件可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

## sourcemap

* eval:  使用eval包裹<font>模块</font>代码 (在模块末尾添加模块来源<font color=red>//# souceURL</font>)
映射到转换后的代码(可执行的js文件)，而不是映射到原始代码(vue文件)，所以不能正确的显示错误行数。

```js
webpackJsonp([1],[
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceURL=webpack:///./src/js/index.js?'
    )
  },
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceURL=webpack:///./src/static/css/app.less?./~/.npminstall/css-loader/0.23.1/css-loader!./~/.npminstall/postcss-loader/1.1.1/postcss-loader!./~/.npminstall/less-loader/2.2.3/less-loader'
    )
  },
  function(module,exports,__webpack_require__){
    eval(
      ...
      //# sourceURL=webpack:///./src/tmpl/appTemplate.tpl?"
    )
  },
...])
```
* inline 是以dateURL的形式添加map，不额外生成map文件
* cheap 是没有列信息 (并 不包含 loader 的 sourcemap（譬如 babel 的 sourcemap）)
* module 是包含了loader的sourcemap
* source-map 映射到源文件
* hidden 只有错误信息 没有源代码

如如下组合

* souce-map 外部
  * 错误代码准确信息 和 源代码位置
* eval-source-map：内链
  * 每一个文件都生成对应的source-map在eval中。
  * 错误代码准确信息 和 源代码位置
* cheap-source-map 外部
  * 错误代码准确信息 和 源代码位置
  * 只能到行
* hidden-source-map 外部 （与 source-map 相比少了末尾的sourceUrl）
  * 错误代码原因,但是没有错误位置
  * 不能追踪源代码错误，只能提示到构建后代码的错误位置
* nosource-source-map 外部
  * 错误代码准确信息 但没有任何源代码信息


## 使用webpack开发时，你用过哪些可以提高效率的插件 

* webpack-dashboard：可以更友好的展示相关打包信息。 (分析工具)
* webpack-merge：提取公共配置，减少重复配置代码
* speed-measure-webpack-plugin：简称 SMP，分析出 Webpack 打包过程中 Loader 和 Plugin 的耗时，有助于找到构建过程中的性能瓶颈。
* size-plugin：监控资源体积变化，尽早发现问题
* HotModuleReplacementPlugin：模块热替换

## Webpack 的HMR热更新原理

![](/images/webpack/hrm.webp)

> 为什么需要两个hash?
lastHash currentHash

lashHash = currentHash = hash1
第一次客户端和服务端是一致都是hash1
但修改代码，重新编译了
1. 重新得到一个新的hash2
2. 还会创建一个hash1的补丁包，包里会说明hash1到hash2哪些代码块发送了改变，以及发生了哪些变量。

[Webpack HMR 原理解析](https://zhuanlan.zhihu.com/p/30669007)

## 文件指纹是什么？（hash、chunkHash、contentHash）

* Hash：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改
  * 问题：重新打包，所有缓存失效（可能只改一个文件）
* Chunkhash：根据chunk生成hash，如果打开来源同一个chunk，那么hash一样
* Contenthash：根据文件内容来定义 hash，文件内容不变，则 contenthash 不变


## webpackPreload webpackPrefetch

* 懒加载 preload： 当文件需要时加载 （点击 加载某个模块）
* 预加载 prefetch: 等其他资源加载完毕，等浏览器空闲 再加载。

```js
document.getElementById('btn').onclick = function() {
  import(/* webpackChunkNmae: 'test', webpackPrefetch: true */'./test').then()
}
```

## bundle 原理

[webpack 打包原理](https://segmentfault.com/a/1190000021494964)

## babel原理

* 解析（Parsing）：将代码字符串解析成抽象语法树。 Babylon.parse(code)
* 转换（Transformation）：对抽象语法树进行转换操作。babel-traverse
* 生成（Code Generation）: 根据变换后的抽象语法树再生成代码字符串。 babel-generator

## webpack 优化

* loader oneof
* loader rules -> exclude include 
* babel 缓存 cacheDirectory: true，让第二次打包速度更快
* thread-loader 多进程打包 （进程开销也需要消耗）

* tree shaking
* 缓存 (hash-chunkhash-contenthash)
* webpack-parallel-uglify-plugin 多进程
* split code
* 懒加载webpackPreload / 预加载webpackPrepetch
* externals (或 dll)

## webpack5

* 更好的tree shaking （webpack4 模块导出的默认没有shaking）
* 缓存 （持久缓存 提高构建）
* spit chunk -> miniSize: 3000  ~ miniSize: { javascript: 3000, style: 5000 }
* cache: { type: 'filesystem'// 磁盘缓存和内存memory缓存 }

## 参考

* [loader为什么是自右向左执行的？](https://juejin.cn/post/6844903693070909447#heading-11)
* [bilibili webpack热更新](https://www.bilibili.com/video/BV13p4y1x78y?from=search&seid=4745697833291182911)
* [从零实现webpack热更新HMR](https://juejin.cn/post/6844904020528594957#heading-42)
