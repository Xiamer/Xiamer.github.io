
---
title: js 继承
date: 2020-12-20 23:13:04
categories: 
- web前端
tags:
- js

---

## 继承相关分类和关系图

![](/images/js-extend.png)


## 继承方式

### 1. 原型链继承 (prototype)

核心：<font color="#dd0000">将父类的实例作为子类的原型</font>

```js
Child.prototype = new Parent() 
// 所有涉及到原型链继承的继承方式都要修改子类构造函数的指向，否则子类实例的构造函数会指向Parent。
// 构造函数prototype的constructor应指向构造函数本身。
Child.prototype.constructor = Child;
```

e.g. 
```js
function Parent () {
  this.names = ['xiamer', 'yian'];
}
Parent.prototype.getName = function () {
  console.log(this.name);
}
function Child () {
}
Child.prototype = new Parent();
Child.prototype.constructor = Child;

const child1 = new Child();
child1.names.push('body');
console.log(child1.names); // ['xiamer', 'yian', 'body']
const child2 = new Child();
console.log(child2.names); // ['xiamer', 'yian', 'body']

```

优点：

* 父类方法可以复用

缺点： 
* <font color="#dd0000">父类的 引用属性 会被所有子类实例共享</font>
  * `child1.names === child1.__proto__.names === Child.prototype.names`
  * `child2.names === child2.__proto__.names === Child.prototype.names`
  * 同一地址引用
* <font color="#dd0000">子类构建实例时不能向父类传递参数</font>
  * 因Parent是提前new


### 2. 构造函数继承 (Parent.call(this))

核心：<font color="#dd0000">将父类构造函数的内容复制给了子类的构造函数。这是所有继承中唯一一个不涉及到prototype的继承。</font>

```js
function Child() {
  Parent.call(this);
}
```

e.g.

```js
function Parent () {
  this.names = ['xiamer', 'yian'];
}
function Child () {
  Parent.call(this);
}

const child1 = new Child();
child1.names.push('body');
console.log(child1.names); // ["xiamer", "yian", "body"]
const child2 = new Child();
console.log(child2.names); // ["xiamer", "yian"]
```

优点：
* 避免了引用类型的属性被所有实例共享
* 可以在 Child 中向 Parent 传参

缺点： 
* <font color="#dd0000">方法在构造函数中定义，每次创建实例都会创建一遍方法。</font>

### 3. 组合继承 (prototype && call)

核心：<font color="#dd0000">原型链继承和构造函数继承的组合，兼具了二者的优点。</font>

```js
function Child() {
    Parent.call(this) // 第二次调用Parent
}
Child.prototype = new Parent() // 第一次调用Parent
Child.prototype.constructor = Child;
```

e.g.

```js
function Parent (name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.getName = function () {
  console.log(this.name)
}
function Child (name, age) {
  Parent.call(this, name); // 第二次调用Parent
  this.age = age;
}
Child.prototype = new Parent(); // 第一次调用Parent
Child.prototype.constructor = Child;

const child1 = new Child('kevin', '18');
child1.colors.push('black');
console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]
const child2 = new Child('daisy', '20');
console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

优点

* 父类的方法可以被复用
* 父类的引用属性不会被共享
* 子类构建实例时可以向父类传递参数

缺点
* <font color="#dd0000">调用了两次父类的构造函数</font>
* 第一次给子类的原型添加了父类的name, color属性，第二次又给子类的构造函数添加了父类的name, color属性，从而覆盖了子类原型中的同名参数。这种被覆盖的情况造成了性能上的浪费。

### 4. 原型式继承(Object.create())

核心：
<font color="#dd0000">ES5 Object.create 的模拟实现，原型式继承的object方法本质上是对参数对象的一个浅复制。</font>

```js
function createObj(o) {
  function F(){}
  F.prototype = o;
  return new F();
}
```

e.g.

```js
function createObj(o){
  function F(){}
  F.prototype = o;
  return new F();
}

let person = {
  name: 'kevin',
  friends: ['daisy', 'kelly']
}
let person1 = createObj(person);
let person2 = createObj(person);
// !!! 其实是给person1赋name属性，而不是改了 person1.name -> person1.__proto__.name
person1.name = 'person1';
console.log(person2.name); // kevin
// person1.firends -> person1.__proto__.firends
person1.firends.push('taylor');
console.log(person2.friends); // ["daisy", "kelly", "taylor"]
console.log(person1.friends); // ["daisy", "kelly", "taylor"]

// es6 写法
let person1 = Object.create(person);
let person2 = Object.create(person);
```

优点：
* 父类方法可以复用

缺点：
* <font color="#dd0000">父类的引用属性会被所有子类实例共享</font>
* <font color="#dd0000">子类构建实例时不能向父类传递参数</font>

### 5. 寄生式继承(Object.create() && clone.fn)

核心：<font color="#dd0000">使用原型式继承获得一个目标对象的浅复制，然后增强这个浅复制的能力。</font>

```js
function createObj (o) {
  var clone = Object.create(o);
  clone.sayName = function () {
    console.log('hi');
  }
  return clone;
}
```

e.g.

```js
function createObjExt(original) { 
  var clone = Object.create(original);    // 通过调用函数创建一个新对象
  clone.sayHi = function(){      // 以某种方式来增强这个对象
      console.log("hi");
  };
  return clone;                  // 返回这个对象
}

var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = createObjExt(person);
anotherPerson.sayHi(); //  "hi"
```

缺点：
* <font color="#dd0000">跟借用构造函数模式一样，每次创建对象都会创建一遍方法。</font>

### 6. 寄生组合继承

核心： <font color="#dd0000">对父类原型的复制，所以不包含父类的构造函数，也就不会调用两次父类的构造函数造成浪费</font>

e.g.

```js
function inheritPrototype(subType, superType){
    var prototype = Object.create(superType.prototype); // 创建了父类原型的浅复制
    prototype.constructor = subType;                    // 修正原型的构造函数
    subType.prototype = prototype;                      // 将子类的原型替换为这个原型
}

function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    alert(this.name);
};

function SubType(name, age){
    SuperType.call(this, name);
    this.age = age;
}
// 核心：因为是对父类原型的复制，所以不包含父类的构造函数，也就不会调用两次父类的构造函数造成浪费
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function(){
  alert(this.age);
}
```
优点：
* <font color="#dd0000">完美的继承方式</font>

### 7. 寄生组合继承(ES6 Class extends)

核心： <font color="#dd0000">ES6继承的结果和寄生组合继承相似，本质上，ES6继承是一种语法糖。但是，寄生组合继承是先创建子类实例this对象，然后再对其增强；而ES6先将父类实例对象的属性和方法，加到this上面（所以必须先调用super方法），然后再用子类的构造函数修改this。</font>

```js
class A {}
class B extends A {
  constructor() {
    // super代表的是父类构造函数，但是返回的是子类的实例。比如A是B的父类，那么super的功能相当于A.prototype.constructor.call(this)。
    // super的使用方式：1、当函数使用。2、当对象使用
    super();
  }
}
```
super的使用方式：1、当函数使用。
e.g.
```js
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

super的使用方式：2、当对象使用 (当对象使用时，相当于引用a原型上的方法)
e.g.
```js
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```


ES6实现继承的具体原理

```js
class A {
}
class B {
}
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);
// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);
```

ES6继承与ES5继承的异同

相同点：本质上ES6继承是ES5继承的语法糖

不同点：
* ES6继承中子类的构造函数的原型链指向父类的构造函数，ES5中使用的是构造函数复制，没有原型链指向。
* ES6子类实例的构建，基于父类实例，ES5中不是。

#### `类的prototype和__proto__方法`

* `在ES5中，每个实例对应的__proto__都是指向对应构造函数的prototype方法。`
* `class中，存在两条链级关系，他们分别是`
 * `子类的__proto__指向父类`
 * `子类prototype属性的__proto__指向父类的prototype属性`

```js
class A {
}

class B extends A {
}

B.__proto__ === A    // true
B.prototype.__proto__ === A.prototype  // true
```

## 参考链接
* [es5 继承](https://segmentfault.com/a/1190000015727237)
* [es6 class继承](https://juejin.cn/post/6844903966396907527)