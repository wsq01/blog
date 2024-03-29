---
title: CSS 变量
date: 2020-04-10 18:01:52
tags: [CSS]
categories: 
 - 前端
 - CSS
---

# 变量的声明
CSS中原生的变量定义语法是：`--*`，变量使用语法是：`var(--*)`，其中`*`表示我们的变量名称。关于命名这个东西，各种语言都有些显示，例如CSS选择器不能是数字开头，JS中的变量是不能直接数值的，但是，在CSS变量中，这些限制通通没有。

但是，不能包含`$，[，^，(，%`等字符，普通字符局限在只要是“数字[`0-9`]”“字母[`a-zA-Z`]”“下划线`_`”和“短横线`-`”这些组合，但是可以是中文，日文或者韩文。
```css
:root {
  --1: #369;
  --深蓝: #369;
}
body {
  background-color: var(--1);
}

div {
  background-color: var(--深蓝);
}
```
你可能会问，为什么选择两根连词线（--）表示变量？因为`$foo`被 Sass 用掉了，`@foo`被 Less 用掉了。为了不产生冲突，官方的 CSS 变量就改用两根连词线了。

各种值都可以放入 CSS 变量。
```css
:root{
  --main-color: #4d4e53;
  --main-bg: rgb(255, 255, 255);
  --logo-border-color: rebeccapurple;

  --header-height: 68px;
  --content-padding: 10px 20px;

  --base-line-height: 1.428571429;
  --transition-duration: .35s;
  --external-link: "external link";
  --margin-top: calc(2vh + 20px);
}
```
# var() 函数
`var()`函数用于读取变量。
```css
a {
  color: var(--foo);
  text-decoration-color: var(--bar);
}
```
`var()`函数还可以使用第二个参数，表示变量的默认值。如果该变量不存在，就会使用这个默认值。
```css
color: var(--foo, #7F583F);
```
第二个参数不处理内部的逗号或空格，都视作参数的一部分。
```css
var(--font-stack, "Roboto", "Helvetica");
var(--pad, 10px 15px 20px);
```
`var()`函数还可以用在变量的声明。
```css
:root {
  --primary-color: red;
  --logo-text: var(--primary-color);
}
```
注意，变量值只能用作属性值，不能用作属性名。
```css
.foo {
  --side: margin-top;
  /* 无效 */
  var(--side): 20px;
}
```
上面代码中，变量`--side`用作属性名，这是无效的。
# 变量值的类型
如果变量值是一个字符串，可以与其他字符串拼接。
```css
--bar: 'hello';
--foo: var(--bar)' world';
```
如果变量值是数值，不能与数值单位直接连用。
```css
.foo {
  --gap: 20;
  /* 无效 */
  margin-top: var(--gap)px;
}
```
上面代码中，数值与单位直接写在一起，这是无效的。必须使用`calc()`函数，将它们连接。
```css
.foo {
  --gap: 20;
  margin-top: calc(var(--gap) * 1px);
}
```
如果变量值带有单位，就不能写成字符串。
```css
/* 无效 */
.foo {
  --foo: '20px';
  font-size: var(--foo);
}

/* 有效 */
.foo {
  --foo: 20px;
  font-size: var(--foo);
}
```
# 作用域
同一个 CSS 变量，可以在多个选择器内声明。读取的时候，优先级最高的声明生效。这与 CSS 的"层叠"规则是一致的。

下面是一个例子。
```html
<style>
  :root { --color: blue; }
  div { --color: green; }
  #alert { --color: red; }
  * { color: var(--color); }
</style>

<p>蓝色</p>
<div>绿色</div>
<div id="alert">红色</div>
```
上面代码中，三个选择器都声明了`--color`变量。不同元素读取这个变量的时候，会采用优先级最高的规则，因此三段文字的颜色是不一样的。

这就是说，变量的作用域就是它所在的选择器的有效范围。
```css
body {
  --foo: #7F583F;
}

.content {
  --bar: #F7EFD2;
}
```
上面代码中，变量`--foo`的作用域是`body`选择器的生效范围，`--bar`的作用域是`.content`选择器的生效范围。

由于这个原因，全局的变量通常放在根元素`:root`里面，确保任何选择器都可以读取它们。
```css
:root {
  --main-color: #06c;
}
```
# 响应式布局
CSS 是动态的，页面的任何变化，都会导致采用的规则变化。

利用这个特点，可以在响应式布局的`media`命令里面声明变量，使得不同的屏幕宽度有不同的变量值。
```css
body {
  --primary: #7F583F;
  --secondary: #F7EFD2;
}

a {
  color: var(--primary);
  text-decoration-color: var(--secondary);
}

@media screen and (min-width: 768px) {
  body {
    --primary:  #F7EFD2;
    --secondary: #7F583F;
  }
}
```
# JavaScript 操作
JavaScript 操作 CSS 变量的写法如下。
```js
// 设置变量
document.body.style.setProperty('--primary', '#7F583F');

// 读取变量
document.body.style.getPropertyValue('--primary').trim();
// '#7F583F'

// 删除变量
document.body.style.removeProperty('--primary');
```
这意味着，JavaScript 可以将任意值存入样式表。下面是一个监听事件的例子，事件信息被存入 CSS 变量。
```js
const docStyle = document.documentElement.style;

document.addEventListener('mousemove', (e) => {
  docStyle.setProperty('--mouse-x', e.clientX);
  docStyle.setProperty('--mouse-y', e.clientY);
});
```
那些对 CSS 无用的信息，也可以放入 CSS 变量。
```
--foo: if(x > 5) this.width = 10;
```
上面代码中，`--foo`的值在 CSS 里面是无效语句，但是可以被 JavaScript 读取。这意味着，可以把样式设置写在 CSS 变量中，让 JavaScript 读取。

所以，CSS 变量提供了 JavaScript 与 CSS 通信的一种途径。
