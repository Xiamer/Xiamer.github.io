---
title: js中__proto__和prototype的关系
date: 2019-05-19 12:13:48
tags:
---

本文内容主要来自[js中__proto__和prototype的区别和关系？](https://www.zhihu.com/question/34183746)作者[doris](https://www.zhihu.com/people/doris-53-22/activities)的回答，主要用于以后知识点回顾。

## 对象的隐式对象(_proto_)指向构造函数的原型对象(proptotype)
   在JS里，万物皆对象。方法（Function）是对象，方法的原型(Function.prototype)是对象。因此，它们都会具有对象共有的特点。即：对象具有属性__proto__，可称为隐式原型，一个对象的隐式原型指向构造该对象的构造函数的原型，这也保证了实例能够访问在构造函数原型中定义的属性和方法。

   ```
   function Persion(){ this.name = 'mingming' };
   Persion.prototype.say = function() {return 'say'};
   Persion.prototype.name = 'prohonghong';
   let p1 = new Persion();
   p1.__proto__ === Persion.prototype; // true
   Persion.__proto__ === Function.prototype;  // true
   Persion.prototype.__proto__ === Object.prototype; // true
   Object.prototype.__proto__; // null
   p1.say(); // say
   p1.name; // mingming
   p1.__proto__.name; // prohonghong
   ```
  
## Function原型对象(prototype)的构造器(constructor)指回原构造函数
 方法这个特殊的对象，除了和其他对象一样有上述_proto_属性之外，还有自己特有的属性——原型属性（prototype），这个属性是一个指针，指向一个对象，这个对象的用途就是包含所有实例共享的属性和方法（我们把这个对象叫做原型对象）。原型对象也有一个属性，叫做constructor，这个属性包含了一个指针，指回原构造函数。

  ```
  Persion.prototype.constructor === Persion // true
  Persion.constructor === Function // true
  Function.prototype.constructor === Function // true
  ```


## 原型链关系图


![](/images/prototype.jpg)

 1. 构造函数Foo()
   
    构造函数的原型属性Foo.prototype指向了原型对象，在原型对象里有共有的方法，所有构造函数声明的实例（这里是f1，f2）都可以共享这个方法。
 2. 原型对象Foo.prototype
   
    Foo.prototype保存着实例共享的方法，有一个指针constructor指回构造函数。
 3. 实例
   
    f1和f2是Foo这个对象的两个实例，这两个对象也有属性__proto__，指向构造函数的原型对象，这样子就可以像上面1所说的访问原型对象的所有方法啦。

   
  另外：构造函数Foo()除了是方法，也是对象啊，它也有__proto__属性，指向它的构造函数的原型对象。
  函数的构造函数是Function，因此这里的__proto__指向了Function.prototype。其实除了Foo()，Function(), Object()也是一样的道理。
  原型对象也是对象啊，它的__proto__属性，又指向谁呢？同理，指向它的构造函数的原型对象呗。这里是Object.prototype.最后，Object.prototype的__proto__属性指向null。

## 参考链接
* [https://www.zhihu.com/question/34183746](https://www.zhihu.com/question/34183746)