---
title: JS代码片段(Object、Function)
date: 2019-10-05 19:21:45
tags: [javascript, 前端, JS code]
categories: javascript
---


# Object
## deepClone
创建对象的深度克隆。
使用递归。使用`object.assign()`和空对象创建原始对象的浅克隆。使用`object.keys()`和`array.prototype.foreach()`确定需要深度克隆哪些键值对。
```js
const deepClone = obj => {
  let clone = Object.assign({}, obj);
  Object.keys(clone).forEach(
    key => (clone[key] = typeof obj[key] === 'object' ? deepClone(obj[key]) : obj[key])
  );
  return Array.isArray(obj) && obj.length
    ? (clone.length = obj.length) && Array.from(clone)
    : Array.isArray(obj)
      ? Array.from(obj)
      : clone;
};

const a = { foo: 'bar', obj: { a: 1, b: 2 } };
const b = deepClone(a); // a !== b, a.obj !== b.obj
```
## equals
对两个值进行深入比较，以确定它们是否等效。
```js
const equals = (a, b) => {
  if (a === b) return true;
  if (a instanceof Date && b instanceof Date) return a.getTime() === b.getTime();
  if (!a || !b || (typeof a !== 'object' && typeof b !== 'object')) return a === b;
  if (a === null || a === undefined || b === null || b === undefined) return false;
  if (a.prototype !== b.prototype) return false;
  let keys = Object.keys(a);
  if (keys.length !== Object.keys(b).length) return false;
  return keys.every(k => equals(a[k], b[k]));
};

equals({ a: [2, { e: 3 }], b: [4], c: 'foo' }, { a: [2, { e: 3 }], b: [4], c: 'foo' }); // true
```
## flattenObject
扁平化对象。
```js
const flattenObject = (obj, prefix = '') =>
  Object.keys(obj).reduce((acc, k) => {
    const pre = prefix.length ? prefix + '.' : '';
    if (typeof obj[k] === 'object') Object.assign(acc, flattenObject(obj[k], pre + k));
    else acc[pre + k] = obj[k];
    return acc;
  }, {});

flattenObject({ a: { b: { c: 1 } }, d: 1 }); // { 'a.b.c': 1, d: 1 }
```
## objectToQueryString
将对象转换为查询字符串。
```js
const objectToQueryString = queryParameters => {
  return queryParameters
    ? Object.entries(queryParameters).reduce((queryString, [key, val], index) => {
      const symbol = queryString.length === 0 ? '?' : '&';
      queryString += typeof val === 'string' ? `${symbol}${key}=${val}` : '';
      return queryString;
    }, '')
    : '';
};

objectToQueryString({ page: '1', size: '2kg', key: undefined }); // '?page=1&size=2kg'
```
## pick
从对象中选取与给定键对应的键值对。
使用`array.prototype.reduce()`将筛选/选取的键转换回具有相应键值对的对象（如果该键存在于对象中）。
```js
const pick = (obj, arr) =>
  arr.reduce((acc, curr) => (curr in obj && (acc[curr] = obj[curr]), acc), {});

pick({ a: 1, b: '2', c: 3 }, ['a', 'c']); // { 'a': 1, 'c': 3 }
```
## renameKeys
用提供的值替换多个对象键的名称。
```js
const renameKeys = (keysMap, obj) =>
  Object.keys(obj).reduce(
    (acc, key) => ({
      ...acc,
      ...{ [keysMap[key] || key]: obj[key] }
    }),
    {}
  );

const obj = { name: 'Bobo', job: 'Front-End Master', shoeSize: 100 };
renameKeys({ name: 'firstName', job: 'passion' }, obj); // { firstName: 'Bobo', passion: 'Front-End Master', shoeSize: 100 }
```
## shallowClone
创建一个对象的浅拷贝。
使用`Object.assign()`和一个空对象(`{}`)来创建原始对象的浅拷贝。
```js
const shallowClone = obj => Object.assign({}, obj);

const a = { x: true, y: 1 };
const b = shallowClone(a); // a !== b
```
## size
获取数组，对象或字符串的大小。
获取`val`（`array`，`object`或`string`）的类型。 对于数组使用`length`属性。 对于对象，使用`length`或`size`如果可用的话，或者对象的键的数量。对于字符串，使用根据`val`创建的`Blob`对象的`size`。
```js
const size = val =>
  Array.isArray(val)
    ? val.length
    : val && typeof val === 'object'
      ? val.size || val.length || Object.keys(val).length
      : typeof val === 'string'
        ? new Blob([val]).size
        : 0;

size([1, 2, 3, 4, 5]); // 5
size('size'); // 4
size({ one: 1, two: 2, three: 3 }); // 3
```
## unflattenObject
```js
const unflattenObject = obj =>
  Object.keys(obj).reduce((acc, k) => {
    if (k.indexOf('.') !== -1) {
      const keys = k.split('.');
      Object.assign(
        acc,
        JSON.parse(
          '{' +
            keys.map((v, i) => (i !== keys.length - 1 ? `"${v}":{` : `"${v}":`)).join('') +
            obj[k] +
            '}'.repeat(keys.length)
        )
      );
    } else acc[k] = obj[k];
    return acc;
  }, {});

unflattenObject({ 'a.b.c': 1, d: 1 }); // { a: { b: { c: 1 } }, d: 1 }
```
# Function
## curry
柯里化一个函数。
使用递归。 如果提供的参数(`args`)数量足够，调用传递函数`fn`。否则返回一个柯里化后的函数 fn ，期望剩下的参数。如果你想柯里化一个接受可变参数数量的函数(可变参数数量的函数，例如 Math.min() )，你可以选择将参数个数传递给第二个参数 arity。
```js
const curry = (fn, arity = fn.length, ...args) =>
  arity <= args.length ? fn(...args) : curry.bind(null, fn, arity, ...args);

curry(Math.pow)(2)(10); // 1024
curry(Math.min, 3)(10)(50)(2); // 2
```
## once
确保函数只被调用一次。
使用一个闭包，使用一个成为`called`的标志，并在第一次调用该函数时将其设置为`rue`，以防止它被再次调用。 为了允许函数改变它的`this`上下文（比如在一个事件监听器中），必须使用`function`关键字，并且提供的函数必须应用上下文。 允许使用`rest`（剩余）/`spread`（展开） (`...`) 运算符为函数提供任意数量的参数。
```js
const once = fn => {
  let called = false;
  return function(...args) {
    if (called) return;
    called = true;
    return fn.apply(this, args);
  };
};

const startApp = function(event) {
  console.log(this, event); // document.body, MouseEvent
};
document.body.addEventListener('click', once(startApp));
// only runs `startApp` once upon click
```
## sleep
延迟异步函数的执行。
延迟执行`async`函数的一部分，通过把它放到`sleep`状态，返回一个`Promise`。
```js
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

async function sleepyWork() {
  console.log("I'm going to sleep for 1 second.");
  await sleep(1000);
  console.log('I woke up after 1 second.');
}
```
## throttle
节流。
```js
const throttle = (fn, wait) => {
  let inThrottle, lastFn, lastTime;
  return function() {
    const context = this,
      args = arguments;
    if (!inThrottle) {
      fn.apply(context, args);
      lastTime = Date.now();
      inThrottle = true;
    } else {
      clearTimeout(lastFn);
      lastFn = setTimeout(function() {
        if (Date.now() - lastTime >= wait) {
          fn.apply(context, args);
          lastTime = Date.now();
        }
      }, Math.max(wait - (Date.now() - lastTime), 0));
    }
  };
};

window.addEventListener(
  'resize',
  throttle(function(evt) {
    console.log(window.innerWidth);
    console.log(window.innerHeight);
  }, 250)
); // Will log the window dimensions at most every 250ms
```