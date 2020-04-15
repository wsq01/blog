---
title: vue生命周期深入
date: 2020-04-14 11:42:05
tags: vue
---

# 生命周期钩子函数
生命周期：Vue实例从开始创建、初始化数据、编译模板、挂载Dom→渲染、更新→渲染、卸载等一系列过程，我们称这是Vue的生命周期，各个阶段有相对应的事件钩子。

| 生命周期钩子 | 组件状态 | 最佳实践 |
| :--: | :--: | :--: |
| `beforeCreate`(创建前) | 实例初始化之后，`this`指向创建的实例，不能访问到`data、computed、watch、methods`上的方法和数据| 常用于初始化非响应式变量 |
| `created`(创建后) | 实例创建完成，可访问`data、computed、watch、methods`上的方法和数据，未挂载到DOM，不能访问`$el`属性，`$refs`属性内容为空数组 | 常用于简单的ajax请求，页面的初始化 |
| `beforeMount`(挂载前) | 在挂载开始之前被调用，相关的`render`函数首次被调用，实例已完成以下的配置：编译模板，把`data`里面的数据和模板生成html，注意此时还没有挂载html到页面上 | |
| `mounted`(挂载后) | 实例挂载到DOM上，此时可以通过DOM API获取到DOM节点，`$refs`属性可以访问 | 常用于获取VNode信息和操作，ajax请求 |
| `beforeUpdate`(更新前) | 响应式数据更新前调用，发生在虚拟DOM重新渲染和打补丁之前 | 适合在更新前访问现有的DOM，比如手动移除已添加的事件监听器 |
| `updated`(更新后) | 虚拟DOM重新渲染和打补丁之后，组件DOM已经更新，可执行依赖于DOM的操作 | 避免在这个钩子函数中操作数据，可能陷入死循环 |
| `beforeDestroy`(销毁前) | 实例销毁之前调用，实例仍然完全可用 | 常用于销毁定时器，解绑全局事件，销毁插件对象等 |
| `destroyed`(销毁后) | 实例销毁后调用，调用后，所有的事件监听器会被移除，所有的子实例也会被销毁 | |

# 单个组件的生命周期
```html
<template>
  <div>
    <h3>单组件</h3>
    <el-button @click="dataVar += 1">更新 {{dataVar}}</el-button>
    <el-button @click="handleDestroy">销毁</el-button>
  </div>
</template>
```
```js
export default {
  data() {
    return {
      dataVar: 1
    }
  },
  beforeCreate() {
    this.compName = 'single'
    console.log(`--${this.compName}--beforeCreate`)
  },
  created() {
    console.log(`--${this.compName}--created`)
  },
  beforeMount() {
    console.log(`--${this.compName}--beforeMount`)
  },
  mounted() {
    console.log(`--${this.compName}--mounted`)
  },
  beforeUpdate() {
    console.log(`--${this.compName}--beforeUpdate`)
  },
  updated() {
    console.log(`--${this.compName}--updated`)
  },
  beforeDestroy() {
    console.log(`--${this.compName}--beforeDestroy`)
  },
  destroyed() {
    console.log(`--${this.compName}--destroyed`)
  },
  methods: {
    handleDestroy() {
      this.$destroy()
    }
  }
}
```
初始化组件时，打印：
```js
--single--beforeCreate
--single--created
--single--beforeMount
--single--mounted
```
当`data`中的值变化时，打印：
```js
--single--beforeUpdate
--single--updated
```
当组件销毁时，打印：
```js
--single--beforeDestroy
--single--destroyed
```
从打印结果可以看出:
* 初始化组件时，仅执行了`beforeCreate/Created/beforeMount/mounted`四个钩子函数。
* 当改变`data`中定义的变量（响应式变量）时，会执行`beforeUpdate/updated`钩子函数。
* 当切换组件（当前组件未缓存）时，会执行`beforeDestory/destroyed`钩子函数。
* 初始化和销毁时的生命钩子函数均只会执行一次，`beforeUpdate/updated`可多次执行。

# 父子组件的生命周期
将单组件作为基础组件（由于`props`在`beforeCreate()`中未初始化），需要做如下更改：
```js
props: {
  compName: {
    type: String,
    default: 'single'
  }
},
beforeCreate() {
  // this.compName = 'single'
  // console.log(`--${this.compName}--beforeCreate`)

  console.log(` --data未初始化--beforeCreate`)
}
```
父组件代码如下：
```html
<template>
    <div class="complex">
        <h3>复杂组件</h3>
        <lifecycle-single compName="child"></lifecycle-single>
    </div>
</template>
```
```js
const COMPONENT_NAME = 'complex'

import LifecycleSingle from './LifeCycleSingle'

export default {
  beforeCreate() {
    console.log(`--${COMPONENT_NAME}--beforeCreate`)
  },
  created() {
    console.log(`--${COMPONENT_NAME}--created`)
  },
  beforeMount() {
    console.log(`--${COMPONENT_NAME}--beforeMount`)
  },
  mounted() {
    console.log(`--${COMPONENT_NAME}--mounted`)
  },
  beforeUpdate() {
    console.log(`--${COMPONENT_NAME}--beforeUpdate`)
  },
  updated() {
    console.log(`--${COMPONENT_NAME}--updated`)
  },
  beforeDestroy() {
    console.log(`--${COMPONENT_NAME}--beforeDestroy`)
  },
  destroyed() {
    console.log(`--${COMPONENT_NAME}--destroyed`)
  },
  components: {
    LifecycleSingle
  }
}
```
初始化组件时，打印：
```js
--complex--beforeCreate
--complex--created
--complex--beforeMount
  --data未初始化--beforeCreate
  --child--created
  --child--beforeMount
  --child--mounted
--complex--mounted
--complex--beforeUpdate
--complex--updated
```
当子组件`data`中的值变化时，打印：
```js
--child--beforeUpdate
--child--updated
```
当父组件`data`中的值变化时，打印：
```js
--complex--beforeUpdate
--complex--updated
```
当`props`改变时，打印：
```js
--complex--beforeUpdate
  --child--beforeUpdate
  --child--updated
--complex-updated
```
当子组件销毁时，打印：
```js
--child--beforeDestroy
--child--destroyed
```
当父组件销毁时，打印：
```js
--complex--beforeDestroy
  --child--beforeDestroy
  --child--destroyed
--complex--destroyed
```
从打印结果可以看出：
* 仅当子组件完成挂载后，父组件才会挂载。
* 当子组件完成挂载后，父组件会主动执行一次`beforeUpdate/updated`钩子函数（仅首次）。
* 父子组件在`data`变化中是分别监控的，但是在更新`props`中的数据是关联的。
* 销毁父组件时，先将子组件销毁后才会销毁父组件。

# 兄弟组件的生命周期
在上面的基础上，复杂组件做如下更改：
```html
<template>
  <div class="complex">
    <h3>复杂组件</h3>
    <lifecycle-single compName="cihld1"></lifecycle-single>
    <lifecycle-single compName="child2"></lifecycle-single>
    <el-button @click="dataVar += 1">complex更新 {{dataVar}}</el-button>
    <el-button @click="handleDestroy">complex销毁</el-button>
  </div>
</template>
```
初始化组件时，打印：
```js
--complex--beforeCreate
--complex--created
--complex--beforeMount
  --data未初始化--beforeCreate
  --child1--created
  --child1--beforeMount
  --data未初始化--beforeCreate
  --child2--created
  --child2--beforeMount
  --child1--mounted
  --child2--mounted
--complex--mounted
--complex--beforeUpdate
--complex--updated
```
当`child1`更新和销毁时，打印：
```js
--child1--beforeUpdate
--child1--updated
--child1--beforeDestroy
--child1--destroyed
```
当`child2`更新和销毁时，打印：
```js
--child2--beforeUpdate
--child2--updated
--child2--beforeDestroy
--child2--destroyed
```
当父组件销毁时，打印：
```js
--complex--beforeDestroy
  --child1--beforeDestroy
  --child1--destroyed
  --child2--beforeDestroy
  --child2--destroyed
--complex--destroyed
```
从打印结果可以看出:
* 组件的初始化（`mounted`之前）分开进行，挂载是从上到下依次进行
* 当没有数据关联时，兄弟组件之间的更新和销毁是互不关联的

# mixin的生命周期
在上面的基础上，添加一个mixin.js文件，内容如下：
```js
const COMPONENT_NAME = 'lifecycleMixin'
export default {
  name: COMPONENT_NAME,
  beforeCreate() {
    console.log(`--${COMPONENT_NAME}--beforeCreate`)
  },
  created() {
    console.log(`--${COMPONENT_NAME}--created`)
  },
  beforeMount() {
    console.log(`--${COMPONENT_NAME}--beforeMount`)
  },
  mounted() {
    console.log(`--${COMPONENT_NAME}--mounted`)
  },
  beforeUpdate() {
    console.log(`--${COMPONENT_NAME}--beforeUpdate`)
  },
  updated() {
    console.log(`--${COMPONENT_NAME}--updated`)
  },
  beforeDestroy() {
    console.log(`--${COMPONENT_NAME}--beforeDestroy`)
  },
  destroyed() {
    console.log(`--${COMPONENT_NAME}--destroyed`)
  }
}
```
同样的，复杂组件做如下更改：
```js
import lifecycleMixin from './mixin'

export default {
  mixins: [lifecycleMixin],
  // ...
}
```
组件初始化时，打印：
```js
--lifecycleMixin--beforeCreate
--complex--beforeCreate
--lifecycleMixin--created
--complex--created
--lifecycleMixin--beforeMount
--complex--beforeMount
  --data未初始化--beforeCreate
  --child1--created
  --child1--beforeMount
  --data未初始化--beforeCreate
  --child2--created
  --child2--beforeMount
  --child1--mounted
  --child2--mounted
--lifecycleMixin--mounted
--complex--mounted
--lifecycleMixin--beforeUpdate
--complex--beforeUpdate
--lifecycleMixin--updated
--complex--updated
```
组件销毁时，打印：
```js
--lifecycleMixin--beforeDestroy
--complex--beforeDestroy
  --child1--beforeDestroy
  --child1--destroyed
  --child2--beforeDestroy
  --child2--destroyed
--lifecycleMixin--destroyed
--complex--destroyed
```
从打印结果可以看出:
* `mixin`中的生命周期与引入该组件的生命周期是仅仅关联的，且`mixin`的生命周期优先执行。

