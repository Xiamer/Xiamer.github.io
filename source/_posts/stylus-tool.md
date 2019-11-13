---
title: stylus-tool
date: 2019-08-26 17:36:16
categories: 
- web前端
tags:
- css
  
---

自适应经常用到的方法

```stylus
/*
 * 元素尺寸 vw
 *
 * $n 设计稿元素尺寸
 * $base 设计稿尺寸
 */
size($n, $base) {
  $n / $base * 100vw;
}

/*
 * 求和
 *
 * arguments
 * return px单位尺寸
 */
sum() {
  n = 0;

  for num in arguments {
    n = n + num;
  }

  return unit(n, 'px');
}

/*
 * 元素宽
 *
 * cw 内容区域宽
 * n 元素列数
 * space 横向留白
 */
sizeW(cw, n, space) {
  'calc((%s - %s) / %s)' % (cw space n);
}

/*
 * 元素高
 *
 * cw 内容区域宽
 * n 元素列数
 * space 横向留白
 * ow 元素宽
 * oh 元素高
 */
sizeH(cw, n, space, ow, oh) {
  scale = (ow / oh);
  'calc((%s - %s) / %s  / %s)' % (cw space n scale);
}
```