

## 1. 
`main.ts ./App.vue`标红，提示找不到模块“./App.vue”或其相应的类型声明。ts(2307)

解决：在`vite-env.d.ts`中加入代码：
```
declare module '*.vue' {
  import type { DefineComponent } from 'vue'
  const vueComponent: DefineComponent<{}, {}, any>
  export default vueComponent
}
```
## 2. 
使用`unplugin-auto-import`自动导入 pinia 时，直接使用`const store = createPinia()`提示找不到名称`“createPinia”。ts(2304)`。

解决：`tsconfig.json`中`include`中加入`auto-imports.d.ts`路径。
## 3. 
`vite.config.ts`中`import path from 'path`标红，提示找不到模块“path”或其相应的类型声明。ts(2307)
解决：`yarn add --dev @types/node`，`tsconfig.json`中加入代码：
```
compilerOptions: {
  "baseUrl": "./",
  "paths": {
    "@/*": ["src/*"]
  }
}
```
## 4.
为了从`store`中提取属性时保持其响应性，需要使用`storeToRefs()`
```js
import useUserStore from '@/store/modules/user'
// store是响应式的，可以直接改store.name = 'xxx'
const store = useUserStore()
// name和avatar是响应式的refs
const { name, avatar } = storeToRefs(store)
// 能取值，非响应式的
const { name, avatar } = store
// 直接取值，非响应式的
const name = store.name
// actions可以直接提取
const { login } = store