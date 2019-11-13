---
title: stylus入门
date: 2019-06-26 17:36:16
categories: 
- web前端
tags:
- css
  
---

本文只是简单介绍`stylus`的基本用法，了解更多建议去[官网](http://stylus-lang.com/)查看。
在线编辑 [stylus2css](http://stylus-lang.com/try.html)
## stylus定义
**富于表现力**、**动态的**、**健壮的** CSS
Stylus是一种创新的样式表语言，可编译为CSS。
受SASS的启发，Stylus使用node.js构建，并且能够在浏览器中运行，如此交互式教程所示。

## Stylus的特征
* 冒号可选
* 分号可选
* 逗号可选
* 括号可选
* 变量
* 插值
* 混合书写
* 算术
* 强制类型转换
* 动态导入
* 条件
* 迭代
* 嵌套选择
* 父级参考
* 变量函数调用
* 词法作用域
* 内置函数(>25)
* 内部语言函数
* 压缩可选
* 图像内联可选
* 可执行Stylus
* 健壮的错误报告
* 单行和多行注释
* CSS字面量
* 字符转义
* TextMate捆绑
* 以及其他更多

## Stylus的使用

### 选择器(Selectors)

#### 简写

```stylus
body
  color white
  
// 编译为
body {
  color: #fff;
}
```
`stylus`有严格的缩进，两个空格符。与平时的写法简介了很多，`{}` `:` `;` 都是可选，方便阅读可加上`:`。

#### 规则集

Stylus允许使用多个逗号或换行选择器同时定义属性。
```stylus
// 以下两种写法是等价的
div, p
  color #fff

div
p
  color #fff

// 编译为
div,
p {
  color: #fff;
}
```
当我们需要同时给 a中的b中的 x 和 c中的d中的x 添加属性 可写成
```stylus
foo bar baz,
form input,
> a
  border 1px solid
  
// 编译为
foo bar baz,
form input,
> a {
  border: 1px solid;
}
```

#### 父级引用 &
```stylus
textarea
input
  color #A7A7A7
  &:hover
    color #000

// 编译为
textarea,
input {
  color: #a7a7a7;
}
textarea:hover,
input:hover {
  color: #000;
}
```

## 变量

可以直接指定表达式变量

```stylus
$font-size = 14px
$font = $font-size "Lucida Grande", Arial

body
  font $font sans-serif
  
// 编译为
body {
  font: 14px "Lucida Grande", Arial sans-serif;
}
```
为了区分变量和原有属性，建议变量以`$`开头

## 属性查找
```stylus
#logo
  position: absolute
  top: 50%
  left: 50%
  width: 150px
  height: 80px
  margin-left: -(@width / 2)
  margin-top: -(@height / 2)
  
// 编译为
#logo {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 150px;
  height: 80px;
  margin-left: -75px;
  margin-top: -40px;
}
```
使用`@`+`属性名` 来访问该属性名对应的值

```stylus
body
  color: #f00
  ul
    li
      color: #00f
      a
        background-color: @color
        
// 编译为
body {
  color: #f00;
}
body ul li {
  color: #00f;
}
body ul li a {
  background-color: #00f;
}
```
属性会向上冒泡，找到最近属性名对应的属性值

```stylus
position()
  position: arguments
  z-index: 1 unless @z-index // 指定默认值为1

#logo
  z-index: 20
  position: absolute

#logo2
  position: absolute
  
// 编译为
#logo {
  z-index: 20;
  position: absolute;
}
#logo2 {
  position: absolute;
  z-index: 1;
}
```
`z-index` 和 `position` 先后顺序无关，只与当前选择器所选择的全部属性有关

## 插值

Stylus支持通过使用{}字符包围表达式来插入值，会变成标识符的一部分。例如，-webkit-{'border' + '-radius'}等同于-webkit-border-radius.
```stylus
vendor(prop, args)
  -webkit-{prop} args
  -moz-{prop} args
  {prop} args

border-radius()
  vendor('border-radius', arguments)


button
  border-radius 1px 2px / 3px 4px
  
// 编译为
button {
  -webkit-border-radius: 1px 2px / 3px 4px;
  -moz-border-radius: 1px 2px / 3px 4px;
  border-radius: 1px 2px / 3px 4px;
}
```
可以看做 `border-radius` 混合写入（后面会讲），然后再调用`vendor`

## 运算符(Operators)
```
[]
! ~ + -
is defined
** * / %
+ -
... ..
<= >= < >
in
== is != is not isnt
is a
&& and || or
?:
= := ?= += -= *= /= %=
not
if unless
```
上表运算符优先级，从最高到最低.

### 一级运算符
```stylus
!0
// => true

!!0
// => false

!1
// => false
```
### 铸造

```stylus
body
  n = 5
  foo: (n)em
  foo: (n)%
  foo: (n + 5)%
  foo: (n * 5)px
  foo: unit(n + 5, '%')
  foo: unit(5 + 180 / 2, deg)
```
unit()函数可用来强制后缀

## 混合书写(Mixins)

```stylus 
border-radius(n)
  -webkit-border-radius n
  -moz-border-radius n
  border-radius n

form input[type=button]
  border-radius 5px
  
// 编译为
form input[type=button] {
  -webkit-border-radius: 5px;
  -moz-border-radius: 5px;
  border-radius: 5px;
}
```
`border-radius` 使用了混合 调用了`border-radius` 也可不指定参数，利用局部变量 `arguments`代替

## 方法(Functions)

### 函数

Stylus强大之处就在于其内置的语言函数定义。其定义与混入(mixins)一致；却可以返回值。

```stylus
add(a, b)
  a + b
body 
  padding add(10px, 5)
  
// 编译为
body {
  padding: 15px;
}
```
### 默认值

```stylus
add(a, b = 5)
  a + b
body 
  padding add(10px)
  
// 编译为
body {
  padding: 15px;
}
```

### 多个返回值(可看做数组)
```stylus
sizes = 15px 10px

sizes[0]
// => 15px


sizes()
 15px 10px

sizes()[0]
// => 15px
```

### 变量函数

```
invoke(a, b, fn)
  fn(a, b)

add(a, b)
  a + b

body
  padding invoke(5, 10, add)
  padding invoke(5, 10, sub)
  
// 编译为
body {
  padding: 15;
  padding: -5;
}
```