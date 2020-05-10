---
title: JS代码片段(Math、Date)
date: 2019-10-02 15:11:41
tags: [javascript, 前端, JS code]
categories: javascript
---


# Math
## average
返回两个或两个以上数字的的平均值。
使用`Array.reduce()`将每个值累加到初始值0的累加器， 除以数组长度`length`。
```js
const average = (...nums) => nums.reduce((acc, val) => acc + val, 0) / nums.length;

average(...[1, 2, 3]); // 2
average(1, 2, 3); // 2
```
## averageBy
使用提供的函数将每个元素映射到一个值后，返回一个数组的平均值。
使用`Array.map()`将每个元素映射到由`fn`返回的值，`Array.reduce()`将每个值累加到累加器，用0作为累加器的初始值，再除以数组的`length`。
```js
const averageBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : val => val[fn]).reduce((acc, val) => acc + val, 0) /
  arr.length;

averageBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], o => o.n); // 5
averageBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], 'n'); // 5
```
## clampNumber
返回由边界值`a`和`b`限定范围内的`num`。
如果`num`在限定范围内，则返回`num`。 否则，返回范围内最接近的数字。
```js
const clampNumber = (num, a, b) => Math.max(Math.min(num, Math.max(a, b)), Math.min(a, b));

clampNumber(2, 3, 5); // 3
clampNumber(1, -1, -5); // -1
```
## digitize
将数字转换为数字数组。
将数字转换为字符串，使用展开运算符 (`...`) 构建一个数组。 使用`Array.map()`和`parseInt()`将每个值转换为整数。
```js
const digitize = n => [...`${n}`].map(i => parseInt(i));

digitize(123); // [1, 2, 3]
```
## distance
返回两点之间的欧氏距离。
使用`Math.hypot()`计算两点之间的欧氏距离( `Euclidean distance`)。
```js
const distance = (x0, y0, x1, y1) => Math.hypot(x1 - x0, y1 - y0);

distance(1, 1, 2, 3); // 2.23606797749979
```
## factorial
计算一个数字的阶乘。
使用递归。如果`n`小于或等于1 ，则返回1 。否则返回`n`和`n - 1`的阶乘。如果`n`是负数，则会引发异常。
```js
const factorial = n =>
  n < 0
    ? (() => {
      throw new TypeError('Negative numbers are not allowed!');
    })()
    : n <= 1
      ? 1
      : n * factorial(n - 1);

factorial(6); // 720
```
## fibonacci
生成一个包含 斐波纳契（`fibonacci`）数组，直到该数组有第`n`元素。
创建一个指定长度的空数组，初始化前两个值( 0和1 )。使用`Array.reduce()`向数组中添加值，该值是最后两个值的和，前两个值除外。
```js
const fibonacci = n =>
  Array.from({ length: n }).reduce(
    (acc, val, i) => acc.concat(i > 1 ? acc[i - 1] + acc[i - 2] : i),
    []
  );

fibonacci(6); // [0, 1, 1, 2, 3, 5]
```
## gcd
计算两个或两个以上数字/数字数组的最大公约数。
内部的`_gcd`函数使用递归。基本情况是，当`y`等于0的情况下，返回`x`。否则，返回`y`的最大公约数和`x / y`的其余数。
```js
const gcd = (...arr) => {
  const _gcd = (x, y) => (!y ? x : gcd(y, x % y));
  return [...arr].reduce((a, b) => _gcd(a, b));
};

gcd(8, 36); // 4
gcd(...[12, 8, 32]); // 4
```
## geometricProgression
初始化一个包含指定范围中数字的数组，包含`start`和`end`，两个元素之间的比例是`step`（后一个数是前一个数的`step`倍）。 如果`step`等于1则返回一个错误。
使用`Array.from()`，`Math.log()`和`Math.floor()`来创建一个所需长度的数组，使用`Array.map()`来填充所需的值。省略第二个参数`start`，使用默认值1。省略第三个参数`step`，使用默认值2。
```js
const geometricProgression = (end, start = 1, step = 2) =>
  Array.from({ length: Math.floor(Math.log(end / start) / Math.log(step)) + 1 }).map(
    (v, i) => start * step ** i
  );

geometricProgression(256); // [1, 2, 4, 8, 16, 32, 64, 128, 256]
geometricProgression(256, 3); // [3, 6, 12, 24, 48, 96, 192]
geometricProgression(256, 1, 4); // [1, 4, 16, 64, 256]
```
## inRange
检查给定的数字是否在给定范围内。
使用算术比较来检查给定的数字是否在指定的范围内。如果没有指定第三个参数`end`，则范围被认为是从0到`start`。
```js
const inRange = (n, start, end = null) => {
  if (end && start > end) [end, start] = [start, end];
  return end == null ? n >= 0 && n < start : n >= start && n < end;
};

inRange(3, 2, 5); // true
inRange(3, 4); // true
inRange(2, 3, 5); // false
inRange(3, 2); // false
```
## isDivisible
检查第一个数字参数是否可被第二个数字整除。
使用模运算符(`%`)来检查余数是否等于0 。
```js
const isDivisible = (dividend, divisor) => dividend % divisor === 0;

isDivisible(6, 3); // true
```
## isEven
如果给定的数字是偶数，则返回`true`，否则返回`false`。
使用模运算符(`%`)来检查数字是奇数还是偶数。如果数字是偶数，则返回`true`，如果是奇数，则返回`false`。
```js
const isEven = num => num % 2 === 0;

isEven(3); // false
```
## isPrime
检查提供的整数是否为素数。
检查数字从2到给定数字的平方根。如果它们中的任何一个可以整除给定的数字，则返回`false`，否则返回`true`，除非数字小于2。
```js
const isPrime = num => {
  const boundary = Math.floor(Math.sqrt(num));
  for (var i = 2; i <= boundary; i++) if (num % i === 0) return false;
  return num >= 2;
};

isPrime(11); // true
```
## lcm
返回两个或两个以上数字的最小公倍数。
使用最大公约数（GCD）公式和`lcm(x,y) = x * y / gcd(x,y)`来确定最小公倍数。 GCD公式使用递归。
```js
const lcm = (...arr) => {
  const gcd = (x, y) => (!y ? x : gcd(y, x % y));
  const _lcm = (x, y) => (x * y) / gcd(x, y);
  return [...arr].reduce((a, b) => _lcm(a, b));
};

lcm(12, 7); // 84
lcm(...[1, 3, 4, 5]); // 60
```
## maxBy
使用提供的函数将每个元素映射到一个值后，然后返回数组的最大值。
使用`Array.map()`将每个元素映射到由`fn`，然后用`Math.max()`返回的值来获取最大值。
```js
const maxBy = (arr, fn) => Math.max(...arr.map(typeof fn === 'function' ? fn : val => val[fn]));

maxBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], o => o.n); // 8
maxBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], 'n'); // 8
```
## median
返回数字数组的中值。
找到数字数组的中间值，使用 `Array.sort()`对值进行排序。 如果`length`是奇数，则返回中间值数字，否则返回两个中间值数值的平均值。
```js
const median = arr => {
  const mid = Math.floor(arr.length / 2),
    nums = [...arr].sort((a, b) => a - b);
  return arr.length % 2 !== 0 ? nums[mid] : (nums[mid - 1] + nums[mid]) / 2;
};

median([5, 6, 50, 1, -5]); // 5
```
## minBy
使用提供的函数将每个元素映射到一个值后，然后返回数组的最小值。
使用`Array.map()`将每个元素映射到由`fn`，然后用`Math.min()`返回的值来获取最小值。
```js
const minBy = (arr, fn) => Math.min(...arr.map(typeof fn === 'function' ? fn : val => val[fn]));

minBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], o => o.n); // 2
minBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], 'n'); // 2
```
## percentile
使用百分比表示给定数组中有多少个数字小于或等于给定值。
使用`Array.reduce()`来计算有多少个数字小于等于该值，并用百分比表示。
```js
const percentile = (arr, val) =>
  (100 * arr.reduce((acc, v) => acc + (v < val ? 1 : 0) + (v === val ? 0.5 : 0), 0)) / arr.length;

percentile([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 6); // 55
```
## powerset
返回给定数组的`powerset`(幂集)。
使用`Array.reduce()`与`Array.map()`结合来遍历元素，并将其组合成一个包含所有排列组合的数组。
```js
const powerset = arr => arr.reduce((a, v) => a.concat(a.map(r => [v].concat(r))), [[]]);

powerset([1, 2]); // [[], [1], [2], [2, 1]]
```
## randomIntArrayInRange
返回指定范围内`n`个随机整数的数组。
使用`array.from()`创建一个特定长度的空数组`math.random()`，生成一个随机数并将其映射到所需的范围，使用`math.floor()`将其设置为整数。
```js
const randomIntArrayInRange = (min, max, n = 1) =>
  Array.from({ length: n }, () => Math.floor(Math.random() * (max - min + 1)) + min);

randomIntArrayInRange(12, 35, 10); // [ 34, 14, 27, 17, 30, 27, 20, 26, 21, 14 ]
```
## randomIntegerInRange
返回指定范围内的随机整数。
使用`Math.random()`生成一个随机数并将其映射到所需的范围，使用`Math.floor()`使其成为一个整数。
```js
const randomIntegerInRange = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;

randomIntegerInRange(0, 5); // 2
```
## randomNumberInRange
返回指定范围内的一个随机数。
使用`Math.random()`生成一个随机值，使用乘法将其映射到所需的范围。
```js
const randomNumberInRange = (min, max) => Math.random() * (max - min) + min;

randomNumberInRange(2, 10); // 6.0211363285087005
```
## round
将数字四舍五入到指定的小数位数。
使用`Math.round()`和模板字面量将数字四舍五入为指定的小数位数。 省略第二个参数`decimals`，数字将被四舍五入到一个整数。
```js
const round = (n, decimals = 0) => Number(`${Math.round(`${n}e${decimals}`)}e-${decimals}`);

round(1.005, 2); // 1.01
```
## sum
返回两个或两个以上数字/数字数组中元素之和。
使用`Array.reduce()`将每个值添加到累加器，并且累加器初始值为0。
```js
const sum = (...arr) => [...arr].reduce((acc, val) => acc + val, 0);

sum(1, 2, 3, 4); // 10
sum(...[1, 2, 3, 4]); // 10
```
## sumBy
使用提供的函数将每个元素映射到一个值之后，然后返回数组的和。
使用`Array.map()`将每个元素映射到由`fn`返回的值，`Array.reduce()`将每个值添加到一个累加器，并且累加器初始值为0。
```js
const sumBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : val => val[fn]).reduce((acc, val) => acc + val, 0);

sumBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], o => o.n); // 20
sumBy([{ n: 4 }, { n: 2 }, { n: 8 }, { n: 6 }], 'n'); // 20
```
## toSafeInteger
将值转换为安全整数。
使用`Math.max()`和`Math.min()`来找到最接近的安全值。 使用`Math.round()`转换为一个整数。
```js
const toSafeInteger = num =>
  Math.round(Math.max(Math.min(num, Number.MAX_SAFE_INTEGER), Number.MIN_SAFE_INTEGER));

toSafeInteger('3.2'); // 3
toSafeInteger(Infinity); // 9007199254740991
```
# Date
## formatDuration
返回给定毫秒数的可读格式。
用适当的值来划分`ms`，以获得`day，hour，minute，second`和`millisecond`的适当值。通过`Array.filter()`使用`Object.entries()`只保留非零值。使用`Array.map()`为每个值创建字符串，并且适当复数化。使用`String.join(', ')`将这些值组合成一个字符串。
```js
const formatDuration = ms => {
  if (ms < 0) ms = -ms;
  const time = {
    day: Math.floor(ms / 86400000),
    hour: Math.floor(ms / 3600000) % 24,
    minute: Math.floor(ms / 60000) % 60,
    second: Math.floor(ms / 1000) % 60,
    millisecond: Math.floor(ms) % 1000
  };
  return Object.entries(time)
    .filter(val => val[1] !== 0)
    .map(([key, val]) => `${val} ${key}${val !== 1 ? 's' : ''}`)
    .join(', ');
};

formatDuration(1001); // '1 second, 1 millisecond'
formatDuration(34325055574); 
// '397 days, 6 hours, 44 minutes, 15 seconds, 574 milliseconds'
```
## getColonTimeFromDate
从日期对象返回格式为`hh:mm:ss`的字符串。
使用`date.prototype.toTimeString()`和`string.prototype.slice()`获取给定日期对象的`hh:mm:ss`部分。
```js
const getColonTimeFromDate = date => date.toTimeString().slice(0, 8);

getColonTimeFromDate(new Date()); // "08:38:00"
```
## getDaysDiffBetweenDates
返回两个日期之间相差的天数。
计算`Date`对象之间的差异(以天为单位)。
```js
const getDaysDiffBetweenDates = (dateInitial, dateFinal) =>
  (dateFinal - dateInitial) / (1000 * 3600 * 24);

getDaysDiffBetweenDates(new Date('2017-12-13'), new Date('2017-12-22')); // 9
```
## isSameDate
判断两个日期是否相同
```js
const isSameDate = (dateA, dateB) => dateA.toISOString() === dateB.toISOString();

isSameDate(new Date(2010, 10, 20), new Date(2010, 10, 20)); // true
```
## isBeforeDate
检查一个日期是否在另一个日期之前。
```js
const isBeforeDate = (dateA, dateB) => dateA < dateB;

isBeforeDate(new Date(2010, 10, 20), new Date(2010, 10, 21)); // true
```
## isAfterDate
检查一个日期是否在另一个日期之后。
```js
const isAfterDate = (dateA, dateB) => dateA > dateB;

isAfterDate(new Date(2010, 10, 21), new Date(2010, 10, 20)); // true
```
## isWeekend
判断特定的日期是否是周六日
```js
const isWeekend = (t = new Date()) => {
  return t.getDay() % 6 === 0;
};

isWeekend()
```
## isWeekday
判断是否是工作日
```js
const isWeekday = (t = new Date()) => {
  return t.getDay() % 6 !== 0;
};

isWeekday()
```
## tomorrow
以字符串形式返回明天日期表示。
使用`new Date()`获取今天的日期，加上86400000秒（24小时），使用`Date.toISOString()`将`Date`对象转换为字符串。
```js
const tomorrow = () => {
  let t = new Date();
  t.setDate(t.getDate() + 1);
  return t.toISOString().split('T')[0];
};

tomorrow(); // 2020-10-18
```
## yesterday
返回昨天的日期。
```js
const yesterday = () => {
  let t = new Date();
  t.setDate(t.getDate() - 1);
  return t.toISOString().split('T')[0];
};

yesterday() // // 2020-10-17
```
## maxDate
返回给定日期中的最大日期。
```js
const maxDate = dates => new Date(Math.max(...dates));

const array = [
  new Date(2017, 4, 13),
  new Date(2018, 2, 12),
  new Date(2016, 0, 10),
  new Date(2016, 0, 9)
];
maxDate(array); // 2018-03-11T22:00:00.000Z
```
## minDate
返回给定日期中的最小日期
```js
const minDate = dates => new Date(Math.min(...dates));

const array = [
  new Date(2017, 4, 13),
  new Date(2018, 2, 12),
  new Date(2016, 0, 10),
  new Date(2016, 0, 9)
];
minDate(array); // 2016-01-08T22:00:00.000Z
```