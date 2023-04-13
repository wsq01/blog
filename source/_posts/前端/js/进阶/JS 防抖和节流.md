---
title: JS防抖和节流
date: 2019-09-09 19:38:18
tags: JS
categories: 
 - 前端
 - JS
---

事件的触发权很多时候都属于用户，有些情况下会产生问题：
* 向后台发送数据，用户频繁触发，对服务器造成压力
* 一些浏览器事件：`window.onresize`、`window.mousemove`等，触发的频率非常高，会造成浏览器性能问题

如果碰到这些问题，那就需要用到函数节流和防抖了。
# 函数节流(throttle)
函数节流：一个函数执行一次后，只有大于设定的执行周期后才会执行第二次。
有个需要频繁触发函数，出于优化性能角度，在规定时间内，只让函数触发的第一次生效，后面不生效。
## 如何实现
#### 使用时间戳
其原理是用时间戳来判断是否已到回调该执行时间，记录上次执行的时间戳，然后每次触发`scroll`事件执行回调，回调中判断当前时间戳距离上次执行时间戳的间隔是否已经到达 规定时间段，如果是，则执行，并更新上次执行的时间戳，如此循环。
```js
const throttle = (() => {
  // 记录上一次函数触发的时间
  let last = 0
  return (callback, wait = 800) => {
    // 记录当前函数触发的时间
    let now = +new Date()
    if (now - last > wait) {
      callback()
      last = now
    }
  }
})()

throttle(() => { console.log('scroll事件被触发了' + Date.now()) }, 200)
```
#### 使用定时器
当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行，直到定时器执行，然后执行函数，清空定时器，这样就可以设置下个定时器。
```js
function throttle(fn, delay) {
  var timer = null;
  var lastTime = 0;

  return function() {
    if (!timer) {
      timer = setTimeout(() => {
        timer = null;
        fn.apply(this, arguments);
      }, delay)
    }
  }
}
```

![](https://upload-images.jianshu.io/upload_images/3534846-15d3f2fb17430362?imageMogr2/auto-orient/strip)

上例中用到了闭包的特性--可以使变量`lastTime`的值长期保存在内存中。
## 函数节流的应用场景
需要间隔一定时间触发回调来控制函数调用频率：
* DOM 元素的拖拽功能实现（`mousemove`）
* 搜索联想（`keyup`）
* 计算鼠标移动的距离（`mousemove`）
* Canvas 模拟画板功能（`mousemove`）
* 射击游戏的`mousedown/keydown`事件（单位时间只能发射一颗子弹）
* 监听滚动事件判断是否到页面底部自动加载更多：给`scroll`加了`debounce`后，只有用户停止滚动后，才会判断是否到了页面底部；如果是`throttle`的话，只要页面滚动就会间隔一段时间判断一次

# 函数防抖(debounce)
函数防抖：一个需要频繁触发的函数，在规定时间内，只让最后一次生效，前面的不生效。
## 如何实现
其原理就是第一次调用函数，创建一个定时器，在指定的时间间隔之后运行代码。当第二次调用该函数时，它会清除前一次的定时器并设置另一个。如果前一个定时器已经执行过了，这个操作就没有任何意义。然而，如果前一个定时器尚未执行，其实就是将其替换为一个新的定时器，然后延迟一定时间再执行。

```html
<button id='btn'>按钮</button>
```
```js
function bindClick() {
  console.log('点击事件被触发' + Date.now())
}
document.getElementById('btn').onclick = debounce(bindClick, 1000)
```
```js
// 第一版
const debounce = (() => {
  // 记录上一次的延时器
  let timer = null
  return (callback, wait = 800) => {
    // 清除上一次延时器
    timer && clearTimeout(timer)
    timer = setTimeout(callback, wait)
  }
})()

debounce(() => { console.log('加载数据') }, 500)
```
如果我们在`onclick`事件中使用`console.log(this)`，在不使用`debounce`函数的时候，`this`指向`button`按钮。但是如果使用`debounce`函数，`this`就会指向`Window`对象！所以我们需要将`this`指向正确的对象。
```js
// 第二版
function debounce(fn, delay) {
  // 记录上一次的延时器
  var timer = null;
  return () => {
    // 清除上一次延时器
    clearTimeout(timer)
    timer = setTimeout(() => {
      fn.apply(this)
    }, delay)
  }
}
```
JavaScript 在事件处理函数中会提供事件对象`event`，我们修改下`bindClick`函数：
```js
function bindClick(e) {
    console.log(e);
    console.log(this);
};
```
如果我们不使用`debouce`函数，会打印`Event`对象，但是在`debounce`函数中，会打印`undefined`，所以我们需要在`debounce`函数里处理下参数问题。
```js
// 第三版
function debounce(fn, delay) {
  // 记录上一次的延时器
  var timer = null;
  return function() {
    // 清除上一次延时器
    clearTimeout(timer)
    timer = setTimeout(() => {
      fn.apply(this, arguments)
    }, delay)
  }
}
```

[![](https://upload-images.jianshu.io/upload_images/3534846-ff82b940cfa30285?imageMogr2/auto-orient/strip)](https://camo.githubusercontent.com/40b8a595e6b9d4c3bd9e7e13546671792b994c23/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31312f32312f313637333661353432636131373039393f773d36363726683d31363026663d67696626733d3738323031) 

上例中也用到了闭包的特性--可以使变量`timer`的值长期保存在内存中。
## 函数防抖的应用场景
对于连续的事件响应我们只需要执行一次回调：
* 每次`resize/scroll`触发统计事件
* 文本输入的验证（连续输入文字后发送AJAX请求进行验证，验证一次就好）

# 总结
函数节流和函数去抖的核心其实就是限制某一个方法被频繁触发，而一个方法之所以会被频繁触发，大多数情况下是因为DOM事件的监听回调，而这也是函数节流以及防抖多数情况下的应用场景。
