

JavaScript 数组在 TypeScript 里面分成两种类型，分别是数组（`array`）和元组（`tuple`）。
# 数组类型
## 简介
TypeScript 数组有一个根本特征：所有成员的类型必须相同，但是成员数量是不确定的，可以是无限数量的成员，也可以是零成员。

数组的类型有两种写法。第一种写法是在数组成员的类型后面，加上一对方括号。
```ts
let arr:number[] = [1, 2, 3];
```
上面示例中，数组`arr`的类型是`number[]`，其中`number`表示数组成员类型是`number`。

如果数组成员的类型比较复杂，可以写在圆括号里面。
```ts
let arr:(number|string)[];
```
上面示例中，数组`arr`的成员类型是`number|string`。

这个例子里面的圆括号是必须的，否则因为竖杠（`|`）的优先级低于`[]`，TypeScript 会把`number|string[]`理解成`number`和`string[]`的联合类型。

如果数组成员可以是任意类型，写成`any[]`。当然，这种写法是应该避免的。
```ts
let arr:any[];
```
数组类型的第二种写法是使用 TypeScipt 内置的`Array`接口。
```ts
let arr:Array<number> = [1, 2, 3];
```
上面示例中，数组`arr`的类型是`Array<number>`，其中`number`表示成员类型是`number`。

这种写法对于成员类型比较复杂的数组，代码可读性会稍微好一些。
```ts
let arr:Array<number|string>;
```
这种写法本质上属于泛型，这里只要知道怎么写就可以了。

数组类型声明了以后，成员数量是不限制的，任意数量的成员都可以，也可以是空数组。
```ts
let arr:number[];
arr = [];
arr = [1];
arr = [1, 2];
arr = [1, 2, 3];
```
上面示例中，数组`arr`无论有多少个成员，都是正确的。

这种规定的隐藏含义就是，数组的成员是可以动态变化的。
```ts
let arr:number[] = [1, 2, 3];

arr[3] = 4;
arr.length = 2;

arr // [1, 2]
```
正是由于成员数量可以动态变化，所以 TypeScript 不会对数组边界进行检查，越界访问数组并不会报错。
```ts
let arr:number[] = [1, 2, 3];
let foo = arr[3]; // 正确
```
上面示例中，变量`foo`的值是一个不存在的数组成员，TypeScript 并不会报错。

TypeScript 允许使用方括号读取数组成员的类型。
```ts
type Names = string[];
type Name = Names[0]; // string
```
上面示例中，类型`Names`是字符串数组，那么`Names[0]`返回的类型就是`string`。

由于数组成员的索引类型都是`number`，所以读取成员类型也可以写成下面这样。
```ts
type Names = string[];
type Name = Names[number]; // string
```
上面示例中，`Names[number]`表示数组`Names`所有数值索引的成员类型，所以返回`string`。
## 数组的类型推断
如果数组变量没有声明类型，TypeScript 就会推断数组成员的类型。这时，推断行为会因为值的不同，而有所不同。

如果变量的初始值是空数组，那么 TypeScript 会推断数组类型是`any[]`。
```ts
// 推断为 any[]
const arr = [];
```
后面，为这个数组赋值时，TypeScript 会自动更新类型推断。
```ts
// 推断为 any[]
const arr = [];

// 推断类型为 number[]
arr.push(123);

// 推断类型为 (string | number)[]
arr.push('abc');
```
上面示例中，数组变量`arr`的初始值是空数值，然后随着新成员的加入，TypeScript 会自动修改推断的数组类型。

但是，类型推断的自动更新只发生初始值为空数组的情况。如果初始值不是空数组，类型推断就不会更新。
```ts
// 推断类型为 number[]
const arr = [123];

arr.push('abc'); // 报错
```
上面示例中，数组变量`arr`的初始值是`[123]`，TypeScript 就推断成员类型为`number`。新成员如果不是这个类型，TypeScript 就会报错，而不会更新类型推断。
## 只读数组，const 断言
JavaScript 规定，`const`命令声明的数组变量是可以改变成员的。
```ts
const arr = [0, 1];
arr[0] = 2;
```
上面示例中，修改`const`命令声明的数组的成员是允许的。

但是，很多时候确实有声明为只读数组的需求，即不允许变动数组成员。

TypeScript 允许声明只读数组，方法是在数组类型前面加上`readonly`关键字。
```ts
const arr:readonly number[] = [0, 1];

arr[1] = 2; // 报错
arr.push(3); // 报错
delete arr[0]; // 报错
```
上面示例中，`arr`是一个只读数组，删除、修改、新增数组成员都会报错。

TypeScript 将`readonly number[]`与`number[]`视为两种不一样的类型，后者是前者的子类型。

这是因为只读数组没有pop()、push()之类会改变原数组的方法，所以`number[]`的方法数量要多于`readonly number[]`，这意味着`number[]`其实是`readonly number[]`的子类型。

我们知道，子类型继承了父类型的所有特征，并加上了自己的特征，所以子类型`number[]`可以用于所有使用父类型的场合，反过来就不行。
```ts
let a1:number[] = [0, 1];
let a2:readonly number[] = a1; // 正确

a1 = a2; // 报错
```
上面示例中，子类型`number[]`可以赋值给父类型`readonly number[]`，但是反过来就会报错。

由于只读数组是数组的父类型，所以它不能代替数组。这一点很容易产生令人困惑的报错。
```ts
function getSum(s:number[]) {
  // ...
}

const arr:readonly number[] = [1, 2, 3];

getSum(arr) // 报错
```
上面示例中，函数`getSum()`的参数`s`是一个数组，传入只读数组就会报错。原因就是只读数组是数组的父类型，父类型不能替代子类型。这个问题的解决方法是使用类型断言`getSum(arr as number[]`)。

注意，`readonly`关键字不能与数组的泛型写法一起使用。
```ts
// 报错
const arr:readonly Array<number> = [0, 1];
```
上面示例中，`readonly`与数组的泛型写法一起使用，就会报错。

实际上，TypeScript 提供了两个专门的泛型，用来生成只读数组的类型。
```ts
const a1:ReadonlyArray<number> = [0, 1];

const a2:Readonly<number[]> = [0, 1];
```
上面示例中，泛型`ReadonlyArray<T>`和`Readonly<T[]>`都可以用来生成只读数组类型。两者尖括号里面的写法不一样，`Readonly<T[]>`的尖括号里面是整个数组（`number[]`），而`ReadonlyArray<T>`的尖括号里面是数组成员（`number`）。

只读数组还有一种声明方法，就是使用“const 断言”。
```ts
const arr = [0, 1] as const;

arr[0] = [2]; // 报错 
```
上面示例中，as const告诉 TypeScript，推断类型时要把变量arr推断为只读数组，从而使得数组成员无法改变。
## 多维数组
TypeScript 使用T[][]的形式，表示二维数组，T是最底层数组成员的类型。
```ts
var multi:number[][] =
  [[1,2,3], [23,24,25]];
```
上面示例中，变量multi的类型是number[][]，表示它是一个二维数组，最底层的数组成员类型是number。
# 元组类型
## 简介
元组（`tuple`）是 TypeScript 特有的数据类型，JavaScript 没有单独区分这种类型。它表示成员类型可以自由设置的数组，即数组的各个成员的类型可以不同。

元组必须明确声明每个成员的类型。
```
const s:[string, string, boolean]
  = ['a', 'b', true];
```
上面示例中，元组s的前两个成员的类型是string，最后一个成员的类型是boolean。

元组类型的写法，与上一章的数组有一个重大差异。数组的成员类型写在方括号外面（`number[]`），元组的成员类型是写在方括号里面（`[number]`）。

TypeScript 的区分方法是，成员类型写在方括号里面的就是元组，写在外面的就是数组。
```
let a:[number] = [1];
```
上面示例中，变量a是一个元组，只有一个成员，类型是number。

使用元组时，必须明确给出类型声明（上例的[number]），不能省略，否则 TypeScript 会把一个值自动推断为数组。
```
// a 的类型为 (number | boolean)[]
let a = [1, true];
```
上面示例中，变量a的值其实是一个元组，但是 TypeScript 会将其推断为一个联合类型的数组，即a的类型为(number | boolean)[]。

元组成员的类型可以添加问号后缀（?），表示该成员是可选的。
```
let a:[number, number?] = [1];
```
上面示例中，元组a的第二个成员是可选的，可以省略。

注意，问号只能用于元组的尾部成员，也就是说，所有可选成员必须在必选成员之后。
```
type myTuple = [
  number,
  number,
  number?,
  string?
];
```
上面示例中，元组myTuple的最后两个成员是可选的。也就是说，它的成员数量可能有两个、三个和四个。

由于需要声明每个成员的类型，所以大多数情况下，元组的成员数量是有限的，从类型声明就可以明确知道，元组包含多少个成员，越界的成员会报错。
```
let x:[string, string] = ['a', 'b'];

x[2] = 'c'; // 报错
```
上面示例中，变量x是一个只有两个成员的元组，如果对第三个成员赋值就报错了。

但是，使用扩展运算符（...），可以表示不限成员数量的元组。
```
type NamedNums = [
  string,
  ...number[]
];

const a:NamedNums = ['A', 1, 2];
const b:NamedNums = ['B', 1, 2, 3];
```
上面示例中，元组类型NamedNums的第一个成员是字符串，后面的成员使用扩展运算符来展开一个数组，从而实现了不定数量的成员。

扩展运算符用在元组的任意位置都可以，但是它后面只能是数组或元组。
```
type t1 = [string, number, ...boolean[]];
type t2 = [string, ...boolean[], number];
type t3 = [...boolean[], string, number];
```
上面示例中，扩展运算符分别在元组的尾部、中部和头部。

如果不确定元组成员的类型和数量，可以写成下面这样。
```
type Tuple = [...any[]];
```
上面示例中，元组Tuple可以放置任意数量和类型的成员。但是这样写，也就失去了使用元组和 TypeScript 的意义。

元组可以通过方括号，读取成员类型。
```
type Tuple = [string, number];
type Age = Tuple[1]; // number
```
上面示例中，Tuple[1]返回1号位置的成员类型。

由于元组的成员都是数值索引，即索引类型都是number，所以可以像下面这样读取。
```
type Tuple = [string, number, Date];
type TupleEl = Tuple[number];  // string|number|Date
```
上面示例中，Tuple[number]表示元组Tuple的所有数值索引的成员类型，所以返回string|number|Date，即这个类型是三种值的联合类型。
## 只读元组
元组也可以是只读的，不允许修改，有两种写法。
```
// 写法一
type t = readonly [number, string]

// 写法二
type t = Readonly<[number, string]>
```
上面示例中，两种写法都可以得到只读元组，其中写法二是一个泛型，用到了工具类型Readonly<T>。

跟数组一样，只读元组是元组的父类型。所以，元组可以替代只读元组，而只读元组不能替代元组。
```
type t1 = readonly [number, number];
type t2 = [number, number];

let x:t2 = [1, 2];
let y:t1 = x; // 正确

x = y; // 报错
```
上面示例中，类型t1是只读元组，类型t2是普通元组。t2类型可以赋值给t1类型，反过来就会报错。

由于只读元组不能替代元组，所以会产生一些令人困惑的报错。
```
function distanceFromOrigin([x, y]:[number, number]) {
  return Math.sqrt(x**2 + y**2);
}

let point = [3, 4] as const;

distanceFromOrigin(point); // 报错
```
上面示例中，函数distanceFromOrigin()的参数是一个元组，传入只读元组就会报错，因为只读元组不能替代元组。

读者可能注意到了，上例中[3, 4] as const的写法，在上一章讲到，生成的是只读数组，其实生成的同时也是只读元组。因为它生成的实际上是一个只读的“值类型”readonly [3, 4]，把它解读成只读数组或只读元组都可以。

上面示例报错的解决方法，就是使用类型断言，在最后一行将传入的参数断言为普通元组，详见《类型断言》一章。
```
distanceFromOrigin(
  point as [number, number]
)
```
## 成员数量的推断
如果没有可选成员和扩展运算符，TypeScript 会推断出元组的成员数量（即元组长度）。
```
function f(point: [number, number]) {
  if (point.length === 3) {  // 报错
    // ...
  }
}
```
上面示例会报错，原因是 TypeScript 发现元组point的长度是2，不可能等于3，这个判断无意义。

如果包含了可选成员，TypeScript 会推断出可能的成员数量。
```
function f(
  point:[number, number?, number?]
) {
  if (point.length === 4) {  // 报错
    // ...
  }
}
```
上面示例会报错，原因是 TypeScript 发现point.length的类型是1|2|3，不可能等于4。

如果使用了扩展运算符，TypeScript 就无法推断出成员数量。
```
const myTuple:[...string[]]
  = ['a', 'b', 'c'];

if (myTuple.length === 4) { // 正确
  // ...
}
```
上面示例中，myTuple只有三个成员，但是 TypeScript 推断不出它的成员数量，因为它的类型用到了扩展运算符，TypeScript 把myTuple当成数组看待，而数组的成员数量是不确定的。

一旦扩展运算符使得元组的成员数量无法推断，TypeScript 内部就会把该元组当成数组处理。
## 扩展运算符与成员数量
扩展运算符（...）将数组（注意，不是元组）转换成一个逗号分隔的序列，这时 TypeScript 会认为这个序列的成员数量是不确定的，因为数组的成员数量是不确定的。

这导致如果函数调用时，使用扩展运算符传入函数参数，可能发生参数数量与数组长度不匹配的报错。
```
const arr = [1, 2];

function add(x:number, y:number){
  // ...
}

add(...arr) // 报错
```
上面示例会报错，原因是函数add()只能接受两个参数，但是传入的是...arr，TypeScript 认为转换后的参数个数是不确定的。

有些函数可以接受任意数量的参数，这时使用扩展运算符就不会报错。
```
const arr = [1, 2, 3];
console.log(...arr) // 正确
```
上面示例中，console.log()可以接受任意数量的参数，所以传入...arr就不会报错。

解决这个问题的一个方法，就是把成员数量不确定的数组，写成成员数量确定的元组，再使用扩展运算符。
```
const arr:[number, number] = [1, 2];

function add(x:number, y:number){
  // ...
}

add(...arr) // 正确
```
上面示例中，arr是一个拥有两个成员的元组，所以 TypeScript 能够确定...arr可以匹配函数add()的参数数量，就不会报错了。

另一种写法是使用as const断言。
```
const arr = [1, 2] as const;
```
上面这种写法也可以，因为 TypeScript 会认为arr的类型是readonly [1, 2]，这是一个只读的值类型，可以当作数组，也可以当作元组。