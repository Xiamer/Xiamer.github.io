---
title: vue 组件生成文档
date: 2021-06-25 14:27:22
tags: vue
---

## 思路

组件库搭建完毕后，如何让使用者像其他`ui库`有良好的文档，方便大家使用。
像 `jsDoc`自动生成文档，社区也会有 `vue component` 生成文档。

因每个组件文档内容有部分定制化，我们需要需要做3件事

1. 将组件`prop` `emit` `slot` 注释生成 `json`
2. 将 `json` 生成 `markdown`
3. 合并原有 `markdown` 和 新生成的`markdown`

## 组件注释生成json

借助于[vue-docgen-api](https://github.com/vue-styleguidist/vue-styleguidist/tree/dev/packages/vue-docgen-api)生成`json`,注释的写法和`jsDoc`写法很类似。

### props 注释
```js
  props: {
    /**
     * Model example
     * @model
     */
    value: {
      type: String,
    },
    /**
     * this is string && values
     * @values small, normal, large
     */
    size: {
      type: String,
      default: "normal",
    },
    /**
     * this is a string or number
     */
    msg: {
      type: [String, Number],
      default: '',
    },
    /**
     * this is boolean
     */
    bool: {
      type: Boolean,
      default: false
    },
    /**
     * this is object
     */
    obj: {
      type: Object,
      default() {
        return {a: {a: 1}}
      },
    },
    /**
     * this is fn
     */
    onClick: {
      type: Function,
      default(event) {
        console.log("You have clicked me!", event.target);
      },
    },
    /**
     * get columns list
     */
    columns: [Array]
  },
```

### slot 注释

```html
<template>
  <div class="Xh-demo">
    <p style="color: red;">我是demo组件2222s</p>
    <!-- @slot 我是默认 -->
    <slot></slot>

    <!-- @slot 我是头部 -->
    <slot name="header"></slot>

    <table class="grid">
      <!-- -->
    </table>

    <!--
      @slot 我是底部
      @binding list 一组数据
      @binding age 年龄 
		-->
    <slot name="footer" :list="list" :age="age" />
  </div>
</template>
```

### emit 注释

```js
  methods: {
    /**
     * Sets the order
     */
    sortBy (key) {
      this.sortKey = key;
      this.sortOrders[key] = this.sortOrders[key] * -1;

      /**
       * Success event.
       *
       * @property {object} 当前选中的对象
       * @property {string} 当前选中的值
       */
      this.$emit("success", {a: {b: 2}}, 'demo');
    },

    emitSuccess() {
      /**
       * Error event.
       */
      this.$emit('error')
    },

    hiddenMethod: function () {},
  }
```
### 使用vue-docgen-api

```js
// 安装
npm install vue-docgen-api --save-dev

// 使用
var vueDocs = require('vue-docgen-api')
vueDocs.parse(filePath).then(json => {
  console.log(json)
})
```

将生成如下json

```js
{
//  displayName: 'XhDemo',
  // description: 'This is test demo',
  // tags: {},
  // exportName: 'default',
  props:
   [ { name: 'v-model',
       description: 'Model example',
       tags: [Object],
       type: [Object] },
     { name: 'size',
       description: 'this is string && values',
       tags: {},
       values: [Array],
       type: [Object],
       defaultValue: [Object] },
     { name: 'msg',
       description: 'this is a string or number',
       type: [Object],
       defaultValue: [Object] },
     { name: 'bool',
       description: 'this is boolean',
       type: [Object],
       defaultValue: [Object] },
     { name: 'obj',
       description: 'this is object',
       type: [Object],
       defaultValue: [Object] },
     { name: 'onClick',
       description: 'this is fn',
       type: [Object],
       defaultValue: [Object] },
     { name: 'columns',
       description: 'get columns list',
       type: [Object] } ],
  events:
   [ { name: 'success',
       description: 'Success event.',
       type: [Object],
       properties: [Array] },
     { name: 'error', description: 'Error event.', type: undefined } ],
  methods: undefined,
  slots:
   [ { name: 'default', description: '我是默认' },
     { name: 'header', description: '我是头部' },
     { name: 'footer',
       scoped: true,
       description: '我是底部',
       bindings: [Array] } ] }
}
```

## json 生成 markdown

[json2md](https://github.com/IonicaBizau/json2md)可以将`json`生成`markdown`，但当表格数据大于一行时，生成的有bug。已有[mr](https://github.com/IonicaBizau/json2md/pull/81)，作者还没合。
其实我们只需将数组的数据生成`markdown`。可借助于[tablemark](https://github.com/citycide/tablemark)生成。

```js
tablemark([
  { name: 'Bob', age: 21, isCool: false },
  { name: 'Sarah', age: 22, isCool: true },
  { name: 'Lee', age: 23, isCool: true }
], {
  columns: [
    'first name',
    { name: 'how old', align: 'center' },
    'are they cool'
  ]
})

// | first name | how old | are they cool |
// | ---------- | :-----: | ------------- |
// | Bob        |   21    | false         |
// | Sarah      |   22    | true          |
// | Lee        |   23    | true          |
```

## 合并原有 `markdown` 和 新生成的`markdown`

通过 `fs` 模块 读取原内容，然后再去做追加,完整代码如下

```js
// bin/docs-all.js
const fs = require('fs')
const { generatorComMd } = require('./docs-base')

fs.readFile(process.cwd() + '/components.json', 'utf8', function (err, data) {
  if (err) console.log(err);
  try {
    const arr = Object.keys(JSON.parse(data));
    arr.forEach(key => {
      generatorComMd(key)
    });
  } catch (error) {
    console.log('err', err)
  }
});

// bin/docs-base.js
const fs = require('fs')
const vueDocs = require('vue-docgen-api')
const tablemark = require('tablemark')

const MD_VALUE = `<!-- 以下为自动生成文档 -->`


/**
 * 生成文档的md内容 , props slot event(emit)
 * 
 * @param {string} path 
 */
 async function generatorComMd(filename) {
  const comppath = process.cwd() + `/packages/${filename}/src/main.vue`
  console.log(`[info]: 检查路径 ${comppath}`)
  if (fs.existsSync(comppath)) {
    const json = await vueDocs.parse(comppath)

    console.log(json)

    const propsMd = json.props && generatorPropsMd(json.props) || ''
    const eventMd = json.events && generatorEventMd(json.events) || ''
    const slotsMd = json.slots && generatorSlotsMd(json.slots) || ''

    const md = `${MD_VALUE}\n${propsMd}${eventMd}${slotsMd}`

    // console.log('[success]: 文档内容已生成。')
    // console.log('------------ 开始合并文档 ------------')
    const mdPath = process.cwd() + `/examples/docs/${filename}.md`
    console.log(`[info]: 检查路径 ${mdPath}`)
    if (fs.existsSync(mdPath)) {
      fs.readFile(mdPath, 'utf8', function (err, data) {
        if (err) {
          return console.log(err);
        } else {
          let content = data
          if (data.includes(MD_VALUE)) {
            content = content.split(MD_VALUE)[0] + md
          } else {
            const prefix = content.endsWith(`\n`) ? `` : `\n`
            content = `${content}${prefix}${md}`
          }
          fs.writeFile(mdPath, content, 'utf8', function (err) {
            if (err) return console.log('写入文件异常', err);
            console.log(`[success]: 组件 ${filename} 文档生成成功!`)
          });
        }
      });
    } else {
      console.error(`[error]: 文件不存在，请检查文件路径`)
    }
  } else {
    console.error(`[error]: 文件不存在，请检查文件路径`)
  }
}


/**
 * 
 * @param {Array} aSlotsList 
 * @returns slots markdown
 */
function generatorSlotsMd(aSlotsList) {
  const slotsTable = aSlotsList.map(v => {
    let bindParam = '-'
    if (v.bindings) {
      bindParam = v.bindings.map(v => {
        return `参数${v.name}:  ${v.description}`
      }).join('<br />')
    }
    return {
      name: v.name,
      desc: v.description,
      bindParam
    }
  })

  const slotsMd = tablemark(slotsTable, {
    columns: ['name', '说明', '绑定参数']
  })
  return `### Slot\n${slotsMd}\n`
}


/**
 * 
 * @param {Array} aPropsList 
 * @returns props markdown
 */
function generatorPropsMd(aPropsList) {
  const propsTable = aPropsList.map(v => {
    return {
      name: v.name,
      desc: v.description,
      type: v.type.name,
      values: v.values || '-',
      defaultValue: v.defaultValue && v.type.name !== 'func' ? v.defaultValue.value : '-',
    }
  })

  const propsMd = tablemark(propsTable, {
    columns: ['参数', '说明', '类型', '可选值', '默认值']
  })
  return `### Attribute\n${propsMd}\n`
}


/**
 * 
 * @param {Array} aEmitList
 * @returns emit markdown
 */
function generatorEventMd(aEmitList) {
  const emitTable = aEmitList.map(v => {
    let cbParams = '-'
    if (v.properties) {
      cbParams = v.properties.map((v, i) => {
        return `参数${i} ${v.type.names}:  ${v.description}`
      }).join('<br />')
    }
    return {
      name: v.name,
      desc: v.description,
      cbParams: cbParams
    }
  })
  const emitMd = tablemark(emitTable, {
    columns: ['事件名称', '说明', '回调参数']
  })
  return `### Methods\n${emitMd}\n`
}


module.exports = {
  generatorComMd
}
```

## 实现结果

执行 `node bin/docs-all.js` 最终实现结果如下

![](/images/vue-gen-docs.png)


## 参考


 * [vue-docgen-api](https://github.com/vue-styleguidist/vue-styleguidist/tree/dev/packages/vue-docgen-api)
 * [tablemark](https://github.com/citycide/tablemark)
 * [element ui](https://element.eleme.cn/#/zh-CN/component/table)