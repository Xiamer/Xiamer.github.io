
---
title: nuxt 缓存 实战
date: 2020-06-28 15:11:10
categories: 
- 前端
tags:
- 缓存
---

## 页面缓存

在不依赖于登录态以及过多参数的情况下，如果并发量很大，可以考虑使用页面级别的缓存。

### 使用 lru-cache 缓存

#### LRUCache 介绍

LRUCache（Least-Recently-Used） 替换掉最近最少使用的对象.

代码简单实现

```js

var LRUCache = function(capacity) {
  this.cache = new Map()
  this.capacity = capacity
}

LRUCache.prototype.get = function(key) {
  if (this.cache.has(key)) {
    // 存在即更新
    let temp = this.cache.get(key)
    this.cache.delete(key)
    this.cache.set(key, temp)
    return temp
  }
  return -1
}

LRUCache.prototype.put = function(key, value) {
  if (this.cache.has(key)) {
    //  存在即更新（删除后加入）
    this.cache.delete(key)
  } else if (this.cache.size >= this.capacity) {
    // 不存在即加入 
    // this.cache.keys() 返回的是 Iterator
    // 缓存超过最大值，则移除最近没有使用的
    this.cache.delete(this.cache.keys().next().value)
  }
  this.cache.set(key, value)
}

let lru = new LRUCache(2)
lru.put('a', 1)
lru.put('b', 2)
lru.put('a', 3)
lru.cache.keys() // MapIterator {"b", "a"}
lru.get(b)       // 2
lru.get(a)       // 3
```

#### 使用 `lur-cache` 缓存页面

1. 新建 `pageServerCache.js`

```js
const LRU = require('lru-cache');
export const cachePage = new LRU({
  max: 100, // 缓存队列长度 最大缓存数量
  maxAge: 1000 * 10, // 缓存时间 单位：毫秒
});

export default function(req, res, next) {
  const url = req._parsedOriginalUrl;
  const pathname = url.pathname;
  // 本地开发环境不做页面缓存(也可开启开发环境进行调试)
  if (process.env.NODE_ENV !== 'development') {

    // 清理缓存
    if (pathname === '/cleancache') {
      cachePage.reset();
      res.statusCode = 200;
    }

    // 只有首页才进行缓存 页面可自己进行配置
    if (['/'].indexOf(pathname) > -1) {
      const existsHtml = cachePage.get('indexPage');
      if (existsHtml) {
        //  如果没有Content-Type:text/html 的 header，gtmetrix网站无法做测评
        res.setHeader('Content-Type', ' text/html; charset=utf-8');
        return res.end(existsHtml.html, 'utf-8');
      } else {
        // 缓存原先的end方案
        res.original_end = res.end;
        // 重写res.end方案，由此nuxt调用res.end实际上是调用该方法，
        res.end = function(data) {
          if (res.statusCode === 200) {
            // 设置缓存
            cachePage.set('indexPage', {
              html: data,
            });
          }
          res.original_end(data, 'utf-8');
        };
      }
    }
  }
  next();
}
```

2. 在`nuxt.config.js`中配置serverMiddleware，引入

```js
  /*
   ** 服务器端中间件--针对首页做缓存
   */
  serverMiddleware: [
    {
      path: '/',
      handler: '~/utils/server-middleware/pageServerCache.js',
    },
  ]
```


### 使用 `redis` 缓存页面

```js
// 配置信息
const Url = require('url');
const zlib = require('zlib');
const redis = require('redis');
const md5 = require('md5');


const request = require('request-json');



const {
  env: { ENV }
} = process;

// redis 客户端实例
let client;

if (ENV !== 'dev') {
  let url = 'redis://xxx.redis.rds.aliyuncs.com';

  if (ENV === 'production') {
    url = 'redis://xxx.redis.rds.aliyuncs.com/1';
  }

  client = redis.createClient({
    url
  });

  client.on('error', function (err) {
    console.error(err);
    // 可做错误消息通知，如：钉钉、email 等
  });
}

const nuxtConfig = {

  serverMiddleware: [
    //  ...,

    function (req, res, next) {
      // 开发环境不做处理
      if (process.env.ENV === 'dev') {
        return next();
      }

      // eslint-disable-next-line
      const query = Url.parse(req.url, true).query;
      const md5OfUrl = md5(`${req.headers.host}${req.url}`);

      if (query.nocache === 'true') {
        // 剔除 ?nocache=true or &nocache=true or nocache=true&
        if (req.url.endsWith('?nocache=true')) {
          req.url = req.url.replace('?nocache=true', '');
        } else if (req.url.endsWith('&nocache=true')) {
          req.url = req.url.replace('&nocache=true', '');
        } else if (req.url.includes('nocache=true&')) {
          req.url = req.url.replace('nocache=true&', '');
        }

        return next();
      }

      const startTime = new Date();

      // redis 断开的情况下，不作处理
      if (client && client.connected) {
        client.hgetall(md5OfUrl, function (err, reply) {
          if (!err && reply && reply.html) {
            // 返回缓存的静态化页面
            const html = zlib
              .gunzipSync(Buffer.from(reply.html, 'binary'))
              .toString();
            res.writeHead(200, {
              'Content-Type': 'text/html'
            });
            res.write(html);
            res.end();

            const timeStamp = new Date(+reply.time);

            // 将过期时间记录到日志
            const strTimeStamp = `记录时刻：${timeStamp.toLocaleDateString()} | ${timeStamp.toLocaleTimeString()}`;

            logTime(
              startTime,
              `${req.headers.host}${req.url} ${md5OfUrl} ${strTimeStamp}`
            );


            // 检查时间是否过期，缓存有效期 10 分钟
            const nowTime = new Date();

            if (nowTime - timeStamp > 1000 * 60 * 10) {
              const freshUrl =
                req.url +
                (req.url.includes('?') ? '&nocache=true' : '?nocache=true');
              console.log('刷新缓存', freshUrl);
              request.get(`https://${req.headers.host}${freshUrl}`);
            }
          } else {
            next();
          }
        });
      } else {
        next();
      }
    }
  ],

  hooks: {
    'render:routeDone'(url, result, context) {
      const md5OfUrl = md5(`${context.req.headers.host}${url}`);

      // 获取 ssr 时间
      const sHeader = context.res._header || '';
      const sTime = sHeader.substring(
        sHeader.indexOf('dur=') + 4,
        sHeader.indexOf(';desc=')
      );
      console.log(
        '页面渲染',
        sTime + 'ms',
        `${context.req.headers.host}${url}`,
        md5OfUrl
      );

      // redis 断开的情况下，不做处理
      // 开发环境中 redis 不开启
      if (!client || !client.connected) return;

      const compressStr = zlib.gzipSync(result.html).toString('binary');

      // 需要存储的哈希结构
      const value = {
        html: compressStr,
        time: new Date().getTime()
      };

      client.hmset(md5OfUrl, value, function (err) {
        if (!err) {
          console.log(
            '压缩效率',
            `${result.html.length}/${compressStr.length}=${result.html.length /
            compressStr.length}`,
            `${context.req.headers.host}${url}`,
            md5OfUrl
          );
        }
      });
    }
  },
};


function logTime(startTime, ext) {
  const nowTime = new Date();
  console.log(`返回缓存 ${nowTime - startTime}ms ${ext}`);
}

export default nuxtConfig;
```

使用 `redis`时，一定要处理好异常信息。



## 组件级别缓存 

配置项`nuxt.config.js`的配置大概长这样子:

```js
const LRU = require('lru-cache')
module.exports = {
  render: {
    bundleRenderer: {
      cache: LRU({
        max: 1000,                     // 最大的缓存个数
        maxAge: 1000 * 60 * 15        // 缓存15分钟
      })
    }
  }
}
```

并不是说配了该项就实现了组件级别的缓存，还需要在需做缓存的`vue`组件上增加`name`以及`serverCacheKey`字段，以确定缓存的唯一键值，比如：

```js
export default {
  name: 'AppHeader',
  props: ['type'],
  serverCacheKey: props => props.type
}
```

上述组件会根据父组件传下来的`type`值去做缓存，键值是：`AppHeader::${props.type}`，由此，新的请求到来时，只要父组件传下来的type属性之前处理过，就可以复用之前的渲染缓存结果，以增进性能

从该例子可以看出，如果该组件除了依赖父组件的`type`属性，还依赖于别的属性，`serverCacheKey`这里也要做出相应的改变，因此，如果组件依赖于很多的全局状态，或者，依赖的状态取值非常多，意味需要缓存会被频繁被设置而导致溢出，其实就没有多大意义了，在lru-cache的配置中，设置的最大缓存个数是1000，超出部分就会被清掉

其次，不应该缓存可能对渲染上下文产生副作用的子组件，比如，组件的created与beforeCreated的钩子在服务端也会走，组件被缓存后就不会执行了，这些可能影响到渲染上下文的地方也要小心，更多内容请参考：[组件级别缓存](https://ssr.vuejs.org/zh/guide/caching.html#%E7%BB%84%E4%BB%B6%E7%BA%A7%E5%88%AB%E7%BC%93%E5%AD%98-component-level-caching)

一般来说，比较适合的场景是v-for大量数据的渲染，因为循环操作比较耗cpu


## API级别缓存


在服务端渲染的场景中，往往会将请求放在服务端去做，渲染完页面再返回给浏览器，而有些接口是可以去做缓存的，比如，不依赖登录态且不依赖过多参数的接口或者是单纯获取配置数据的接口等，接口的处理也是需要时间的，对接口的缓存可以加快每个请求的处理速度，更快地释放掉请求，从而增进性能


api的请求使用`axios`，`axios`即可以在服务端使用也可是在浏览器使用

```js
import axios from 'axios'
import md5 from 'md5'
import LRU from 'lru-cache'

// 给api加3秒缓存
const CACHED = LRU({
  max: 1000,
  maxAge: 1000 * 3
})

function request (config) {
  let key
  // 服务端才加缓存，浏览器端就不管了
  if (config.cache && !process.browser) {
    const { params = {}, data = {} } = config
    key = md5(config.url + JSON.stringify(params) + JSON.stringify(data))
    if (CACHED.has(key)) {
      // 缓存命中
      return Promise.resolve(CACHED.get(key))
    }
  }
  return axios(config)
    .then(rsp => {
      if (config.cache && !process.browser) {
        // 返回结果前先设置缓存
        CACHED.set(key, rsp.data)
      }
      return rsp.data
    })
}


const api = {
  getGames: params => request({
    url: '/gameInfo/gatGames',
    params,
    cache: true
  })
}
```




## 参考链接

* [redis](https://www.npmjs.com/package/redis)
* [vue ssr cache](https://ssr.vuejs.org/zh/guide/caching.html)
