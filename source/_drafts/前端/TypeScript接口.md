

# 简介
`interface`是对象的模板，可以看作是一种类型约定，中文译为“接口”。使用了某个模板的对象，就拥有了指定的类型结构。
```ts
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}
```
上面示例中，定义了一个接口`Person`，它指定一个对象模板，拥有三个属性`firstName、lastName`和`age`。任何实现这个接口的对象，都必须部署这三个属性，并且必须符合规定的类型。

实现该接口很简单，只要指定它作为对象的类型即可。
```ts
const p:Person = {
  firstName: 'John',
  lastName: 'Smith',
  age: 25
};
```
上面示例中，变量`p`的类型就是接口`Person`，所以必须符合`Person`指定的结构。

方括号运算符可以取出`interface`某个属性的类型。
```ts
interface Foo {
  a: string;
}

type A = Foo['a']; // string
```
上面示例中，`Foo['a']`返回属性`a`的类型，所以类型`A`就是`string`。

`interface`可以表示对象的各种语法，它的成员有5种形式：对象属性、对象的属性索引、对象方法、函数、构造函数。

## 对象属性
```ts
interface Point {
  x: number;
  y: number;
}
```
上面示例中，`x`和`y`都是对象的属性，分别使用冒号指定每个属性的类型。

属性之间使用分号或逗号分隔，最后一个属性结尾的分号或逗号可以省略。

如果属性是可选的，就在属性名后面加一个问号。
```ts
interface Foo {
  x?: string;
}
```
如果属性是只读的，需要加上`readonly`修饰符。
```ts
interface A {
  readonly a: string;
}
```
## 对象的属性索引
```ts
interface A {
  [prop: string]: number;
}
```
上面示例中，`[prop: string]`就是属性的字符串索引，表示属性名只要是字符串，都符合类型要求。

属性索引共有`string、number`和`symbol`三种类型。

一个接口中，最多只能定义一个字符串索引。字符串索引会约束该类型中所有名字为字符串的属性。
```ts
interface MyObj {
  [prop: string]: number;

  a: boolean;      // 编译错误
}
```
上面示例中，属性索引指定所有名称为字符串的属性，它们的属性值必须是数值（`number`）。属性a的值为布尔值就报错了。

属性的数值索引，其实是指定数组的类型。
```ts
interface A {
  [prop: number]: string;
}

const obj:A = ['a', 'b', 'c'];
```
上面示例中，`[prop: number]`表示属性名的类型是数值，所以可以用数组对变量obj赋值。

同样的，一个接口中最多只能定义一个数值索引。数值索引会约束所有名称为数值的属性。

如果一个`interface`同时定义了字符串索引和数值索引，那么数值索性必须服从于字符串索引。因为在 JavaScript 中，数值属性名最终是自动转换成字符串属性名。
```ts
interface A {
  [prop: string]: number;
  [prop: number]: string; // 报错
}

interface B {
  [prop: string]: number;
  [prop: number]: number; // 正确
}
```
上面示例中，数值索引的属性值类型与字符串索引不一致，就会报错。数值索引必须兼容字符串索引的类型声明。
## 对象的方法
对象的方法共有三种写法。
```ts
// 写法一
interface A {
  f(x: boolean): string;
}

// 写法二
interface B {
  f: (x: boolean) => string;
}

// 写法三
interface C {
  f: { (x: boolean): string };
}
```
属性名可以采用表达式，所以下面的写法也是可以的。
```ts
const f = 'f';

interface A {
  [f](x: boolean): string;
}
```
类型方法可以重载。
```ts
interface A {
  f(): number;
  f(x: boolean): boolean;
  f(x: string, y: string): string;
}
```
`interface`里面的函数重载，不需要给出实现。但是，由于对象内部定义方法时，无法使用函数重载的语法，所以需要额外在对象外部给出函数方法的实现。
```ts
interface A {
  f(): number;
  f(x: boolean): boolean;
  f(x: string, y: string): string;
}

function MyFunc(): number;
function MyFunc(x: boolean): boolean;
function MyFunc(x: string, y: string): string;
function MyFunc(
  x?:boolean|string, y?:string
):number|boolean|string {
  if (x === undefined && y === undefined) return 1;
  if (typeof x === 'boolean' && y === undefined) return true;
  if (typeof x === 'string' && typeof y === 'string') return 'hello';
  throw new Error('wrong parameters');  
}

const a:A = {
  f: MyFunc
}
```
上面示例中，接口A的方法f()有函数重载，需要额外定义一个函数MyFunc()实现这个重载，然后部署接口A的对象a的属性f等于函数MyFunc()就可以了。

## 函数

`interface`也可以用来声明独立的函数。
```ts
interface Add {
  (x:number, y:number): number;
}

const myAdd:Add = (x,y) => x + y;
```
上面示例中，接口Add声明了一个函数类型。

## 构造函数
`interface`内部可以使用`new`关键字，表示构造函数。
```ts
interface ErrorConstructor {
  new (message?: string): Error;
}
```
上面示例中，接口`ErrorConstructor`内部有`new`命令，表示它是一个构造函数。
