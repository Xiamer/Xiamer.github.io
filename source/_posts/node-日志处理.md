
---
title: node 日志处理 winston
date: 2020-09-23 17:42:14
tags:
---

对于一个服务端项目，日志尤为重要。遇到问题，查看`log`快速定位问题。
node有一些好用的日志库，例如 [winston](https://github.com/winstonjs/winston)、[log4js](https://github.com/log4js-node/log4js-node)、[bunyan](https://github.com/trentm/node-bunyan) 等等，本文主要介绍 `winston`。


## 基本使用

```js
// index.js
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  transports: [
    new winston.transports.Console({
    }),
    new winston.transports.File({
      filename: 'info.log',
      level: 'info'
    })
  ]
});

logger.error('error'); 
logger.info('info');

// node index.js
```
`node index.js`执行后，控制台会打印
```
``js
{"message":"error","level":"error"}
{"message":"info","level":"info"}
```

同时生成`info.log`文件
```js
{"message":"error","level":"error"}
{"message":"info","level":"info"}
```

## Transport


winston将输出的目标抽象成`Transport`对象。winston提供了四种基础的`Transport`：

1. `winston.transports.Console(options)`

输出到控制台，其中`options`：

`level`，日志级别，默认`'info'`, 只有低于等于当前`level`时，才会进入该`Transport`。
`timestamp`，布尔值，`true`则自动加入时间戳；函数，返回的值作为`timestamp`加入到`formatter`的`options`中，默认值是`false`。
`json`，布尔值，是否以JSON格式输出，默认是`false`。
`stringify`，布尔值，是否调用`JSON.stringify()`输出，默认是`false`。
`prettyPrint`，布尔值，是否格式化输出，默认是`false`。
`depth`，数值，对象显示的深度，默认是null，即不限制。
`formatter`，格式化函数，用这个函数的返回值代替输出，默认是undefined。

2. `winston.transports.File(options)`

输出到文件，其中`options`：

`level`，同`Console`。
`name`，字符串，这个`Transport`对象的标识。
`timestamp`，同Console，默认值是true。
`filename`，字符串，输出的文件路径。
`maxsize`，数值，当文件大小超过这个数值则会回滚。
`maxFiles`，数值，最大的文件个数。
`json`，同Console，默认值是true。
`zippedArchive`，布尔值，是否压缩回滚的日志文件。

3. `winston.transports.Http(options)`

输出到`Http`服务，其中`options`：

`host`，字符串，服务的主机名，默认是`'localhost'`。
`port`，数值，服务的端口号，默认是`80`或者`443`（使用`ssl`）。
`path`，字符串，服务的路径，默认是`'/'`。
`auth`，对象，提供`username`和`password`进行`HTTP`鉴权，默认是`undefined`。
`ssl`，布尔值，是否使用`HTTPS`，默认是`false`。


`winston`还提供了一个根据日期处理的`Transport`，可以安时间<font color=red>切割日志</font>，以另一个npm库`winston-daily-rotate-file`提供：

基本功能同`File`，其中：

`datePattern`，字符串，日期格式，默认是`'yyyy-MM-dd'`。
`prepend`，布尔值，文件名是否以日期开始，默认是`false`。
`localTime`，布尔值，是否使用本地时间，默认是`false`，即使用UTC时间。
`maxDays`，数值，日志最多保存天数，默认是0，即不删除任何日志。
`createTree`，布尔值，是否使用文件夹存储日志。


## nuxt 日志处理

```js
// nuxt.config.js
const {loggerOptions} = require('./winston-log')
module.exports = {
  // ...,
  modules: [
    // ...,
    ['nuxt-winston-log']
  ],
    winstonLog: {
    useDefaultLogger: false,
    autoCreateLogPath: false,
    loggerOptions
  },
}
```

```js
const util = require('util');
const DailyRotateFile = require('winston-daily-rotate-file');
const { createLogger, format, transports } = require('winston');
const { combine, timestamp, label, printf, prettyPrint } = format;

// 可修改console.log 使console.log 直接上报。
// const infoLog = console.log
// console.log = function(...arg) {
//   process.winstonLog && process.winstonLog.info(...arg)
//   infoLog.call(console, ...arg)
// }

// const errorLog = console.error
// console.error = function(...arg) {
//   process.winstonLog && process.winstonLog.error(...arg)
//   errorLog.call(console, ...arg)
// }


function transform(info, opts) {
  const args = info[Symbol.for('splat')];
  if (args) { info.message = util.format(info.message, ...args); }
  return info;
}

function utilFormatter() { return {transform}}

const loggerOptions = {
  level: 'info',
  transports: [
    new DailyRotateFile({
      level: 'info',
      filename: 'info-%DATE%.log',
      dirname: './logs',
      datePattern: 'YYYY-MM-DD', // 日志文件
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d',
      format: combine(
        label({ label: 'info-label' }),
        timestamp({format: 'YYYY-MM-DD HH:mm:ss'}),
        utilFormatter(),
        printf(
          ({level, message, label, timestamp}) => `${timestamp} ${label || '-'} ${level}: ${message}`
          )
          ,
        // prettyPrint()
        ),
    }),
    new DailyRotateFile({
      level: 'error',
      filename: 'error-%DATE%.log',
      dirname: './logs',
      datePattern: 'YYYY-MM-DD', // 日志文件
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d',
      format: combine(
        label({ label: 'error-label' }),
        timestamp({format: 'YYYY-MM-DD HH:mm:ss'}),
        utilFormatter(),
        printf(
          ({level, message, label, timestamp}) => `${timestamp} ${label || '-'} ${level}: ${message}`
          )
        // prettyPrint()
        ),
    })
  ]
}

module.exports = {
  loggerOptions
}

```

## 自定义transport

`winston`中的`level`上小于等于`level`会触发`transport`，若想等于某`level`时，触发我们可自定义`transport`

```js
const winston = require('winston');
const Transport = require('winston-transport');

// Inherit from `winston-transport` so you can take advantage
// of the base functionality and `.exceptions.handle()`.
//
class CustomTransport extends Transport {
  constructor(opts) {
    super(opts);

    //
    // Consume any custom options here. e.g.:
    // - Connection information for databases
    // - Authentication information for APIs (e.g. loggly, papertrail,
    //   logentries, etc.).
    //
  }

  log(info, callback) {
    setImmediate(() => {
      // 触发log，仅当level===error时触发。
      if (info.level === 'error') {
      this.emit('logged', info);
      }
    });

    // Perform the writing to the remote service

    callback();
  }
};

const transport = new CustomTransport();
transport.on('logged', (info) => {
  // Verification that log was called on your transport
  // console.log(`Logging! It's happening!`, info);
});

// Create a logger and consume an instance of your transport
const logger = winston.createLogger({
  level: 'info',
  transports: [transport]
});

logger.error('hello')
logger.info('word')
```

## node项目日志实战

```js
// log.js
const BRAND_ENV = process.env.BRAND;    // 环境变量
const ROBOT_ID = process.env.ROBOT_ID || 1;    // 环境变量
const NODE_ENV = process.env.NODE_ENV;
const TYPE = process.env.TYPE
const VPS = process.env.VPS || 'all';
const path = require('path');
const fs = require('fs');
const util = require('util');
const { createLogger, format, transports } = require('winston');
const { combine, timestamp, label, printf } = format;

const myFormat = printf(info => {
  return `${(new Date(info.timestamp)).toLocaleString('chinese', { hour12: false })}: ${info.message}`;
});
let logger;
let sTodayDate;
if (NODE_ENV === 'production') {
  // 今天的日期
  sTodayDate = (new Date()).toLocaleDateString();
  // logger对象，调用logger.info即可输出日志
  logger = fnBsnCreateLogger(sTodayDate);
  // 重载logger.info
  fnBsnOverLoadLogger();

  // 重载console.log
  console.log = function () {
    logger.info.apply(logger, formatArgs(arguments));
  };
  // 把logger挂载到全局(全局可以访问logger)
  global.logger = logger;
}

function fnBsnOverLoadLogger() {
  // 重载logger.info，在logger执行前判断需不需要新建新一天的日志文件
  logger.info = (function (oriLogFunc) {
    return function (str) {
      // 如果到了新的一天
      if ((new Date()).toLocaleDateString() !== sTodayDate) {
        // 更新logger的配置
        sTodayDate = (new Date()).toLocaleDateString();
        logger = fnBsnCreateLogger(sTodayDate);
        // 重载logger.info
        fnBsnOverLoadLogger();
      }
      oriLogFunc.call(logger, str);
    }
  })(logger.info);
}

/**
 * 创建logger对象
 * @param {string} date 日期
 */
function fnBsnCreateLogger(date) {
  let sFileName = '';
  if (TYPE === 'xxxx') {
    createFolder(`./log/xxxx-${BRAND_ENV.toLowerCase()}-R${ROBOT_ID}/${date}.log`);
    sFileName = `./log/xxxx-${BRAND_ENV.toLowerCase()}-R${ROBOT_ID}/${date}.log`;
  }
  else {
    createFolder(`./log/logistic-${VPS}/${date}.log`);
    sFileName = `./log/logistic-${VPS}/${date}.log`;
  }
  return createLogger({
    format: combine(
      timestamp(),
      myFormat
    ),
    transports: [
      new transports.Console(),
      new transports.File({
        filename: sFileName
      })
    ]
  });
}

// 根据文件路径创建文件夹
function createFolder(to) {
  let sep = path.sep
  let folders = path.dirname(to).split(sep);
  let p = '';
  while (folders.length) {
    p += folders.shift() + sep;
    if (!fs.existsSync(p)) {
      fs.mkdirSync(p);
    }
  }
};

// 把多个参数如 ('test', 'test') => 'test test'
function formatArgs(args) {
  return [util.format.apply(util.format, Array.prototype.slice.call(args))];
}

```


## 参考

* [github winston](https://github.com/winstonjs/winston)
* [How to make `winston` logging library work like `console.log`?](https://stackoverflow.com/questions/55387738/how-to-make-winston-logging-library-work-like-console-log)
* [Node.js日志神器 winston](https://juejin.cn/post/6865926810061045774)