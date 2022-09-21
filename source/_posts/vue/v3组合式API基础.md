---
title: v3组合式API基础
date: 2022-07-23 17:52:35
tags: vue
categories: vue
---


# 创建一个 Vue 应用
## 应用实例
每个 Vue 应用都是通过`createApp`函数创建的一个新的应用实例：
```ts
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})
```
## 根组件
我们传入`createApp`的对象实际上是一个组件，每个应用都需要一个“根组件”，其他组件将作为其子组件。

如果你使用的是单文件组件，我们可以直接从另一个文件中导入根组件。
```ts
import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
```
## 挂载应用
应用实例必须在调用了`.mount()`方法后才会渲染出来。该方法接收一个“容器”参数，可以是一个实际的 DOM 元素或是一个 CSS 选择器字符串：
```html
<div id="app"></div>
```
```ts
app.mount('#app')
```
应用根组件的内容将会被渲染在容器元素里面。容器元素自己将不会被视为应用的一部分。

`.mount()`方法应该始终在整个应用配置和资源注册完成后被调用。同时应注意，不同于其他资源注册方法，它的返回值是根组件实例而非应用实例。
## 多个应用实例
你不必再受限于一个页面只能拥有一个应用实例。`createApp` API 允许多个 Vue 应用共存于同一个页面上，而且每个应用都拥有自己的用于配置和全局资源的作用域。
```ts
const app1 = createApp({
  /* ... */
})
app1.mount('#container-1')

const app2 = createApp({
  /* ... */
})
app2.mount('#container-2')
```
# 响应式基础
## 声明响应式状态
我们可以使用`reactive()`函数创建一个响应式对象或数组：
```
<template>
  <button @click="increment">
    { { state.count } }
  </button>
</template>

<script setup>
import { reactive } from 'vue'

const state = reactive({ count: 0 })

function increment() {
  state.count++
}
</script>

```
响应式对象其实是 JavaScript `Proxy`，其行为表现与一般对象相似。不同之处在于 Vue 能够跟踪对响应式对象`property`的访问与更改操作。
#### DOM 更新时机
当你更改响应式状态后，DOM 也会自动更新。然而，你得注意 DOM 的更新并不是同步的。相反，Vue 将缓冲它们直到更新周期的 “下个时机” 以确保无论你进行了多少次声明更改，每个组件都只需要更新一次。

若要等待一个状态改变后的 DOM 更新完成，可以使用`nextTick()`这个全局 API：
```ts
import { nextTick } from 'vue'

function increment() {
  state.count++
  nextTick(() => {
    // 访问更新后的 DOM
  })
}
```
#### 深层响应性
在 Vue 中，状态都是默认深层响应式的。这意味着即使在更改深层次的对象或数组，你的改动也能被检测到。
```ts
import { reactive } from 'vue'

const obj = reactive({
  nested: { count: 0 },
  arr: ['foo', 'bar']
})

function mutateDeeply() {
  // 以下都会按照期望工作
  obj.nested.count++
  obj.arr.push('baz')
}
```
#### 响应式代理 vs 原始对象
`reactive()`返回的是一个原始对象的`Proxy`，它和原始对象是不相等的：
```js
const raw = {}
const proxy = reactive(raw)

// 代理和原始对象不是全等的
console.log(proxy === raw) // false
```
只有代理是响应式的，更改原始对象不会触发更新。因此，使用 Vue 的响应式系统的最佳实践是 仅使用你声明对象的代理版本。

为保证访问代理的一致性，对同一个对象调用`reactive()`会总是返回同样的代理，而对一个已存在代理调用`reactive()`也是返回同样的代理：
```js
// 在同一个对象上调用 reactive() 会返回相同的代理
console.log(reactive(raw) === proxy) // true

// 在一个代理上调用 reactive() 会返回它自己
console.log(reactive(proxy) === proxy) // true
```
这个规则对嵌套对象也适用。依靠深层响应性，响应式对象内的嵌套对象依然是代理：
```js
const proxy = reactive({})

const raw = {}
proxy.nested = raw

console.log(proxy.nested === raw) // false
```
#### reactive() 的局限性
`reactive()` API 有两条限制：
* 仅对对象类型有效（对象、数组和`Map、Set`这样的集合类型），而对`string、number`和`boolean`这样的原始类型无效。
* 因为 Vue 的响应式系统是通过`property`访问进行追踪的，因此我们必须始终保持对该响应式对象的相同引用。这意味着我们不可以随意地“替换”一个响应式对象，因为这将导致对初始引用的响应性连接丢失：

```js
let state = reactive({ count: 0 })

// 上面的引用 ({ count: 0 }) 将不再被追踪（响应性连接已丢失！）
state = reactive({ count: 1 })
```
同时这也意味着当我们将响应式对象的 property 赋值或解构至本地变量时，或是将该 property 传入一个函数时，我们会失去响应性：
```js
const state = reactive({ count: 0 })

// n 是一个局部变量，同 state.count
// 失去响应性连接
let n = state.count
// 不影响原始的 state
n++

// count 也和 state.count 失去了响应性连接
let { count } = state
// 不会影响原始的 state
count++

// 该函数接收一个普通数字，并且
// 将无法跟踪 state.count 的变化
callSomeFunction(state.count)
```
## ref() 定义响应式变量
为了解决`reactive()`带来的限制，Vue 也提供了一个`ref()`方法来允许我们创建可以使用任何值类型的响应式`ref`：
```js
import { ref } from 'vue'

const count = ref(0)
```
`ref()`从参数中获取到值，将其包装为一个带`.value property`的`ref`对象：
```js
const count = ref(0)

console.log(count) // { value: 0 }
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```
和响应式对象的`property`类似，`ref`的`.value property`也是响应式的。同时，当值为对象类型时，会用`reactive()`自动转换它的`.value`。

一个包含对象类型值的`ref`可以响应式地替换整个对象：
```js
const objectRef = ref({ count: 0 })

// 这是响应式的替换
objectRef.value = { count: 1 }
```
`ref`被传递给函数或是从一般对象上被解构时，不会丢失响应性：
```js
const obj = {
  foo: ref(1),
  bar: ref(2)
}

// 该函数接收一个 ref
// 需要通过 .value 取值
// 但它会保持响应性
callSomeFunction(obj.foo)

// 仍然是响应式的
const { foo, bar } = obj
```
#### ref 在模板中的解包
当`ref`在模板中作为顶层`property`被访问时，它们会被自动“解包”，所以不需要使用`.value`。下面是之前的计数器例子，用`ref()`代替：
```js
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">
    { { count } } <!-- 无需 .value -->
  </button>
</template>
```
请注意，仅当`ref`是模板渲染上下文的顶层`property`时才适用自动“解包”。 例如，`foo`是顶层`property`，但`object.foo`不是。

所以我们给出以下`object`：
```
const object = { foo: ref(1) }
```
下面的表达式将不会像预期的那样工作：
```js
{ { object.foo + 1 } }
```
渲染的结果会是一个`[object Object]`，因为`object.foo`是一个`ref`对象。我们可以通过让`foo`成为顶级`property`来解决这个问题：
```js
const { foo } = object
{ { foo + 1 } }
```
现在渲染结果将是 2。

需要注意的是，如果一个`ref`是文本插值（即一个`{ { } }`符号）计算的最终值，它也将被解包。因此下面的渲染结果将为 1：
```
{ { object.foo } }
```
这只是文本插值的一个方便功能，相当于`{ { object.foo.value } }`。
#### ref 在响应式对象中的解包
当一个`ref`作为一个响应式对象的`property`被访问或更改时，它会自动解包，因此会表现得和一般的`property`一样：
```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```
如果将一个新的`ref`赋值给一个关联了已有`ref`的`property`，那么它会替换掉旧的`ref`：
```
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
// 原始 ref 现在已经和 state.count 失去联系
console.log(count.value) // 1
```
#### 数组和集合类型的 ref 解包
不像响应式对象，当`ref`作为响应式数组或像`Map`这种原生集合类型的元素被访问时，不会进行解包。
```
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```
# 计算属性
模板中的表达式虽然方便，但也只能用来做简单的操作。如果在模板中写太多逻辑，会让模板变得臃肿，难以维护。比如说，我们有这样一个包含嵌套数组的对象：
```
const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})
```
我们想根据`author`是否已有一些书籍来展示不同的信息：
```html
<p>Has published books:</p>
<span>{ {author.books.length> 0 ? 'Yes' : 'No'} }</span>
```
这里的模板看起来有些复杂。我们必须认真看好一会儿才能明白它的计算依赖于`author.books`。更重要的是，如果在模板中需要不止一次这样的计算，我们可能不想写重复的代码。

推荐使用计算属性来描述依赖响应式状态的复杂逻辑。
```
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// 一个计算属性 ref
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
  <p>Has published books:</p>
  <span>{ { publishedBooksMessage } }</span>
</template>
```
我们在这里定义了一个计算属性`publishedBooksMessage`。`computed()`方法期望接收一个`getter`函数，返回值为一个计算属性`ref`。和其他一般的`ref`类似，可以通过`publishedBooksMessage.value`访问计算结果。计算属性`ref`也会在模板中自动解包，因此在模板表达式中引用时无需添加`.value`。

Vue 的计算属性会自动追踪响应式依赖。它会检测到`publishedBooksMessage`依赖于`author.books`，所以当`author.books`改变时，任何依赖于`publishedBooksMessage`的绑定都会同时更新。
# 侦听器
## 基本示例
计算属性允许我们声明性地计算推导值。然而，在有些情况下，为了应对一些状态的变化，我们需要运行些“副作用”：例如更改 DOM，或者根据异步操作的结果，去修改另一处的状态。

在组合式 API 中，我们可以使用`watch`函数在每次响应式状态发生变化时触发回调函数：
```
<script setup>
import { ref, watch } from 'vue'

const question = ref('')
const answer = ref('Questions usually contain a question mark. ;-)')

// 可以直接侦听一个 ref
watch(question, async (newQuestion, oldQuestion) => {
  if (newQuestion.indexOf('?') > -1) {
    answer.value = 'Thinking...'
    try {
      const res = await fetch('https://yesno.wtf/api')
      answer.value = (await res.json()).answer
    } catch (error) {
      answer.value = 'Error! Could not reach the API. ' + error
    }
  }
})
</script>

<template>
  <p>
    Ask a yes/no question:
    <input v-model="question" />
  </p>
  <p>{ { answer } }</p>
</template>
```
## 侦听来源类型
`watch`的第一个参数可以是不同形式的“来源”：它可以是一个`ref`(包括计算属性)、一个响应式对象、一个`getter`函数、或多个来源组成的数组：
```
const x = ref(0)
const y = ref(0)

// 单个 ref
watch(x, (newX) => {
  console.log(`x is ${newX}`)
})

// getter 函数
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`sum of x + y is: ${sum}`)
  }
)

// 多个来源组成的数组
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x is ${newX} and y is ${newY}`)
})
```
注意，你不能侦听响应式对象的`property`，例如:
```
const obj = reactive({ count: 0 })

// 这不起作用，因为你向 watch() 传入了一个 number
watch(obj.count, (count) => {
  console.log(`count is: ${count}`)
})
```
而是用`getter`函数：
```
// 提供一个 getter 函数
watch(
  () => obj.count,
  (count) => {
    console.log(`count is: ${count}`)
  }
)
```
## 深层侦听器
直接给`watch()`传入一个响应式对象，会隐式地创建一个深层侦听器——该回调函数在所有嵌套的变更时都会被触发：
```
const obj = reactive({ count: 0 })

watch(obj, (newValue, oldValue) => {
  // 在嵌套的 property 变更时触发
  // 注意：`newValue` 此处和 `oldValue` 是相等的
  // 因为它们是同一个对象！
})

obj.count++
```
这不同于返回响应式对象的`getter`函数：只有在`getter`函数返回不同的对象时，才会触发回调：
```
watch(
  () => state.someObject,
  () => {
    // 仅当 state.someObject 被替换时触发
  }
)
```
然而，在上面的例子里，你可以显式地加上`deep`选项，强制转成深层侦听器：
```
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // 注意：`newValue` 此处和 `oldValue` 是相等的
    // *除非* state.someObject 被整个替换了
  },
  { deep: true }
)
```
## watchEffect()
`watch()`是懒执行的：仅在侦听源变化时，才会执行回调。但在某些场景中，我们希望在创建侦听器时，立即执行一遍回调。举个例子，我们想请求一些初始数据，然后在相关状态更改时重新请求数据。我们可以这样写：
```
const url = ref('https://...')
const data = ref(null)

async function fetchData() {
  const response = await fetch(url.value)
  data.value = await response.json()
}

// 立即获取
fetchData()
// ...再侦听 url 变化
watch(url, fetchData)
```
这段代码还可以用`watchEffect`函数来简化。`watchEffect()`会立即执行一遍回调函数，如果这时函数产生了副作用，Vue 会自动追踪副作用的依赖关系，自动分析出响应源。上面的例子可以重写为：
```
watchEffect(async () => {
  const response = await fetch(url.value)
  data.value = await response.json()
})
```
这个例子中，回调会立即执行。在执行期间，它会自动追踪`url.value`作为依赖（近似于计算属性）。每当`url.value`变化时，回调会再次执行。
## watch vs watchEffect
`watch`和`watchEffect`都能响应式地执行有副作用的回调。它们之间的主要区别是追踪响应式依赖的方式：
* `watch`只追踪明确侦听的源。它不会追踪任何在回调中访问到的东西。另外，仅在响应源确实改变时才会触发回调。`watch`会避免在发生副作用时追踪依赖，因此，我们能更加精确地控制回调函数的触发时机。
* `watchEffect`，则会在副作用发生期间追踪依赖。它会在同步执行过程中，自动追踪所有能访问到的响应式`property`。这更方便，而且代码往往更简洁，但其响应性依赖关系不那么明确。

## 回调的刷新时机
当你更改了响应式状态，它可能会同时触发 Vue 组件更新和侦听器回调。

默认情况下，用户创建的侦听器回调，都会在 Vue 组件更新之前被调用。这意味着你在侦听器回调中访问的 DOM 将是被 Vue 更新之前的状态。

如果想在侦听器回调中能访问被 Vue 更新之后的 DOM，你需要指明`flush: 'post'`选项：
```
watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
```
后置刷新的`watchEffect()`有个更方便的别名`watchPostEffect()`：
```
import { watchPostEffect } from 'vue'

watchPostEffect(() => {
  /* 在 Vue 更新后执行 */
})
```
## 停止侦听器
在`<script setup>`中用同步语句创建的侦听器，会自动绑定到宿主组件实例上，并且会在宿主组件卸载时自动停止。因此，在大多数情况下，你无需关心怎么停止一个侦听器。

一个关键点是，侦听器必须用同步语句创建：如果用异步回调创建一个侦听器，那么它不会绑定到当前组件上，你必须手动停止它，以防内存泄漏。如下方这个例子：
```
<script setup>
import { watchEffect } from 'vue'

// 它会自动停止
watchEffect(() => {})

// ...这个则不会！
setTimeout(() => {
  watchEffect(() => {})
}, 100)
</script>
```
要手动停止一个侦听器，请调用`watch`或`watchEffect`返回的函数：
```
const unwatch = watchEffect(() => {})

// ...当该侦听器不再需要时
unwatch()
```
注意，需要异步创建侦听器的情况很少，请尽可能选择同步创建。如果需要等待一些异步数据，你可以使用条件式的侦听逻辑：
```
// 需要异步请求得到的数据
const data = ref(null)

watchEffect(() => {
  if (data.value) {
    // 数据加载后执行某些操作...
  }
})
```
# 模板 ref
虽然 Vue 的声明性渲染模型为你抽象了大部分对 DOM 的直接操作，但在某些情况下，我们仍然需要直接访问底层 DOM 元素。要实现这一点，我们可以使用特殊的`ref attribute`：
```
<input ref="input">
```
`ref`是一个特殊的`attribute`，和`v-for`中的`key`类似。它允许我们在一个特定的 DOM 元素或子组件实例被挂载后，获得对它的直接引用。这可能很有用，比如说在组件挂载时编程式地聚焦到一个`input`元素上，或在一个元素上初始化一个第三方库。
## 访问模板 ref
为了通过组合式 API 获得该模板`ref`，我们需要声明一个同名的`ref`：
```
<script setup>
import { ref, onMounted } from 'vue'

// 声明一个 ref 来存放该元素的引用
// 必须和模板 ref 同名
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```
如果你正试图观察一个模板`ref`的变化，确保考虑到`ref`的值为`null`的情况：
```
watchEffect(() => {
  if (input.value) {
    input.value.focus()
  } else {
    // 此时还未挂载，或此元素已经被卸载（例如通过 v-if 控制）
  }
})
```
## v-for 中的 ref
当`ref`在`v-for`中使用时，相应的`ref`中包含的值是一个数组，它将在元素被挂载后填充：
```
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      { { item } }
    </li>
  </ul>
</template>
```
应该注意的是，`ref`数组不能保证与源数组相同的顺序。
## 函数型 ref
除了使用字符串值作名字，`ref attribute`还可以绑定为一个函数，会在每次组件更新时都被调用。函数接受该元素引用作为第一个参数：
```
<input :ref="(el) => { /* 将 el 分配给 property 或 ref */ }">
```
如果你正在使用一个动态的`:ref`绑定，我们也可以传一个函数。当元素卸载时，这个`el`参数会是`null`。你当然也可以使用一个方法而不是内联函数。
## 组件上的 ref
`ref`也可以被用在一个子组件上。此时`ref`中引用的是组件实例：
```
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const child = ref(null)

onMounted(() => {
  // child.value 是 <Child /> 组件的实例
})
</script>

<template>
  <Child ref="child" />
</template>
```
如果一个子组件使用的是选项式 API 或没有使用`<script setup>`，被引用的组件实例和该子组件的`this`完全一致，这意味着父组件对子组件的每一个属性和方法都有完全的访问权。这使得在父组件和子组件之间创建紧密耦合的实现细节变得很容易，当然也因此，应该只在绝对需要时才使用组件引用。大多数情况下，你应该首先使用标准的`props`和`emit`接口来实现父子组件交互。

有一个例外的情况，使用了`<script setup>`的组件是默认私有的：一个父组件无法访问到一个使用了`<script setup>`的子组件中的任何东西，除非子组件在其中通过`defineExpose`宏显式暴露：
```
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```
当父组件通过模板`ref`获取到了该组件的实例时，得到的实例类型为`{ a: number, b: number }` (`ref`都会自动解包，和一般的实例一样)。
