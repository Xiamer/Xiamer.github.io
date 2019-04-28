---
title: css 20分钟玩转 grid
date: 2018-04-28 15:37:34
tags:
---
## 简介

 CSS网格布局擅长于将一个页面划分为几个主要区域，以及定义这些区域的大小、位置、层次等关系。
![网页基本布局](https://upload-images.jianshu.io/upload_images/8493649-885b3b6e160fee71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

学习grid布局之前，请先了解 [flex](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex) 布局，有一定相似性，但这个更强大一些。


## 属性
### `display` 设置网格布局
![](https://upload-images.jianshu.io/upload_images/8493649-5df87aa0ccf1ca1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  display: grid;
}
```
如上图所示，默认为块级元素，独占一行。
![](https://upload-images.jianshu.io/upload_images/8493649-4f89f07ad4fe6dcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  display: inline-grid;
}
```
如上图所示，也可设为行内块元素，与其他元素并排。

### `grid-template` 设置网格区域
设置每一列宽：`grid-template-columns: <track-size> ... | <line-name> <track-size> ...;`
设置每一行高：`grid-template-rows: <track-size> ... | <line-name> <track-size> ...;`
设置网格布局允许指定区域：`grid-template-areas:  "<grid-area-name> | . | none | ..."
    "...";`
 **<track-size>** - can be a length, a percentage, or a fraction of the free space in the grid (using the `[fr](https://css-tricks.com/snippets/css/complete-guide-grid/#fr-unit)` unit)
**<line-name>** - an arbitrary name of your choosing
**<grid-area-name>** - the name of a grid area specified with `[grid-area]

![](https://upload-images.jianshu.io/upload_images/8493649-7c7d15a741fe2fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  display: grid;
  grid-template-columns: 50px 50px 50px;
  grid-template-rows: 50px 50px 50px;
  /* grid-template-columns: 33.33% 33.33% 33.33%;
  grid-template-rows: 33.33% 33.33% 33.33%; */
}
```
如上图所示，设置每一列为50px，每一行为50px，也可设为33.33%，等价的。
```
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 50px);
}
```
也可使用`repeat`属性值，第一个为参数为重复的次数，第二个参数为重复的值。
```
.wrapper {
  display: grid;
  /* grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 1fr 1fr 1fr; */
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
}
```
也可使用`fr`属性值(fraction,每一分块区域)，如果两列的宽度分别为`1fr`和`3fr`，就表示后者是前者的三倍。

![](https://upload-images.jianshu.io/upload_images/8493649-0135a11dbeeb924b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  display: grid;
  grid-template-columns: 50px 2fr 1fr;
  grid-template-rows: repeat(3, 1fr);
}
```
如上图所示，我们经常会结合`px`和`fr`一起用，第一列`50px`,剩下两列分三份分`100px`，第二列站`2份`，第三列站`1份`，因此上面列宽依次为`50px 66.66px 33.33px`。

![](https://upload-images.jianshu.io/upload_images/8493649-bbd2e1631bb034ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
  display: grid;
  grid-template-columns: 50px minmax(40px, 100px) 20px;
  grid-template-rows: repeat(3, 1fr);
  /* grid-template-columns: 50px minmax(40px, 100px) minmax(20px, 40px);
  grid-template-rows: repeat(3, 1fr); */
}
```
如上图所示，可设置`minmax `属性值来设置大小，中间这一列最小值为40px，最大值为100px，因此上面列宽依次为`50px 80px 20px`。

![](https://upload-images.jianshu.io/upload_images/8493649-57ccef27a077380a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
   display: grid;
   grid-template-columns: 50px minmax(40px, 100px) minmax(20px, 40px);     
   grid-template-rows: repeat(3, 1fr);
}
```
如上图所示，列宽依次是50px 60px 40px,是怎样计算呢？
[how-the-minmax-function-works](https://bitsofco.de/how-the-minmax-function-works/)
![](https://upload-images.jianshu.io/upload_images/8493649-1c34527b6051081b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
   display: grid;
   grid-template-columns: 20px auto auto;
   grid-template-rows: repeat(3, 1fr);
}
```
如上图所示，可设置`auto`属性值，让浏览器绝对长度，因此上面列宽依次为`20px 65px 65px`。

![](https://upload-images.jianshu.io/upload_images/8493649-24b825a16023eac4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
.grid-template-areas {
  display: grid;
  grid-template-columns: 50px 50px 50px 50px;
  grid-template-rows: auto;
  grid-template-areas:
    'header header header header'
    'main main . sidebar'
    'footer footer footer footer';
}
.item-1 {
  grid-area: header;
}
.item-2 {
  grid-area: main;
}
.item-3 {
  grid-area: sidebar;
}
.item-4 {
  grid-area: footer;
}
```
如上图所示，`grid-template-areas`设置`4x3`网格， `grid-area`对应所选区域，`.`代表不填充，留空白。

### `grid-gap`设置行与行间距、列与列间距
` grid-gap: <grid-row-gap> <grid-column-gap>;`
![](https://upload-images.jianshu.io/upload_images/8493649-9ffff07912c337e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.container {
  display: grid;
  grid-template-columns: 100px 50px 100px;
  grid-template-rows: 80px auto 80px;
  grid-gap: 15px 10px;
  /* grid-row-gap: 15px;
  grid-column-gap: 10px; */
}
```
如上图所示，设置行间距为`15px`，列间距为`10px`

4. `grid-auto-flow` 设置排列方式，默认“先行后列”
` grid-auto-flow: row | column | row dense | column dense`

![](https://upload-images.jianshu.io/upload_images/8493649-4b25db5a7438e8d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
}
.item-1 {
  /* 代表所占列的网格线为1～3，grid-row 不写默认为所占行的网格线为1～2 */
  grid-column: 1 / 3; 
}
.item-2 {
  grid-column: 1 / 3;
}
```
如上图所示，为默认排列，先行后列，放不下就放到下一行。


![](https://upload-images.jianshu.io/upload_images/8493649-ce609ebafcebbb6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  grid-auto-flow: row dense;
}
.item-1 {
  /* 代表所占列的网格线1～3，grid-row 不写默认为所占行的网格线1～2 */
  grid-column: 1 / 3;
}
.item-2 {
  grid-column: 1 / 3;
}
```
上图所排为默认排列，先行后列，并且紧凑，尽量不留空隙。


### `place-items` 设置网格内容的位置
`place-items: <align-items> <justify-items>;` **先竖直后水平**
水平位置：`justify-items: start | end | center | stretch;`
竖直位置：`align-items: start | end | center | stretch;`

* start：对齐网格的起始边缘
* end：对齐网格的结束边缘
* center：网格内部居中
* stretch：拉伸，占满网格的整个宽度（默认值）。

![](https://upload-images.jianshu.io/upload_images/8493649-d82baa82a027a9e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 50px);
  place-items: start end;
}
```
如上图所示，竖直位置内容从上到下排列，水平位置内容从左到右排列。

![](https://upload-images.jianshu.io/upload_images/8493649-b7d6bdb3a93a9198.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 50px);
  place-items: start stretch;
}
```
如上图所示，竖直位置内容从上到下排列，水平位置拉伸占满网格的整个宽度。

### `place-content`设置整个内容区域在容器里面的位置
`place-content: <align-content> <justify-content>`
 水平位置：`justify-content: start | end | center | stretch | space-around | space-between | space-evenly;`
垂直位置： ` align-content:start | end | center | stretch | space-around | space-between | space-evenly; `
* start: 将网格对齐以与容器的起始边缘齐平
* end: 将网格对齐以与容器的末端边缘齐平
* stretch: 调整网格的大小填充容器的整个宽度(默认值)
* space-around: 每个网格两侧的间隔相等，两端到容器的间隔是网格的一半
* space-between: 网格与网格的间隔相等，网格与容器边框之间没有间隔
* space-evenly: 每个网格之间间距相同，包括到两端距离

![](https://upload-images.jianshu.io/upload_images/8493649-abf0ef4df884f055.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  box-shadow: 0px 0px 2px 1px #cccccc;
  width: 250px;
  height: 250px;
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 50px);
  place-content: end end;
}
```
如上图所示，整个内容区在容器的水平方向末端，竖直方向末端。

![](https://upload-images.jianshu.io/upload_images/8493649-b7230192eec730bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  box-shadow: 0px 0px 2px 1px #cccccc;
  width: 250px;
  height: 250px;
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 50px);
  place-content: space-evenly space-evenly;
}
```
如上图所示， 每个网格之间间距相同，包括到两端距离。

### `grid-auto-rows grid-auto-column`设置超出定义网格的网格大小

![](https://upload-images.jianshu.io/upload_images/8493649-9d41f461805abd46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
  box-shadow: 0px 0px 2px 1px #cccccc;
  width: 250px;
  height: 250px;
  box-sizing: border-box;
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 50px);
  grid-auto-rows: 80px;
}
```
如上图所示，定义一个3x3网格，第10个网格超出区域，超出行的高度为80px。

### `grid-row grid-column`设置网格自身起始、终止线
`grid-column: grid-column-start / grid-column-end`
`grid-row: grid-row-start / grid-row-end`
`grid-column: <start-line> / <end-line> | <start-line> / span <value>`
 `grid-row: <start-line> / <end-line> | <start-line> / span <value>`

![](https://upload-images.jianshu.io/upload_images/8493649-86c2b5a45133869a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.wrapper {
  box-shadow: 0px 0px 2px 1px #cccccc;
  width: 250px;
  height: 250px;
  box-sizing: border-box;
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(4, 50px);
}
.item-1 {
  /* 简写 */
  grid-row: 2 / 4;
  grid-column: 2 / 4;
  /* 拆开写相同*/
  /* grid-row-start: 2;
  grid-row-end: 4;
  grid-column-start: 2;
  grid-column-end: 4; */
}
```
如上图所示，定义一个3x3网格，第一个网格水平方向起始线为2，终止线为4，竖直方向起始线为2，终止线为4，剩下的网格会按默认依次排列。

### `place-self` 设置单元网格内容位置。
`place-self: <align-self> <justify-self>;`
`justify-self: start | end | center | stretch;`
`align-self: start | end | center | stretch;`
![](https://upload-images.jianshu.io/upload_images/8493649-bde3b637ac548cf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
.wrapper {
  display: grid;
  grid-template-columns: repeat(3, 50px);
  grid-template-rows: repeat(3, 30px);
}
.item-1 {
  place-self: center center;
}
```

如上图所示，`item-1`在第一个网格上下左右中间位置。

## 示例相关代码
[https://github.com/Xiamer/css-example/tree/master/grid](https://github.com/Xiamer/css-example/tree/master/grid)

## grid 兼容性
![](https://upload-images.jianshu.io/upload_images/8493649-c96f58d5ea6a86b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
[https://caniuse.com/#search=css%20grid](https://caniuse.com/#search=css%20grid)


## 参考链接
* [https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Grid_Layout](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Grid_Layout)
* [https://css-tricks.com/snippets/css/complete-guide-grid/#prop-grid-template-areas](https://css-tricks.com/snippets/css/complete-guide-grid/#prop-grid-template-areas)
* [http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html](http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html)

