---
title: JS代码片段(Type)
date: 2019-10-06 21:31:05
tags: [javascript, JS code, JS进阶]
categories: javascript
---


# Type
## getType
返回值的原生类型。
返回值小写的构造函数名称，如果值为`undefined`或`null`，则返回`undefined`或`null`。
```js
const getType = v =>
  v === undefined ? 'undefined' : v === null ? 'null' : v.constructor.name.toLowerCase()

getType(new Set([1, 2, 3])) // 'set'
```
## is
检查提供的值是否属于指定类型。
使用`array.prototype.includes()`确保该值不是`undefined`或`null`，并将该值的构造函数属性与类型进行比较，以检查提供的值是否属于指定的类型。
```js
const is = (type, val) => ![, null].includes(val) && val.constructor === type;

is(Array, [1]); // true
is(ArrayBuffer, new ArrayBuffer()); // true
is(Map, new Map()); // true
is(RegExp, /./g); // true
is(Set, new Set()); // true
is(WeakMap, new WeakMap()); // true
is(WeakSet, new WeakSet()); // true
is(String, ''); // true
is(String, new String('')); // true
is(Number, 1); // true
is(Number, new Number(1)); // true
is(Boolean, true); // true
is(Boolean, new Boolean(true)); // true
```
## isArray
检查给定的参数是否是一个数组。
使用`Array.isArray()`来检查一个值是否为一个数组。
```js
const isArray = val => Array.isArray(val)

isArray([1]) // true
```
## isArrayLike
检查提供的参数是否是类数组（即可迭代）。
检查提供的参数是否不为空，以及其`symbol.iterator`属性是否为函数。
```js
const isArrayLike = obj => obj != null && typeof obj[Symbol.iterator] === 'function';

isArrayLike(document.querySelectorAll('.className')) // true
isArrayLike('abc') // true
isArrayLike(null) // false
```
## isBoolean
检查给定的参数是否是一个原生的布尔值。
使用`typeof`来检查一个值是否为一个布尔值。
```js
const isBoolean = val => typeof val === 'boolean';

isBoolean(null); // false
isBoolean(false); // true
```
## isEmpty
如果值为空对象、集合、没有可枚举属性或是任何不被视为集合的类型，则返回`true`。
检查提供的值是否为空或其长度是否等于0。
```js
const isEmpty = val => val == null || !(Object.keys(val) || val).length;

isEmpty([]); // true
isEmpty({}); // true
isEmpty(''); // true
isEmpty([1, 2]); // false
isEmpty({ a: 1, b: 2 }); // false
isEmpty('text'); // false
isEmpty(123); // true - type is not considered a collection
isEmpty(true); // true - type is not considered a collection
```
## isFunction
检查给定的参数是否是一个函数。
使用`typeof`来检查一个值是否为一个函数。
```js
const isFunction = val => typeof val === 'function';

isFunction('x'); // false
isFunction(x => x); // true
```
## isNil
如果指定的值为空或未定义，则返回`true`，否则返回`false`。
```js
const isNil = val => val === undefined || val === null;

isNil(null); // true
isNil(undefined); // true
```
## isNull
如果指定的值为`null`，则返回`true`；否则返回`false`。
使用严格等号运算符来检查`val`值是否等于`null`。
```js
const isNull = val => val === null;

isNull(null); // true
```
## isNumber
检查给定的参数是否是一个数字。
使用`typeof`来检查一个值是否为一个数字。
```js
const isNumber = val => typeof val === 'number' && val === val;

isNumber(1); // true
isNumber('1'); // false
isNumber(NaN); // false
```
## isObject
如果传递的值是一个对象，则返回一个布尔值。
使用`Object`构造函数为给定值创建对象包装。 如果该值为`null`或`undefined`，则创建并返回一个空对象。否则，返回一个对应于给定值的类型的对象。
```js
const isObject = obj => obj === Object(obj);

isObject([1, 2, 3, 4]); // true
isObject([]); // true
isObject(['Hello!']); // true
isObject({ a: 1 }); // true
isObject({}); // true
isObject(true); // false
```
其他方法：
```js
const isObject = obj => obj !== null && (typeof obj === 'object' || typeof obj === 'function');
```
## isObjectLike
检查值是否类对象。
```js
const isObjectLike = val => val !== null && typeof val === 'object';

isObjectLike({}); // true
isObjectLike([1, 2, 3]); // true
isObjectLike(x => x); // false
isObjectLike(null); // false
```
## isPlainObject
检查提供的值是否是由对象构造函数创建的对象。
```js
const isPlainObject = val => !!val && typeof val === 'object' && val.constructor === Object;

isPlainObject({ a: 1 }); // true
isPlainObject(new Map()); // false
```
## isPrimitive
返回一个布尔值，确定传递的值是否为原始值。
```js
const isPrimitive = val => Object(val) !== val;

isPrimitive(null); // true
isPrimitive(50); // true
isPrimitive('Hello!'); // true
isPrimitive(false); // true
isPrimitive(Symbol()); // true
isPrimitive([]); // false
```
其他方法：
```js
const isPrimitive = val => {
  return typeof val === 'string' ||
    typeof val === 'number' ||
    typeof val === 'boolean' ||
    typeof val === 'undefined' ||
    typeof val === 'symbol' ||
    val === null
}
```
## isPromiseLike
如果一个对象看起来像一个`Promise`，则返回`true`，否则返回`false` 。
检查对象是不为`null`，它的`typeof`是否匹配`object`或`function` ，如果它有`.then`属性，这也是一个`function` 。
```js
const isPromiseLike = obj =>
  obj !== null &&
  (typeof obj === 'object' || typeof obj === 'function') &&
  typeof obj.then === 'function';

isPromiseLike({
  then: function() {
    return '';
  }
}); // true
isPromiseLike(null); // false
isPromiseLike({}); // false
```
## isString
检查给定的参数是否是一个字符串。
使用`typeof`来检查一个值是否为一个字符串。
```js
const isString = val => typeof val === 'string';

isString('10'); // true
```
## isSymbol
检查给定的参数是否是一个`symbol`。
使用`typeof`来检查一个值是否为一个`symbol`。
```js
const isSymbol = val => typeof val === 'symbol';

isSymbol(Symbol('x')); // true
```
## isUndefined
检查给定的参数是否是一个`undefined`。
使用严格等号运算符来检查`val`值是否等于`undefined`。
```js
const isUndefined = val => val === undefined;

isUndefined(undefined); // true
```
## isValidJSON
检查提供的参数是否是有效的JSON。
使用`JSON.parse()`和`try... catch`块来检查提供的参数是否是有效的JSON。
```js
const isValidJSON = str => {
  try {
    JSON.parse(str);
    return true;
  } catch (e) {
    return false;
  }
};

isValidJSON('{"name":"Adam","age":20}'); // true
isValidJSON('{"name":"Adam",age:"20"}'); // false
isValidJSON(null); // true
```
## castArray
如果提供的值不是数组，则将其强制转换为数组。
使用`array.prototype.isarray()`确定`val`是一个数组，并将其按原样返回，或相应地封装在数组中。
```js
const castArray = val => (Array.isArray(val) ? val : [val]);

castArray('foo'); // ['foo']
castArray([1]); // [1]
```
## cloneRegExp
克隆一个正则表达式。
使用`new RegExp()`，`RegExp.source`和`RegExp.flags`来克隆给定的正则表达式。
```js
const cloneRegExp = regExp => new RegExp(regExp.source, regExp.flags);

const regExp = /lorem ipsum/gi;
const regExp2 = cloneRegExp(regExp); // /lorem ipsum/gi
```
## coalesce
返回第一个非`null/undefined`的参数。
```js
const coalesce = (...args) => args.find(_ => ![undefined, null].includes(_));

coalesce(null, undefined, '', NaN, 'Waldo'); // ""
```
## coalesceFactory
返回自定义的`coalesce`函数，该函数返回从提供的参数验证函数返回`true`的第一个参数。
使用`Array.find()`返回从提供的参数验证函数返回`true`的第一个参数。
```js
const coalesceFactory = valid => (...args) => args.find(valid);

const customCoalesce = coalesceFactory(_ => ![null, undefined, '', NaN].includes(_));
customCoalesce(undefined, null, NaN, '', 'Waldo'); // "Waldo"
```
## extendHex
将3位数的颜色代码扩展为6位数的颜色代码。
使用`Array.map(), String.split()`和`Array.join()`加入映射数组，将3位RGB十六进制颜色代码转换为6位数形式。`Array.slice()`用于从字符串开始删除#，因为输出中已经默认添加了。
```js
const extendHex = shortHex =>
  '#' +
  shortHex
    .slice(shortHex.startsWith('#') ? 1 : 0)
    .split('')
    .map(x => x + x)
    .join('');

extendHex('#03f'); // '#0033ff'
extendHex('05a'); // '#0055aa'
```
## httpGet
向传递的URL发出一个`GET`请求。
使用`XMLHttpRequest` web API向给定的`url`发出`get`请求。 通过调用给定的`callback`和`responseText`来处理`onload`事件。 通过运行提供的`err`函数，处理`onerror`事件。省略第四个参数`err`，默认将错误记录到控制台的`error`流。
```js
const httpGet = (url, callback, err = console.error) => {
  const request = new XMLHttpRequest();
  request.open('GET', url, true);
  request.onload = () => callback(request.responseText);
  request.onerror = () => err(request);
  request.send();
};

httpGet(
  'https://jsonplaceholder.typicode.com/posts/1',
  console.log
);
```
## httpPost
对传递的URL发出一个`POST`请求。
使用`XMLHttpRequest` web api对给定的`url`发出一个`post`请求。 用`setRequestHeader`方法设置`HTTP`请求头的值。通过调用给定的`callback`和`responseText`来处理`onload`事件。 通过运行提供的`err`函数，处理`onerror`事件。 省略第三个参数`data`，不发送数据到提供的`url` 。 省略第四个参数`err`，默认将错误记录到控制台的`error`流。
```js
const httpPost = (url, data, callback, err = console.error) => {
  const request = new XMLHttpRequest();
  request.open('POST', url, true);
  request.setRequestHeader('Content-type', 'application/json; charset=utf-8');
  request.onload = () => callback(request.responseText);
  request.onerror = () => err(request);
  request.send(data);
};

const newPost = {
  userId: 1,
  id: 1337,
  title: 'Foo',
  body: 'bar bar bar'
};
const data = JSON.stringify(newPost);
httpPost(
  'https://jsonplaceholder.typicode.com/posts',
  data,
  console.log
);
httpPost(
  'https://jsonplaceholder.typicode.com/posts',
  null, // does not send a body
  console.log
);
```
## isBrowser
确定当前运行时环境是否为浏览器。
```js
const isBrowser = () => ![typeof window, typeof document].includes('undefined');

isBrowser(); // true (browser)
isBrowser(); // false (Node)
```

## prettyBytes
将字节数转换为可读的字符串。
根据存取字节数常用的单位构建一个数组字典。使用`Number.toPrecision()`将数字截断为一定数量的数字。将构建、美化好的字符串返回，并考虑提供的选项以及是否为负值。省略第二个参数`precision`，使用默认精度为3的数字。省略第三个参数`addSpace`，默认情况下在数字和单位之间添加空格。
```js
const prettyBytes = (num, precision = 3, addSpace = true) => {
  const UNITS = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'];
  if (Math.abs(num) < 1) return num + (addSpace ? ' ' : '') + UNITS[0];
  const exponent = Math.min(Math.floor(Math.log10(num < 0 ? -num : num) / 3), UNITS.length - 1);
  const n = Number(((num < 0 ? -num : num) / 1000 ** exponent).toPrecision(precision));
  return (num < 0 ? '-' : '') + n + (addSpace ? ' ' : '') + UNITS[exponent];
};

prettyBytes(1000); // "1 KB"
prettyBytes(-27145424323.5821, 5); // "-27.145 GB"
prettyBytes(123456789, 3, false); // "123MB"
```
## randomHexColorCode
生成一个随机的十六进制颜色代码。
使用`Math.random`生成一个随机的24位（6x4位）十六进制数。使用位操作符，然后使用`toString(16)`将其转换为十六进制字符串。
```js
const randomHexColorCode = () => {
  let n = (Math.random() * 0xfffff * 1000000).toString(16);
  return '#' + n.slice(0, 6);
};

randomHexColorCode(); // "#e34155"
```
## serializeCookie
将一个`cookie name-value`对序列化为`Set-Cookie`头字符串。
使用模板字面量和`encodeURIComponent()`来创建合适的字符串。
```
const serializeCookie = (name, val) => `${encodeURIComponent(name)}=${encodeURIComponent(val)}`;

serializeCookie('foo', 'bar'); // 'foo=bar'
```
## timeTaken
计算一个函数执行的时间。
使用`console.time()`和`console.timeEnd()`来测量开始和结束时间之间的差，以确定回调执行的时间。
```js
const timeTaken = callback => {
  console.time('timeTaken');
  const r = callback();
  console.timeEnd('timeTaken');
  return r;
};

timeTaken(() => Math.pow(2, 10)); 
// 1024, (logged): timeTaken: 0.02099609375ms
```
