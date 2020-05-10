---
title: JS深拷贝和浅拷贝
date: 2019-07-15 17:08:16
tags: [javascript, JS进阶]
categories: javascript
---


# 数据类型
数据分为基本数据类型(`String`，`Number`，`Boolean`，`Null`，`Undefined`，`Symbol`)和对象数据类型。
* 基本数据类型的特点：直接存储在栈(`stack`)中的数据
* 引用数据类型的特点：**存储的是该对象在栈中引用，真实的数据存放在堆内存里**

引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

![](http://upload-images.jianshu.io/upload_images/3534846-b71d714f367d14c7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 浅拷贝与深拷贝
深拷贝和浅拷贝是只针对`Object`和`Array`这样的引用数据类型的。
深拷贝和浅拷贝的示意图大致如下：

![](http://upload-images.jianshu.io/upload_images/3534846-162d55fd3baeaa8f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

浅拷贝只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。
# 赋值和浅拷贝的区别
当我们把一个对象赋值给一个新的变量时，**赋的其实是该对象的在栈中的地址，而不是堆中的数据**。也就是两个对象指向的是同一个存储空间，无论哪个对象发生改变，其实都是改变的存储空间的内容，因此，两个对象是联动的。

浅拷贝是按位拷贝对象，**它会创建一个新对象**，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。即默认拷贝构造函数只是对对象进行浅拷贝复制(逐个成员依次拷贝)，即只复制对象空间而不复制资源。

我们先来看两个例子，对比赋值与浅拷贝会对原对象带来哪些改变。
```js
// 对象赋值
var obj1 = {    
  'name' : 'zhangsan',    
  'age' :  '18',    
  'language' : [1,[2,3],[4,5]],
};
var obj2 = obj1;
obj2.name = "lisi";
obj2.language[1] = ["二","三"];
console.log('obj1', obj1)
console.log('obj2', obj2)
```

![](http://upload-images.jianshu.io/upload_images/3534846-a36719bccb28b4ea?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```js
// 浅拷贝
var obj1 = {    
  'name' : 'zhangsan',
  'age' :  '18',    
  'language' : [1,[2,3],[4,5]],
};
var obj3 = shallowCopy(obj1);
obj3.name = "lisi";
obj3.language[1] = ["二","三"]; 
function shallowCopy(src) {    
  var dst = {};  
  for (var prop in src) {        
    if (src.hasOwnProperty(prop)) {
      dst[prop] = src[prop];
    }
  }    
  return dst;
}
console.log('obj1',obj1)
console.log('obj3',obj3)
```

![](http://upload-images.jianshu.io/upload_images/3534846-3ad34a6707bd2483?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面例子中，`obj1`是原始数据，`obj2`是赋值操作得到，而`obj3`浅拷贝得到。我们可以很清晰看到对原始数据的影响，具体请看下表：

![](http://upload-images.jianshu.io/upload_images/3534846-4797068d8de133aa?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 浅拷贝的实现方式
## 1、Object.assign()
`Object.assign()`方法可以把任意多个的源对象自身的可枚举属性拷贝给目标对象，然后返回目标对象。但是`Object.assign()`进行的是浅拷贝，拷贝的是对象的属性的引用，而不是对象本身。
```js
var obj = { a: {a: "kobe", b: 39} };
var initalObj = Object.assign({}, obj);
initalObj.a.a = "wade";
console.log(obj.a.a); //wade
```
注意：当`object`只有一层的时候，是深拷贝。
```js
let obj = {username: 'kobe'};
let obj2 = Object.assign({}, obj);
obj2.username = 'wade';
console.log(obj); //{username: "kobe"}
```
## 2、Array.prototype.concat()
```js
let arr = [1, 3, {    
  username: 'kobe'
}];
let arr2 = arr.concat();    
arr2[2].username = 'wade';
console.log(arr);
```
修改新对象会改到原对象：

![](http://upload-images.jianshu.io/upload_images/3534846-8065f72d7ab4eec9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3、Array.prototype.slice()
```js
let arr = [1, 3, {
  username: ' kobe'
}];
let arr3 = arr.slice();
arr3[2].username = 'wade'
console.log(arr);
```
同样修改新对象会改到原对象：

![](http://upload-images.jianshu.io/upload_images/3534846-c5930a7f1b762e6b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于Array的`slice`和`concat`方法的补充说明：Array的`slice`和`concat`方法不修改原数组，只会返回一个浅复制了原数组中的元素的一个新数组。
原数组的元素会按照下述规则拷贝：
*   如果该元素是个对象引用(不是实际的对象)，`slice`会拷贝这个对象引用到新的数组里。两个对象引用都引用了同一个对象。如果被引用的对象发生改变，则新的和原来的数组中的这个元素也会发生改变。
*   对于字符串、数字及布尔值来说（不是`String`、`Number`或者`Boolean`对象），`slice`会拷贝这些值到新的数组里。在别的数组里修改这些字符串或数字或是布尔值，将不会影响另一个数组。

```js
let arr = [1, 3, {
  username: ' kobe'
}];
let arr3 = arr.slice();
arr3[1] = 2;
console.log(arr, arr3);
```

![](http://upload-images.jianshu.io/upload_images/3534846-3163efd685bda110?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、深拷贝的实现方式
### 1、JSON.parse(JSON.stringify())
```js
let arr = [1, 3, {
    username: ' kobe'
}];
let arr4 = JSON.parse(JSON.stringify(arr));
arr4[2].username = 'duncan'; 
console.log(arr, arr4);
```

![](http://upload-images.jianshu.io/upload_images/3534846-87acfa35bcffe5d8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原理： 用`JSON.stringify`将对象转成JSON字符串，再用`JSON.parse()`把字符串解析成对象，一去一来，新的对象产生了，而且对象会开辟新的栈，实现深拷贝。

这种方法虽然可以实现数组或对象深拷贝，但不能处理函数。
```js
let arr = [1, 3, {
  username: ' kobe'
}, function(){}];

let arr4 = JSON.parse(JSON.stringify(arr));
arr4[2].username = 'duncan'; 
console.log(arr, arr4);
```

![](http://upload-images.jianshu.io/upload_images/3534846-c808f47a5e63a2fa?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是因为`JSON.stringify()`方法是将一个JavaScript值(对象或者数组)转换为一个JSON字符串，不能接受函数。
## 2、手写递归方法
递归方法实现深度克隆原理：遍历对象、数组直到里边都是基本数据类型，然后再去复制，就是深度拷贝。
```js
//简便方法
function deepClone( obj ) {
  let target = Array.isArray( obj ) ? [] : {}
  for ( var k in obj ) {
    typeof obj[ k ] === 'object' ? target[ k ] = deepClone(obj[ k ]) : target[ k ] = obj[ k ]
  }
  return target
}
```
```js
const deepClone = obj => {
  if (obj === null) return null;
  let clone = Object.assign({}, obj);
  Object.keys(clone).forEach(key => {
    clone[key] = typeof obj[key] === 'object' ? deepClone(obj[key]) : obj[key]
  });
  return Array.isArray(obj) && obj.length ? (clone.length = obj.length) && Array.from(clone) : Array.isArray(obj) ? Array.from(obj) : clone;
};
```
