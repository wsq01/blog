

## 组件引入
当使用`setup`的时候，组件直接引入就可以了，不需要再自己手动注册
```vue
<template>
  <Child />
</template>

<script setup lang="ts">
import Child from "./Child.vue";
</script>
```
## ref 和 reactive
`ref`一般用于基本的数据类型，`reactive`一般用于对象。
```vue
<template>
  <h1>{{ title }}</h1>
  <div>
    {{ data }}
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue"

const title = ref("title")

const data = reactive({
  userName: "xiaoming",
  age: 18
})
</script>
```
## defineEmits 和 defineProps 获取父组件传过来值和事件
```ts
// 第一种不带默认值props
const props = defineProps<{
  foo: string
  bar?: number
}>()
// 第二种带默认值props

export interface ChildProps {
  foo: string
  bar?: number
}

const props = withDefaults(defineProps<ChildProps>(), {
  foo: "1qsd"
  bar?: 3
})

// 第一种获取事件
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 第二种获取事件
const emit = defineEmits(["dosth"])
```
## 使用 useAttrs 和 useSlots
useAttrs 可以获取父组件传过来的 id 、class 等值。useSlots 可以获得插槽的内容。例子中，我们使用 useAttrs 获取父组件传过来的 id 、class、useSlots 获取插槽的内容。

父组件：
```vue
<template>
  <div class="father">{{ fatherRef }}</div>
  <Child :fatherRef="fatherRef" @changeVal="changeVal" class="btn" id="111">
    <template #test1>
      <div>1223</div>
    </template>
  </Child>
</template>

<script setup lang="ts">
import { ref } from "vue";

import Child from "./Child.vue";

const fatherRef = ref("1");

function changeVal(val: string) {
  fatherRef.value = val;
}
</script>

<style lang="scss" scoped>
.father {
  margin-top: 40px;
  margin-bottom: 40px;
}
.btn {
  font-size: 20px;
  color: red;
}
</style>
```
子组件：
```vue
<template>
  <!-- <div class="child">{{ props.fatherRef }}</div> -->
  <div v-bind="attrs">
    <slot name="test1">11</slot>
    <input type="text" v-model="inputVal" />
  </div>
</template>

<script setup lang="ts">
import { computed, useAttrs, useSlots } from "vue";

const props = defineProps<{
  fatherRef: string;
}>();

const emits = defineEmits(["changeVal"]);

const slots = useSlots();

const attrs = useAttrs();

console.log(122, attrs, slots);

const inputVal = computed({
  get() {
    return props.fatherRef;
  },

  set(val: string) {
    emits("changeVal", val);
  },
});
</script>
```
## 使用自定义指令
在 setup 里边自定义指令的时候，只需要遵循vNameOfDirective  这样的命名规范就可以了

比如如下自定义 focus 指令，命名就是 vMyFocus，使用的就是 v-my-focus

自定义指令
```vue
<script setup lang="ts">
const vMyFocus = {
  onMounted: (el: HTMLInputElement) => {
    el.focus();
    // 在元素上做些操作
  },
};
</script>
<template>
  <input v-my-focus value="111" />
</template>
```
