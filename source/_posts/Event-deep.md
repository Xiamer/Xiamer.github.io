---
title: JS Event deep
date: 2019-05-09 17:54:15
tags:
---

## 进程和线程

进程(Process)是系统资源分配和调度的单元。一个运行着的程序就对应了一个进程。一个进程包括了运行中的程序和程序所使用到的内存和系统资源。如果是单核CPU的话，在同一时间内，有且只有一个进程在运行。但是，单核CPU也能实现多任务同时运行，比如你边听网易云音乐的每日推荐歌曲，边在网易有道云笔记上写博文。这算开了两个进程（多进程），那运行的机制就是一会儿播放一下歌，一会儿响应一下你的打字，但由于CPU切换的速度很快，你根本感觉不到，以至于你认为这两个进程是在同时运行的。进程之间是资源隔离的。

那线程(Thread)是什么？线程是进程下的执行者，一个进程至少会开启一个线程（主线程），也可以开启多个线程。比如网易云音乐一边播放音频，一边显示歌词。多进程的运行其实也就是通过进程中的线程来执行的。一个进程下的线程是共享资源的。当多个线程同时操作同一个资源的时候，就出现资源争抢的问题。这又是另外一个问题了。

操作系统的设计，可以归结为三点：

1. 以多进程形式，允许多个任务同时运行；

2. 以多线程形式，允许单个任务分成不同的部分运行；

3. 提供协调机制，一方面防止进程之间和线程之间产生冲突，另一方面允许进程之间和线程之间共享资源。

## Javascript是单线程

其实这与它的用途有关。作为浏览器脚本语言，JavaScript 的主要用途是与用户互动，以及操作 DOM。若以多线程的方式操作这些 DOM，则可能出现操作的冲突。假设有两个线程同时操作一个 DOM 元素，线程1 要求浏览器删除 DOM，而线程2 却要求修改 DOM 样式，这时浏览器就无法决定采用哪个线程的操作。当然，我们可以为浏览器引入“锁”的机制来解决这些冲突，但这会大大提高复杂性，所以 JavaScript 从诞生开始就选择了单线程执行。

另外，因为 JavaScript 是单线程的，在某一时刻内只能执行特定的一个任务，并且会阻塞其它任务执行。那么对于类似 I/O 等耗时的任务，就没必要等待他们执行完后才继续后面的操作。在这些任务完成前，JavaScript 完全可以往下执行其他操作，当这些耗时的任务完成后则以回调的方式执行相应处理。这些就是 JavaScript 与生俱来的特性：异步与回调。

## 浏览器不是单线程的

虽然JS运行在浏览器中，是单线程的，每个window一个JS线程，但浏览器不是单线程的，例如Webkit或是Gecko引擎，都可能有如下线程：

* javascript引擎线程
* 界面渲染线程
* 浏览器事件触发线程
* Http请求线程

## 同步任务和异步任务

单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。

JavaScript语言的设计者意识到，这时主线程完全可以不管IO设备，挂起处于等待中的任务，先运行排在后面的任务。等到IO设备返回了结果，再回过头，把挂起的任务继续执行下去。

于是，所有任务可以分成两种，一种是**同步任务**（synchronous），另一种是**异步任务**（asynchronous）。同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

![](https://user-gold-cdn.xitu.io/2017/11/21/15fdd88994142347?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

具体来说，异步执行的运行机制如下。（同步执行也是如此，因为它可以被视为没有异步任务的异步执行。）

  1. 所有同步任务都在主线程上执行，形成一个**执行栈**（execution context stack）。

  2. 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。

  3. 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。

  4. 主线程不断重复上面的第三步。

## 宏任务队列和微任务队列

任务队列：**宏任务队列**（macro tasks）和**微任务队列**（micro tasks）。宏任务队列可以有多个，微任务队列只有一个。

* 宏任务：宿主(浏览器/node)发起的任务，主要有script（全局任务）, setTimeout, setInterval, setImmediate, I/O, UI rendering.
* 微任务：js引擎发起的任务，主要有process.nextTick, Promise, Object.observer(已废弃), MutationObserver.
  
> **一段代码块就是一个宏任务。**所以一般执行代码块的时候，也就是程序执行进入主线程了，主线程会根据不同的代码再分微任务和宏任务等待主线程执行完成后，不停地循环执行。
> 主线程（宏任务） => 微任务 => 宏任务 => 主线程


![](https://user-gold-cdn.xitu.io/2017/11/21/15fdcea13361a1ec?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**主要部分**：  开始-> 取task queue(macro task)第一个task执行(script) -> 取micro task全部任务依次执行 -> 取task queue下一个任务执行 -> 再次取出micro task全部任务执行 -> … 另外在处理microtask期间，如果有新添加的microtasks，也会被添加到队列的末尾并执行。js引擎存在**monitoring process进程**， 会不停的监听`task queue`


细节可以参考 [Tasks和Microtasks](https://github.com/creeperyang/blog/issues/21)


列子1：
```
console.log('script start');

// 微任务
Promise.resolve().then(() => {
    console.log('p 1');
});

// 宏任务
setTimeout(() => {
    console.log('setTimeout');
}, 0);

var s = new Date();
while(new Date() - s < 50); // 阻塞50ms

// 微任务
Promise.resolve().then(() => {
    console.log('p 2');
});

console.log('script ent');


/*** output ***/

// one macro task
script start
script ent

// all micro tasks
p 1
p 2

// one macro task again
setTimeout
```

上面之所以加50ms的阻塞，是因为 `setTimeout` 的 delayTime 最少是 4ms. 为了避免认为 `setTimeout` 是因为4ms的延迟而后面才被执行的，我们加了50ms阻塞。

例子2:

```
async function async1(){
    console.log('async1 start')
    await async2()
    console.log('async1 end')
}
async function async2(){
    console.log('async2')
}
console.log('script start')
setTimeout(function(){
    console.log('setTimeout') 
},0)  
async1();
new Promise(function(resolve){
    console.log('promise1')
    resolve();
}).then(function(){
    console.log('promise2')
})
console.log('script end')
```
* 定义函数async1、async2。输出'script start'
* 将 setTimeout 里面的回调函数(宏任务)添加到**下一轮任务队列**。因为这段代码前面没有执行任何的异步操作且等待时间为0s。所以回调函数会被立刻放到下一轮任务队列的开头。
* 执行async1。我们知道async函数里面await标记之前的语句和 await 后面的语句是同步执行的。所以这里先后输出"async1 start",’async2 start‘. (根据 TC39 最近决议，await将直接使用 Promise.resolve() 相同语义。)
* 这时暂停执行下面的语句，下面的语句被放到当前队列的最后。
继续执行同步任务。
* 输出 ‘Promise1’。将then里面的函数放在当前队列的最后。
* 然后输出‘script end’,注意这时只是同步任务执行完了，当前任务队列的任务还没有执行完毕，还有两个微任务被添加进来了!队列是先进先出的结构，所以这里先输出 ‘async1 end’ 再输出 ‘Promise2’,这时第一轮任务队列才真算执行完了。
* 然后执行下一个任务列表的任务。执行setTimeout里面的异步函数。输出‘setTimeout’。

作者：sliiva
链接：https://juejin.im/post/5c8a024d51882546be0a3082
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
  
## Event Loop

主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）。

![](https://image-static.segmentfault.com/402/025/4020255170-59bc9e1671029)


## NodeJs 的 Event Loop

Node.js也是单线程的Event Loop，但是它的运行机制不同于浏览器环境。

![](https://www.ruanyifeng.com/blogimg/asset/2014/bg2014100803.png)

1. V8引擎解析JavaScript脚本。
2. 解析后的代码，调用Node API。
3. libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎。
4. V8引擎再将结果返回给用户。

除了setTimeout和setInterval这两个方法，Node.js还提供了另外两个与"任务队列"有关的方法: **process.nextTick**和**setImmediate**。process.nextTick方法可以在当前"执行栈"的尾部----下一次Event Loop（主线程读取"任务队列"）之前----触发回调函数。也就是说，它指定的任务总是发生在所有异步任务之前。setImmediate方法则是在当前"任务队列"的尾部添加事件，也就是说，它指定的任务总是在下一次Event Loop时执行。


### Node.js Event Loop原理

node.js的特点是事件驱动，非阻塞单线程。当应用程序需要I/O操作的时候，线程并不会阻塞，而是把I/O操作交给底层库（LIBUV）。此时node线程会去处理其他任务，当底层库处理完I/O操作后，会将主动权交还给Node线程，所以Event Loop的用处是调度线程，例如：当底层库处理I/O操作后调度Node线程处理后续工作，所以虽然node是单线程，但是底层库处理操作依然是多线程

### Node Event Loop的事件处理机制

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

   * timers: 这个阶段执行setTimeout()和setInterval()设定的回调。
   * pending callbacks: 上一轮循环中有少数的 I/O callback 会被延迟到这一轮的这一阶段执行。
   * idle, prepare: 仅内部使用。
   * poll: 执行 I/O callback，在适当的条件下会阻塞在这个阶段
   * check: 执行setImmediate()设定的回调。
   * close callbacks: 执行比如socket.on('close', ...)的回调。


## 参考链接
* [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
* [JavaScript 异步、栈、事件循环、任务队列](https://segmentfault.com/a/1190000011198232)
* [通过microtasks和macrotasks看JavaScript异步任务执行顺序](https://tuobaye.com/2017/10/24/%E9%80%9A%E8%BF%87microtasks%E5%92%8Cmacrotasks%E7%9C%8BJavaScript%E5%BC%82%E6%AD%A5%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F/)
* [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)
* [从面试题看 JS 事件循环与 macro micro 任务队列](https://juejin.im/post/5c8a024d51882546be0a3082)
* [深入核心，详解事件循环机制](https://www.jianshu.com/p/12b9f73c5a4f)
* [node基础面试事件环？微任务、宏任务？](https://juejin.im/post/5b35cdfa51882574c020d685)
* [The Node.js Event Loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
* [详解 setTimeout、setImmediate、process.nextTick 的区别](https://www.cnblogs.com/onepixel/articles/7605465.html)
* [node.js在线](https://repl.it/repls/NauticalVastFont)