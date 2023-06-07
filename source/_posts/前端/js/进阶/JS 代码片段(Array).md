---
title: JS代码片段(Array)
date: 2019-09-06 19:08:18
tags: JS
categories: 
 - 前端
 - JS
---

## all
函数封装了`every`函数，判断条件默认为元素转为`boolean`值，如果都为`true`，则返回`true`。否则，返回`false`。
```js
const all = (arr, fn = Boolean) => arr.every(fn);

all([4, 2, 3], x => x > 1); // true
all([1, 2, 3]); // true
```
## allEqual
检查数组中的所有元素是否相等。
```js
const allEqual = arr => arr.every(val => val === arr[0]);

allEqual([1, 2, 3, 4, 5, 6]); // false
allEqual([1, 1, 1, 1]); // true
```

## any
检查数组中的所有元素是否基于`fn`返回`true`。只要数组元素有一个转为`true`，则返回`true`。省略第二个参数`fn`，将布尔值用作默认值。
```js
const any = (arr, fn = Boolean) => arr.some(fn);

any([0, 1, 2, 0], x => x >= 2); // true
any([0, 0, 1, 0]); // true
```
## arrayMax
返回数组中的最大值。
`Math.max()`返回参数中最大的值。如果没有参数，返回`-Infinity`。如果有某个参数为`NaN`，或是不能转换成数字的非数字值，则返回`NaN`。
```js
const arrayMax = arr => Math.max(...arr);
arrayMax([10, 1, 5]) // 10
```
## arrayMin
返回数组中的最小值。
```js
const arrayMin = arr => Math.min(...arr);
arrayMin([10, 1, 5]) // 1
```
## chunk
把一个数组分块成指定大小的小数组。
使用`Array.from()`创建一个新的数组，它的长度与将要生成的`chunk`(块) 数量相匹配。 使用`Array.slice()`将新数组的每个元素映射到长度为`size`的`chunk`中。 如果原始数组不能均匀分割，最后的`chunk`将包含剩余的元素。
```js
const chunk = (arr, size) =>
  Array.from({ length: Math.ceil(arr.length / size) }, (v, i) =>
    arr.slice(i * size, i * size + size)
  );

chunk([1, 2, 3, 4, 5], 2); // [[1,2],[3,4],[5]]
```
## compact
过滤掉数组中所有假值元素。
使用`Array.filter()`过滤掉数组中所有假值元素(`false, null, 0, "", undefined, NaN`)。
```js
const compact = arr => arr.filter(Boolean);

compact([0, 1, false, 2, '', 3, 'a', 'e' * 23, NaN, 's', 34]);
// [ 1, 2, 3, 'a', 's', 34 ]
```
## countOccurrences
计数数组中某个值的出现次数。
使用`Array.reduce()`在每次遇到数组中的特定值时递增计数器。
```js
const countOccurrences = (arr, val) => arr.reduce((a, v) => (v === val ? a + 1 : a), 0);

countOccurrences([1, 1, 2, 1, 2, 3], 1); // 3
```
## deepFlatten
深度平铺数组。
使用递归。 通过空数组(`[]`) 使用`Array.concat()`，结合展开运算符(`...`) 来平铺数组。 递归平铺每个数组元素。
```js
const deepFlatten = arr => [].concat(...arr.map(v => Array.isArray(v) ? deepFlatten(v) : v));

deepFlatten([1, [2], [[3], 4], 5]); // [1,2,3,4,5]
```
其他方法：
```js
const deepFlatten = (arr) => {
  while(arr.some(item => Array.isArray(item))) {
    arr = [].contact(...arr);
  }
  return arr;
}
deepFlatten([1, [2], [[3], 4], 5]); // [1,2,3,4,5]
```
```js
// ES6 flat()
[1, [2, [3]]].flat(Infinity) // [1, 2, 3]
```
## difference
返回两个数组之间的差异。
根据数组`b`创建一个`Set`对象，然后在数组`a`上使用`Array.filter()`方法，过滤出数组`b`中不包含的值。
```js
const difference = (a, b) => {
  const s = new Set(b);
  return a.filter(x => !s.has(x));
};

difference([1, 2, 3], [1, 2, 4]); // [3]
```
## differenceWith
过滤出数组中比较函数不返回`true`的所有值。 类似于`difference`，除了接受一个`comparator`（比较函数）。
使用`Array.filter()`和`Array.findIndex()`来查找合适的值。
```js
const differenceWith = (arr, val, comp) => arr.filter(a => val.findIndex(b => comp(a, b)) === -1);

differenceWith([1, 1.2, 1.5, 3, 0], [1.9, 3, 0], (a, b) => Math.round(a) === Math.round(b)); // [1, 1.2]
```
## drop
从数组开头删除`n`位元素。
```js
const drop = (arr, n = 1) => arr.slice(n);

drop([1, 2, 3]); // [2,3]
drop([1, 2, 3], 2); // [3]
drop([1, 2, 3], 42); // []
```
## dropRight
返回从右开始删除`n`个元素的新数组。
检查`n`是否小于给定数组的长度，并且使用`Array.slice()`来从右开始删除指定数量的元素。
```js
const dropRight = (arr, n = 1) => arr.slice(0, -n);

dropRight([1, 2, 3]); // [1,2]
dropRight([1, 2, 3], 2); // [1]
dropRight([1, 2, 3], 42); // []
```
## dropWhile
从数组左边开始删除元素，判断条件为`func`。一直删除到数组左边直至遇到能让`func`变为`true`的元素。
```js
const dropWhile = (arr, func) => {
  while (arr.length > 0 && !func(arr[0])) arr = arr.slice(1);
  return arr;
};
```
## dropRightWhile
从数组右边开始删除元素，判断条件为`func`。一直删除到数组右边直至遇到能让`func`变为`true`的元素。
```js
const dropRightWhile = (arr, func) => {
  let rightIndex = arr.length;
  while (rightIndex-- && !func(arr[rightIndex]));
  return arr.slice(0, rightIndex + 1);
};

dropRightWhile([1, 2, 3, 4], n => n < 3); // [1, 2]
```
## everyNth
返回数组中的每个第`n`个元素。
使用`Array.filter()`创建一个包含给定数组的每个第`n`个元素的新数组。
```js
const everyNth = (arr, nth) => arr.filter((e, i) => i % nth === nth - 1);

everyNth([1, 2, 3, 4, 5, 6], 2); // [ 2, 4, 6 ]
```
## filterFalsy
去除数组中能转换为`false`的元素。
```js
const filterFalsy = arr => arr.filter(Boolean);

filterFalsy(['', true, {}, false, 'sample', 1, 0]); // [true, {}, 'sample', 1]
```
## filterNonUnique
去掉数组中的非唯一值。
```js
const filterNonUnique = arr => arr.filter(i => arr.indexOf(i) === arr.lastIndexOf(i));

filterNonUnique([1, 2, 2, 3, 4, 4, 5]); // [1, 3, 5]
```
## filterNonUniqueBy
扩展了`filterNonUnique`方法，通过方法`fn`进行判断。`fn`四个参数是比较两个元素的值和索引（方便`fn`方法的扩展）
```js
(i === j) === fn(v, x, i, j)
```
注意`arr.every()`，当数组内所有索引和值比较过，都为`true`，那个元素才是独一无二的。
```js
const filterNonUniqueBy = (arr, fn) =>
  arr.filter((v, i) => arr.every((x, j) => (i === j) === fn(v, x, i, j)));

filterNonUniqueBy(
  [
    { id: 0, value: 'a' },
    { id: 1, value: 'b' },
    { id: 2, value: 'c' },
    { id: 1, value: 'd' },
    { id: 0, value: 'e' }
  ],
  (a, b) => a.id == b.id
); // [ { id: 2, value: 'c' } ]
```
## findLast
寻找数组最后一个能满足`fn`的元素。
```js
const findLast = (arr, fn) => arr.filter(fn).pop();

findLast([1, 2, 3, 4], n => n % 2 === 1); // 3
```
## findLastIndex
寻找数组最后一个能满足`fn`的元素的索引。
```js
const findLastIndex = (arr, fn) =>
  arr
    .map((val, i) => [i, val])
    .filter(([i, val]) => fn(val, i, arr))
    .pop()[0];

findLastIndex([1, 2, 3, 4], n => n % 2 === 1); // 2 (index of the value 3)
```
## flatten
根据指定的`depth`平铺数组。
每次递归，使`depth`减1 。使用`Array.reduce()`和`Array.concat()`来合并元素或数组。默认情况下， `depth`等于1时停递归。省略第二个参数`depth`，只能平铺1层的深度 (单层平铺)。
```js
const flatten = (arr, depth = 1) =>
  arr.reduce((a, v) => a.concat(depth > 1 && Array.isArray(v) ? flatten(v, depth - 1) : v), []);

flatten([1, [2], 3, 4]); // [1, 2, 3, 4]
flatten([1, [2, [3, [4, 5], 6], 7], 8], 2); // [1, 2, 3, [4, 5], 6, 7, 8]
```
ES6方法：
```js
[1, 2, [3, [4, 5]]].flat() // [1, 2, 3, [4, 5]]
[1, 2, [3, [4, 5]]].flat(2) // [1, 2, 3, 4, 5]
[1, [2, [3]]].flat(Infinity) // [1, 2, 3]
```
## forEachRight
从数组的最后一个元素开始，为每个数组元素执行一次提供的函数。
使用`Array.slice(0)`克隆给定的数组，`Array.reverse()`反转数组，`Array.forEach()`遍历这个反向数组。
```js
const forEachRight = (arr, callback) =>
  arr
    .slice(0)
    .reverse()
    .forEach(callback);

forEachRight([1, 2, 3, 4], val => console.log(val)); // '4', '3', '2', '1'
```
## groupBy
根据给定的函数对数组元素进行分组。
使用`Array.map()`将数组的值映射到函数或属性名称。使用`Array.reduce()`来创建一个对象，其中的`key`是从映射结果中产生。
```js
const groupBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : val => val[fn]).reduce((acc, val, i) => {
    acc[val] = (acc[val] || []).concat(arr[i]);
    return acc;
  }, {});

groupBy([6.1, 4.2, 6.3], Math.floor); // {4: [4.2], 6: [6.1, 6.3]}
groupBy(['one', 'two', 'three'], 'length'); // {3: ['one', 'two'], 5: ['three']}
```
## head
获取数组的第一个元素。
使用`arr[0]`返回传递数组的第一个元素。
```js
const head = arr => arr[0];

head([1, 2, 3]); // 1
```
## indexOfAll
返回数组中所有`val`的索引。 如果`val`从不出现，则返回`[]`。
使用`Array.forEach()`循环元素和`Array.push()`来存储匹配元素的索引。返回索引数组。
```js
const indexOfAll = (arr, val) => arr.reduce((acc, el, i) => (el === val ? [...acc, i] : acc), []);

indexOfAll([1, 2, 3, 1, 2, 3], 1); // [0,3]
indexOfAll([1, 2, 3], 4); // []
```
## initial
返回一个数组中除了最后一个元素以外的所有元素。
使用`arr.slice(0,-1)`返回排除了最后一个元素的数组。
```js
const initial = arr => arr.slice(0, -1);

initial([1, 2, 3]); // [1,2]
```
## initialize2DArray
初始化一个给定行数和列数，及值的二维数组。
使用`Array.map()`生成`h`行，其中每个行都是一个长度为`w`的新数组。 如果未提供值`val`，则默认为`null`。
```js
const initialize2DArray = (w, h, val = null) =>
  Array.from({ length: h }).map(() => Array.from({ length: w }).fill(val));

initialize2DArray(2, 2, 0); // [[0,0], [0,0]]
```
## initializeArrayWithRange
初始化一个数组，该数组包含指定范围内的数字，包括`start`和`end`，数字间隔为`step`。
使用`Array.from(Math.ceil((end+1-start)/step))`创建一个所需长度的数组（元素的数量等于 (`end-start)/step`或者`(end+1-start)/step`包括`end`）， 用`Array.map()`填充在这个范围内要求的值。你可以省略`start`来使用默认值0。 您可以省略`step`使用默认值1 。
```js
const initializeArrayWithRange = (end, start = 0, step = 1) =>
  Array.from({ length: Math.ceil((end - start + 1) / step) }, (v, i) => i * step + start);

initializeArrayWithRange(5); // [0,1,2,3,4,5]
initializeArrayWithRange(7, 3); // [3,4,5,6,7]
initializeArrayWithRange(9, 0, 2); // [0,2,4,6,8]
```
## initializeArrayWithValues
使用指定的值初始化和填充数组。
使用`Array(n)`创建所需长度的数组，使用`fill(v)`以填充所需的值。你可以忽略`val`，使用默认值0。
```
const initializeArrayWithValues = (n, val = 0) => Array(n).fill(val);

initializeArrayWithValues(5, 2); // [2, 2, 2, 2, 2]
```
## intersection
返回两个数组中都存在的元素列表。
根据数组 `b`创建一个`Set`对象，然后在数组`a`上使用`Array.filter()`方法，只保留数组`b`中也包含的值。
```js
const intersection = (a, b) => {
  const s = new Set(b);
  return a.filter(x => s.has(x));
};

intersection([1, 2, 3], [4, 3, 2]); // [2, 3]
```
## join
将数组的所有元素拼接成一个字符串并返回此字符串。 使用分隔符和结束分隔符。
使用`Array.reduce()`将元素拼接成一个字符串。 省略第二个参数`separator`，则默认使用分隔符`,`。 省略第三个参数`end`，默认使用与`separator`相同的值。
```js
const join = (arr, separator = ',', end = separator) =>
  arr.reduce(
    (acc, val, i) =>
      i === arr.length - 2
        ? acc + val + end
        : i === arr.length - 1
          ? acc + val
          : acc + val + separator,
    ''
  );

join(['pen', 'pineapple', 'apple', 'pen'], ',', '&'); // "pen,pineapple,apple&pen"
join(['pen', 'pineapple', 'apple', 'pen'], ','); // "pen,pineapple,apple,pen"
join(['pen', 'pineapple', 'apple', 'pen']); // "pen,pineapple,apple,pen"
```
## last
返回数组中的最后一个元素。
使用`arr.length - 1`来计算给定数组的最后一个元素的索引并返回。
```js
const last = arr => arr[arr.length - 1];

last([1, 2, 3]); // 3
```
## longestItem
获取任何数量的可迭代对象或具有`length`属性的对象，并返回最长的一个。
使用`Array.sort()`按`length`对所有参数进行排序，返回第一个（最长）元素。
```js
const longestItem = (...vals) => vals.reduce((a, x) => (x.length > a.length ? x : a));

longestItem('this', 'is', 'a', 'testcase'); // 'testcase'
longestItem(...['a', 'ab', 'abc']); // 'abc'
longestItem(...['a', 'ab', 'abc'], 'abcd'); // 'abcd'
longestItem([1, 2, 3], [1, 2], [1, 2, 3, 4, 5]); // [1, 2, 3, 4, 5]
longestItem([1, 2, 3], 'foobar'); // 'foobar'
```
## mapObject
使用一个函数将数组的值映射到对象，其键值对中，原始值作为键，映射值作为值。
使用一个匿名的内部函数作用域来声明一个`undefined`的内存空间，使用闭包来存储返回值。 使用一个新的`Array`来存储带有函数映射的数组和一个逗号运算符来返回第二个步骤，而不需要从一个上下文移动到另一个上下文（由于闭包和操作顺序）。
```js
const mapObject = (arr, fn) =>
  (a => (
    (a = [arr, arr.map(fn)]), a[0].reduce((acc, val, ind) => ((acc[val] = a[1][ind]), acc), {})
  ))();

const squareIt = arr => mapObject(arr, a => a * a);
squareIt([1, 2, 3]); // { 1: 1, 2: 4, 3: 9 }
```
## maxN
从提供的数组中返回`n`个最大元素。如果`n`大于或等于提供的数组长度，则返回原数组（按降序排列）。
结合使用`Array.sort()`与展开操作符(`...`) ，创建一个数组的浅克隆，并按降序排列。使用`Array.slice()`以获得指定的元素个数。 忽略第二个参数`n`，默认获取单个元素（以数组的形式）。
```js
const maxN = (arr, n = 1) => [...arr].sort((a, b) => b - a).slice(0, n);

maxN([1, 2, 3]); // [3]
maxN([1, 2, 3], 2); // [3,2]
```
## minN
从提供的数组中返回`n`个最小元素。如果`n`大于或等于提供的数组长度，则返回原数组（按降序排列）。
结合使用`Array.sort()`与展开操作符(`...`) ，创建一个数组的浅克隆，并按降序排列。使用`Array.slice()`以获得指定的元素个数。 忽略第二个参数`n`，默认获取单个元素（以数组的形式）。
```js
const minN = (arr, n = 1) => [...arr].sort((a, b) => a - b).slice(0, n);

minN([1, 2, 3]); // [1]
minN([1, 2, 3], 2); // [1,2]
```
## nthElement
返回数组的第`n`个元素。
使用`Array.slice()`获取数组的第`n`个元素。如果索引超出范围，则返回`[]`。省略第二个参数`n`，将得到数组的第一个元素。
```js
const nthElement = (arr, n = 0) => (n === -1 ? arr.slice(n) : arr.slice(n, n + 1))[0];

nthElement(['a', 'b', 'c'], 1); // 'b'
nthElement(['a', 'b', 'b'], -3); // 'a'
```
## partition
根据所提供的函数对每个元素进行迭代，将这些元素分成两个数组。
使用`Array.reduce()`创建两个数组的数组。 使用`Array.push()`将`fn`返回为`true`的元素添加到第一个数组，而`fn`返回`false`的元素到第二个元素。
```js
const partition = (arr, fn) =>
  arr.reduce(
    (acc, val, i, arr) => {
      acc[fn(val, i, arr) ? 0 : 1].push(val);
      return acc;
    },
    [[], []]
  );

const users = [{ user: 'barney', age: 36, active: false }, { user: 'fred', age: 40, active: true }];
partition(users, o => o.active); 
// [[{ 'user': 'fred', 'age': 40, 'active': true }],[{ 'user': 'barney', 'age': 36, 'active': false }]]
```
## pull
改变原始数组，过滤掉指定的值。
使用`Array.filter()`和`Array.includes()`来剔除指定的值。使用`Array.length = 0`将数组中的长度重置为零，并且通过`Array.push()`只使用`pulled`值重新填充数组。
```js
const pull = (arr, ...args) => {
  let argState = Array.isArray(args[0]) ? args[0] : args;
  let pulled = arr.filter((v, i) => !argState.includes(v));
  arr.length = 0;
  pulled.forEach(v => arr.push(v));
};

let myArray = ['a', 'b', 'c', 'a', 'b', 'c'];
pull(myArray, 'a', 'c'); // myArray = [ 'b', 'b' ]
```
## pullAtIndex
改变原始数组，过滤掉指定索引的值。
使用`Array.filter()`和`Array.includes()`来剔除指定的值。使用`Array.length = 0`将数组中的长度重置为零，并且通过`Array.push()`只使用`pulled`值重新填充数组。使用`Array.push()`来跟踪`pulled`值。
```js
const pullAtIndex = (arr, pullArr) => {
  let removed = [];
  let pulled = arr
    .map((v, i) => (pullArr.includes(i) ? removed.push(v) : v))
    .filter((v, i) => !pullArr.includes(i));
  arr.length = 0;
  pulled.forEach(v => arr.push(v));
  return removed;
};

let myArray = ['a', 'b', 'c', 'd'];
let pulled = pullAtIndex(myArray, [1, 3]); // myArray = [ 'a', 'c' ] , pulled = [ 'b', 'd' ]
```
## pullAtValue
改变原始数组，过滤出指定的值。 返回删除的元素。
使用`Array.filter()`和`Array.includes()`来剔除指定的值。使用`Array.length = 0`将数组中的长度重置为零， 并且通过`Array.push()`只使用`pulled`值重新填充数组。使用`Array.push()`来跟踪`pulled`值。
```js
const pullAtValue = (arr, pullArr) => {
  let removed = [],
    pushToRemove = arr.forEach((v, i) => (pullArr.includes(v) ? removed.push(v) : v)),
    mutateTo = arr.filter((v, i) => !pullArr.includes(v));
  arr.length = 0;
  mutateTo.forEach(v => arr.push(v));
  return removed;
};

let myArray = ['a', 'b', 'c', 'd'];
let pulled = pullAtValue(myArray, ['b', 'd']); // myArray = [ 'a', 'c' ] , pulled = [ 'b', 'd' ]
```
## reducedFilter
根据条件过滤一个对象数组，同时过滤掉未指定的键（`key`）。
使用`Array.filter()`根据断言`fn`过滤数组，以便返回条件为真值（`truthy`）的对象。在过滤出来的数组上，使用`Array.map()`和`Array.reduce()`返回新的对象来过滤掉`keys`参数中未提供的键。
```js
const reducedFilter = (data, keys, fn) =>
  data.filter(fn).map(el =>
    keys.reduce((acc, key) => {
      acc[key] = el[key];
      return acc;
    }, {})
  );

const data = [
  {
    id: 1,
    name: 'john',
    age: 24
  },
  {
    id: 2,
    name: 'mike',
    age: 50
  }
];

reducedFilter(data, ['id', 'name'], item => item.age > 24); // [{ id: 2, name: 'mike'}]
```
## remove
从数组中移除给定函数返回`false`的元素。
使用`Array.filter()`和`Array.reduce()`来查找返回真值的数组元素，使用`Array.splice()`来移除元素。`func`有三个参数`(value, index, array)`。
```js
const remove = (arr, func) =>
  Array.isArray(arr)
    ? arr.filter(func).reduce((acc, val) => {
      arr.splice(arr.indexOf(val), 1);
      return acc.concat(val);
    }, [])
    : [];

remove([1, 2, 3, 4], n => n % 2 === 0); // [2, 4]
```
## sample
从数组中随机返回一个元素。
使用`Math.random()`生成一个随机数，乘以`length`，并使用`Math.floor()`舍去小数获得到最接近的整数。这个方法也适用于字符串。
```js
const sample = arr => arr[Math.floor(Math.random() * arr.length)];

sample([3, 7, 9, 11]); // 9
```
## sampleSize
从`array`中获取`n`个唯一键随机元素。
使用`Fisher-Yates`（洗牌算法）对数组进行打乱。使用`Array.slice()`获取第一个`n`元素。 省略第二个参数，`n`从数组中随机取得1个元素。
```js
const sampleSize = ([...arr], n = 1) => {
  let m = arr.length;
  while (m) {
    const i = Math.floor(Math.random() * m--);
    [arr[m], arr[i]] = [arr[i], arr[m]];
  }
  return arr.slice(0, n);
};

sampleSize([1, 2, 3], 2); // [3,1]
sampleSize([1, 2, 3], 4); // [2,3,1]
```
## shank
与`array.prototype.splice()`具有相同的功能，但返回新数组而不是改变原始数组。
```js
const shank = (arr, index = 0, delCount = 0, ...elements) =>
  arr
    .slice(0, index)
    .concat(elements)
    .concat(arr.slice(index + delCount));

const names = ['alpha', 'bravo', 'charlie'];
const namesAndDelta = shank(names, 1, 0, 'delta'); // [ 'alpha', 'delta', 'bravo', 'charlie' ]
const namesNoBravo = shank(names, 1, 1); // [ 'alpha', 'charlie' ]
console.log(names); // ['alpha', 'bravo', 'charlie']
```
## shuffle
随机排列指定数组的值，返回一个新的数组。
使用`Fisher-Yates`算法对数组元素进行重新排序。
```js
const shuffle = ([...arr]) => {
  let m = arr.length;
  while (m) {
    const i = Math.floor(Math.random() * m--);
    [arr[m], arr[i]] = [arr[i], arr[m]];
  }
  return arr;
};

const foo = [1, 2, 3];
shuffle(foo); // [2, 3, 1], foo = [1, 2, 3]
```
## similarity
返回存在于两个数组中的元素数组。
使用`Array.filter()`移除不在`values`中的值，使用`Array.includes()`确定。
```
const similarity = (arr, values) => arr.filter(v => values.includes(v));

similarity([1, 2, 3], [1, 2, 4]); // [1, 2]
```
## symmetricDifference
返回两个数组之间的差集。
根据每个数组创建一个`Set`，然后在每个数组上使用`Array.filter()`，只保留另一个数组不包含的值。
```js
const symmetricDifference = (a, b) => {
  const sA = new Set(a),
    sB = new Set(b);
  return [...a.filter(x => !sB.has(x)), ...b.filter(x => !sA.has(x))];
};

symmetricDifference([1, 2, 3], [1, 2, 4]); // [3, 4]
symmetricDifference([1, 2, 2], [1, 3, 1]); // [2, 2, 3]
```
## symmetricDifferenceBy
将提供的函数应用于两个数组的每个数组元素后，返回两个数组之间的差集。
通过对每个数组的元素应用`fn`创建一个集合，然后对每个元素使用`array.prototype.filter()`来只保留不包含在另一个数组中的值。
```js
const symmetricDifferenceBy = (a, b, fn) => {
  const sA = new Set(a.map(v => fn(v))),
    sB = new Set(b.map(v => fn(v)));
  return [...a.filter(x => !sB.has(fn(x))), ...b.filter(x => !sA.has(fn(x)))];
};

symmetricDifferenceBy([2.1, 1.2], [2.3, 3.4], Math.floor); // [ 1.2, 3.4 ]
```
## tail
返回剔除第一个元素后的数组。
如果数组的`length`大于1，则返回`arr.slice(1)`，否则返回整个数组。
```js
const tail = arr => (arr.length > 1 ? arr.slice(1) : arr);

tail([1, 2, 3]); // [2,3]
tail([1]); // [1]
```
## take
从一个给定的数组中创建一个前`n`个元素的数组。
使用`Array.slice()`创建一个数组包含第一个元素开始，到`n`个元素结束的数组。
```js
const take = (arr, n = 1) => arr.slice(0, n);

take([1, 2, 3], 5); // [1, 2, 3]
take([1, 2, 3], 0); // []
```
## takeRight
从一个给定的数组中创建一个后`n`个元素的数组。
使用`Array.slice()`来创建一个从第`n`个元素开始至末尾的数组。
```js
const takeRight = (arr, n = 1) => arr.slice(arr.length - n, arr.length);

takeRight([1, 2, 3], 2); // [ 2, 3 ]
takeRight([1, 2, 3]); // [3]
```
## takeWhile
移除数组中的元素，直到传递的函数返回`true`。返回删除的元素。
循环数组，使用`for…of`循环`array.prototype.entries()`，直到函数返回的值为`true`。使用`array.prototype.slice()`返回删除的元素。
```js
const takeWhile = (arr, func) => {
  for (const [i, val] of arr.entries()) if (func(val)) return arr.slice(0, i);
  return arr;
};

takeWhile([1, 2, 3, 4], n => n >= 3); // [1, 2]
```
## takeRightWhile
从数组末尾移除元素，直到传递的函数返回`true`。返回删除的元素。
使用`array.prototype.reducceright()`循环数组，并在函数返回错误值时累积元素。
```js
const takeRightWhile = (arr, func) =>
  arr.reduceRight((acc, el) => (func(el) ? acc : [el, ...acc]), []);

takeRightWhile([1, 2, 3, 4], n => n < 3); // [3, 4]
```
## union
返回两个数组中任何一个中存在的每个元素一次。
创建一个包含A和B所有值的集合，并转换为数组。
```js
const union = (a, b) => Array.from(new Set([...a, ...b]));

union([1, 2, 3], [4, 3, 2]); // [1,2,3,4]
```
## unionBy
将提供的函数应用于两个数组的每个数组元素之后，返回两个数组中任何一个数组中存在的每个元素一次。
```js
const unionBy = (a, b, fn) => {
  const s = new Set(a.map(fn));
  return Array.from(new Set([...a, ...b.filter(x => !s.has(fn(x)))]));
};

unionBy([2.1], [1.2, 2.3], Math.floor); // [2.1, 1.2]
```
## uniqueElements
返回数组的所有唯一值。
```js
const uniqueElements = arr => [...new Set(arr)];

uniqueElements([1, 2, 2, 3, 4, 4, 5]); // [1, 2, 3, 4, 5]
```
## without
从数组中排除给定值。
使用`Array.filter()`创建一个不包括所有给定值的数组。
```js
const without = (arr, ...args) => arr.filter(v => !args.includes(v));

without([2, 1, 2, 3], 1, 2); // [3]
```
## xProd
通过从数组中创建每个可能的对，从提供的两个数组中创建一个新数组。
使用`array.prototype.reduce()、array.prototype.map()`和`array.prototype.concat()`从两个数组的元素中生成每个可能的对，并将它们保存在一个数组中。
```js
const xProd = (a, b) => a.reduce((acc, x) => acc.concat(b.map(y => [x, y])), []);

xProd([1, 2], ['a', 'b']); // [[1, 'a'], [1, 'b'], [2, 'a'], [2, 'b']]
```
## zip
创建一组元素，根据原始数组中的位置进行分组。
```js
const zip = (...arrays) => {
  const maxLength = Math.max(...arrays.map(x => x.length));
  return Array.from({ length: maxLength }).map((_, i) => {
    return Array.from({ length: arrays.length }, (_, k) => arrays[k][i]);
  });
};

zip(['a', 'b'], [1, 2], [true, false]); // [['a', 1, true], ['b', 2, false]]
zip(['a'], [1, 2], [true, false]); // [['a', 1, true], [undefined, 2, false]]
```
## zipObject
给定一个有效属性标识符数组和一个值数组，返回一个将属性与值关联的对象。
由于一个对象可以有未定义的值，但不存在未定义的属性，该属性数组用于使用`Array.reduce()`来决定结果对象的结构。
```js
const zipObject = (props, values) =>
  props.reduce((obj, prop, index) => ((obj[prop] = values[index]), obj), {});

zipObject(['a', 'b', 'c'], [1, 2]); // {a: 1, b: 2, c: undefined}
zipObject(['a', 'b'], [1, 2, 3]); // {a: 1, b: 2}
```