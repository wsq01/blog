---
title: ES6 Module的加载实现
date: 2019-07-11 11:02:24
tags: [javascript, ES6]
categories: [javascript, ES6]
---

# 浏览器加载
## 传统方法
HTML网页中，浏览器通过`<script>`标签加载JavaScript脚本。
```js
<!-- 页面内嵌的脚本 -->
<script type="application/javascript">
  // module code
</script>
<!-- 外部脚本 -->
<script type="application/javascript" src="path/to/myModule.js">
</script>
```
上面代码中，由于浏览器脚本的默认语言是JavaScript，因此`type="application/javascript"`可以省略。
默认情况下，浏览器是同步加载JavaScript脚本，即渲染引擎遇到`<script>`标签就会停下来，等到执行完脚本，再继续向下渲染。如果是外部脚本，还必须加入脚本下载的时间。
如果脚本体积很大，下载和执行的时间就会很长，因此造成浏览器堵塞，用户会感觉到浏览器“卡死”了，没有任何响应。这显然是很不好的体验，所以浏览器允许脚本异步加载，下面就是两种异步加载的语法。
```js
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```
上面代码中，`<script>`标签打开`defer`或`async`属性，脚本就会异步加载。渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接执行后面的命令。
`defer`与`async`的区别是：`defer`要等到整个页面在内存中正常渲染结束（DOM结构完全生成，以及其他脚本执行完成），才会执行；`async`一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。一句话，`defer`是渲染完再执行，`async`是下载完就执行。另外，如果有多个`defer`脚本，会按照它们在页面出现的顺序加载，而多个`async`脚本是不能保证加载顺序的。
## 加载规则
浏览器加载ES6模块，也使用`<script>`标签，但是要加入`type="module"`属性。
```js
<script type="module" src="./foo.js"></script>
```
浏览器对于带有`type="module"`的`<script>`，都是异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了`<script>`标签的`defer`属性。
如果网页有多个`<script type="module">`，它们会按照在页面出现的顺序依次执行。
`<script>`标签的`async`属性也可以打开，这时只要加载完成，渲染引擎就会中断渲染立即执行。执行完成后，再恢复渲染。
```js
<script type="module" src="./foo.js" async></script>
```
一旦使用了`async`属性，`<script type="module">`就不会按照在页面出现的顺序执行，而是只要该模块加载完成，就执行该模块。
ES6模块也允许内嵌在网页中，语法行为与加载外部脚本完全一致。
```js
<script type="module">
  import utils from "./utils.js";
  // other code
</script>
```
对于外部的模块脚本（上例是`foo.js`），有几点需要注意。
* 代码是在模块作用域之中运行，而不是在全局作用域运行。模块内部的顶层变量，外部不可见。
* 模块脚本自动采用严格模式，不管有没有声明`use strict`。
* 模块之中，可以使用`import`命令加载其他模块（`.js`后缀不可省略，需要提供绝对URL或相对URL），也可以使用`export`命令输出对外接口。
* 模块之中，顶层的`this`关键字返回`undefined`，而不是指向`window`。也就是说，在模块顶层使用`this`关键字，是无意义的。
* 同一个模块如果加载多次，将只执行一次。

下面是一个示例模块。
```js
import utils from 'https://example.com/js/utils.js';
const x = 1;
console.log(x === window.x); //false
console.log(this === undefined); // true
```
利用顶层的`this`等于`undefined`这个语法点，可以侦测当前代码是否在ES6模块之中。
```js
const isNotModuleScript = this !== undefined;
```
# ES6模块与CommonJS模块的差异
ES6模块与CommonJS模块完全不同。它们有两个重大差异。
* CommonJS模块输出的是一个值的拷贝，ES6模块输出的是值的引用
* CommonJS模块是运行时加载，ES6模块是编译时输出接口

第二个差异是因为CommonJS加载的是一个对象（即`module.exports`属性），该对象只有在脚本运行完才会生成。而ES6模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。
CommonJS模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。
```js
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
// main.js
var mod = require('./lib');
console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```
上面代码说明，`lib.js`模块加载以后，它的内部变化就影响不到输出的`mod.counter`了。这是因为`mod.counter`是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。
```js
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```
上面代码中，输出的`counter`属性实际上是一个取值器函数。现在再执行`main.js`，就可以正确读取内部变量`counter`的变动了。
```js
$ node main.js
3
4
```
ES6模块的运行机制与CommonJS不一样。JS引擎对脚本静态分析的时候，遇到模块加载命令`import`，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。
因此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。
```js
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}
// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```
再举一个例子。
```js
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
// m2.js
import {foo} from './m1.js';
console.log(foo);
setTimeout(() => console.log(foo), 500);
```
上面代码中，`m1.js`的变量`foo`，在刚加载时等于`bar`，过了500毫秒，又变为等于`baz`。
运行结果：
```
$ babel-node m2.js
bar
baz
```
上面代码表明，ES6模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。
由于ES6输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。
```js
// lib.js
export let obj = {};
// main.js
import { obj } from './lib';
obj.prop = 123; // OK
obj = {}; // TypeError
```
最后，`export`通过接口，输出的是同一个值。不同的脚本加载这个接口，得到的都是同样的实例。
```js
// mod.js
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}
export let c = new C();
```
上面的脚本`mod.js`，输出的是一个C的实例。不同的脚本加载这个模块，得到的都是同一个实例。
```js
// x.js
import {c} from './mod';
c.add();
// y.js
import {c} from './mod';
c.show();
// main.js
import './x';
import './y';
```
现在执行`main.js`，输出的是1。
```
$ babel-node main.js
1
```
这就证明了`x.js`和`y.js`加载的都是`c`的同一个实例。
# Node加载
## 概述
Node对ES6模块的处理比较麻烦，因为它有自己的CommonJS模块格式，与ES6模块格式是不兼容的。目前的解决方案是，将两者分开，ES6模块和CommonJS采用各自的加载方案。
Node要求ES6模块采用`.mjs`后缀文件名。也就是说，只要脚本文件里面使用`import`或者`export`命令，那么就必须采用`.mjs`后缀名。`require`命令不能加载`.mjs`文件，会报错，只有`import`命令才可以加载`.mjs`文件。反过来，`.mjs`文件里面也不能使用`require`命令，必须使用`import`。
目前，这项功能还在试验阶段。安装Node v8.5.0或以上版本，要用`--experimental-modules`参数才能打开该功能。
```
$ node --experimental-modules my-app.mjs
```
为了与浏览器的`import`加载规则相同，Node的`.mjs`文件支持URL路径。
```js
import './foo?query=1'; // 加载 ./foo 传入参数 ?query=1
```
上面代码中，脚本路径带有参数`?query=1`，Node会按URL规则解读。同一个脚本只要参数不同，就会被加载多次，并且保存成不同的缓存。由于这个原因，只要文件名中含有:、%、#、?等特殊字符，最好对这些字符进行转义。
目前，Node的`import`命令只支持加载本地模块（`file:`协议），不支持加载远程模块。
如果模块名不含路径，那么`import`命令会去`node_modules`目录寻找这个模块。
```js
import 'baz';
import 'abc/123';
```
如果模块名包含路径，那么`import`命令会按照路径去寻找这个名字的脚本文件。
```js
import 'file:///etc/config/app.json';
import './foo';
import './foo?search';
import '../bar';
import '/baz';
```
如果脚本文件省略了后缀名，比如`import './foo'`，Node会依次尝试四个后缀名：`./foo.mjs`、`./foo.js`、`./foo.json`、`./foo.node`。如果这些脚本文件都不存在，Node就会去加载`./foo/package.json`的`main`字段指定的脚本。如果`./foo/package.json`不存在或者没有`main`字段，那么就会依次加载`./foo/index.mjs`、`./foo/index.js`、`./foo/index.json`、`./foo/index.node`。如果以上四个文件还是都不存在，就会抛出错误。
最后，Node的`import`命令是异步加载，这一点与浏览器的处理方法相同。
## 内部变量
ES6模块应该是通用的，同一个模块不用修改，就可以用在浏览器环境和服务器环境。为了达到这个目标，Node规定ES6模块之中不能使用CommonJS模块的特有的一些内部变量。
首先，就是`this`关键字。ES6模块之中，顶层的`this`指向`undefined`；CommonJS模块的顶层`this`指向当前模块，这是两者的一个重大差异。
其次，以下这些顶层变量在ES6模块之中都是不存在的。
```
arguments
require
module
exports
__filename
__dirname
```
如果你一定要使用这些变量，有一个变通方法，就是写一个CommonJS模块输出这些变量，然后再用 ES6模块加载这个CommonJS模块。但是这样一来，该ES6模块就不能直接用于浏览器环境了，所以不推荐这样做。
```js
// expose.js
module.exports = {__dirname};
// use.mjs
import expose from './expose.js';
const {__dirname} = expose;
```
上面代码中，`expose.js`是一个CommonJS模块，输出变量`__dirname`，该变量在ES6模块之中不存在。ES6模块加载`expose.js`，就可以得到`__dirname`。
## ES6模块加载CommonJS模块
CommonJS模块的输出都定义在`module.exports`这个属性上面。Node的`import`命令加载CommonJS模块，Node会自动将`module.exports`属性，当作模块的默认输出，即等同于`export default xxx`。
下面是一个CommonJS模块。
```js
// a.js
module.exports = {
  foo: 'hello',
  bar: 'world'
};
// 等同于
export default {
  foo: 'hello',
  bar: 'world'
};
```
`import`命令加载上面的模块，`module.exports`会被视为默认输出，即`import`命令实际上输入的是这样一个对象`{ default: module.exports }`。
所以，一共有三种写法，可以拿到CommonJS模块的`module.exports`。
```js
// 写法一
import baz from './a';
// baz = {foo: 'hello', bar: 'world'};
// 写法二
import {default as baz} from './a';
// baz = {foo: 'hello', bar: 'world'};
// 写法三
import * as baz from './a';
// baz = {
//   get default() {return module.exports;},
//   get foo() {return this.default.foo}.bind(baz),
//   get bar() {return this.default.bar}.bind(baz)
// }
```
上面代码的第三种写法，可以通过`baz.default`拿到`module.exports`。`foo`属性和`bar`属性就是可以通过这种方法拿到了`module.exports`。
下面是一些例子。
```js
// b.js
module.exports = null;
// es.js
import foo from './b';
// foo = null;
import * as bar from './b';
// bar = { default:null };
```
上面代码中，`es.js`采用第二种写法时，要通过`bar.default`这样的写法，才能拿到`module.exports`。
```js
// c.js
module.exports = function two() {
  return 2;
};
// es.js
import foo from './c';
foo(); // 2
import * as bar from './c';
bar.default(); // 2
bar(); // throws, bar is not a function
```
上面代码中，`bar`本身是一个对象，不能当作函数调用，只能通过`bar.default`调用。
CommonJS模块的输出缓存机制，在ES6加载方式下依然有效。
```js
// foo.js
module.exports = 123;
setTimeout(_ => module.exports = null);
```
上面代码中，对于加载`foo.js`的脚本，`module.exports`将一直是123，而不会变成`null`。
由于ES6模块是编译时确定输出接口，CommonJS模块是运行时确定输出接口，所以采用`import`命令加载CommonJS模块时，不允许采用下面的写法。
```js
// 不正确
import { readFile } from 'fs';
```
上面的写法不正确，因为`fs`是CommonJS格式，只有在运行时才能确定`readFile`接口，而`import`命令要求编译时就确定这个接口。解决方法就是改为整体输入。
```js
// 正确的写法一
import * as express from 'express';
const app = express.default();
// 正确的写法二
import express from 'express';
const app = express();
```
## CommonJS模块加载ES6模块
CommonJS模块加载ES6模块，不能使用`require`命令，而要使用`import()`函数。ES6模块的所有输出接口，会成为输入对象的属性。
```js
// es.mjs
let foo = { bar: 'my-default' };
export default foo;
// cjs.js
const es_namespace = await import('./es.mjs');
// es_namespace = {
//   get default() {
//     ...
//   }
// }
console.log(es_namespace.default);
// { bar:'my-default' }
```
上面代码中，`default`接口变成了`es_namespace.default`属性。
下面是另一个例子。
```js
// es.js
export let foo = { bar:'my-default' };
export { foo as bar };
export function f() {};
export class c {};
// cjs.js
const es_namespace = await import('./es');
// es_namespace = {
//   get foo() {return foo;}
//   get bar() {return foo;}
//   get f() {return f;}
//   get c() {return c;}
// }
```
# 循环加载
循环加载指的是，`a`脚本的执行依赖`b`脚本，而`b`脚本的执行又依赖`a`脚本。
```js
// a.js
var b = require('b');
// b.js
var a = require('a');
```
通常，循环加载表示存在强耦合，如果处理不好，还可能导致递归加载，使得程序无法执行，因此应该避免出现。
目前最常见的两种模块格式CommonJS和ES6，处理“循环加载”的方法是不一样的，返回的结果也不一样。
## CommonJS模块的加载原理
CommonJS的一个模块，就是一个脚本文件。`require`命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。
```js
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```
上面代码就是Node内部加载模块后生成的一个对象。该对象的`id`属性是模块名，`exports`属性是模块输出的各个接口，`loaded`属性是一个布尔值，表示该模块的脚本是否执行完毕。其他还有很多属性，这里都省略了。
以后需要用到这个模块的时候，就会到`exports`属性上面取值。即使再次执行`require`命令，也不会再次执行该模块，而是到缓存之中取值。也就是说，CommonJS模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。
## CommonJS模块的循环加载
CommonJS模块的重要特性是加载时执行，即脚本代码在`require`的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。
```js
// a.js
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');
```
上面代码之中，`a.js`脚本先输出一个`done`变量，然后加载另一个脚本文件`b.js`。注意，此时`a.js`代码就停在这里，等待`b.js`执行完毕，再往下执行。
```js
// b.js
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');
```
上面代码之中，`b.js`执行到第二行，就会去加载`a.js`，这时，就发生了“循环加载”。系统会去`a.js`模块对应对象的`exports`属性取值，可是因为`a.js`还没有执行完，从`exports`属性只能取回已经执行的部分，而不是最后的值。
`a.js`已经执行的部分，只有一行。
```js
exports.done = false;
```
因此，对于`b.js`来说，它从a.js只输入一个变量`done`，值为`false`。
然后，`b.js`接着往下执行，等到全部执行完毕，再把执行权交还给`a.js`。于是，`a.js`接着往下执行，直到执行完毕。我们写一个脚本`main.js`，验证这个过程。
```js
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```
执行`main.js`，运行结果如下。
```
$ node main.js

在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true
```
上面的代码证明了两件事。一是，在`b.js`之中，`a.js`没有执行完毕，只执行了第一行。二是，`main.js`执行到第二行时，不会再次执行`b.js`，而是输出缓存的`b.js`的执行结果，即它的第四行。
```js
exports.done = true;
```
总之，CommonJS输入的是被输出值的拷贝，不是引用。
另外，由于CommonJS模块遇到循环加载时，返回的是当前已经执行的部分的值，而不是代码全部执行后的值，两者可能会有差异。所以，输入变量的时候，必须非常小心。
```js
var a = require('a'); // 安全的写法
var foo = require('a').foo; // 危险的写法
exports.good = function (arg) {
  return a.foo('good', arg); // 使用的是 a.foo 的最新值
};
exports.bad = function (arg) {
  return foo('bad', arg); // 使用的是一个部分加载时的值
};
```
上面代码中，如果发生循环加载，`require('a').foo`的值很可能后面会被改写，改用`require('a')`会更保险一点。
## ES6模块的循环加载
ES6处理循环加载与CommonJS有本质的不同。ES6模块是动态引用，如果使用`import`从一个模块加载变量（即`import foo from 'foo'`），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。
```js
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';
// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo);
export let bar = 'bar';
```
上面代码中，`a.mjs`加载`b.mjs`，`b.mjs`又加载`a.mjs`，构成循环加载。执行`a.mjs`，结果如下。
```
$ node --experimental-modules a.mjs
b.mjs
ReferenceError: foo is not defined
```
上面代码中，执行`a.mjs`以后会报错，`foo`变量未定义，这是为什么？
让我们一行行来看，ES6循环加载是怎么处理的。首先，执行`a.mjs`以后，引擎发现它加载了`b.mjs`，因此会优先执行`b.mjs`，然后再执行`a.mjs`。接着，执行`b.mjs`的时候，已知它从`a.mjs`输入了`foo`接口，这时不会去执行`a.mjs`，而是认为这个接口已经存在了，继续往下执行。执行到第三行`console.log(foo)`的时候，才发现这个接口根本没定义，因此报错。
解决这个问题的方法，就是让`b.mjs`运行的时候，`foo`已经有定义了。这可以通过将`foo`写成函数来解决。
```js
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
function foo() { return 'foo' }
export {foo};
// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo());
function bar() { return 'bar' }
export {bar};
```
这时再执行`a.mjs`就可以得到预期结果。
```
$ node --experimental-modules a.mjs
b.mjs
foo
a.mjs
bar
```
这是因为函数具有提升作用，在执行`import {bar} from './b'`时，函数`foo`就已经有定义了，所以`b.mjs`加载的时候不会报错。这也意味着，如果把函数`foo`改写成函数表达式，也会报错。
```js
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
const foo = () => 'foo';
export {foo};
```
上面代码的第四行，改成了函数表达式，就不具有提升作用，执行就会报错。
我们再来看ES6模块加载器SystemJS给出的一个例子。
```js
// even.js
import { odd } from './odd'
export var counter = 0;
export function even(n) {
  counter++;
  return n === 0 || odd(n - 1);
}
// odd.js
import { even } from './even';
export function odd(n) {
  return n !== 0 && even(n - 1);
}
```
上面代码中，`even.js`里面的函数`even`有一个参数`n`，只要不等于0，就会减去1，传入加载的`odd()`。`odd.js`也会做类似操作。
运行上面这段代码，结果如下。
```
$ babel-node
> import * as m from './even.js';
> m.even(10);
true
> m.counter
6
> m.even(20)
true
> m.counter
17
```
上面代码中，参数`n`从10变为0的过程中，`even()`一共会执行6次，所以变量`counter`等于6。第二次调用`even()`时，参数`n`从20变为0，`even()`一共会执行11次，加上前面的6次，所以变量`counter`等于17。
这个例子要是改写成CommonJS，就根本无法执行，会报错。
```js
// even.js
var odd = require('./odd');
var counter = 0;
exports.counter = counter;
exports.even = function (n) {
  counter++;
  return n == 0 || odd(n - 1);
}
// odd.js
var even = require('./even').even;
module.exports = function (n) {
  return n != 0 && even(n - 1);
}
```
上面代码中，`even.js`加载`odd.js`，而`odd.js`又去加载`even.js`，形成“循环加载”。这时，执行引擎就会输出`even.js`已经执行的部分（不存在任何结果），所以在`odd.js`之中，变量`even`等于`undefined`，等到后面调用`even(n-1)`就会报错。
```
$ node
> var m = require('./even');
> m.even(10)
TypeError: even is not a function
```
