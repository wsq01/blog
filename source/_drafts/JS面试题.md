---
title: JS面试题.md
tags: javascript
---

## typeof相关
```js
typeof console.log // "function"
typeof NaN // "number"
typeof new Function(); // "function"
typeof new Date(); // "object"
typeof new Boolean(true); // "object"
typeof new Number(1); // "object"
typeof new String("abc"); // "object"
typeof /\*.js/; // "object"
typeof (typeof 1) === 'string'; // typeof返回的肯定是一个字符串
typeof Symbol.iterator; // 'symbol'
typeof undefined === 'undefined'; // true
typeof Math.sin === 'function'; // true
typeof isNaN; // 'function'
typeof isNaN(); // 'boolean'
```

```js
['1', '2', '3'].map(parseInt) // 1, NaN, NaN
```

总结：
`typeof`运算符返回7种类型:`undefined,string,number,boolean,symbol,object,function`。
前5个是值类型，后4个是引用类型，`typeof`只能区分值类型的详细类型，对引用类型无法区分,只能区分`function`类型。
## 数值相关
```js
parseInt("0a"); // 0
// parseInt("0a")==>parseInt("0")==>parseInt("0",10)==>0
parseInt("010a"); // 8
// parseInt("010a")==>parseInt("010")==>parseInt("10",8)==>8
parseInt("0xt"); // NaN
// parseInt("0xt")==>parseInt("",16)==>NaN

Number.MIN_VALUE > 0 // true
```

## 作用域相关
#### 下面代码输出什么
```js
var a = 10;
(function () {
    console.log(a)
    a = 5
    console.log(window.a)
    var a = 20;
    console.log(a)
})()
```
依次输出：`undefined -> 10 -> 20`

解析：
在立即执行函数中，`var a = 20;`语句定义了一个局部变量`a`，由于js的变量声明提升机制，局部变量`a`的声明会被提升至立即执行函数的函数体最上方，且由于这样的提升并不包括赋值，因此第一条打印语句会打印`undefined`，最后一条语句会打印20。

由于变量声明提升，`a = 5;`这条语句执行时，局部的变量`a`已经声明，因此它产生的效果是对局部的变量`a`赋值，此时`window.a`依旧是最开始赋值的10。
#### 下面代码输出什么
```js
var name = 'Tom';
(function() {
if (typeof name == 'undefined') {
  name = 'Jack';
  console.log('Goodbye ' + name);
} else {
  console.log('Hello ' + name);
}
})();
// Hello Tom
```
解析：
1、首先在进入函数作用域当中，获取`name`属性
2、在当前作用域没有找到`name`
3、通过作用域链找到最外层，得到`name`属性
4、执行`else`的内容，得到`Hello Tom`

## 连续赋值
#### 下面代码输出什么
```js
var a = {n: 1};
var b = a;
a.x = a = {n: 2};

console.log(a.x) 	
console.log(b.x)
// undefined {n: 2}
```
解析：
1、第3行，`.`运算符优先级比`=`高，所以这里首先执行`a.x`，相当于为`a`（或者`b`）所指向的`{n: 1}`对象新增了一个属性`x`，即此时对象将变为`{n: 1, x: undefined}`。
2、`=`从右往左进行赋值，`a = {n: 2}`，此时`a`的引用已经变成了`{n: 2}`这个对象，`a = {n: 2}`这条赋值语句返回`{n: 2}`。
3、执行`a.x = {n：2}`，此时因为`a.x`已经绑定到了`{n: 1 , x: undefined}`这个内存地址，也就是`b.x`，于是`{ n: 1, x: undefined} => {n: 1, x: { n: 2}}`，即`b.x = { n: 2 }`。
