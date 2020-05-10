


# 组件注册
## 组件名
在注册一个组件的时候，我们始终需要给它一个名字。
```js
Vue.component('my-component-name', { /* ... */ })
```
该组件名就是`Vue.component`的第一个参数。
## 组件名大小写
定义组件名的方式有两种：
#### 使用短横线分隔命名
```js
Vue.component('my-component-name', { /* ... */ })
```
当使用短横线分隔命名定义一个组件时，必须在引用这个自定义元素时使用短横线分隔命名，例如 `<my-component-name>`。
#### 使用驼峰式命名
```js
Vue.component('MyComponentName', { /* ... */ })
```
当使用驼峰式命名定义一个组件时，引用这个自定义元素时两种命名法都可以使用。也就是说`<my-component-name>`和`<MyComponentName>`都是可接受的。注意，尽管如此，直接在DOM(即非字符串的模板)中使用时只有短横线分隔命名是有效的。
## 全局注册
```js
Vue.component('my-component-name', {
  // ... 选项 ...
})
```
这些组件是全局注册的。也就是说它们在注册之后可以用在任何新创建的`Vue`根实例 (`new Vue`)的模板中。
```js
Vue.component('component-a', { /* ... */ })
Vue.component('component-b', { /* ... */ })
new Vue({ el: '#app' })

<div id="app">
  <component-a></component-a>
  <component-b></component-b>
</div>
```
在所有子组件中也是如此，也就是说这两个组件在各自内部也都可以相互使用。
## 局部注册
通过一个普通的JavaScript对象来定义组件。
```js
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
```
然后在`components`选项中定义要使用的组件。
```js
new Vue({
  el: '#app'
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```
对于`components`对象中的每个属性来说，其属性名就是自定义元素的名字，其属性值就是这个组件的选项对象。
注意局部注册的组件在其子组件中不可用。例如，如果你希望`ComponentA`在`ComponentB`中可用，则你需要这样写：
```js
var ComponentA = { /* ... */ }
var ComponentB = {
  components: {
    'component-a':ComponentA
  },
  // ...
}
```
或者通过Babel和webpack使用ES2015模块。
```js
import ComponentA from './ComponentA.vue'

export default {
  components: {
    ComponentA
  },
  // ...
}
```
在对象中放一个类似`ComponentA`的变量名其实是`ComponentA: ComponentA`的缩写，即这个变量名同时是：
* 用在模板中的自定义元素的名称
* 包含了这个组件选项的变量名

# 模块系统
## 在模块系统中局部注册
创建一个`components`目录，并将每个组件放置在其各自的文件中。
然后在局部注册之前导入每个你想使用的组件。例如，在一个假设的`ComponentB.js`或`ComponentB.vue`文件中：
```js
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  },
  // ...
}
```
现在`ComponentA`和`ComponentC`都可以在`ComponentB`的模板中使用了。
## 基础组件的自动化全局注册
可能你的许多组件只是包裹了一个输入框或按钮之类的元素，是相对通用的。我们有时候会把它们称为基础组件，它们会在各个组件中被频繁的用到。
所以会导致很多组件里都会有一个包含基础组件的长列表。
```js
import BaseButton from './BaseButton.vue'
import BaseIcon from './BaseIcon.vue'
import BaseInput from './BaseInput.vue'

export default {
  components: {
    BaseButton,
    BaseIcon,
    BaseInput
  }
}
```
而只是用于模板中的一小部分。
```
<BaseInput v-model="searchText" @keydown.enter="search"/>
<BaseButton @click="search">
  <BaseIcon name="search"/>
</BaseButton>
```
如果你使用了webpack，那么就可以使用`require.context`只全局注册这些非常通用的基础组件。这里有一份可以让你在应用入口文件 (比如 `src/main.js`) 中全局导入基础组件的示例代码：
```js
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'
const requireComponent = require.context(
  './components',  // 其组件目录的相对路径
  false,  // 是否查询其子目录
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)
requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)
  // 获取组件的PascalCase命名
  const componentName = upperFirst(
    camelCase(
      // 剥去文件名开头的 `'./` 和结尾的扩展名
      fileName.replace(/^\.\/(.*)\.\w+$/, '$1')
    )
  )
  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过export default导出的，
    // 那么就会优先使用.default，否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```
全局注册的行为必须在根`Vue`实例(通过`new Vue`)创建之前发生。
# Prop
## Prop的大小写
HTML中的特性名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。这意味着当你使用DOM中的模板时，驼峰命名法的`prop`名需要使用其等价的短横线分隔命名。
```js
Vue.component('blog-post', {
  // 在JavaScript中是驼峰命名法的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
<!-- 在 HTML中是短横线分隔命名的 -->
<blog-post post-title="hello!"></blog-post>
```
如果使用字符串模板，那么这个限制就不存在了。
## 静态和动态的Prop
可以像这样给`prop`传入一个静态的值。
```html
<blog-post title="My journey with Vue"></blog-post>
```
`prop`还可以通过`v-bind`动态赋值。
```html
<blog-post v-bind:title="post.title"></blog-post>
```
任何类型的值都可以传给一个`prop`。
#### 传入一个数字
```html
<!-- 即便42是静态的，我们仍然需要v-bind来告诉Vue -->
<!-- 这是一个JavaScript表达式而不是一个字符串 -->
<blog-post v-bind:likes="42"></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:likes="post.likes"></blog-post>
```
#### 传入一个布尔值
```html
<!-- 包含该prop没有值的情况在内，都意味着true -->
<blog-post favorited></blog-post>
<!-- 即便false是静态的，我们仍然需要v-bind来告诉Vue -->
<!-- 这是一个JavaScript表达式而不是一个字符串。-->
<base-input v-bind:favorited="false">
<!-- 用一个变量进行动态赋值。-->
<base-input v-bind:favorited="post.currentUserFavorited">
```
#### 传入一个数组
```html
<!-- 即便数组是静态的，我们仍然需要v-bind来告诉Vue -->
<!-- 这是一个JavaScript表达式而不是一个字符串 -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```
#### 传入一个对象
```html
<!-- 即便对象是静态的，我们仍然需要v-bind来告诉Vue -->
<!-- 这是一个JavaScript表达式而不是一个字符串 -->
<blog-post v-bind:comments="{ id: 1, title: 'My Journey with Vue' }"></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:post="post"></blog-post>
```
#### 传入一个对象的所有属性
如果你想要将一个对象的所有属性都作为`prop`传入，可以使用不带参数的`v-bind`(取代`v-bind:prop-name`)。
```html
post: {
  id: 1,
  title: 'My Journey with Vue'
}
//下面的模板：
<blog-post v-bind="post"></blog-post>
//等价于：
<blog-post v-bind:id="post.id" v-bind:title="post.title"></blog-post>
```
## 单向数据流
所有的`prop`都使得其父子`prop`之间形成了一个单向下行绑定：父级`prop`的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。
额外的，每次父级组件发生更新时，子组件中所有的`prop`都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变`prop`。如果你这样做了，`Vue`会在浏览器的控制台中发出警告。
这里有两种常见的试图改变一个`prop`的情形：
1. 这个`prop`用来传递一个初始值；这个子组件接下来希望将其作为一个本地的`prop`数据来使用。在这种情况下，最好定义一个本地的`data`属性并将这个`prop`用作其初始值。
```js
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```
2. 这个`prop`以一种原始的值传入且需要进行转换。在这种情况下，最好使用这个`prop`的值来定义一个计算属性。
```js
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```
注意在JavaScript中对象和数组是通过引用传入的，所以对于一个数组或对象类型的`prop`来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。
## Prop验证
我们可以为组件的`prop`指定需求。如果有一个需求没有被满足，则`Vue`会在浏览器控制台中警告你。
为了定制`prop`的验证方式，可以为`props`中的值提供一个带有验证需求的对象，而不是一个字符串数组。
```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 匹配任何类型)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组且一定会从一个工厂函数返回默认值
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```
当`prop`验证失败的时候，(开发环境构建版本的)`Vue`将会产生一个控制台的警告。
注意那些`prop`会在一个组件实例创建之前进行验证，所以实例的属性 (如`data`、`computed`等) 在`default`或`validator`函数中是不可用的。
## 类型检查
`type`可以是下列原生构造函数中的一个：`String`、`Number`、`Boolean`、`Function`、`Object`、`Array`、`Symbol`。
额外的，`type`还可以是一个自定义的构造函数，并且通过`instanceof`来进行检查确认。例如，给定下列现成的构造函数：
```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```
你可以使用：
```js
Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```
来验证`author prop`的值是否是通过`new Person`创建的。
## 非Prop的特性
一个非`prop`特性是指传向一个组件，但是该组件并没有相应`prop`定义的特性。
因为显式定义的`prop`适用于向一个子组件传入信息，然而组件库的作者并不总能预见组件会被用于怎样的场景。这也是为什么组件可以接受任意的特性，而这些特性会被添加到这个组件的根元素上。
例如，想象一下你通过一个Bootstrap插件使用了一个第三方的`<bootstrap-data-input>`组件，这个插件需要在其`<input>`上用到一个`data-date-picker`特性。我们可以将这个特性添加到你的组件实例上：
```html
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```
然后这个`data-date-picker="activated"`特性就会自动添加到`<bootstrap-date-input>`的根元素上。
#### 替换/合并已有的特性
想象一下 `<bootstrap-date-input>` 的模板是这样的：
```html
<input type="date" class="form-control">
```
为了给我们的日期选择器插件定制一个主题，我们可能需要像这样添加一个特别的类名：
```html
<bootstrap-date-input
 data-date-picker="activated"
 class="date-picker-theme-dark"
></bootstrap-date-input>
```
在这种情况下，我们定义了两个不同的 `class` 的值：
* `form-control`，这是在组件的模板内设置好的
* `date-picker-theme-dark`，这是从组件的父级传入的

对于绝大多数特性来说，从外部提供给组件的值会替换掉组件内部设置好的值。所以如果传入 `type="text"` 就会替换掉 `type="date"` 并把它破坏！庆幸的是，`class` 和 `style` 特性会稍微智能一些，即两边的值会被合并起来，从而得到最终的值：`form-control date-picker-theme-dark`。
#### 禁用特性继承
如果你不希望组件的根元素继承特性，你可以设置在组件的选项中设置`inheritAttrs: false`。例如：
```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```
这尤其适合配合实例的`$attrs`属性使用，该属性包含了传递给一个组件的特性名和特性值，例如：
```js
{
 class: 'username-input',
 placeholder: 'Enter your username'
}
```
有了 `inheritAttrs: false` 和 `$attrs`，你就可以手动决定这些特性会被赋予哪个元素。在撰写基础组件的时候是常会用到的：
```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```
这个模式允许你在使用基础组件的时候更像是使用原始的HTML元素，而不会担心哪个元素是真正的根元素：
```html
<base-input
  v-model="username"
  class="username-input"
  placeholder="Enter your username"
></base-input>
```
# 插槽
## 插槽内容
`Vue`实现了一套内容分发的API，将`<slot>`元素作为承载分发内容的出口。
它允许你像这样合成组件：
```html
<navigation-link url="/profile">
  Your Profile
</navigation-link>
```
然后你在`<navigation-link>`的模板中可能会写为：
```html
<a v-bind:href="url" class="nav-link">
  <slot></slot>
</a>
```
当组件渲染的时候，这个`<slot>`元素将会被替换为`Your Profile`。插槽内可以包含任何模板代码，包括 HTML。
```html
<navigation-link url="/profile">
  <!-- 添加一个 Font Awesome 图标 -->
  <span class="fa fa-user"></span>
  Your Profile
</navigation-link>
```
甚至其它的组件。
```html
<navigation-link url="/profile">
  <!-- 添加一个图标的组件 -->
  <font-awesome-icon name="user"></font-awesome-icon>
  Your Profile
</navigation-link>
```
如果`<navigation-link>`没有包含一个`<slot>`元素，则任何传入它的内容都会被抛弃。
## 具名插槽
有些时候我们需要多个插槽。例如，一个假设的`<base-layout>`组件多模板如下：
```html
<div class="container">
  <header>
    <!-- 我们希望把页头放这里 -->
  </header>
  <main>
    <!-- 我们希望把主要内容放这里 -->
  </main>
  <footer>
    <!-- 我们希望把页脚放这里 -->
  </footer>
</div>
```
对于这样的情况，`<slot>`元素有一个特殊的特性：`name`。这个特性可以用来定义额外的插槽。
```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```
在向具名插槽提供内容的时候，我们可以在一个父组件的`<template>`元素上使用`slot`特性。
```html
<base-layout>
  <template slot="header">
    <h1>Here might be a page title</h1>
  </template>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
  <template slot="footer">
    <p>Here's some contact info</p>
  </template>
</base-layout>
```
另一种`slot`特性的用法是直接用在一个普通的元素上：
```html
<base-layout>
  <h1 slot="header">Here might be a page title</h1>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
  <p slot="footer">Here's some contact info</p>
</base-layout>
```
我们还是可以保留一个未命名插槽，这个插槽是默认插槽，也就是说它会作为所有未匹配到插槽的内容的统一出口。上述两个示例渲染出来的HTML都将会是：
```html
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```
## 插槽的默认内容
有的时候为插槽提供默认的内容是很有用的。例如，一个`<submit-button>`组件可能希望这个按钮的默认内容是`Submit`，但是同时允许用户覆写为`Save、Upload`或别的内容。
可以在`<slot>`标签内部指定默认的内容来做到这一点。
```html
<button type="submit">
  <slot>Submit</slot>
</button>
```
如果父组件为这个插槽提供了内容，则默认的内容会被替换掉。
# 自定义事件
## 事件名
跟组件和`prop`不同，事件名不存在任何自动化的大小写转换。而是触发的事件名需要完全匹配监听这个事件所用的名称。如果触发一个驼峰式命名名字的事件：
```js
this.$emit('myEvent')
```
则监听这个名字的短横线分隔命名版本是不会有任何效果的：
```html
<my-component v-on:my-event="doSomething"></my-component>
```
跟组件和`prop`不同，事件名不会被用作一个JavaScript变量名或属性名，所以就没有理由使用驼峰式命名了。并且`v-on`事件监听器在DOM模板中会被自动转换为全小写 (因为HTML是大小写不敏感的)，所以`v-on:myEvent`将会变成`v-on:myevent`——导致`myEvent`不可能被监听到。
因此，我们推荐你始终使用短横线分隔命名的事件名。
## 自定义组件的v-model
一个组件上的`v-model`默认会利用名为 `value`的`prop`和名为`input`的事件，但是像单选框、复选框等类型的输入控件可能会将`value`特性用于不同的目的。`model`选项可以用来避免这样的冲突：
```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```
现在在这个组件上使用`v-model`的时候：
```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```
这里的`lovingVue`的值将会传入这个名为`checked`的`prop`。同时当`<base-checkbox>`触发一个`change`事件并附带一个新的值的时候，这个`lovingVue`的属性将会被更新。
注意你仍然需要在组件的`props`选项里声明`checked`这个`prop`。
## 将原生事件绑定到组件
要在一个组件的根元素上直接监听一个原生事件。可以使用`-on`的`.native`修饰符：
```html
<base-input v-on:focus.native="onFocus"></base-input>
```
有的时候这是很有用的，不过在你尝试监听一个类似`<input>`的非常特定的元素时，这并不是个好主意。比如上述`<base-input>`组件可能做了如下重构，所以根元素实际上是一个`<label>`元素：
```html
<label>
  {{ label }}
  <input
    v-bind="$attrs"
    v-bind:value="value"
    v-on:input="$emit('input', $event.target.value)"
  >
</label>
```
这时，父级的`.native`监听器将静默失败。它不会产生任何报错，但是`onFocus`处理函数不会被调用。
为了解决这个问题，Vue提供了一个`$listeners`属性，它是一个对象，里面包含了作用在这个组件上的所有监听器。
```js
{
  focus: function (event) { /* ... */ }
  input: function (value) { /* ... */ },
}
```
有了这个`$listeners`属性，你就可以配合`v-on="$listeners"`将所有的事件监听器指向这个组件的某个特定的子元素。对于类似`<input>`的你希望它也可以配合`v-model`工作的组件来说，为这些监听器创建一个类似下述`inputListeners`的计算属性通常是非常有用的：
```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` 将所有的对象合并为一个新对象
      return Object.assign({},
        // 我们从父级添加所有的监听器
        this.$listeners,
        // 然后我们添加自定义监听器，
        // 或覆写一些监听器的行为
        {
          // 这里确保组件配合 `v-model` 的工作
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```
现在`<base-input>`组件是一个完全透明的包裹器了，也就是说它可以完全像一个普通的`<input>`元素一样使用了：所有跟它相同的特性和监听器的都可以工作。
## .sync修饰符
在有些情况下，我们可能需要对一个`prop`进行“双向绑定”。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。
我们推荐以`update:my-prop-name`的模式触发事件取而代之。举个例子，在一个包含`title prop`的假设的组件中，我们可以用以下方法表达对其赋新值的意图：
```js
this.$emit('update:title', newTitle)
```
然后父组件可以监听那个事件并根据需要更新一个本地的数据属性。例如：
```html
<text-document
 v-bind:title="doc.title"
 v-on:update:title="doc.title = $event"
></text-document>
```
为了方便起见，我们为这种模式提供一个缩写，即 `.sync` 修饰符：
```html
<text-document v-bind:title.sync="doc.title"></text-document>
```
当我们用一个对象同时设置多个`prop`的时候，也可以将这个 `.sync` 修饰符和 `v-bind` 配合使用：
```html
<text-document v-bind.sync="doc"></text-document>
```
这样会把`doc`对象中的每一个属性 (如`title`) 都作为一个独立的`prop`传进去，然后各自添加用于更新的`v-on`监听器。
将`v-bind.sync`用在一个字面量的对象上，例如`v-bind.sync=”{ title: doc.title }”`，是无法正常工作的，因为在解析一个像这样的复杂表达式的时候，有很多边缘情况需要考虑。
