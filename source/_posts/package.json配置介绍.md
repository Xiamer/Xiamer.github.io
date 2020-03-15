---
title: package.json 配置介绍
date: 2020-03-15 16:03:08
categories:
- nodejs
tags:
- nodejs
- npm
---

## 概述

每个项目的根目录下面，一般都有一个package.json文件，定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证等元数据）。npm install命令根据这个配置文件，自动下载所需的模块，也就是配置项目所需的运行和开发环境。

## 字段简介

### name 
name: "name"
项目名称。若打算发包，name 和 version 是必填，组成唯一标识符。

### version
version: "1.0.0"
版本号，遵守“大版本.次要版本.小版本”的格式。

### description
description: "description"
项目描述，方便在npm search 查找。

### keyword
keyword: ["keyword1", "keyword2"]
关键词，方便在npm search 查找。

### homepage
homepage："https://github.com/owner/project#readme"
项目首页的网址

### bugs
```json
{
  "url": "https://github.com/owner/project/issues",
  "email": "project@hostname.com"
}
```
项目问题收集的网址或电子邮件地址。

### license
license: "ISC"
许可证，可设置为 `UNLICENSED` 不允许他人使用。

### author and contributors

```json

{
  "author": "xg <xiaoguang_10@qq.com> (https://www.a.com)",
  "contributors": [
    { "name": "xg", "email": "xiaoguang_10@qq.com", "url": "https://www.a.com"}
  ]
}

```
项目贡献,数组的元素可设置为String，npm 会将会解析。如： "xg <xiaoguang_10@qq.com> (https://www.a.com)"

### files

file: ['package.json', 'README', '....']
可选字段，项目包含的一组文件，当你的package被引入（dependency）时所包含的文件。如果是文件夹，文件夹下的文件也会被包含。如果需要把某些文件不包含在项目中，添加一个”.npmignore”文件。这个文件和”gitignore”类似。

### main

"main": "lib/index.js"

main字段指定了模块的**入口文件**。就是说，如果你的模块名叫"moduleName"，用户安装了它，并且调用了 require("moduleName")，则这个main字段指定的模块的导出对象会被返回。例如node_modules中引入的模块指定主入口文件。这个字段的默认值是模块根目录下面的index.js。
定义了 npm 包的入口文件，browser 环境和 node 环境均可使用。


### browser

```json
"browser": {
  "./lib/index.js": "./lib/index.browser.js", // browser+cjs
  "./lib/index.mjs": "./lib/index.browser.mjs"  // browser+mjs
}
```

browser指定该模板供**浏览器**使用的版本。Browserify这样的浏览器打包工具，通过它就知道该打包那个文件。
定义 npm 包在 browser 环境下的入口文件。

### bin

bin项用来指定各个内部命令对应的可执行文件的位置。
这里提供一个pm2的bin例子:

```json
{
  "bin": {
    "pm2": "./bin/pm2",
    "pm2-dev": "./bin/pm2-dev",
    "pm2-docker": "./bin/pm2-docker",
    "pm2-runtime": "./bin/pm2-runtime"
  }
}
```

输入pm2实际上是运行{模块所在目录}/bin/pm2。以此类推.

带有bin信息的包, 在局部安装后, 可执行文件会在./node_modules/.bin下,

如果是全局安装, 可执行文件会在 $PATH 里对应npm那个目录下.

### man

```json
{ 
  "name" : "foo",
  "version" : "0.1.1",
  "description" : "foo",
  "main" : "foo.js",
  "man" : "./man/doc.1"
}
```

制定一个或通过数组制定一些文件来让linux下的man命令查找文档地址。
如果只有一个文件被指定的话，安装后直接使用man+模块名称，而不管man指定的文件的实际名称
通过man foo命令会得到 ./man/doc.1 文件的内容。
如果man文件名称不是以模块名称开头的，安装的时候会给加上模块名称前缀。因此，下面这段配置：

```json
{ 
  "name" : "foo",
  "version" : "1.2.3",
  "description" : "foo",
  "main" : "foo.js",
  "man" : [ "./man/foo.1", "./man/bar.1" ]
}
```

会创建一些文件来作为man foo和man foo-bar命令的结果。
man文件必须以数字结尾，或者如果被压缩了，以.gz结尾。数字表示文件将被安装到man的哪个部分。

```json
{ 
  "name" : "foo",
  "version" : "0.1.1",
  "description" : "foo",
  "main" : "foo.js",
  "man" : [ "./man/foo.1", "./man/foo.2" ]
}
```

会创建 man foo 和 man 2 foo 两条命令。
可查考 [linux man](https://man.linuxde.net/man)

### directories

CommonJs通过directories来制定一些方法来描述模块的结构，看看npm的package.json文件https://registry.npmjs.org/npm/latest ，可以发现里边有这个字段的内容。

目前这个配置没有任何作用，将来可能会有作用。

#### directories.lib

告诉用户模块中lib目录在哪，这个配置目前没有任何作用，但是对使用模块的人来说是一个很有用的信息。

#### directories.bin

如果你在这里指定了bin目录，这个配置下面的文件会被加入到bin路径下，如果你已经在package.json中配置了bin目录，那么这里的配置将不起任何作用。

#### directories.man

指定一个目录，目录里边都是man文件，这是一种配置man文件的语法糖。

#### directories.doc

在这个目录里边放一些markdown文件，可能最终有一天它们会被友好的展现出来（应该是在npm的网站上）

#### directories.example

放一些示例脚本，以后某一天会有用。

#### directories.test

放一些测试脚本，目前尚未公开，但可能会在将来公开

### repository

指定一个代码存放地址，对想要为你的项目贡献代码的人有帮助。

```json
{
  "repository": 
  { 
    "type" : "git", 
    "url" : "https://github.com/npm/npm.git"
  },
  "repository" :
  {
     "type" : "svn",
     "url" : "https://v8.googlecode.com/svn/trunk/"
  }
}

```

若你的模块放在GitHub, GitHub gist, Bitbucket, or GitLab的仓库里，npm install的时候可以使用缩写标记来完成

```code
"repository": "npm/npm"

"repository": "gist:11081aaa281"

"repository": "bitbucket:example/repo"

"repository": "gitlab:another/repo"
```

### script 

scripts属性是一个对象，里边指定了项目的生命周期个各个环节需要执行的命令。key是生命周期中的事件，value是要执行的命令。

详见 https://docs.npmjs.com/misc/scripts

### config

用来设置一些项目不怎么变化的项目配置，例如port等。
用户用的时候可以使用如下用法：

```code
http.createServer(...).listen(process.env.npm_package_config_port)

{ 
  "name" : "foo",
  "config" : { "port" : "8080" }
}
```

可以通过 `npm_package_config_port` 获取变量。
可以通过npm config set foo:port 80来修改config。
可查看 [npm-config](https://docs.npmjs.com/misc/config)和[npm-script](https://docs.npmjs.com/misc/scripts)获取更多信息。

### dependencies

dependencies属性是一个对象，配置模块依赖的模块列表，key是模块名称，value是版本范围，版本范围是一个字符，可以被一个或多个空格分割。
dependencies也可以被指定为一个git地址或者一个压缩包地址。

简而言之：**dependencies字段指定了项目运行所依赖的模块**。

不要把测试工具或transpilers写到dependencies中。 下面是一些写法，详见https://docs.npmjs.com/misc/semver

* version 精确匹配版本
* `>`version 必须大于某个版本
* `>`=version 大于等于
* <version 小于
* <=versionversion 小于
* ~version "约等于"，具体规则详见semver文档
* ^version "兼容版本"具体规则详见semver文档
* 1.2.x 仅一点二点几的版本
* http://... 见下面url作为denpendencies的说明
* 任何版本
* "" 空字符，和*相同
* version1 - version2 相当于 >=version1 <=version2.
* range1 || range2 范围1和范围2满足任意一个都行
* git... 见下面git url作为denpendencies的说明
* user/repo See 见下面GitHub仓库的说明
* tag 发布的一个特殊的标签，见npm-tag的文档 https://docs.npmjs.com/getting-started/using-tags
* path/path/path 见下面本地模块的说明

下面的写法都是可以的

```json
{ 
  "dependencies" :
  { "foo" : "1.0.0 - 2.9999.9999",
  "bar" : ">=1.0.2 <2.1.2",
  "baz" : ">1.0.2 <=2.3.4",
  "boo" : "2.0.1",
  "qux" : "<1.0.0 || >=2.3.1 <2.4.5 || >=2.5.2 <3.0.0",
  "asd" : "http://asdf.com/asdf.tar.gz",
  "til" : "~1.2",
  "elf" : "~1.2.3",
  "two" : "2.x",
  "thr" : "3.3.x",
  "lat" : "latest",
  "dyl" : "file:../dyl"
  }
}
```


### devDependencies

如果有人想要下载并使用你的模块，也许他们并不希望或需要下载一些你在开发过程中使用的额外的测试或者文档框架。
在这种情况下，最好的方法是把这些依赖添加到devDependencies属性的对象中。
这些模块会在npm link或者npm install的时候被安装，也可以像其他npm配置一样被管理，详见npm的config文档。
对于一些跨平台的构建任务，例如把CoffeeScript编译成JavaScript，就可以通过在package.json的script属性里边配置prepublish脚本来完成这个任务，然后需要依赖的coffee-script模块就写在devDependencies属性中。

简而言之：**devDependencies指定项目开发所需要的模块**。

### URLs as Dependencies

在版本范围的地方可以写一个url指向一个压缩包，模块安装的时候会把这个压缩包下载下来安装到模块本地。

### Git URLs as Dependencies

```code
git://github.com/user/project.git#commit-ish
git+ssh://user@hostname:project.git#commit-ish
git+ssh://user@hostname/project.git#commit-ish
git+http://user@hostname/project/blah.git#commit-ish
git+https://user@hostname/project/blah.git#commit-ish
```

commit-ish 可以是任意标签，哈希值，或者可以检出的分支，默认是master分支

### GitHub URLs

支持github的 username/modulename 的写法，#后边可以加后缀写明分支hash或标签：

```json
{
  "name": "foo",
  "version": "0.0.0",
  "dependencies": {
    "express": "expressjs/express",
    "mocha": "mochajs/mocha#4727d357ea",
    "module": "user/repo#feature\/branch"
  }
}
```

### peerDependencies

有时候做一些插件开发，比如grunt等工具的插件，它们往往是在grunt的某个版本的基础上开发的，而在他们的代码中并不会出现require("grunt")这样的依赖，dependencies配置里边也不会写上grunt的依赖，为了说明此模块只能作为插件跑在宿主的某个版本范围下，可以配置peerDependencies：

```json
{
  "name": "tea-latte",
  "version": "1.3.5",
  "peerDependencies": {
    "tea": "2.x"
  }
}
```

上面这个配置确保再npm install的时候tea-latte会和2.x版本的tea一起安装，而且它们两个的依赖关系是同级的：
├── tea-latte@1.3.5
└── tea@2.2.0
这个配置的目的是让npm知道，如果要使用此插件模块，请确保安装了兼容版本的宿主模块。


### Local Paths

npm2.0.0版本以上可以提供一个本地路径来安装一个本地的模块，通过npm install xxx --save 来安装，格式如下：

```code
../foo/bar
~/foo/bar
./foo/bar
/foo/bar
```

package.json 生成的相对路径如下:

```json
{
  "name": "baz",
  "dependencies": {
    "bar": "file:../foo/bar"
  }
}
```

这种属性在离线开发或者测试需要用npm install的情况，又不想自己搞一个npm server的时候有用，但是发布模块到公共仓库时不应该使用这种属性。


### bundledDependencies

指定发布的时候会被一起打包的模块。

### optionalDependencies

如果一个依赖模块可以被使用， 同时你也希望在该模块找不到或无法获取时npm继续运行，你可以把这个模块依赖放到optionalDependencies配置中。这个配置的写法和dependencies的写法一样，不同的是这里边写的模块安装失败不会导致npm install失败。
当然，这种模块就需要你自己在代码中处理模块确实的情况了，例如：

```js
try {
  var foo = require('foo')
  var fooVersion = require('foo/package.json').version
} catch (er) {
  foo = null
}
if ( notGoodFooVersion(fooVersion) ) {
  foo = null
}

// .. then later in your program ..

if (foo) {
  foo.doFooThings()
}
```

optionalDependencies 中的配置会覆盖dependencies中的配置，最好只在一个地方写。

### engines

你可以指定项目运行的node版本范围，如下：
{ "engines" : { "node" : ">=0.10.3 <0.12" } }
和dependencies一样，如果你不指定版本范围或者指定为*，任何版本的node都可以。
也可以指定一些npm版本可以正确的安装你的模块，例如：
{ "engines" : { "npm" : "~1.0.20" } }
要注意的是，除非你设置了**engine-strict**属性，**engines属性是仅供参考**的。

### os

可以指定你的模块只能在哪个操作系统上跑：
"os" : [ "darwin", "linux" ]
也可以指定黑名单而不是白名单：
"os" : [ "!win32" ]
服务的操作系统是由process.platform来判断的，这个属性允许黑白名单同时存在.

### cpu

限制模块只能在某某cpu架构下运行
"cpu" : [ "x64", "ia32" ]
同样可以设置黑名单:
"cpu" : [ "!arm", "!mips" ]
cpu架构通过 `process.arch` 判断

### preferGlobal

如果您的软件包主要用于安装到全局的命令行应用程序，那么该值设置为true ，如果它被安装在本地，则提供一个警告。实际上该配置并没有阻止用户把模块安装到本地，只是防止该模块被错误的使用引起一些问题。

### private

如果这个属性被设置为true，npm将拒绝发布它，这是为了防止一个私有模块被无意间发布出去。如果你只想让模块被发布到一个特定的npm仓库，如一个内部的仓库，可与在下面的publishConfig中配置仓库参数。

### publishConfig

这个配置是会在模块发布时用到的一些值的集合。如果你不想模块被默认被标记为最新的，或者默认发布到公共仓库，可以在这里配置tag或仓库地址。

### DEFAULT VALUES

npm设置了一些默认参数，如：
"scripts": {"start": "node server.js"}
如果模块根目录下有一个server.js文件，那么npm start会默认运行这个文件。
"scripts":{"preinstall": "node-gyp rebuild"}
如果模块根目录下有binding.gyp, npm将默认用node-gyp来编译preinstall的脚本
"contributors": [...]
若模块根目录下有AUTHORS 文件，则npm会按Name (url)格式解析每一行的数据添加到contributors中，可以用#添加行注释

### 查看更多

* [semver](https://docs.npmjs.com/misc/semver)
* [npm-init](https://docs.npmjs.com/cli/init)
* [npm-version](https://docs.npmjs.com/cli/version)
* [npm-uninstall](https://docs.npmjs.com/cli/uninstall)
* [npm-publish](https://docs.npmjs.com/cli/publish)
* [npm-install](https://docs.npmjs.com/cli/publish)
* [npm-help](https://docs.npmjs.com/cli/publish)


## 参考链接

* [docs npm](https://docs.npmjs.com/)
* [docs npm package.json](https://docs.npmjs.com/files/package.json)
* [zoucz blog npm package.json](https://zoucz.com/blog/2016/02/17/npm-package/)