---
title: JS执行机制——执行上下文
date: 2019-11-03 11:43:26
tags: [javascript, JS执行机制]
categories: [javascript, JS执行机制]
---

# 变量提升（Hoisting）
我们先来看看什么是 JavaScript 中的声明和赋值。
```js
var myname = '张三'
```
这段代码你可以把它看成是两行代码组成的：
```js
var myname // 声明部分
myname = '张三' // 赋值部分
```

{% asset_img img1.png %}

上面是变量的声明和赋值，下面是函数的声明和赋值：
```js
function foo() {
  console.log('foo')
}
 
var bar = function() {
  console.log('bar')
}
```
第一个函数`foo`是一个完整的函数声明，也就是说没有涉及到赋值操作；第二个函数是先声明变量`bar`，再把`function(){console.log('bar')}`赋值给`bar`。

{% asset_img img2.png 函数的声明和赋值 %}

所谓的变量提升，是指在 JavaScript 代码执行过程中，JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的“行为”。变量被提升后，会给变量设置默认值，这个默认值就是我们熟悉的`undefined`。
```js
/*
* 变量提升部分
*/
// 把变量 myname 提升到开头，
// 同时给 myname 赋值为 undefined
var myname = undefined
// 把函数 showName 提升到开头
function showName() {
  console.log('showName 被调用');
}
 
/*
* 可执行代码部分
*/
showName()
console.log(myname)
// 去掉 var 声明部分，保留赋值语句
myname = '极客时间'
```
为了模拟变量提升的效果，对代码做以下调整，如下图：

{% asset_img img3.png 模拟变量提升示意图 %}

通过这段模拟的变量提升代码，明白了可以在定义之前使用变量或者函数的原因：函数和变量在执行之前都提升到了代码开头。
# JavaScript 代码的执行流程
从概念的字面意义上来看，“变量提升”意味着变量和函数的声明会在物理层面移动到代码的最前面，正如我们所模拟的那样。但，这并不准确。实际上变量和函数声明在代码里的位置是不会改变的，而且是在编译阶段被 JavaScript 引擎放入内存中。一段 JavaScript 代码在执行之前需要被 JavaScript 引擎编译，编译完成之后，才会进入执行阶段。大致流程：

{% asset_img img4.png JavaScript 的执行流程图 %}

## 1. 编译阶段
那么编译阶段和变量提升存在什么关系呢？

为了搞清楚这个问题，我们还是回过头来看上面那段模拟变量提升的代码，为了方便，可以把这段代码分成两部分。

第一部分：变量提升部分的代码。
```js
var myname = undefined
function showName() {
  console.log('函数 showName 被执行');
}
```
第二部分：执行部分的代码。
```js
showName()
console.log(myname)
myname = '极客时间'
```
下面我们就可以把 JavaScript 的执行流程细化，如下图所示：

{% asset_img img5.png JavaScript 执行流程细化图 %}

从上图可以看出，输入一段代码，经过编译后，会生成两部分内容：执行上下文（`Execution context`）和可执行代码。

执行上下文是 JavaScript 执行一段代码时的运行环境，比如调用一个函数，就会进入这个函数的执行上下文，确定该函数在执行期间用到的诸如`this`、变量、对象以及函数等。

在执行上下文中存在一个变量环境的对象（`Viriable Environment`），该对象中保存了变量提升的内容，比如上面代码中的变量`myname`和函数`showName`，都保存在该对象中。

你可以简单地把变量环境对象看成是如下结构：
```js
VariableEnvironment:
    myname -> undefined, 
    showName -> function : {console.log(myname)
```
了解完变量环境对象的结构后，接下来，我们再结合下面这段代码来分析下是如何生成变量环境对象的。
```js
showName()
console.log(myname)
var myname = '极客时间'
function showName() {
  console.log('函数 showName 被执行');
}
```
我们可以一行一行来分析上述代码：
* 第 1 行和第 2 行，由于这两行代码不是声明操作，所以 JavaScript 引擎不会做任何处理；
* 第 3 行，由于这行是经过`var`声明的，因此 JavaScript 引擎将在环境对象中创建一个名为`myname`的属性，并使用`undefined`对其初始化；
* 第 4 行，JavaScript 引擎发现了一个通过`function`定义的函数，所以它将函数定义存储到堆 (HEAP）中，并在环境对象中创建一个`showName`的属性，然后将该属性值指向堆中函数的位置。

这样就生成了变量环境对象。接下来 JavaScript 引擎会把声明以外的代码编译为字节码，你可以类比如下的模拟代码：
```js
showName()
console.log(myname)
myname = '极客时间'
```
现在有了执行上下文和可执行代码了，那么接下来就到了执行阶段了。
## 2. 执行阶段
JavaScript 引擎开始执行“可执行代码”，按照顺序一行一行地执行。下面我们就来一行一行分析下这个执行过程：
* 当执行到`showName`函数时，JavaScript 引擎便开始在变量环境对象中查找该函数，由于变量环境对象中存在该函数的引用，所以 JavaScript 引擎便开始执行该函数，并输出“函数`showName`被执行”结果。
* 接下来打印`“myname”`信息，JavaScript 引擎继续在变量环境对象中查找该对象，由于变量环境存在`myname`变量，并且其值为 `undefined`，所以这时候就输出`undefined`。
* 接下来执行第 3 行，把“极客时间”赋给`myname`变量，赋值后变量环境中的`myname`属性值改变为“极客时间”，变量环境如下所示：
```js
VariableEnvironment:
    myname -> " 极客时间 ", 
    showName ->function : {console.log(myname)
```

以上就是一段代码的编译和执行流程。实际上，编译阶段和执行阶段都是非常复杂的，包括了词法分析、语法解析、代码优化、代码生成等。
# 代码中出现相同的变量或者函数怎么办？
现在你已经知道了，在执行一段 JavaScript 代码之前，会编译代码，并将代码中的函数和变量保存到执行上下文的变量环境中，那么如果代码中出现了重名的函数或者变量，JavaScript 引擎会如何处理？
```js
function showName() {
  console.log('张三');
}
showName();
function showName() {
  console.log('李四');
}
showName(); 
```
完整执行流程：
* 首先是编译阶段。遇到了第一个`showName`函数，会将该函数体存放到变量环境中。接下来是第二个`showName`函数，继续存放至变量环境中，但是变量环境中已经存在一个`showName`函数了，此时，第二个`showName`函数会将第一个`showName`函数覆盖掉。这样变量环境中就只存在第二个`showName`函数了。
* 接下来是执行阶段。先执行第一个`showName`函数，但由于是从变量环境中查找`showName`函数，而变量环境中只保存了第二个`showName`函数，所以最终调用的是第二个函数，打印的内容是“李四”。第二次执行`showName`函数也是走同样的流程，所以输出的结果也是“李四”。

综上所述，一段代码如果定义了两个相同名字的函数，那么最终生效的是最后一个函数。
# 总结
* JavaScript 代码执行过程中，需要先做变量提升，而之所以需要实现变量提升，是因为 JavaScript 代码在执行之前需要先编译。
* 在编译阶段，变量和函数会被存放到变量环境中，变量的默认值会被设置为`undefined`；在代码执行阶段，JavaScript 引擎会从变量环境中去查找自定义的变量和函数。
* 如果在编译阶段，存在两个相同的函数，那么最终存放在变量环境中的是最后定义的那个，这是因为后定义的会覆盖掉之前定义的。