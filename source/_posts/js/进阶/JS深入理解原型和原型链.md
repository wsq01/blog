---
title: JS深入理解原型和原型链
date: 2020-04-13 09:22:50
tags: [javascript, JS进阶]
categories: javascript
---

# 原型
## 原型对象的理解
#### 函数对象的 prototype 属性
我们创建的每一个函数都有一个`prototype`属性，这个属性是一个指针，指向一个对象。这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法，简单来说，该函数实例化的所有对象的`__proto__`的属性指向这个对象，它是该函数所有实例化对象的原型。
```js
function Person() {

}

// 为原型对象添加方法
Person.prototype.sayName = function() {
  alert(this.name);
}
```
{% asset_img img1.png %}
#### constructor 属性
当函数创建，`prototype`属性指向一个原型对象时，在默认情况下，这个原型对象将会获得一个`constructor`属性，这个属性是一个指针，指向`prototype`所在的函数对象。
也就是`Person.prototype.constructor`指向`Person`函数对象。
{% asset_img img2.png %}
#### 对象的`__proto__`属性
当我们调用构造函数创建一个新实例后，在这个实例的内部将包含一个指针，指向构造函数的原型对象，这个指针叫`[[Prototype]]`。所有的对象都含有`[[Prototype]]`属性，指向它们的原型对象。可以通过对象的`__proto__`属性来访问。
```js
var student = new Person();

console.log(student.__proto__ === Person.prototype); // true
```
从上面我们可以看出，这个连接是存在于实例与构造函数的原型对象之间的，而不是存在于实例和构造函数之间的。

{% asset_img img3.png %}

#### prototype和__proto__的区别

{% asset_img 8.png %}

```js
var a = {};
console.log(a.prototype);  //undefined
console.log(a.__proto__);  //Object {}

var b = function(){}
console.log(b.prototype);  //b {}
console.log(b.__proto__);  //function() {}
```
#### __proto__属性指向谁
`__proto__`的指向取决于对象创建时的实现方式。

{% asset_img 9.png %}

```js
/*1、字面量方式*/
var a = {};
console.log(a.__proto__);  //Object {}

console.log(a.__proto__ === a.constructor.prototype); //true

/*2、构造器方式*/
var A = function(){};
var a = new A();
console.log(a.__proto__); //A {}

console.log(a.__proto__ === a.constructor.prototype); //true

/*3、Object.create()方式*/
var a1 = {a:1}
var a2 = Object.create(a1);
console.log(a2.__proto__); //Object {a: 1}

console.log(a2.__proto__ === a1.constructor.prototype); //false
```
## 原型属性
#### 属性访问
每当代码读取对象的某个属性时，首先会在对象本身搜索这个属性，如果找到该属性就返回该属性的值，如果没有找到，则继续搜索该对象对应的原型对象，以此类推下去。
因为这样的搜索过程，因此我们如果在实例中添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性，因为在实例中搜索到该属性后就不会再向后搜索了。
#### 属性判断
在属性确认存在的情况下，我们可以使用`hasOwnProperty()`方法来判断一个属性是存在于实例中，还是存在于原型中。这个方法只有在给定属性存在于实例中时，才会返回`true`。
判断一个属性是否存在，我们可以使用`in`操作符，它会在对象能够访问给定属性时返回`true`，无论该属性存在于实例还是原型中。
```js
function hasPrototypeProperty(object, name){
    return !object.hasOwnProperty(name) && (name in object);
}

function Person() {};

Person.prototype.name = "laker" ;

var student = new Person();

console.log(student.name); // laker
console.log(hasPrototypeProperty(student, "name")); // false


student.name = "xiaoming";
console.log(student.name); //xiaoming 屏蔽了原型对象中的 name 属性
console.log(hasPrototypeProperty(student, "name")); // true
```
#### for-in 循环
在使用`for-in`循环时，返回的是所有能够通过对象访问的、可枚举的属性，其中包括了存在于实例中的属性，也包括了存在于原型中的属性。
需要注意的一点是，屏蔽了实例中不可枚举属性的实例属性也会在`for-in`循环中返回。
#### 所有属性获取
如果想要获得对象上所有可枚举的实例属性，可以使用`Object.keys()`方法，这个方法接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组。
如果想要获取所有的实例属性，无论它是否可以枚举，我们可以使用`Object.getOwnPropertyNames()`方法。
# 原型链
## 原型链理解
原型链是实现继承的主要方法。其基本思想是利用一个引用类型继承另一个引用类型的属性和方法。

原型链的主要实现方法是让构造函数的`prototype`对象等于另一个类型的实例，此时的`prototype`对象因为是实例，因此将包含一个指向另一个原型的指针，相应地另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与类型的链条。这就是原型链的基本概念。
```js
function Super(){};
function Middle(){};
function Sub(){};
Middle.prototype = new Super();
Sub.prototype = new Middle();
var suber = new Sub();
```
{% asset_img img4.png %}
## 默认的原型
所有的引用类型都继承了`Object`，这个继承就是通过原型链来实现的。所有函数的默认原型都是`Object`的实例，因此默认原型都会包含一个内部指针，指向`Object.Prototype`。这也正是所有的自定义类型都会继承`toString()`、`valueOf()`等默认方法的根本原因。
{% asset_img img5.png %}
`Object.prototype`就是原型链的终点了，`Object.prototype.__proto__`返回的是一个`null`空对象，这就意味着原型链的结束。
## 函数也是对象
其实上面的原型链还少了一部分。因为在 JavaScript 中，函数也是对象，那么函数也应该有它的原型对象。
```js
function Foo(who) {
this.me = who;
}

Foo.prototype.identify = function() {
return "I am " + this.me;
};

function Bar(who) {
Foo.call( this, who );
}

Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```
我们先看一下不包含函数原型的以上实例所包含的原型链的关系图：

{% asset_img img6.png %}

下面是加入了函数原型后的原型链的关系图。

{% asset_img img7.png %}

## 示例
```js
var A = function(){};
var a = new A();
console.log(a.__proto__); //A {}（即构造器function A 的原型对象）
console.log(a.__proto__.__proto__); //Object {}（即构造器function Object 的原型对象）
console.log(a.__proto__.__proto__.__proto__); //null
```

{% asset_img 10.png %}
