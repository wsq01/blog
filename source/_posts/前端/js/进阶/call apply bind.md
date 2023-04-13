---
title: call apply bind
date: 2020-03-12 10:25:14
tags: JS
categories: 
 - 前端
 - JS
---

`call，apply，bind`这三个函数是`Function`原型上的方法`Function.prototype.call()`，`Function.prototype.apply`，`Function.prototype.bind()`，所有的函数都是`Funciton`的实例，因此所有的函数可以调用`call，apply，bind`这三个方法。

# call，apply，bind 在用法上的异同
## 相同点
* 三者都可以改变函数的`this`对象指向。
* 三者第一个参数都是`this`要指向的对象，如果如果没有这个参数或参数为`undefined`或`null`，则默认指向全局`window`。

## 不同点
* 函数调用`call`，`apply`方法时，返回的是调用函数的返回值。而`bind`是返回一个新的函数，你需要再加一个小括号来调用。
* 三者都可以传参，`call`接受的是一系列参数，`apply`接受的是一个数组，`bind`可以分为多次传入。

# Function.prototype.call(thisArg [, arg1, arg2, ...])
`call()`方法调用一个函数，第一个参数是`this`的指向，后面传入的是一个参数列表（注意和`apply`传参的区别）。当第一个参数为`null`或`undefined`的时候，表示指向`window`（在浏览器中），使用`call`方法改变`this`指向后原函数会立即执行，且此方法只是临时改变`this`指向一次。
## 示例
```js
var name = 'zhangsan';
var product = {
  name: 'lisi',
};
function log(...args) {
  console.log(this.name, ...args);
}

log(1, 2, 3);               // zhangsan 1 2 3
log.call(null, 1, 2, 3);    // zhangsan 1 2 3
log.call(product, 1, 2, 3); // lisi 1 2 3
```
# Function.prototype.apply(thisArg [, Array])
`apply()`方法调用一个函数，第一个参数是`this`的指向，后面传入的是一个数组（或类数组对象）形式的参数列表（注意和`call`传参的区别）。当第一个参数为`null`或`undefined`的时候，表示指向`window`（在浏览器中），使用`apply`方法改变`this`指向后原函数会立即执行，且此方法只是临时改变`this`指向一次。
## 示例
```js
var name = 'zhangsan';
var product = {
  name: 'lisi',
};

function log(...args){
  console.log(this.name, ...args);
}

log([1, 2, 3]);                 // zhangsan [1 2 3]
log.apply(null, [1, 2, 3]);     // zhangsan 1 2 3
log.apply(product, [1, 2, 3]);  // lisi 1 2 3
```
# Function.prototype.bind(thisArg [, arg1, arg2, ...])
`bind`方法的第一参数是`this`的指向，后面传入的也是一个参数列表(但是这个参数列表可以分多次传入，`call`则必须一次性传入所有参数)，但是它改变`this`指向后不会立即执行，而是返回一个永久改变`this`指向的函数。
## 示例
```js
// 例一
this.x = 9;

var module = {
  x: 81,
  getX: function() { return this.x; }
};
module.getX();  // 返回 81 （通过对象调用函数， 上下文为该对象）
var retrieveX = module.getX;  // 获取对象中函数的引用地址
retrieveX();    // 返回 9, 在这种情况下， "this" 指向全局作用域（在全局对象下调用函数）
// 永久为函数 boundGetX 绑定 this 上下文
var boundGetX = retrieveX.bind(module);
boundGetX();   // 返回 81 （函数 this 上下文永久绑定为 module）
```
```js
// 例二为回调函数绑定 this 上下文
var x = 10;
var obj = {
  x: 20,
  get: ffunction(){
    console.log(this.x);
  }
};
// 将对象中方法取出（函数的引用地址），作为回调函数， 又因为 setTimeout 回调函数执行的上下文是 window
setTimeout(obj.get, 1000);            // 打印 10
// 将对象中方法取出（函数的引用地址），作为回调函数并绑定 this 上下文
setTimeout(obj.get.bind(obj), 1000);  // 打印 20
```
# 手写 call，apply，bind
## 实现一个call
通过在`thisArg`上临时添加`func`，然后直接调用`thisArg.func()`。
```js
Function.prototype.myCall = function(thisArg, ...args) {
  // 这里的 this 其实就是 func.myCall(thisArg, ...args) 中的 func，因为 myCall 是通过 func 调用的嘛
  const func = this;

  if (thisArg === undefined || thisArg === null) {
    // 如果 thisArg 是 undefined 或则 null，this 指向全局对象，直接调用就可以达到指向全局对象的目的了
    return func(...args);
  }
​
  const tempFunc = Symbol('Temp property');
  // 在 thisArg 上临时绑定 func
  thisArg[tempFunc] = func;
​
  // 通过 thisArg 调用 func 来达到改变 this 指向的作用
  const result = thisArg[tempFunc](...args);
​
  // 删除临时属性
  Reflect.deleteProperty(thisArg, tempFunc);
  return result;
}
​
function printName() {
  console.log(this.name);
}
​
console.log(printName.myCall({ name: 'ly' })); // => ly
```
## 实现一个apply
过程和`call`类似，只是参数不同。
```js
Function.prototype.myApply = function(thisArg, args) {
  if (thisArg === undefined || thisArg === null) {
    // 如果 thisArg 是 undefined 或则 null，this 指向全局对象，直接调用就可以达到指向全局对象的目的了
    return tempFunc(args);
  }
​
  // 这里的 this 其实就是 func.myCall(thisArg, ...args) 中的 func，因为 myCall 是通过 func 调用的嘛
  const func = this;
​
  const tempFunc = Symbol('Temp property');
  // 在 thisArg 上临时绑定 func
  thisArg[tempFunc] = func;
​
  // 通过 thisArg 调用 func 来达到改变 this 指向的作用
  const result = thisArg[tempFunc](args);
​
  // 删除临时属性
  delete thisArg[tempFunc];
  return result;
}
​
function printName() {
  console.log(this.name);
}
​
console.log(printName.myCall({ name: 'ly' })); // => ly
```
## 实现一个bind
`bind()`方法创建一个新的函数，在`bind()`被调用时，这个新函数的`this`被指定为`bind()`的第一个参数，而其余参数将作为新函数的参数，供调用时使用。
```js
Function.prototype.bind = function(thisArg /* , args */) {
  // 将this和arguments的值保存至变量中以便在后面嵌套的函数中可以使用它们
  var self = this, boundArgs = arguments;

  // bind()方法的返回值是一个函数
  return function() {
    // 创建一个实参列表，将传入bind()的第二个及后续的实参都传入这个函数
    var i, args = [];
    for(i = 1; i < boundArgs.length; i++){ args.push(boundArgs[i]) }
    for(i = 0; i < arguments.length; i++){ args.push(arguments[i]) }
    // 现在将self作为o的方法来调用，传入这些实参
    return self.apply(thisArg, args)
  }
}
const obj = { a: 1, b: 2 }
const fn = function(b, c) {
  return this.a + b + c;
}
const fn1 = fn.bind(obj, 2);
fn1(3); // 6
```
# apply call bind 的一些运用
## 类数组转为数组
```js
// 方法一：
const obj = {0: 'q', 1: 'i', 2: 'q', 3: 'a',  4:'n', 5: 'y', 6:'i', 7:'n', length: 8};
const arr = [];
Array.prototype.push.apply(arr, obj);
console.log(arr); // ["q", "i", "q", "a", "n", "y", "i", "n"]
// 方法二：
const obj = {0: 'q', 1: 'i', length: 2};
const arr = Array.prototype.slice.call(obj);  // [q, i]
```
## 为伪数组添加新的元素
```js
// 方法一: 当然你也可以使用 apply
const obj = {0: 'q', length: 1};
Array.prototype.push.call(obj, 'i', 'a', 'n');
console.log(obj);   // {0: 'q', 1: 'i', 2: 'a', 3: 'n'}
// 方法二：
const obj = {0: 'q', length: 1};
const push = Array.prototype.push.bind(obj);
push('i', 'a', 'n');
console.log(obj);   // {0: 'q', 1: 'i', 2: 'a', 3: 'n'}
```
## 求数组中最大值(最小值一样做法)
```js
const arr = [1,2,3,4,5,6];
const max = Math.max.apply(null, arr);
// 或 const max = Math.max.call(null, ...arr)
console.log(max); // 6
```
## 数组合并追加
```js
const arr = [1, 2];
const brr = [3, 4];
Array.prototype.push.apply(arr, brr);
// 或者 Array.prototype.push.call(arr, ...brr);
// 当然还可以这样 arr.push(...brr);
console.log(arr); // [1, 2, 3, 4]
```