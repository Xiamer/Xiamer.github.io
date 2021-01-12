---
title: 发布订阅模式vs观察者模式
date: 2020-12-30 23:13:04
categories: 
- web前端
tags:
- js
---
## 发布订阅模式

发布订阅模式：基于一个事件（主题）通道，希望接收通知的对象 `Subscriber` 通过自定义事件订阅主题，被激活事件的对象 `Publisher` 通过发布主题事件的方式通知各个订阅该主题的 `Subscriber` 对象。

发布订阅模式与观察者模式的不同，“第三者” （事件中心）出现。**目标对象并不直接通知观察者，而是通过事件中心(eventpool)来派发通知。**

代码实现

```javascript
class EventEmitter {  constructor() {
    this._eventpool = {};
  }

  // 订阅
  on(event, callback) {
    this._eventpool[event] ? this._eventpool[event].push(callback) : this._eventpool[event] = [callback]
  }

  // 发布
  emit(event, ...args) {
    this._eventpool[event] && this._eventpool[event].forEach(cb => cb(...args))
  }

  // 取消订阅
  off(event, callback) {
    const cbList = this._eventpool[event]
    if (!cbList) return false
    if (callback) {
      const i = cbList.indexOf(callback)
      if(i>= 0) this._eventpool[event].splice(i, 1)
    } else {
      delete this._eventpool[event]
    }
  }

  // 只订阅一次
  once(event, callback) {
    // 该写法 无法删除 event 对应的 callback，传进on时为无名函数
    // this.on(event, (...args) => {
    //   this.off(event, callback)
    // })
    const self = this
    function fn(...arg) {
      callback(...arg)
      self.off(event, fn);
    }
    this.on(event, fn)
  }
}
```

e.g:

```javascript
function a(q, p) { console.log('a', q, p) };
function b(q, p) { console.log('b', q, p) };
function c(q, p) { console.log('c', q, p) };

const ee = new EventEmitter()
ee.on('event1', a)
ee.on('event1', b)
ee.once('event1', c)
ee.emit('event1', 1, 2)
// a 1 2
// b 1 2
// c 1 2 
ee.emit('event1', 1, 2)
// a 1 2 
// b 1 2 
```

发布订阅模式中，订阅者各自实现不同的逻辑，且只接收自己对应的事件通知。实现你想要的 “不一样”。

## 观察者模式

观察者模式：**定义了对象间一种一对多的依赖关系**，当目标对象 Subject 的状态发生改变时，所有依赖它的对象 Observer 都会得到通知。

`Subject` 添加一系列 `Observer`， `Subject` 负责维护与这些 `Observer` 之间的联系。

```javascript
// 目标者类
class Subject {
  constructor() {
    this.observers = [];  // 观察者列表
  }
  // 添加
  add(observer) {
    this.observers.push(observer);
  }
  // 删除
  remove(observer) {
    let idx = this.observers.findIndex(item => item === observer);
    idx > -1 && this.observers.splice(idx, 1);
  }
  // 通知
  notify() {
    for (let observer of this.observers) {
      observer.update();
    }
  }
}

// 观察者类
class Observer {
  constructor(food) {
    this.food = food;
  }
  // 目标对象更新时触发的回调
  update() {
    console.log(`今天吃：${this.food}`);
  }
}


// 实例化目标者
let subject = new Subject();

// 实例化两个观察者
let obs1 = new Observer('螃蟹');
let obs2 = new Observer('虾');

// 向目标者添加观察者
subject.add(obs1);
subject.add(obs2);

// 目标者通知更新
subject.notify();  
// 输出：
// 今天吃：螃蟹
// 今天吃：虾


```

**优点：**

* 目标者与观察者，功能耦合度降低，专注自身功能逻辑；

- 观察者被动接收更新，时间上解耦，实时接收目标者更新状态。

**缺点**：

1. 观察者模式虽然实现了对象间依赖关系的低耦合，但却不能对事件通知进行细分管控，如 “筛选通知”，“指定主题事件通知”

## 观察者模式 vs 发布订阅模式

![](/images/observer-publish.jpg)
**相同**：

- 都是定义一个一对多的依赖关系，有关状态发生变更时执行相应的通知。

**差异：**

- 在**观察者**模式中，观察者是知道Subject的，Subject一直保持对观察者进行记录。然而，在**发布订阅**模式中，发布者和订阅者**不知道对方的存在**。它们只有通过消息代理进行通信。
- 在**发布订阅**模式中，组件是松散耦合的，正好和观察者模式相反。

## 参考链接

* [观察者模式、发布订阅模式、vue 双向绑定 ](https://segmentfault.com/a/1190000019722065)
