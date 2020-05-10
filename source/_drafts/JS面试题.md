---
title: JS面试题.md
tags: [javascript, 前端, 面试]
categories: javascript
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
#### 判断下面的值
```js
parseInt("0a"); // 0
// parseInt("0a")==>parseInt("0")==>parseInt("0",10)==>0
parseInt("010a"); // 8
// parseInt("010a")==>parseInt("010")==>parseInt("10",8)==>8
parseInt("0xt"); // NaN
// parseInt("0xt")==>parseInt("",16)==>NaN

Number.MIN_VALUE > 0 // true
```
#### 下面代码输出什么
```js
var two   = 0.2
var one   = 0.1
var eight = 0.8
var six   = 0.6
[two - one == one, eight - six == two] // [true, false]
```
#### 表达式的结果是什么
```js
var a = 111111111111111110000, b = 1111;
a + b;
/*
A: 111111111111111111111
B: 111111111111111110000
C: NaN
D: Infinity
*/
```
答案：B

解析：
精度最多只能到53个二进制位，这意味着，绝对值小于2的53次方的整数，即-2<sup>53</sup>到2<sup>53</sup>，都可以精确表示。
大于2<sup>53</sup>的数值，都无法保持精度。由于2<sup>53</sup>是一个16位的十进制数值，所以简单的法则就是，JavaScript 对15位的十进制数都可以精确处理。大于2<sup>53</sup>以后，多出来的有效数字（最后三位的111）都会无法保存，变成0。

#### 下面代码输出什么
```js
[1 < 2 < 3, 3 < 2 < 1] // [true, true]
```


## 运算符相关
#### 下面代码输出什么
```js
var val = 'smtg';
console.log('Value is ' + (val === 'smtg') ? 'Something' : 'Nothing');
// Something
```
解析：
`+`运算符的优先级比三目运算符的优先级高。
#### 下面代码输出什么
```js
function isOdd(num) {
    return num % 2 == 1;
}
function isEven(num) {
    return num % 2 == 0;
}
function isSane(num) {
    return isEven(num) || isOdd(num);
}
var values = [7, 4, '13', -9, Infinity];
values.map(isSane);
// [true, true, true, false, false]
```
解析：
余数运算符（`%`）返回前一个运算子被后一个运算子除，所得的余数，运算结果的正负号由第一个运算子的正负号决定，因此`-9 % 2 === -1`。
`Infinity % 2`返回`NaN`。
#### 下面代码输出什么
```js
[] == [] // false
```
解析：
两个复合类型（对象、数组、函数）的数据比较时，不是比较它们的值是否相等，而是比较它们是否指向同一个地址。

#### 下面代码输出什么
```js
(function(){
  var x = y = 1;
})();
console.log(y);
console.log(x);
// 1, error
```
解析：
赋值运算符的计算顺序是从右开始计算，即右结合。
`y`是全局变量。
#### 下面代码输出什么
```js
var a = [1, 2, 3],
    b = [1, 2, 3],
    c = [1, 2, 4];
a ==  b
a === b
a >   c
a <   c
// false, false, false, true
```
解析：
* 非相等运算符：如果运算子是对象，会转为原始类型的值，再进行比较。对象转换成原始类型的值，算法是先调用`valueOf`方法；如果返回的还是对象，再接着调用`toString`方法。
* 非相等运算符：两个字符串进行比较时，字符串按照字典顺序进行比较。
* 相等运算符和严格相等运算符：两个复合类型（对象、数组、函数）的数据比较时，不是比较它们的值是否相等，而是比较它们是否指向同一个地址。
* 数组的`valueOf`方法返回数组本身，数组的`toString`方法返回数组的字符串形式。
* 对于两个对象的比较，严格相等运算符比较的是地址，而大于或小于运算符比较的是值。



## 标准库相关
#### 下面代码输出什么
```js
function showCase(value) {
  switch(value) {
  case 'A':
    console.log('Case A');
    break;
  case 'B':
    console.log('Case B');
    break;
  case undefined:
    console.log('undefined');
    break;
  default:
    console.log('Do not know!');
  }
}
showCase(new String('A')); // Do not know
```
解析：
需要注意的是，`switch`语句后面的表达式，与case语句后面的表示式比较运行结果时，采用的是严格相等运算符（`===`）。
`new String('A')`返回的是一个对象。
#### 下面代码输出什么
```js
function showCase2(value) {
  switch(value) {
  case 'A':
    console.log('Case A');
    break;
  case 'B':
    console.log('Case B');
    break;
  case undefined:
    console.log('undefined');
    break;
  default:
    console.log('Do not know!');
  }
}
showCase2(String('A')); // Case A
```
解析：
`String`函数可以将任意类型的值转化成字符串，因此`String('A') === 'A' => true`。

#### 下面代码输出什么
```js
var ary = [0,1,2];
ary[10] = 10;
ary.filter(function(x) { return x === undefined;});
// []
```
解析：
当数组的某个位置是空元素，即两个逗号之间没有任何值，我们称该数组存在空位。
数组的某个位置是空位，与某个位置是`undefined`，是不一样的。如果是空位，使用数组的`forEach`方法、`for...in`结构、`Object.keys`等方法进行遍历，空位都会被跳过。如果某个位置是`undefined`，遍历的时候就不会被跳过。

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
