---
title: JS null、undefined和布尔值
date: 2018-03-02 16:11:14
tags: JS
categories: 
 - 前端
 - JS
---

# 数据类型概述
## 简介
JavaScript语言的每一个值，都属于某一种数据类型。JavaScript的数据类型共有六种。（ES6又新增了第七种`Symbol`类型的值。）
- 数值（`number`）：整数和小数
- 字符串（`string`）：文本。
- 布尔值（`boolean`）：表示真伪的两个特殊值，即`true`（真）和`false`（假）
- `undefined`：表示“未定义”或不存在，即由于目前没有定义，所以此处暂时没有任何值
- `null`：表示空值，即此处的值为空。
- 对象（`object`）：各种值组成的集合。

通常，数值、字符串、布尔值这三种类型，合称为原始类型的值，即它们是最基本的数据类型，不能再细分了。对象则称为合成类型的值，因为一个对象往往是多个原始类型的值的合成，可以看作是一个存放各种值的容器。至于`undefined`和`null`，一般将它们看成两个特殊值。

对象是最复杂的数据类型，又可以分成三个子类型。
- 狭义的对象（`object`）
- 数组（`array`）
- 函数（`function`）

函数其实是处理数据的方法，JavaScript把它当成一种数据类型，可以赋值给变量，这为JavaScript的“函数式编程”奠定了基础。
## typeof 运算符
JavaScript有三种方法，可以确定一个值到底是什么类型。
- `typeof`运算符
- `instanceof`运算符
- `Object.prototype.toString`方法

`typeof`运算符可以返回一个值的数据类型。

数值、字符串、布尔值分别返回`number`、`string`、`boolean`。
```javascript
typeof 123 // "number"
typeof '123' // "string"
typeof false // "boolean"
```
函数返回`function`。
```javascript
function f() {}
typeof f // "function"
```
`undefined`返回`undefined`。
```javascript
typeof undefined // "undefined"
```
利用这一点，`typeof`可以用来检查一个没有声明的变量，而不报错。
```javascript
v // ReferenceError: v is not defined
typeof v  // "undefined"
```
实际编程中，这个特点通常用在判断语句。
```javascript
// 错误的写法
if (v) {
  // ...
}
// ReferenceError: v is not defined

// 正确的写法
if (typeof v === "undefined") {
  // ...
}
```
对象返回`object`。
```javascript
typeof window // "object"
typeof {} // "object"
typeof [] // "object"
```
上面代码中，空数组（`[]`）的类型也是`object`，这表示在JavaScript内部，数组本质上只是一种特殊的对象。这里顺便提一下，`instanceof`运算符可以区分数组和对象。
```javascript
var o = {};
var a = [];

o instanceof Array // false
a instanceof Array // true
```
`null`返回`object`。
```javascript
typeof null // "object"
```
`null`的类型是`object`，这是由于历史原因造成的。1995年的JavaScript语言第一版，只设计了五种数据类型（对象、整数、浮点数、字符串和布尔值），没考虑`null`，只把它当作`object`的一种特殊值。后来`null`独立出来，作为一种单独的数据类型，为了兼容以前的代码，`typeof null`返回`object`就没法改变了。
# null, undefined
## 概述
`null`与`undefined`都可以表示“没有”，含义非常相似。将一个变量赋值为`undefined`或`null`，语法效果几乎没区别。

既然含义与用法都差不多，为什么要同时设置两个这样的值，这不是无端增加复杂度吗？这与历史原因有关。

1995 年 JavaScript 诞生时，最初像 Java 一样，只设置了`null`表示"无"。根据 C 语言的传统，`null`可以自动转为`0`。
```javascript
Number(null) // 0
5 + null // 5
```
但是，JavaScript 的设计者 Brendan Eich，觉得这样做还不够。首先，第一版的JavaScript里面，`null`就像在Java里一样，被当成一个对象，Brendan Eich 觉得表示“无”的值最好不是对象。其次，那时的JavaScript 不包括错误处理机制，Brendan Eich 觉得，如果`null`自动转为 0，很不容易发现错误。

因此，他又设计了一个`undefined`。区别是这样的：`null`是一个表示“空”的对象，转为数值时为`0`；`undefined`是一个表示"此处无定义"的原始值，转为数值时为`NaN`。
```javascript
Number(undefined) // NaN
5 + undefined // NaN
```
## 用法和含义

`null`表示该变量有意缺少对象指向，即该处的值现在为空。`null`常在返回类型应是一个对象，但没有关联的值的地方使用。

`undefined`表示尚未初始化的变量的值。`undefined`是全局对象的一个属性。也就是说，它是全局作用域的一个变量。
```javascript
// 变量声明了，但没有赋值
var i;
i // undefined

// 调用函数时，应该提供的参数没有提供，该参数等于 undefined
function f(x) {
  return x;
}
f() // undefined

// 对象没有赋值的属性
var  o = new Object();
o.p // undefined

// 函数没有返回值时，默认返回 undefined
function f() {}
f() // undefined
```

`undefined`
* 这个变量从根本上就没有定义
* 隐藏式 空值

`null`
* 这个值虽然定义了，但它并未指向任何内存中的对象
* 声明式 空值

## 表现形式
```js
typeof null  // 'object'
typeof undefined  // 'undefined'

null == undefined  // true
null === undefined  // false
!!null === !!undefined  // true

let a = undefined + 1  // NaN
let b = null + 1  // 1
Number(undefined)  // NaN
Number(null)  // 0

JSON.stringify({a: undefined})  // '{}'
JSON.stringify({b: null})  // '{b: null}'
JSON.stringify({a: undefined, b: null})  // '{b: null}'

function test(n) {
    let undefined = 'test'
    return n === undefined
}

test()           // false
test(undefined)  // false
test('test')     // ture

let undefined = 'test'  // Uncaught SyntaxError: Identifier 'undefined' has already been declared
```
在`if`语句中，它们都会被自动转为`false`。
```javascript
if (!undefined) {
  console.log('undefined is false');
}
// undefined is false

if (!null) {
  console.log('null is false');
}
// null is false
```
## 深入探索
#### 为什么 typeof null 是 object？
`typeof null`输出为`object`其实是一个底层的错误，但直到现阶段都无法被修复。

原因是，在 JavaScript 初始版本中，值以 32 位存储。前 3 位表示数据类型的标记，其余位则是值。

对于所有的对象，它的前 3 位都以 000 作为类型标记位。在 JavaScript 早期版本中，`null`被认为是一个特殊的值，用来对应 C 中的空指针 。但 JavaScript 中没有 C 中的指针，所以`null`意味着什么都没有或者`void`并以全 0(32个) 表示。

因此每当 JavaScript 读取`null`时，它前端的 3 位将它视为对象类型 ，这也是为什么`typeof null`返回`object`的原因。
#### 为什么 null + 1 和 undefined + 1 表现不同？
在执行加法运算前，隐式类型转换会尝试将表达式中的变量转换为`number`类型。`null`转化为`number`时，会转换成 0。`undefined`转换为`number`时，会转换为`NaN`。

至于为什么执行如此的转换方式，我猜测是 JavaScript 早期的一个糟糕设计。
#### 为什么 JSON.stringify 会将值为 undefined 的内容删除？
JSON 会将`undefined`对应的`key`删除，这是 JSON 自身的转换原则。

在`undefined`的情况下，有无该条数据是没有区别的，因为他们在表现形式上并无不同：
```js
let obj1 = { a: undefined }
let obj2 = {}

console.log(obj1.a)  // undefined
console.log(obj2.a)  // undefined
```
#### 为什么 let undefiend='test' 可以覆盖掉 JavaScript 自身的 undefined？
JavaScript 对于`undefined`的限制方式为全局创建了一个只读的`undefined`，但是并没有彻底禁止局部`undefined`变量的定义。

据说在 JavaScript 高版本禁止了该操作，但没有准确的依据。

请在任何时候，都不要进行`undefined`变量的覆盖，就算是你的 JSON 转换将`undefined`转换为`''`。也不要通过该操作进行，这将是及其危险的行为。
# 布尔值
布尔值代表“真”和“假”两个状态。“真”用关键字`true`表示，“假”用关键字`false`表示。布尔值只有这两个值。

下列运算符会返回布尔值：
- 前置逻辑运算符： `!`
- 相等运算符：`===`，`!==`，`==`，`!=`
- 比较运算符：`>`，`>=`，`<`，`<=`

如果JavaScript预期某个位置应该是布尔值，会将该位置上现有的值自动转为布尔值。转换规则是除了下面六个值被转为`false`，其他值都视为`true`。
- `undefined`
- `null`
- `false`
- `0`
- `NaN`
- `""`或`''`（空字符串）

布尔值往往用于程序流程的控制。
```javascript
if ('') {
  console.log('true');
}
// 没有任何输出
```
上面代码中，`if`命令后面的判断条件，预期应该是一个布尔值，所以JavaScript自动将空字符串，转为布尔值`false`，导致程序不会进入代码块，所以没有任何输出。

注意，空数组（`[]`）和空对象（`{}`）对应的布尔值，都是`true`。
```javascript
if ([]) {
  console.log('true');
}
// true

if ({}) {
  console.log('true');
}
// true
```