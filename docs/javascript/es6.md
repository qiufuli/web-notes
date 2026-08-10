# ES6 知识点

# <font color="#e96900">let 和 const</font>

> es6 新增两个声明变量的方式 let 和 const ，它的用法类似于 var，但有本质上的不同

## [let 和 const 的相同点](#list1)

1. `let/const` 有自己的**块级作用域**
2. `let/const` 在**相同作用域**下不能重复**声明**（使用 var let const 任意一种重复声明都报错）
3. `let/const` **不存在变量声明提升**，声明必须在调用之前

## [let 和 const 的不同点](#list2)

1. `const`声明必须有初始值` const a =1`; `let`声明可以没有初始值 `let a`;
2. `const`声明后不能重新**赋值** `const a = 1; a=2` 报错，`let `声明后可以重新赋值 `let a = 1; a= 2` 正确;

（**当前作用域以及子作用域中**`const`都不能重复**赋值**）

（**当前作用域以及子作用域中** `let`都可以重新**赋值** 并且子作用域中赋值后也会作用到`let`声明的变量上）

<font color="#ff0000">重新赋值 和 重新声明 是两回事哈！！！</font>

## [let 和 const 声明不在 window 上](#list3)

> var 和 function 声明的全局变量 可以直接用 window.xxx 访问，但 let、const、class 声明的全局变量 window.xxx 访问不到（undefined）<br>也就是说，从 es6 开始，全局变量将逐步与顶层对象（window）的属性脱钩

---

<div id="list1"></div>

<font style="color:#34495e;font-size:1.25rem;font-weight:600">let 和 const 的相同点</font>

<font style="background-color:#ffec45">1、let/const 有自己的**块级作用域**</font>

```js
{
  let a = 10
  var b = 10
}
console.log(b) // 10
console.log(a) //a is not defined
```

上面代码在代码块之中，分别用`let`和`var`声明了两个变量。然后在代码块之外调用这两个变量，结果`let`声明的变量报错，`var`声明的变量返回了正确的值。这表明，`let`声明的变量只在它所在的代码块有效。

`for`循环的计数器，就很合适使用`let`命令。类似闭包

```js
var a = []
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i)
  }
}
a[6]() // 10

//=============================================
var a = []
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i)
  }
}
a[6]() // 6
```

上面第一段代码中，变量`i`是`var`命令声明的，在全局范围内都有效，所以全局只有一个变量`i`。每一次循环，变量`i`的值都会发生改变，而循环内被赋给数组`a`的函数内部的`console.log(i)`，里面的`i`指向的就是全局的`i`。也就是说，所有数组`a`的成员里面的`i`，指向的都是同一个 i，导致运行时输出的是最后一轮的`i`的值，也就是 `10`。

上面第二段代码中，变量`i`是`let`声明的，当前的`i`只在本轮循环有效，所以每一次循环的`i`其实都是一个新的变量，所以最后输出的是`6`。这种方式我们也常说成**块级作用域**。你可能会问，如果每一轮循环的变量`i`都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 `JavaScript` 引擎内部会记住上一轮循环的值，初始化本轮的变量`i`时，就在上一轮循环的基础上进行计算

`for`循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```js
for (let i = 0; i < 3; i++) {
  let i = 'abc'
  console.log(i)
}
// abc
// abc
// abc
```

上面代码正确运行，输出了 3 次 abc。这表明函数内部的变量`i`与循环变量`i`不在同一个作用域，有各自单独的作用域。

---

<font style="background-color:#ffec45">2、let/const 在**相同作用域**下不能重复**声明**（使用 var let const 任意一种重复声明都报错）</font>

```js
// 报错
function func() {
  let a = 10
  var a = 1
}

// 报错
function func() {
  let a = 10
  let a = 1
}
```

因此，不能在函数内部重新声明参数。

```js
function func(arg) {
  let arg
}
func() // 报错

function func(arg) {
  {
    let arg
  }
}
func() // 不报错
```

---

<font style="background-color:#ffec45">3、let/const **不存在变量声明提升**，声明必须在调用之前</font>

```js
// var 的情况
console.log(foo) // 输出undefined
var foo = 2

// let 的情况
console.log(bar) // 报错ReferenceError
let bar = 2
```

上面代码中，变量`foo`用`var`命令声明，会发生变量提升，即脚本开始运行时，变量`foo`已经存在了，但是没有值，所以会输出`undefined`。变量`bar`用`let`命令声明，不会发生变量提升。这表示在声明它之前，变量`bar`是不存在的，这时如果用到它，就会抛出一个错误。

---

**暂时性死区**

只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

```js
var tmp = 123

if (true) {
  tmp = 'abc' // ReferenceError
  let tmp
}
```

上面代码中，存在全局变量`tmp`，但是块级作用域内`let`又声明了一个局部变量`tmp`，导致后者绑定这个块级作用域，所以在`let`声明变量前，对`tmp`赋值会报错。

`ES6` 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

```js
if (true) {
  // TDZ开始
  tmp = 'abc' // ReferenceError
  console.log(tmp) // ReferenceError

  let tmp // TDZ结束
  console.log(tmp) // undefined

  tmp = 123
  console.log(tmp) // 123
}
```

---

<div id="list2"></div>

<font style="color:#34495e;font-size:1.25rem;font-weight:600">let 和 const 的不同点</font>

<font style="background-color:#ffec45">1、const 声明必须有初始值 let 声明可以没有初始值; </font>

```js
const foo; // Missing initializer in const declaration

let a ; //正确
```

<font style="background-color:#ffec45">2、const 声明后不能重新**赋值** ，let 声明后可以重新**赋值**;</font>

**当前作用域以及子作用域中**`const`都不能重复**赋值**

```js
const a = 1
a = 2 //报错

const b = 2
{
  b = 3 // 报错
}
```

**当前作用域以及子作用域中** `let`都可以重新**赋值** 并且子作用域中赋值后也会作用到`let`声明的变量上

```js
let a = 1
a = 2 //正确

let b = 2
{
  b = 3 //正确
}
```

> 所以！ 一般我们用`const`声明项目中不变的常量， `let`声明项目中常用的变量

---

<div id="list3"></div>

<font style="color:#34495e;font-size:1.25rem;font-weight:600">let 和 const 声明不在 window 上</font>

`var` 和`function`声明的全局变量 可以直接用`window.xxx`访问，但`let、const、class`声明的全局变量 `window.xxx `访问不到`（undefined）`<br>也就是说，从`es6`开始，全局变量将逐步与顶层对象（`window`）的属性脱钩

```js
const a = 1
let b = 2
var c = 3
function d() {
  console.log('d')
}

console.log(window.a) //undefined
console.log(window.b) //undefined
console.log(window.c) // 3
console.log(window.d) // function d(){ console.log('d');}
```

# <font color="#e96900">变量的解构赋值</font>

## 1.数组的解构赋值

> ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

### 完全解构

```
let [a, b, c] = [1, 2, 3];


let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

上面代码表示，可以从数组中提取值，按照对应位置，对变量赋值。

**如果解构不成功，变量的值就等于`undefined`。**

### 不完全解构

```js
// 不完全解构
let [x, y] = [1, 2, 3]
console.log(x, y)
let [g, [h], j] = [1, [2, 3], 4]
console.log(g, h, j)
```

如果等号的右边不是数组(或者严格地说，不是可遍历的结构),那么将会报错。

```
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```

对于 Set 结构，也可以使用数组的解构赋值。

```
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
```

### 默认值

解构赋值允许指定默认值。

```
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

注意，ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以，只有当一个数组成员严格等于 undefined，默认值才会生效(`null`的话就不是全等`undefined` ,值为`undefined`等同于没有 ,默认值生效，值为有效或者为`null` ,取给定的值)。

```
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```

上面代码中，如果一个数组成员是`null`，默认值就不会生效，因为`null`不严格等于`undefined`。

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

```
function f() {
  console.log('aaa');
}

let [x = f()] = [1];
```

上面代码中，因为 x 能取到值，所以函数`f`根本不会执行。上面的代码其实等价于下面的代码。

```
let x;
if ([1][0] === undefined) {
  x = f();
} else {
  x = [1][0];
}
```

---

## 2.对象解构赋值

> 解构不仅可以用于数组，还可以用于对象。

### 同名解构

```
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"
```

**对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。**

```
let { bar, foo } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: 'aaa', bar: 'bbb' };
baz // undefined
```

上面代码的第一个例子，等号左边的两个变量的次序，与等号右边两个同名属性的次序不一致，但是对取值完全没有影响。第二个例子的变量没有对应的同名属性，导致取不到值，最后等于`undefined`。

如果解构失败，变量的值等于`undefined`。

```
let {foo} = {bar: 'baz'};
foo // undefined
```

上面代码中，等号右边的对象没有`foo`属性，所以变量 foo 取不到值，所以等于`undefined`。

```
// 例一
let { log, sin, cos } = Math;

// 例二
const { log } = console;
log('hello') // hello
```

上面代码的例一将`Math`对象的对数、正弦、余弦三个方法，赋值到对应的变量上，使用起来就会方便很多。例二将`console.log`赋值到`log`变量。

---

### 不同名解构

如果变量名与属性名不一致，必须写成下面这样。

```
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

这实际上说明，对象的解构赋值是下面形式的简写

```
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };
```

也就是说，对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。**真正被赋值的是后者，而不是前者。**

```
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"
foo // error: foo is not defined
```

上面代码中，`foo`是匹配的模式，`baz`才是变量。真正被赋值的是变量`baz`，而不是模式`foo`。

---

### 嵌套解构

与数组一样，解构也可以用于嵌套结构的对象。

```
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"
```

注意，这时 p 是模式，不是变量，因此不会被赋值。如果 p 也要作为变量赋值，可以写成下面这样。

```
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p, p: [x, { y }] } = obj;
x // "Hello"
y // "World"
p // ["Hello", {y: "World"}]
```

下面是另一个例子。

```
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
line // 1
loc  // Object {start: Object}
start // Object {line: 1, column: 5}
```

上面代码有三次解构赋值，分别是对`loc、start、line`三个属性的解构赋值。注意，最后一次对`line`属性的解构赋值之中，只有`line`是变量，`loc`和`start`都是模式，不是变量。

下面是嵌套赋值的例子。

```
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

obj // {prop:123}
arr // [true]
```

如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错。

```
// 报错
let {foo: {bar}} = {baz: 'baz'};
```

上面代码中，等号左边对象的 foo 属性，对应一个子对象。该子对象的 bar 属性，解构时会报错。原因很简单，因为 foo 这时等于 undefined，再取子属性就会报错。

注意，对象的解构赋值可以取到继承的属性。

```
const obj1 = {};
const obj2 = { foo: 'bar' };
Object.setPrototypeOf(obj1, obj2);

const { foo } = obj1;
foo // "bar"
```

上面代码中，对象`obj1`的原型对象是`obj2`。`foo`属性不是`obj1`自身的属性，而是继承自`obj2`的属性，解构赋值可以取到这个属性。

---

### 默认值

对象的解构也可以指定默认值。

```
var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x: y = 3} = {};
y // 3

var {x: y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```

默认值生效的条件是，对象的属性值严格等于 undefined。

```
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};
x // null
```

上面代码中，属性 x 等于 null，因为 null 与 undefined 不严格相等，所以是个有效的赋值，导致默认值 3 不会生效。

---

### 注意点

如果要将一个已经声明的变量用于解构赋值，必须非常小心。

```
// 错误的写法
let x;
{x} = {x: 1};
// SyntaxError: syntax error
```

上面代码的写法会报错，因为 JavaScript 引擎会将`{x}`理解成一个代码块，从而发生语法错误**。只有不将大括号写在行首，避免 JavaScript 将其解释为代码块，才能解决这个问题**。

```
// 正确的写法
let x;
({x} = {x: 1});
```

上面代码将整个解构赋值语句，放在一个圆括号里面，就可以正确执行。

解构赋值允许等号左边的模式之中，不放置任何变量名。因此，可以写出非常古怪的赋值表达式。

```
({} = [true, false]);
({} = 'abc');
({} = []);
```

上面的表达式虽然毫无意义，但是语法是合法的，可以执行。

由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构

```
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

上面代码对数组进行对象解构。数组 arr 的 0 键对应的值是 1，[arr.length - 1]就是 2 键，对应的值是 3。方括号这种写法，属于“属性名表达式”。

---

## 3.字符串解构赋值

字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

```
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

类似数组的对象都有一个 length 属性，因此还可以对这个属性解构赋值。

```
let {length : len} = 'hello';
len // 5
```

---

## 4.函数参数的解构赋值

函数的参数也可以使用解构赋值。

```
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

上面代码中，函数`add`的参数表面上是一个数组，但在传入参数的那一刻，数组参数就被解构成变量`x`和`y`。对于函数内部的代码来说，它们能感受到的参数就是`x`和`y`。

下面是另一个例子。

```
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

函数参数的解构也可以使用默认值。

```
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

上面代码中，函数`move`的参数是一个对象，通过对这个对象进行解构，得到变量`x`和`y`的值。如果解构失败，`x`和`y`等于默认值。

注意，下面的写法会得到不一样的结果。

```
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

上面代码是为函数`move`的参数指定默认值，而不是为变量`x`和`y`指定默认值，所以会得到与前一种写法不同的结果。

`undefined`就会触发函数参数的默认值。

```
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```

---

## 5.圆括号问题

解构赋值虽然很方便，但是解析起来并不容易。对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。

由此带来的问题是，如果模式中出现圆括号怎么处理。ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

但是，这条规则实际上不那么容易辨别，处理起来相当麻烦。因此，建议只要有可能，就不要在模式中放置圆括号。

### 不能使用圆括号的情况

以下三种解构赋值不得使用圆括号。
（1）变量声明语句

```
// 全部报错
let [(a)] = [1];

let {x: (c)} = {};
let ({x: c}) = {};
let {(x: c)} = {};
let {(x): c} = {};

let { o: ({ p: p }) } = { o: { p: 2 } };
```

上面 6 个语句都会报错，因为它们都是变量声明语句，模式不能使用圆括号。

（2）函数参数
函数参数也属于变量声明，因此不能带有圆括号。

```
// 报错
function f([(z)]) { return z; }
// 报错
function f([z,(x)]) { return x; }
```

（3）赋值语句的模式

```
// 全部报错
({ p: a }) = { p: 42 };
([a]) = [5];
```

上面代码将整个模式放在圆括号之中，导致报错。

```
// 报错
[({ p: a }), { x: c }] = [{}, {}];
```

上面代码将一部分模式放在圆括号之中，导致报错。

### 可以使用圆括号的情况

可以使用圆括号的情况只有一种：赋值语句的非模式部分，可以使用圆括号。

```
let b;
[(b)] = [3]; // 正确


({ p: (d) } = {}); // 正确
[(parseInt.prop)] = [3]; // 正确
```

上面三行语句都可以正确执行，因为首先它们**都是赋值语句，而不是声明语句**；其次它们的圆括号都不属于模式的一部分。第一行语句中，模式是取数组的第一个成员，跟圆括号无关；第二行语句中，模式是 p，而不是 d；第三行语句与第一行语句的性质一致。

---

## 6.用途

（1）交换变量的值

```
let x = 1;
let y = 2;

[x, y] = [y, x];
```

上面代码交换变量 x 和 y 的值，这样的写法不仅简洁，而且易读，语义非常清晰。

（2）从函数返回多个值

函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。

```
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

（3）函数参数的定义

解构赋值可以方便地将一组参数与变量名对应起来。

```
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

（4）提取 JSON 数据

解构赋值对提取 JSON 对象中的数据，尤其有用。

```
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

（5）函数参数的默认值

```
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。

（6）遍历 Map 结构

任何部署了 Iterator 接口的对象，都可以用 for...of 循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

```
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想获取键名，或者只想获取键值，可以写成下面这样。

```
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

（7）输入模块的指定方法
加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

```
const { SourceMapConsumer, SourceNode } = require("source-map");
```

---

# <font color="#e96900">模板字符串</font>

> 模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，**也可以用来定义多行字符串，或者在字符串中嵌入变量（${xxxx}）。**

```
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

上面代码中的模板字符串，都是用反引号表示。如果在模板字符串中需要使用反引号，则前面要用**反斜杠转义**。

```
let greeting = `\`Yo\` World!`;
```

大括号内部可以放入任意的 JavaScript 表达式，可以进行运算，以及引用对象属性。

```
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```

模板字符串之中还能调用函数。

```
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```

如果大括号中的值不是字符串，将按照一般的规则转为字符串。比如，大括号中是一个对象，将默认调用对象的`toString`方法。

**如果模板字符串中的变量没有声明，将报错。**

```
// 变量place没有声明
let msg = `Hello, ${place}`;
// 报错
```

由于模板字符串的大括号内部，就是执行 JavaScript 代码，因此如果大括号内部是一个字符串，将会原样输出。

```
`Hello ${'World'}`
// "Hello World"
```

模板字符串甚至还能嵌套。

```
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;
```

如果需要引用模板字符串本身，在需要时执行，可以写成函数。

```
let func = (name) => `Hello ${name}!`;
func('Jack') // "Hello Jack!"
```

上面代码中，模板字符串写成了一个函数的返回值。执行这个函数，就相当于执行这个模板字符串了。

---

# <font color="#e96900">扩展运算符</font>

> 扩展运算符（spread）是三个点（`...`），将一个数组转为用逗号分隔的参数序列。

```
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
```

该运算符主要用于函数调用。

```
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

const numbers = [4, 38];
add(...numbers) // 42
```

上面代码中，`array.push(...items)和add(...numbers)`这两行，都是函数的调用，它们的都使用了扩展运算符。该运算符将一个数组，变为参数序列。

扩展运算符与正常的函数参数可以结合使用，非常灵活。

```
function f(v, w, x, y, z) { }
const args = [0, 1];
f(-1, ...args, 2, ...[3]);
```

扩展运算符后面还可以放置表达式。

```
const arr = [
  ...(x > 0 ? ['a'] : []),
  'b',
];
```

如果扩展运算符后面是一个空数组，则不产生任何效果。

```
[...[], 1]
// [1]
```

注意，扩展运算符如果放在括号中，JavaScript 引擎就会认为这是函数调用。**如果这时不是函数调用，就会报错。**

```
(...[1, 2])
// Uncaught SyntaxError: Unexpected number

console.log((...[1, 2]))
// Uncaught SyntaxError: Unexpected number

console.log(...[1, 2])
// 1 2
```

上面前两种情况都会报错，因为扩展运算符所在的括号不是函数调用，而第三种情况`console.log(...[1, 2])`就不会报错，因为这时是函数调用。

## 替代函数的 apply 方法

由于扩展运算符可以展开数组，所以不再需要`apply`方法，将数组转为函数的参数了。

```
// ES5 的写法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f.apply(null, args);

// ES6的写法
function f(x, y, z) {
  // ...
}
let args = [0, 1, 2];
f(...args);
```

下面是扩展运算符取代`apply`方法的一个实际的例子，应用`Math.max`方法，简化求出一个数组最大元素的写法。

```
// ES5 的写法
Math.max.apply(null, [14, 3, 77])

// ES6 的写法
Math.max(...[14, 3, 77])

// 等同于
Math.max(14, 3, 77);
```

上面代码中，由于 JavaScript 不提供求数组最大元素的函数，所以只能套用`Math.max`函数，将数组转为一个参数序列，然后求最大值。有了扩展运算符以后，就可以直接用`Math.max`了。

另一个例子是通过 push 函数，将一个数组添加到另一个数组的尾部。

```
// ES5的 写法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);

// ES6 的写法
let arr1 = [0, 1, 2];
let arr2 = [3, 4, 5];
arr1.push(...arr2);
```

上面代码的 ES5 写法中，push 方法的参数不能是数组，所以只好通过`apply`方法变通使用`push`方法。有了扩展运算符，就可以直接将数组传入`push`方法。

下面是另外一个例子。

```
// ES5
new (Date.bind.apply(Date, [null, 2015, 1, 1]))
// ES6
new Date(...[2015, 1, 1]);
```

## 扩展运算符的应用

（1）复制数组

数组是复合的数据类型，直接复制的话，只是复制了指向底层数据结构的指针，而不是克隆一个全新的数组。

```
const a1 = [1, 2];
const a2 = a1;

a2[0] = 2;
a1 // [2, 2]
```

上面代码中，`a2`并不是`a1`的克隆，而是指向同一份数据的另一个指针。修改`a2`，会直接导致`a1`的变化。

ES5 只能用变通方法来复制数组。

```
const a1 = [1, 2];
const a2 = a1.concat();

a2[0] = 2;
a1 // [1, 2]
```

上面代码中，`a1`会返回原数组的克隆，再修改`a2`就不会对`a1`产生影响。

扩展运算符提供了复制数组的简便写法。

```
const a1 = [1, 2];
// 写法一
const a2 = [...a1];
// 写法二
const [...a2] = a1;
```

上面的两种写法，`a2`都是`a1`的克隆。

（2）合并数组

扩展运算符提供了数组合并的新写法。

```
const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];

// ES5 的合并数组
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6 的合并数组
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]
```

不过，这两种方法都是浅拷贝，使用的时候需要注意。

```
const a1 = [{ foo: 1 }];
const a2 = [{ bar: 2 }];

const a3 = a1.concat(a2);
const a4 = [...a1, ...a2];

a3[0] === a1[0] // true
a4[0] === a1[0] // true
```

上面代码中，`a3`和`a4`是用两种不同方法合并而成的新数组，但是它们的成员都是对原数组成员的引用，这就是浅拷贝。如果修改了原数组的成员，会同步反映到新数组。

（3）与解构赋值结合

扩展运算符可以与解构赋值结合起来，用于生成数组。

```
// ES5
a = list[0], rest = list.slice(1)
// ES6
[a, ...rest] = list
```

下面是另外一些例子。

```
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []
```

如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。

```
const [...butLast, last] = [1, 2, 3, 4, 5];
// 报错

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 报错
```

（4）字符串

扩展运算符还可以将字符串转为真正的数组。

```
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

总结

1. 扩展运算符将一个数组转为用逗号分隔的参数序列，所以克隆数组的时候是把参数赋值给新数组，修改新数组对原有数组不产生影响。
2. 如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。

---

# <font color="#e96900">数组实例方法</font>

> 本文涵盖 ES5，ES6 中常用的数组实例，并不单纯的是 ES6 语法

## forEach

`forEach()` 方法指定数组的每项元素都执行一次传入的函数，返回值为`undefined`。

> 语法：arr.forEach(fn, thisArg);

> fn 表示在数组每一项上执行的函数 function(value, index, array)，接受三个参数：
> value 当前正在被处理的元素的值

    index 当前元素的数组索引
    array 数组本身

> thisArg 可选，用来当做 fn 函数内的 this 对象。

`forEach` 将为数组中每一项执行一次 `fn `函数，那些已删除，新增或者从未赋值的项将被跳过（但不包括值为 `undefined` 的项）。

```
var array = [1, 3,undefined,null, 5];
var obj = {name:'cc'};
var sReturn = array.forEach(function(value, index, array){
  array[index] = value * value;
  console.log(this.name); // cc被打印了三次
},obj);
console.log(array); // [1, 9, NaN, 0, 25], 可见原数组改变了  !!!!!!!!!!!!!!!!!!!
console.log(sReturn); // undefined, 可见返回值为undefined   !!!!!!!!!!!!!!!!!!!!!!!!!!
```

注意：`forEach`无法直接退出循环，只能使用`return` 来达到`for`循环中`continue`的效果，并且`forEach`不能在低版本 IE（6~8）中使用

---

## map-映射

`map()` 方法遍历数组，使用传入函数处理每个元素，并返回函数的返回值组成的**新数组**(可理解为将数组的每个元素**映射**成指定的形式，新数组的长度与原数组相同)。

> 语法：arr.map(fn, thisArg) fn 和 thisArg 同 forEach()

```
var arr = [1,2,3,4];
var newArr = arr.map((value,index,array) => {
    return value*10 //新数组为10,20,30,40
})
console.log('newArr',newArr); // [10, 20, 30, 40]
//map遍历数组，返回一个新数组，不改变原数组的值。
```

---

## filter-过滤

`filter()` 方法使用传入的函数测试所有元素，并返回所有通过测试的元素组成的**新数组**。它就好比一个过滤器，筛掉不符合条件的元素。

> 语法：arr.filter(fn, thisArg)

```
var arr = [1,2,3,4];
arr.filter((value,index,array) => {
    return value > 2 //新数组为[3,4]
})
//filter过滤掉数组中不满足条件的值，返回一个新数组，不改变原数组的值。
```

---

## find-返回第一个对象

数组实例的`find`方法，用于找出**第一个符合条件的数组成员**。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回`undefined`。

> 语法：arr.find(fn,thisArg)

```
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```

上面代码中，`find`方法的回调函数可以接受三个参数，依次为当前的值、当前的位置和原数组。

---

## findIndex-返回第一个对象索引值

`findIndex`方法的用法与`find`方法非常类似，返回**第一个符合条件的数组成员的位置**，如果所有成员都不符合条件，则返回`-1`。

```
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

---

## reduce-累加

`reduce()` 方法接收一个方法作为累加器，数组中的每个值(从左至右) 开始合并，最终为**一个值**。

> 语法：arr.reduce(fn, initialValue)

> fn 表示在数组每一项上执行的函数，接受四个参数：
> previousValue 上一次调用回调返回的值，或者是提供的初始值
> value 数组中当前被处理元素的值
> index 当前元素在数组中的索引
> array 数组自身

```
var arr = [1,2,3,4];
arr.reduce((previousValue,value,index,array) => {
    console.log(previousValue)  // 1  3  6  previousValue为上次一计算的结果
    console.log(value)  // 2  3  4
    console.log(index)  // 1  2  3
    return previousValue+value  //最终结果为10
})
//reduce 让数组的前后两项进行某种计算。然后返回其值，并继续计算。
//不改变原数组，返回计算的最终结果，从数组的第二项开始遍历。
```

---

## every--全满足

方法的结果为 `false`；如果全部返回 `true`，那么该方法的结果才为 `true`。

```
var arr = [1,2,3,4];
arr.every((value,index,array) => {
    return value> 1 //结果为false
})
//遍历数组每一项，每一项返回true,则最终结果为true。
//当任何一项返回false时，停止遍历，返回false。不改变原数组
```

---

## some--一个满足

`some()` 方法刚好同 `every()` 方法相反，`some` 测试数组元素时，只要有一个函数返回值为 `true`，则该方法返回 `true`，若全部返回 `false`，则该方法返回 `false`。

`some`方法与`includes`方法有着异曲同工之妙，他们都是探测数组中是否拥有满足条件的元素，一旦找到，便返回`true`。

```
var arr = [1,2,3,4];
arr.some((value,index,array) => {
    return value> 3 //结果为true
})
//遍历数组每一项，只要有一个函数返回值为 true，结果返回true。
//若全部返回 false，则该方法返回 false。
//不改变原数组
```

---

## includes

`Array.prototype.includes`方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的`includes`方法类似。ES2016 引入了该方法。

```
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
```

该方法的第二个参数表示搜索的起始位置，默认为`0`。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为 3），则会重置为从`0`开始。

---

## 数组的空位

数组的空位指，数组的某一个位置没有任何值。比如，`Array`构造函数返回的数组都是空位。

```
Array(3) // [, , ,]
```

注意，空位不是`undefined`，一个位置的值等于`undefined`，依然是有值的。空位是没有任何值。

ES5 对空位的处理，已经很不一致了，大多数情况下会忽略空位。

`forEach(), filter(), reduce(), every() 和some()`都会跳过空位。
`map()`会跳过空位，但会保留这个值
`join()和toString()`会将空位视为`undefined`，而`undefined`和`null`会被处理成空字符串。

```
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```

ES6 则是明确将空位转为`undefined`。

扩展运算符（...）也会将空位转为`undefined`。

```
[...['a',,'b']]
// [ "a", undefined, "b" ]
```

`entries()、keys()、values()、find()和findIndex()`会将空位处理成`undefined`。

```
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

由于空位的处理规则非常不统一，所以建议避免出现空位。

---

# <font color="#e96900">箭头函数</font>

> ES6 允许使用“箭头”（`=>`）定义函数。

```
var f = v => v;

// 等同于
var f = function (v) {
  return v;
};
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```
var f = () => 5;
// 等同于
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用`return`语句返回。

```
var sum = (num1, num2) => { return num1 + num2; }
```

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。

```
// 报错
let getTempItem = id => { id: id, name: "Temp" };

// 不报错
let getTempItem = id => ({ id: id, name: "Temp" });
```

下面是一种特殊情况，虽然可以运行，但会得到错误的结果。

```
let foo = () => { a: 1 };
foo() // undefined
```

上面代码中，原始意图是返回一个对象`{ a: 1 }`，但是由于引擎认为大括号是代码块，所以执行了一行语句`a: 1`。这时，`a`可以被解释为语句的标签，因此实际执行的语句是`1`;，然后函数就结束了，没有返回值。

箭头函数可以与变量解构结合使用。

```
const full = ({ first, last }) => first + ' ' + last;

// 等同于
function full(person) {
  return person.first + ' ' + person.last;
}
```

箭头函数使得表达更加简洁。

```
const isEven = n => n % 2 === 0;
const square = n => n * n;
```

上面代码只用了两行，就定义了两个简单的工具函数。如果不用箭头函数，可能就要占用多行，而且还不如现在这样写醒目。

箭头函数的一个用处是简化回调函数。

```
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);
```

```
// 正常函数写法
var result = values.sort(function (a, b) {
  return a - b;
});

// 箭头函数写法
var result = values.sort((a, b) => a - b);
```

```
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
```

## 使用注意点

箭头函数有几个使用注意点。

（1）函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。

（2）`不可以当作构造函数`，也就是说，不可以使用`new`命令，否则会抛出一个错误。

（3）不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用 rest(扩展运算符) 参数代替。

（4）不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

上面四点中，第一点尤其值得注意。`this`对象的指向是可变的，但是在箭头函数中，它是固定的。

```
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42
```

上面代码中，`setTimeout`的参数是一个箭头函数，这个箭头函数的定义生效是在 foo 函数生成时，而它的真正执行要等到 100 毫秒后。如果是普通函数，执行时`this`应该指向全局对象`window`，这时应该输出`21`。但是，箭头函数导致 this 总是指向函数定义生效时所在的对象（本例是`{id: 42}`），所以输出的是`42`。

箭头函数可以让`setTimeout`里面的`this`，**绑定定义时所在的作用域，而不是指向运行时所在的作用域**。

```
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数
  setInterval(() => this.s1++, 1000);
  // 普通函数
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
```

上面代码中，`Timer`函数内部设置了两个定时器，分别使用了箭头函数和普通函数。前者的 this 绑定定义时所在的作用域（即`Timer`函数），后者的`this`指向运行时所在的作用域（即全局对象）。所以，3100 毫秒之后，`timer.s1`被更新了 3 次，而`timer.s2`一次都没更新。

箭头函数可以让`this`指向固定化，这种特性很有利于封装回调函数。下面是一个例子，DOM 事件的回调函数封装在一个对象里面。

```
var handler = {
  id: '123456',

  init: function() {
    document.addEventListener('click',
      event => this.doSomething(event.type), false);
  },

  doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
};
```

上面代码的`init`方法中，使用了箭头函数，这导致这个箭头函数里面的`this`，总是指向`handler`对象。否则，回调函数运行时，`this.doSomething`这一行会报错，因为此时`this`指向`document`对象。

`this`指向的固定化，并不是因为箭头函数内部有绑定`this`的机制，实际原因是箭头函数根本没有自己的`this`，导致内部的`this`就是外层代码块的`this`。正是因为它没有`this`，所以也就不能用作构造函数。

所以，箭头函数转成 ES5 的代码如下。

```
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```

上面代码中，转换后的 ES5 版本清楚地说明了，箭头函数里面根本没有自己的`this`，而是引用外层的`this`。

```
function foo() {
  return () => {
    return () => {
      return () => {
        console.log('id:', this.id);
      };
    };
  };
}

var f = foo.call({id: 1});

var t1 = f.call({id: 2})()(); // id: 1
var t2 = f().call({id: 3})(); // id: 1
var t3 = f()().call({id: 4}); // id: 1
```

上面代码之中，只有一个`this`，就是函数`foo`的`this`，所以`t1、t2、t3`都输出同样的结果。因为所有的内层函数都是箭头函数，都没有自己的`this`，它们的`this`其实都是最外层`foo`函数的`this`。

除了`this`，以下三个变量在箭头函数之中也是不存在的，指向外层函数的对应变量：`arguments、super、new.target`。

```
function foo() {
  setTimeout(() => {
    console.log('args:', arguments);
  }, 100);
}

foo(2, 4, 6, 8)
// args: [2, 4, 6, 8]
```

上面代码中，箭头函数内部的变量`arguments`，其实是函数`foo`的`arguments`变量。

另外，由于箭头函数没有自己的`this`，所以当然也就不能用`call()、apply()、bind()`这些方法去改变 this 的指向。

```
(function() {
  return [
    (() => this.x).bind({ x: 'inner' })()
  ];
}).call({ x: 'outer' });
// ['outer']
```

上面代码中，箭头函数没有自己的`this`，所以`bind`方法无效，内部的`this`指向外部的`this`。

长期以来，JavaScript 语言的 this 对象一直是一个令人头痛的问题，在对象方法中使用`this`，必须非常小心。箭头函数”绑定”`this`，很大程度上解决了这个困扰。

## 不适用场合

由于箭头函数使得`this`从“动态”变成“静态”，下面两个场合不应该使用箭头函数。

第一个场合是定义对象的方法，且该方法内部包括`this`。

```
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  }
}
```

上面代码中，`cat.jumps()`方法是一个箭头函数，这是错误的。调用`cat.jumps()`时，如果是普通函数，该方法内部的`this`指向`cat`；如果写成上面那样的箭头函数，使得`this`指向全局对象，因此不会得到预期结果。**这是因为对象不构成单独的作用域**，导致`jumps`箭头函数定义时的作用域就是全局作用域。

第二个场合是需要动态`this`的时候，也不应使用箭头函数。

```
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on');
});
```

上面代码运行时，点击按钮会报错，因为`button`的监听函数是一个箭头函数，导致里面的`this`就是全局对象。如果改成普通函数，`this`就会动态指向被点击的按钮对象。

另外，如果函数体很复杂，有许多行，或者函数内部有大量的读写操作，不单纯是为了计算值，这时也不应该使用箭头函数，而是要使用普通函数，这样可以提高代码可读性。

---

# <font color="#e96900">Promise 对象</font>

## Promise 含义

`Promise` 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了`Promise`对象。

所谓`Promise`，简单**说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果**。从语法上说，`Promise` 是一个对象，**从它可以获取异步操作的消息**。`Promise` 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

`Promise`对象有以下两个特点。
（1）对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 `resolved`（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（`Event`）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

有了`Promise`对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，`Promise`对象提供统一的接口，使得控制异步操作更加容易。

`Promise`也有一些缺点。首先，无法取消`Promise`，**一旦新建它就会立即执行，无法中途取消**。其次**，如果不设置回调函数，`Promise`内部抛出的错误，不会反应到外部**。第三，当**处于`pending`状态时，无法得知目前进展到哪一个阶段**（刚刚开始还是即将完成）。

---

## 基本用法

ES6 规定，`Promise`对象是一个构造函数，用来生成`Promise`实例。

```
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

`Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 `pending` 变为 `resolved`），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；

`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从 `pending` 变为 `rejected`），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态变为`rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受`Promise`对象传出的值作为参数。

```
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
})
```

上面代码中，`timeout`方法返回一个`Promise`实例，表示一段时间以后才会发生的结果。过了指定的时间（ms 参数）以后，`Promise`实例的状态变为`resolved`，就会触发`then`方法绑定的回调函数。

**`Promise` 新建后就会立即执行。**

```
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```

上面代码中，`Promise` 新建后立即执行，所以首先输出的是`Promise`。然后，**`then`方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行**，所以`resolved`最后输出。

下面是异步加载图片的例子。

```
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    const image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
```

下面是一个用`Promise`对象实现的 `Ajax` 操作的例子。

```
const getJSON = function(url) {
  const promise = new Promise(function(resolve, reject){
    const handler = function() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

上面代码中，`getJSON`是对 XMLHttpRequest 对象的封装，用于发出一个针对 JSON 数据的 HTTP 请求，并且返回一个`Promise`对象。需要注意的是，在 getJSON 内部，`resolve`函数和`reject`函数调用时，都带有参数。

如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给回调函数。`reject`函数的参数通常是 Error 对象的实例，表示抛出的错误；`resolve`函数的参数除了正常的值以外，还可能是另一个 `Promise` 实例，比如像下面这样。

```
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```

上面代码中，`p1`和`p2`都是 `Promise` 的实例，但是`p2`的`resolve`方法将`p1`作为参数，即一个异步操作的结果是返回另一个异步操作。

注意，这时`p1`的状态就会传递给`p2`，也就是说，`p1`的状态决定了`p2`的状态。如果`p1`的状态是`pending`，那么`p2`的回调函数就会等待`p1`的状态改变；如果`p1`的状态已经是`resolved`或者`rejected`，那么`p2`的回调函数将会立刻执行。

```
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```

上面代码中，`p1`是一个 `Promise`，3 秒之后变为`rejected`。`p2`的状态在 1 秒之后改变，`resolve`方法返回的是`p1`。由于`p2`返回的是另一个 `Promise`，导致`p2`自己的状态无效了，由`p1`的状态决定`p2`的状态。所以，后面的`then`语句都变成针对后者（`p1`）。又过了 2 秒，`p1`变为`rejected`，导致触发`catch`方法指定的回调函数。

注意，调用`resolve`或`reject`并不会终结 `Promise` 的参数函数的执行。

```
new Promise((resolve, reject) => {
  resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
// 2
// 1
```

上面代码中，调用`resolve(1)`以后，后面的`console.log(2)`还是会执行，并且会首先打印出来。这是因为立即 `resolved` 的 `Promise` 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。

一般来说，调用`resolve`或`reject`以后，`Promise` 的使命就完成了，后继操作应该放到`then`方法里面，而不应该直接写在`resolve`或`reject`的后面。所以，最好在它们前面加上`return`语句，这样就不会有意外。

```
new Promise((resolve, reject) => {
  return resolve(1);
  // 后面的语句不会执行
  console.log(2);
})
```

---

## Promise.prototype.then

### 链式调用

Promise 实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为 `Promise` 实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数。

**`then`方法返回(需要写`return` 否则无法链式调用)的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。**

```
getJSON("/posts.json").then(function(json) {
  return json.post;  ======> return Promise.resolve(json.post)
}).then(function(post) {
  // ...
});
```

上面的代码使用`then`方法，依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。

---

## Promise.prototype.catch()

`Promise.prototype.catch`方法是`.then(null, rejection)`或.`then(undefined, rejection)`的别名，**用于指定发生错误时的回调函数**。

```
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```

上面代码中，`getJSON`方法返回一个 Promise 对象，如果该对象状态变为`resolved`，则会调用`then`方法指定的回调函数；如果异步操作抛出错误，状态就会变为`rejected`，就会调用`catch`方法指定的回调函数，处理这个错误。另外，`then`方法指定的回调函数，如果运行中抛出错误，也会被`catch`方法捕获。

```
p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同于
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
```

Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个 catch 语句捕获。

```
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```

上面代码中，一共有三个 Promise 对象：一个由`getJSON`产生，两个由`then`产生。它们之中任何一个抛出的错误，都会被最后一个 catch 捕获。

一般来说，不要在`then`方法里面定义 Reject 状态的回调函数（即 then 的第二个参数），总是使用`catch`方法。

```
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

---

## Promise.prototype.finally()

`finally`方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。该方法是 `ES2018` 引入标准的。

```
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

上面代码中，不管`promise`最后的状态，在执行完`then`或`catch`指定的回调函数以后，都会执行`finally`方法指定的回调函数。

`finally`方法的回调函数不接受任何参数，这意味着没有办法知道，前面的 `Promise` 状态到底是`fulfilled`还是`rejected`。这表明，`finally`方法里面的操作，应该是与状态无关的，不依赖于 `Promise` 的执行结果。

---

## Promise.all()

**谁跑的慢，以谁为准执行回调 (接收的是数组，返回的也是数组)**

`Promise.all`方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

```
const p = Promise.all([p1, p2, p3]);
```

上面代码中，`Promise.all`方法接受一个数组作为参数，`p1、p2、p3`都是 Promise 实例，如果不是，就会先调用下面讲到的`Promise.resolve`方法，将参数转为 Promise 实例，再进一步处理。（`Promise.all`方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。）

`p`的状态由`p1、p2、p3`决定，分成两种情况。

（1）只有`p1、p2、p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfille`d，此时`p1、p2、p3`的返回值组成一个数组，传递给`p`的回调函数。

（2）只要`p1、p2、p3`之中**有一个被`rejected`**，`p`的状态就变成`rejected`，**此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。**

```
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

上面代码中，`promises`是包含 6 个 Promise 实例的数组，只有这 6 个实例的状态都变成`fulfilled`，或者其中有一个变为`rejected`，才会调用 Promise.all 方法后面的回调函数。

---

## Promise.race()

**谁跑的快，以谁为准执行回调 (接收的是数组，返回的是最快一个的结果)**

race 的使用场景：比如我们可以用 race 给某个异步请求设置超时时间，并且在超时后执行相应的操作，代码如下：

```js
//请求某个图片资源
function requestImg() {
  var p = new Promise((resolve, reject) => {
    var img = new Image()
    img.onload = function () {
      resolve(img)
    }
    img.src = '图片的路径'
  })
  return p
}
//延时函数，用于给请求计时
function timeout() {
  var p = new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('图片请求超时')
    }, 5000)
  })
  return p
}
Promise.race([requestImg(), timeout()])
  .then((data) => {
    console.log(data)
  })
  .catch((err) => {
    console.log(err)
  })
```

我们把这两个返回 Promise 对象的函数放进 race，于是他俩就会赛跑，如果 5 秒之内图片请求成功了，那么遍进入 then 方法，执行正常的流程。如果 5 秒钟图片还未成功返回，那么 timeout 就跑赢了，则进入 catch，报出“图片请求超时”的信息。

## Promise.resolve()

**有时需要将现有对象转为 `Promise` 对象**，`Promise.resolve`方法就起到这个作用。

```
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

上面代码将 jQuery 生成的`deferred`对象，转为一个新的 `Promise` 对象。

`Promise.resolve`等价于下面的写法。

```
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

`Promise.resolve`方法的参数分成四种情况。

（1）参数是一个 Promise 实例

如果参数是 Promise 实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。

（2）参数是一个 thenable 对象
`thenable`对象指的是具有`then`方法的对象，比如下面这个对象。

```
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
```

`Promise.resolve`方法会将这个对象转为 `Promise` 对象，然后就立即执行`thenable`对象的 then 方法。

```
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```

上面代码中，`thenable`对象的`then`方法执行后，对象`p1`的状态就变为`resolved`，从而立即执行最后那个`then`方法指定的回调函数，输出 42。

（3）参数不是具有`then`方法的对象，或根本就不是对象

如果参数是一个原始值，或者是一个不具有`then`方法的对象，则`Promise.resolve`方法返回一个新的 `Promise` 对象，状态为`resolved`。

```
const p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```

上面代码生成一个新的 `Promise` 对象的实例`p`。由于字符串`Hello`不属于异步操作（判断方法是字符串对象不具有 `then` 方法），返回 `Promise` 实例的状态从一生成就是`resolved`，所以回调函数会立即执行。`Promise.resolve`方法的参数，会同时传给回调函数。

（4）不带有任何参数
`Promise.resolve()`方法允许调用时不带参数，直接返回一个`resolved`状态的 `Promise` 对象。

所以，如果希望得到一个 `Promise` 对象，比较方便的方法就是直接调用`Promise.resolve()`方法。

```
const p = Promise.resolve();

p.then(function () {
  // ...
});
```

上面代码的变量`p`就是一个 `Promise` 对象。

需要注意的是，立即`resolve()`的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。

```
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
```

上面代码中，`setTimeout(fn, 0)`在下一轮“事件循环”开始时执行，`Promise.resolve()`在本轮“事件循环”结束时执行，`console.log('one')`则是立即执行，因此最先输出。

---

## Promise.reject()

`Promise.reject(reason)`方法也会返回一个新的 Promise 实例，该实例的状态为`rejected`。

```
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```

上面代码生成一个 Promise 对象的实例`p`，状态为`rejected`，回调函数会立即执行。

注意，`Promise.reject()`方法的参数，会原封不动地作为`reject`的理由，变成后续方法的参数。这一点与`Promise.resolve`方法不一致。

```
const thenable = {
  then(resolve, reject) {
    reject('出错了');
  }
};

Promise.reject(thenable)
.catch(e => {
  console.log(e === thenable)
})
// true
```

上面代码中，`Promise.reject`方法的参数是一个`thenable`对象，执行以后，后面`catch`方法的参数不是`reject`抛出的“出错了”这个字符串，而是`thenable`对象。

---

# <font color="#e96900">async 和 await</font>

## async

`async` 函数算是一个语法糖,使异步函数、回调函数在语法上看上去更像同步函数。`async`函数返回一个 `Promise` 对象，可以使用`then`方法添加回调函数。当函数执行的时候，一旦遇到`await`就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。

```
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

函数前面的`async`关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个`Promise`对象。

```
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

上面代码指定 50 毫秒以后，输出 hello world。

由于`async`函数返回的是 `Promise` 对象，可以作为`await`命令的参数。所以，上面的例子也可以写成下面的形式。

```
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

`async` 函数有多种使用形式。

```
// 函数声明
async function foo() {}

// 函数表达式
const foo = async function () {};

// 对象的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭头函数
const foo = async () => {};
```

### 语法

`async`函数的语法规则总体上比较简单，难点是错误处理机制。

`async`函数返回一个 Promise 对象。

`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。

```
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```

上面代码中，函数`f`内部`return`命令返回的值，会被`then`方法回调函数接收到。

`async`函数内部抛出错误，会导致返回的 `Promise` 对象变为`reject`状态。抛出的错误对象会被`catch`方法回调函数接收到。

```
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
```

`async`函数返回的 `Promise` 对象，必须等到内部所有`await`命令后面的 `Promise` 对象执行完，才会发生状态改变，除非遇到`return`语句或者**抛出错误**。也就是说，只有`async`函数内部的**异步操作**执行完，才会执行`then`方法指定的回调函数。

```
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

上面代码中，函数`getTitle`内部有三个操作：抓取网页、取出文本、匹配页面标题。只有这三个操作全部完成，才会执行`then`方法里面的`console.log`。

---

## await

正常情况下，`await`命令后面是一个 `Promise` 对象，返回该对象的结果。如果不是 `Promise` 对象，就直接**返回对应的值**。

```
async function f() {
  // 等同于
  // return 123;
  return await 123;
}

f().then(v => console.log(v))
// 123
```

`await`命令后面的 `Promise` 对象如果变为`reject`状态，则`reject`的参数会被`catch`方法的回调函数接收到。

```
async function f() {
  await Promise.reject('出错了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出错了
```

注意，上面代码中，`await`语句前面没有`return`，但是`reject`方法的参数依然传入了`catch`方法的回调函数。这里如果在`await`前面加上`return`，效果是一样的。

任何一个`await`语句后面的 `Promise` 对象变为`reject`状态，那么整个`async`函数都会中断执行。

```
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
```

上面代码中，第二个`await`语句是不会执行的，因为第一个`await`语句状态变成了`reject`。

有时，我们希望即使前一个异步操作失败，也不要中断后面的异步操作。这时可以将第一个`await`放在`try...catch`结构里面，这样不管这个异步操作是否成功，第二个`await`都会执行。

```
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world
```

另一种方法是`await`后面的 `Promise` 对象再跟一个`catch`方法，处理前面可能出现的错误。

```
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出错了
// hello world
```

## 使用注意点

第一点，前面已经说过，`await`命令后面的`Promise`对象，运行结果可能是`rejected`，所以最好把`await`命令放在`try...catch`代码块中。

```
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

第二点，多个`await`命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。

```
let foo = await getFoo();
let bar = await getBar();
```

上面代码中，`getFoo`和`getBar`是两个独立的异步操作（即互不依赖），被写成继发关系。这样比较耗时，因为只有`getFoo`完成以后，才会执行`getBar`，完全可以让它们同时触发。

```
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

上面两种写法，`getFoo`和`getBar`都是同时触发，这样就会缩短程序的执行时间。

---

# <font color="#e96900">class 类</font>

## class

JavaScript 语言中，生成实例对象的传统方法是通过构造函数。

```
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);
```

上面这种写法跟传统的面向对象语言（比如 C++ 和 Java）差异很大，很容易让新学习这门语言的程序员感到困惑。

ES6 提供了更接近传统语言的写法，引入了 `Class`（类）这个概念，作为对象的模板。通过 class 关键字，可以定义类。

基本上，ES6 的`class`可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的`class`写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用 ES6 的`class`改写，就是下面这样。

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

上面代码定义了一个“类”，可以看到里面有一个`constructor`方法，这就是构造方法，而`this`关键字则代表实例对象。也就是说，ES5 的构造函数`Point`，对应 ES6 的`Point`类的构造方法。

Point 类除了构造方法，还定义了一个`toString`方法。注意，定义“类”的方法的时候，前面不需要加上`function`这个关键字，直接把函数定义放进去了就可以了。另外，**方法之间不需要逗号分隔，加了会报错。**

使用的时候，也是直接对类使用 new 命令，跟构造函数的用法完全一致。

```
class Bar {
  doStuff() {
    console.log('stuff');
  }
}

var b = new Bar();
b.doStuff() // "stuff"
```

构造函数的`prototype`属性，在 ES6 的“类”上面继续存在。事实上，**类的所有方法都定义在类的`prototype`属性上面。**

```
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```

在类的实例上面调用方法，其实就是调用原型上的方法。

```
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true
```

上面代码中，`b`是`B`类的实例，它的`constructor`方法就是`B`类原型的`constructor`方法。

由于类的方法都定义在`prototype`对象上面，所以类的新方法可以添加在`prototype`对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。

```
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```

`prototype`对象的`constructor`属性，直接指向“类”的本身，这与 ES5 的行为是一致的。

```
Point.prototype.constructor === Point // true
```

**另外，类的内部所有定义的方法，都是不可枚举的（non-enumerable）。**

```
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代码中，`toString`方法是`Point`类内部定义的方法，它是不可枚举的。这一点与 ES5 的行为不一致。

```
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面代码中，`toString`方法是`Point`类内部定义的方法，它是不可枚举的。这一点与 ES5 的行为不一致。

```
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

---

### constructor 方法

`constructor`方法是类的默认方法，通过`new`命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个空的`constructor`方法会被默认添加。

```
class Point {
}

// 等同于
class Point {
  constructor() {}
}
```

`constructor`方法默认返回实例对象（即 this），完全可以指定返回另外一个对象。

```
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

上面代码中，`constructor`函数返回一个全新的对象，结果导致实例对象不是`Foo`类的实例。

类必须使用`new`调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用`new`也可以执行。

```
class Foo {
  constructor() {
    return Object.create(null);
  }
}

Foo()
// TypeError: Class constructor Foo cannot be invoked without 'new'
```

### 类的实例

生成类的实例的写法，与 ES5 完全一样，也是使用`new`命令。前面说过，如果忘记加上`new`，像函数那样调用`Class`，将会报错。

```
class Point {
  // ...
}

// 报错
var point = Point(2, 3);

// 正确
var point = new Point(2, 3);
```

与 ES5 一样，实例的属性除非显式定义在其本身（即定义在`this`对象上），否则都是定义在原型上（即定义在`class`上）。

```
//定义类
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

上面代码中，`x`和`y`都是实例对象`point`自身的属性（因为定义在`this`变量上），所以`hasOwnProperty`方法返回`true`，而`toString`是原型对象的属性（因为定义在`Point`类上），所以`hasOwnProperty`方法返回`false`。这些都与 ES5 的行为保持一致。

与 ES5 一样，类的所有实例共享一个原型对象。

```
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
//true
```

---

## 继承

`Class` 可以通过`extends`关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

```
class Point {
}

class ColorPoint extends Point {
}
```

上面代码定义了一个`ColorPoint`类，该类通过`extends`关键字，继承了`Point`类的所有属性和方法。但是由于没有部署任何代码，所以这两个类完全一样，等于复制了一个`Point`类。下面，我们在`ColorPoint`内部加上代码。

```
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```

上面代码中，`constructor`方法和`toString`方法之中，都出现了`super`关键字，它在这里表示父类的构造函数，用来新建父类的`this`对象。

子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错。这是因为子类自己的`this`对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用`super`方法，子类就得不到`this`对象。

```
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```

上面代码中，`ColorPoint`继承了父类`Point`，但是它的构造函数没有调用`super`方法，导致新建实例时报错。

> ES5 的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面（`Parent.apply(this)`）。

> ES6 的继承机制完全不同，实质是先将父类实例对象的属性和方法，加到`this`上面（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。

如果子类没有定义`constructor`方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有`constructor`方法。

```
class ColorPoint extends Point {
}

// 等同于
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}
```

另一个需要注意的地方是，在子类的构造函数中，只有调用`super`之后，才可以使用`this`关键字，否则会报错。这是因为子类实例的构建，基于父类实例，只有`super`方法才能调用父类实例。

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正确
  }
}
```

上面代码中，子类的`constructor`方法没有调用`super`之前，就使用`this`关键字，结果报错，而放在`super`方法之后就是正确的。

下面是生成子类实例的代码。

```
let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true
```

上面代码中，实例对象 cp 同时是 ColorPoint 和 Point 两个类的实例，这与 ES5 的行为完全一致。

最后，父类的静态方法，也会被子类继承。

```
class A {
  static hello() {
    console.log('hello world');
  }
}

class B extends A {
}

B.hello()  // hello world
```

上面代码中，`hello()`是`A`类的静态方法，`B`继承`A`，也继承了`A`的静态方法。

### super 关键字

`super`这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。

#### super 作为函数使用---super 作为函数调用时，代表父类的构造函数,但 super 中的 this 指向子类

第一种情况，`super`作为函数调用时，代表父类的构造函数。ES6 要求，子类的构造函数必须执行一次`super`函数。

```
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

上面代码中，子类 B 的构造函数之中的`super()`，代表调用父类的构造函数。这是必须的，否则 JavaScript 引擎会报错。

> 注意，`super`虽然代表了父类 A 的构造函数，但是返回的是子类 B 的实例，即`super`内部的`this`指的是 B 的实例，因此`super()`在这里相当于`A.prototype.constructor.call(this)`。 比如：调用父类的 this.name 其实打印的是子类中的 this.name

作为函数时，`super()`只能用在子类的构造函数之中，用在其他地方就会报错。

```
class A {}

class B extends A {
  m() {
    super(); // 报错
  }
}
```

上面代码中，`super()`用在`B`类的`m`方法之中，就会造成句法错误。

#### super 作为对象使用---在普通方法中，指向父类的原型对象；在静态方法中，指向父类

第二种情况，`super`作为对象时**，在普通方法中，指向父类的原型对象(A.prototype 不能访问实例属性)；在静态方法中，指向父类。**

```
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

上面代码中，子类`B`当中的`super.p()`，就是将`super`当作一个对象使用。这时，`super`在普通方法之中，指向`A.prototype`，所以`super.p()`就相当于`A.prototype.p()`。

这里需要注意，由于`super`指向**父类的原型对象**，所以定义在**父类实例上的方法或属性**，是无法通过`super`调用的。

```
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

上面代码中，`p`是父类 A 实例的属性，`super.p`就引用不到它。

如果属性定义在父类的原型对象上，`super`就可以取到。

```
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

ES6 规定，在子类普通方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类实例。

```
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();
  }
}

let b = new B();
b.m() // 2
```

上面代码中，`super.print()`虽然调用的是`A.prototype.print()`，但是`A.prototype.print()`内部的`this`指向子类`B`的实例，导致输出的是 2，而不是 1。也就是说，实际上执行的是`super.print.call(this)`。

由于`this`指向子类实例，所以如果通过`super`对某个属性赋值，这时`super`就是`this`，赋值的属性会变成子类实例的属性。

```
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

上面代码中，`super.x`赋值为 3，这时等同于对`this.x`赋值为 3。而当读取`super.x`的时候，读的是`A.prototype.x`，所以返回`undefined`。

如果`super`作为对象，用在静态方法之中，这时`super`将指向父类，而不是父类的原型对象。

```
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

上面代码中，`super`在静态方法之中指向父类，在普通方法之中指向父类的原型对象。

另外，在子类的静态方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类，而不是子类的实例。

```
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```

上面代码中，静态方法 B.m 里面，`super.print`指向父类的静态方法。这个方法里面的 this 指向的是 B，而不是 B 的实例。

---

## 总结

1. `class`类的方法都是定义在`prototype`对象上的。
2. 类的内部所有定义的方法，都是不可枚举的，ES5 的写法是可以枚举的。
3. 类必须使用`new`调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用`new`也可以执行。
4. `super`这个关键字，既可以当作函数使用，也可以当作对象使用。
5. 使用`super`的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。
6. 由于`super`指向**父类的原型对象**，所以定义在**父类实例上的方法或属性**(constructor 中)，是无法通过`super`调用的。
7. `super`在静态方法之中指向**父类**，在普通方法之中指向**父类的原型对象**。
8. ES6 规定，在子类普通方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类实例。

---

# <font color="#e96900">Proxy</font>

`Proxy` 可以理解成，**在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截**，因此提供了一种机制，**可以对外界的访问进行过滤和改写**。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

```js
var obj = new Proxy(
  {},
  {
    get: function (target, key, receiver) {
      console.log(`getting ${key}!`)
      return Reflect.get(target, key, receiver)
    },
    set: function (target, key, value, receiver) {
      console.log(`setting ${key}!`)
      return Reflect.set(target, key, value, receiver)
    },
  }
)
```

上面代码对一个空对象架设了一层拦截，重定义了属性的读取（`get`）和设置（`set`）行为。这里暂时先不解释具体的语法，只看运行结果。对设置了拦截行为的对象`obj`，去读写它的属性，就会得到下面的结果。

```js
obj.count = 1
//  setting count!
++obj.count
//  getting count!
//  setting count!
//  2
```

上面代码说明，`Proxy` 实际上重载（overload）了点运算符，即用自己的定义覆盖了语言的原始定义。

ES6 原生提供 `Proxy` 构造函数，用来生成 `Proxy` 实例。

```js
var proxy = new Proxy(target, handler)
```

`Proxy` 对象的所有用法，都是上面这种形式，不同的只是`handler`参数的写法。其中，`new Proxy()`表示生成一个`Proxy`实例，`target`参数表示**所要拦截的目标对象**，`handler`参数也是一个对象，**用来定制拦截行为**。

下面是另一个拦截读取属性行为的例子。

```js
var proxy = new Proxy(
  {},
  {
    get: function (target, property) {
      return 35
    },
  }
)

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

上面代码中，作为构造函数，`Proxy`接受两个参数。第一个参数是所要代理的目标对象（上例是一个空对象），即如果没有`Proxy`的介入，操作原来要访问的就是这个对象；第二个参数是一个配置对象，对于每一个被代理的操作，需要提供一个对应的处理函数，该函数将拦截对应的操作。比如，上面代码中，配置对象有一个`get`方法，用来拦截对目标对象属性的访问请求。`get`方法的两个参数分别是目标对象和所要访问的属性。可以看到，由于拦截函数总是返回 35，所以访问任何属性都得到 35。

注意，要使得`Proxy`起作用，必须针对**Proxy 实例（上例是 proxy 对象）进行操作**，而**不是针对目标对象**（上例是空对象）进行操作。

如果`handler`没有设置任何拦截，那就等同于直接通向原对象。

```js
var target = {}
var handler = {}
var proxy = new Proxy(target, handler)
proxy.a = 'b'
target.a // "b"
```

上面代码中，`handler`是一个空对象，没有任何拦截效果，访问`proxy`就等同于访问 target。

一个技巧是将 `Proxy` 对象，设置到`object.proxy`属性，从而可以在`object`对象上调用。

```js
var object = { proxy: new Proxy(target, handler) }
```

`Proxy` 实例也可以作为其他对象的原型对象。

```js
var proxy = new Proxy(
  {},
  {
    get: function (target, property) {
      return 35
    },
  }
)

let obj = Object.create(proxy)
obj.time // 35
```

上面代码中，`proxy`对象是`obj`对象的原型，`obj`对象本身并没有`time`属性，所以根据原型链，会在`proxy`对象上读取该属性，导致被拦截。

同一个拦截器函数，可以设置拦截多个操作。

```js
var handler = {
  get: function (target, name) {
    if (name === 'prototype') {
      return Object.prototype
    }
    return 'Hello, ' + name
  },

  apply: function (target, thisBinding, args) {
    return args[0]
  },

  construct: function (target, args) {
    return { value: args[1] }
  },
}

var fproxy = new Proxy(function (x, y) {
  return x + y
}, handler)

fproxy(1, 2) // 1
new fproxy(1, 2) // {value: 2}
fproxy.prototype === Object.prototype // true
fproxy.foo === 'Hello, foo' // true
```

对于可以设置、但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果。

下面是 Proxy 支持的拦截操作一览，一共 13 种。

1. **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
2. **set(target, propKey, value, receiver)**：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
3. **has(target, propKey)**：拦截`propKey in proxy`的操作，返回一个布尔值。
4. **deleteProperty(target, propKey)**：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
5. **ownKeys(target)**：拦截`Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
6. **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
7. **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
8. **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
9. **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
10. **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
11. **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
12. **apply(target, object, args)**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)`。
13. **construct(target, args)**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

---

<div id="list2"></div>

## 2.this 问题

虽然 `Proxy` 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 `Proxy` 代理的情况下，目标对象内部的`this`关键字会指向 `Proxy` 代理。

```js
const target = {
  m: function () {
    console.log(this === proxy)
  },
}
const handler = {}

const proxy = new Proxy(target, handler)

target.m() // false
proxy.m() // true
```

上面代码中，一旦`proxy`代理`target.m`，后者内部的`this`就是指向`proxy`，而不是`target`。

```js
const _name = new WeakMap()

class Person {
  constructor(name) {
    _name.set(this, name)
  }
  get name() {
    return _name.get(this)
  }
}

const jane = new Person('Jane')
jane.name // 'Jane'

const proxy = new Proxy(jane, {})
proxy.name // undefined
```

上面代码中，目标对象`jane`的`name`属性，实际保存在外部`WeakMap`对象`_name`上面，通过`this`键区分。由于通过`proxy.name`访问时，`this`指向`proxy`，导致无法取到值，所以返回`undefined`。

此外，有些原生对象的内部属性，只有通过正确的`this`才能拿到，所以 `Proxy` 也无法代理这些原生对象的属性。

```js
const target = new Date()
const handler = {}
const proxy = new Proxy(target, handler)

proxy.getDate()
// TypeError: this is not a Date object.
```

上面代码中，`getDate`方法只能在`Date`对象实例上面拿到，如果`this`不是`Date`对象实例就会报错。这时，`this`绑定原始对象，就可以解决这个问题。

```js
const target = new Date('2015-01-01')
const handler = {
  get(target, prop) {
    if (prop === 'getDate') {
      return target.getDate.bind(target)
    }
    return Reflect.get(target, prop)
  },
}
const proxy = new Proxy(target, handler)

proxy.getDate() // 1
```

---

<div id="list3"></div>

## 3.实例：Web 服务的客户端

`Proxy` 对象可以拦截目标对象的任意属性，这使得它很合适用来写 Web 服务的客户端。

```js
const service = createWebService('http://example.com/data')

service.employees().then((json) => {
  const employees = JSON.parse(json)
  // ···
})
```

上面代码新建了一个 Web 服务的接口，这个接口返回各种数据。`Proxy` 可以拦截这个对象的任意属性，所以不用为每一种数据写一个适配方法，只要写一个 `Proxy` 拦截就可以了。

```js
function createWebService(baseUrl) {
  return new Proxy(
    {},
    {
      get(target, propKey, receiver) {
        return () => httpGet(baseUrl + '/' + propKey)
      },
    }
  )
}
```

# <font color="#e96900">module 语法</font>

> 首先说明一下 CommonJS 模块规范（node 用的就是这个）是**运行时加载**，es6 模块规范是**编译时加载**
>
> - 运行时加载 ---- 加载的时候引入对应的模块，整体引入，即使你只有几个函数，但也是整体模块都引入进来了
> - 编译时加载 ---- 编译的时候就知道需要引入模块中的那些函数，按需引入，在加载之前就已经确定好引入关系

- node------`require `引入 ，`module.exports/exports`导出
- es6--------`import`引入,`export/export default`导出

## CommonJS 模块规范

> 运行时加载

### exports 和 module.exports

1. `Node`为每个模块提供一个`exports`变量，指向`module.exports` 相当于 `var exports = module.exports;`
   > 指向的是同一个内存地址，但最终输出的是`module.exports`!!! 他们之间的区别就是` exports` 只是 `module.exports`的引用，辅助后者添加内容用的

```js
// 正确用法
exports.x = function(){}
module.exports.y = {}

//错误用法
exports = function(){} 在这里已经切断了exports和module.exports的联系了
```

---

## ES6 规范

> 编译时加载

### export 和 export default

首先我们讲这两个导出，下面我们讲讲它们的区别

1. export 与 export default 均可用于导出常量、函数、文件、模块等
2. 在一个文件或模块中，export、import 可以有多个，export default 仅有一个
3. 通过 export 方式导出，在导入时要加{ }，export default 则不需要
4. export 能直接导出变量表达式，export default 不行。

#### export

```javascript
export var firstName = 'Michael'

var firstName = 'Michael'
export { firstName }

export function multiply(x, y) {
  return x * y
}
```

需要特别注意的是，export 命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```js
// 报错
export 1;

// 报错
var m = 1;
export m;
```

上面两种写法都会报错，因为没有提供对外的接口。第一种写法直接输出 1，第二种写法通过变量 m，还是直接输出 1。1 只是一个值，不是接口。正确的写法是下面这样。

```js
// 写法一
export var m = 1

// 写法二
var m = 1
export { m }

// 写法三
var n = 1
export { n as m }
```

上面三种写法都是正确的，规定了对外的接口`m`。其他脚本可以通过这个接口，取到值 1。它们的实质是，在接口名与模块内部变量之间，建立了一一对应的关系。

同样的，`function`和`class`的输出，也必须遵守这样的写法。

```js
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};
```

#### export default

为了给用户提供方便，让他们不用阅读文档就能加载模块，就要用到`export default`命令，为模块指定默认输出。

```js
// export-default.js
export default function () {
  console.log('foo')
}
```

上面代码是一个模块文件`export-default.js`，它的默认输出是一个函数。

其他模块加载该模块时，`import`命令可以为该匿名函数指定任意名字。

```js
// import-default.js
import customName from './export-default'
customName() // 'foo'
```

上面代码的 import 命令，可以用任意名称指向 export-default.js 输出的方法，这时就不需要知道原模块输出的函数名。需要注意的是，**这时 import 命令后面，不使用大括号**。
`export default`命令用在非匿名函数前，也是可以的。

```js
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;
```

上面代码中，`foo`函数的函数名`foo`，在模块外部是无效的。加载的时候，视同匿名函数加载。

下面比较一下默认输出和正常输出。

```js
// 第一组
export default function crc32() {
  // 输出
  // ...
}

import crc32 from 'crc32' // 输入

// 第二组
export function crc32() {
  // 输出
  // ...
}

import { crc32 } from 'crc32' // 输入
```

上面代码的两组写法，第一组是使用`export default`时，对应的`import`语句不需要使用大括号；第二组是不使用`export default`时，对应的`import`语句需要使用大括号。

`export default`命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此`export default命令`只能使用一次。所以，`import`命令后面才不用加大括号，因为只可能唯一对应`export default命令`。

本质上，`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```js
// modules.js
function add(x, y) {
  return x * y
}
export { add as default }
// 等同于
// export default add;

// app.js
import { default as foo } from 'modules'
// 等同于
// import foo from 'modules';
```

正是因为`export default`命令其实只是输出一个叫做`default`的变量，所以它后面不能跟变量声明语句。

```js
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;
```

同样地，因为`export default`命令的本质是将后面的值，赋给`default`变量，所以可以直接将一个值写在`export default`之后。

```js
// 正确
export default 42;

// 报错
export 42;
```

```js
// 正确
export default 42;

// 报错
export 42;
```

如果想在一条`import`语句中，同时输入默认方法和其他接口，可以写成下面这样。

```js
import _, { each, forEach } from 'lodash'
```

对应上面代码的`export`语句如下。

```js
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

export { each as forEach }
```

`export default`也可以用来输出类。

```js
// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

### import()

> 类似于`require`，运行时加载相应模块 动态加载

#### 适用场合

1. 按需加载，在 vue 中可以实现路由懒加载

```js
button.addEventListener('click', (event) => {
  import('./dialogBox.js')
    .then((dialogBox) => {
      dialogBox.open()
    })
    .catch((error) => {
      /* Error handling */
    })
})
```

2. 条件加载，可以写在条件语句中

```js
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
}

```

3. 动态的模块路径

```js
import(f())
.then(...);

```

#### 注意点

`import()`加载模块成功以后，这个模块会作为一个对象，当作`then`方法的参数。因此，可以使用对象解构赋值的语法，获取输出接口。

```js
import('./myModule.js').then(({ export1, export2 }) => {
  // ...·
})
```

上面代码中，export1 和 export2 都是 myModule.js 的输出接口，可以解构获得。

```js
import('./myModule.js').then((myModule) => {
  console.log(myModule.default)
})
```

上面的代码也可以使用具名输入的形式。

```js
import('./myModule.js').then(({ default: theDefault }) => {
  console.log(theDefault)
})
```

如果想同时加载多个模块，可以采用下面的写法。

```js
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

`import()`也可以用在 async 函数之中。

```js
async function main() {
  const myModule = await import('./myModule.js')
  const { export1, export2 } = await import('./myModule.js')
  const [module1, module2, module3] = await Promise.all([
    import('./module1.js'),
    import('./module2.js'),
    import('./module3.js'),
  ])
}
main()
```

---

# <font color="#e96900">Set 和 Map</font>

> ES6 中新增的数据类型有哪些？
> `Set` 和 `Map` 都是 `ES6 `中新增的数据结构，是对当前 JS 数组和对象这两种重要数据结构的扩展

## Set

> Set 类似于数组，但数组可以允许元素重复，Set 不允许元素重复,一般可以做数组去重

Set 实例不允许元素有重复，可以通过以下示例证明。可以通过一个数组初始化一个 Set 实例，或者通过 add 添加元素，元素不能重复，重复的会被忽略。

```js
// 例1
const set = new Set([1, 2, 3, 4, 4])
console.log(set) // Set(4) {1, 2, 3, 4}

// 例2
const set = new Set()
;[2, 3, 5, 4, 5, 8, 8].forEach((item) => set.add(item))
for (let item of set) {
  console.log(item)
}
// 2 3 5 4 8
```

Set 实例的属性和方法有

1. size：获取元素数量。
2. add(value)：添加元素，返回 Set 实例本身。
3. delete(value)：删除元素，返回一个布尔值，表示删除是否成功。
4. has(value)：返回一个布尔值，表示该值是否是 Set 实例的元素。
5. clear()：清除所有元素，没有返回值。

```js
const s = new Set()
s.add(1).add(2).add(2) // 添加元素

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2)
s.has(2) // false

s.clear()
console.log(s) // Set(0) {}
```

Set 实例的遍历

1. keys()：返回键名的遍历器。
2. values()：返回键值的遍历器。不过由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以 keys()和 values()返回结果一致。
3. entries()：返回键值对的遍历器。
4. forEach()：使用回调函数遍历每个成员

```js
let set = new Set(['aaa', 'bbb', 'ccc'])

for (let item of set.keys()) {
  console.log(item)
}
// aaa
// bbb
// ccc

for (let item of set.values()) {
  console.log(item)
}
// aaa
// bbb
// ccc

for (let item of set.entries()) {
  console.log(item)
}
// ["aaa", "aaa"]
// ["bbb", "bbb"]
// ["ccc", "ccc"]

set.forEach((value, key) => console.log(key + ' : ' + value))
// aaa : aaa
// bbb : bbb
// ccc : ccc
```

### Set 数组去重---严格来说 Set 不允许元素重复本来就可以实现数组去重但是数据类型还是`Set`类型 不是`Array`类型，需要通过`Array.from`转一下

```js
let set = new Set([1,1,2,3,4,5])
console.log('set===>',set) //Set(5) {1, 2, 3, 4, 5}
console.log(set instanceof Set) //true
console.log(set instanceof Array) //false

let arrSet = Array.from(set);
console.log('arrSet===>',arrSet); //arrSet===> (5) [1, 2, 3, 4, 5]

console.log(arrSet instanceof Array) true
```

---

## Map

> `Map` 类似于对象，但普通对象的`key`必须是字符串或者数字，而 `Map` 的`key`可以是任何数据类型

Map 的用法和普通对象基本一致，先看一下它能用非字符串或者数字作为 key 的特性。

```js
const map = new Map()
const obj = { p: 'Hello World' }

map.set(obj, 'OK')
map.get(obj) // "OK"

map.has(obj) // true
map.delete(obj) // true
map.has(obj) // false
```

需要使用 new Map()初始化一个实例

Map 实例的属性和方法如下：

1. size：获取成员的数量
2. set：设置成员 key 和 value
3. get：获取成员属性值
4. has：判断成员是否存在
5. delete：删除成员
6. clear：清空所有

```js
const map = new Map()
map.set('aaa', 100)
map.set('bbb', 200)

map.size // 2

map.get('aaa') // 100

map.has('aaa') // true

map.delete('aaa')
map.has('aaa') // false

map.clear()
```

Map 实例的遍历方法有：

1. keys()：返回键名的遍历器。
2. values()：返回键值的遍历器。
3. entries()：返回所有成员的遍历器。
4. forEach()：遍历 Map 的所有成员

```js
const map = new Map()
map.set('aaa', 100)
map.set('bbb', 200)

for (let key of map.keys()) {
  console.log(key)
}
// "aaa"
// "bbb"

for (let value of map.values()) {
  console.log(value)
}
// 100
// 200

for (let item of map.entries()) {
  console.log(item[0], item[1])
}
// aaa 100
// bbb 200

// 或者
for (let [key, value] of map.entries()) {
  console.log(key, value)
}
// aaa 100
// bbb 200
```
