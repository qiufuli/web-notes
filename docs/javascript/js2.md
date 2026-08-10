# js 进阶

# <font color="#e96900">数组相关</font>

## 哪些方法改变原有数组，哪些方法返回新的数组

改变原数组：

- push()
- pop()
- shift()
- unshift()
- sort()
- reverse()
- splice()----splice(index:从哪开始,howmany：删除几个,item1..item2：添加的元素) ---改变原有数组,返回被删除的元素

不改变原数组

- slice()--- slice(start,end) 返回新数组，不包含 end
- concat()
- join()
- forEach()
- filter()
- map()
- find()
- findIndex()
- every()
- some()
- reduce()

---

## sort()---改变原数组

`sort() `方法用原地算法对数组的元素进行排序，并返回数组（改变原有数组）。默认排序顺序是**先将元素转换为字符串，然后按照字符串的各个字符的 Unicode 位点进行排序**。

```js
arr.sort([compareFunction])

var arr = [3, 10, 2, 6]

arr.sort(function (a, b) {
  return a - b // [2, 3, 6, 10]升序
  return b - a // [10, 6, 3, 2]降序
})
```

**如果不适纯数字，可以比较某个属性，对象可以按照某个属性排序:**
因为`sort`排序是按照字符串的各个字符的 Unicode 位点进行排序，非数字类型也可以进行排序

```js
var items = [
  { name: 'Edward', value: 21 },
  { name: 'Sharpe', value: 37 },
  { name: 'And', value: 45 },
  { name: 'The', value: -12 },
  { name: 'Magnetic' },
  { name: 'Zeros', value: 37 },
]

// sort by value
items.sort(function (a, b) {
  return a.value - b.value // 因为value都是纯数字的
})

// sort by name
items.sort(function (a, b) {
  var nameA = a.name.toUpperCase() // ignore upper and lowercase
  var nameB = b.name.toUpperCase() // ignore upper and lowercase
  if (nameA < nameB) {
    return -1
  }
  if (nameA > nameB) {
    return 1
  }

  // names must be equal

  return 0
})
```

---

## Array.from()

`Array.from()` 方法从一个类似数组或可迭代对象创建一个新的，浅拷贝的数组实例。

```js
Array.from(arrayLike[, mapFn[, thisArg]])
```

- arrayLike 想要转换成数组的伪数组对象或可迭代对象。
- mapFn 可选,如果指定了该参数，新数组中的每个元素会执行该回调函数
- thisArg 可选，执行回调函数 mapFn 时 this 对象。

### 从 String 生成数组

```javascript
Array.from('foo')
// [ "f", "o", "o" ]
```

### 从 Set 生成数组

```javascript
const set = new Set(['foo', 'bar', 'baz', 'foo'])
Array.from(set)
// [ "foo", "bar", "baz" ]
```

### 从 Map 生成数组

```javascript
const map = new Map([
  [1, 2],
  [2, 4],
  [4, 8],
])
Array.from(map)
// [[1, 2], [2, 4], [4, 8]]
const mapper = new Map([
  ['1', 'a'],
  ['2', 'b'],
])
Array.from(mapper.values())
// ['a', 'b'];

Array.from(mapper.keys())
// ['1', '2'];
```

### 从类数组对象（arguments）生成数组

```javascript
function f() {
  return Array.from(arguments)
}

f(1, 2, 3)

// [ 1, 2, 3 ]
```

### 在 Array.from 中使用箭头函数

```javascript
Array.from([1, 2, 3], (x) => x + x)
// [2, 4, 6]
Array.from({ length: 5 }, (v, i) => i)
// [0, 1, 2, 3, 4]
```

### 数组去重合并

```javascript
function combine() {
  let arr = [].concat.apply([], arguments) //没有去重复的新数组
  return Array.from(new Set(arr))
}

var m = [1, 2, 2],
  n = [2, 3, 3]
console.log(combine(m, n)) // [1, 2, 3]
```

---

# <font color="#e96900">对象相关</font>

## Object.assign()

`Object.assign()` 方法用于将**自身所以可枚举属性的值**从一个或多个源对象复制到目标对象。它将返回目标对象。

```js
Object.assign(target, ...sources)
```

```js
const target = { a: 1, b: 2 }
const source = { b: 4, c: 5 }

const res = Object.assign({}, target, source)
console.log('res', res) //{a: 1, b: 4, c: 5} 返回的新对象是目标对象和多个源对象的覆盖组合对象

//target作为目标对象 虽然返回一个新对象 但target的值也同样会被改变
const resTarget = Object.assign(target, source)
resTarget.d = 6
console.log('resTarget', resTarget) //{a: 1, b: 4, c: 5,d:6}
console.log('target', target) //{a: 1, b: 4, c: 5,d:6}
```

上面的例子中，因为`Object.assign()`返回的是目标对象，`resTarget` 的目标对象是`target`，所以改变了`resTarget`，`target`也会随之改变，更推荐`res`的定义方式，将一个空对象作为目标对象。

**目标对象和源对象存在同名属性时，源对象会覆盖目标对象**

`Object.assign` 方法只会拷贝源对象自身的并且可枚举的属性到目标对象。该方法使用源对象的[[Get]]和目标对象的[[Set]]，所以它会调用相关 getter 和 setter。因此，它分配属性，而不仅仅是复制或定义新的属性。如果合并源包含 getter，这可能使其不适合将新属性合并到原型中。为了将属性定义（包括其可枚举性）复制到原型，应使用`Object.getOwnPropertyDescriptor()`和`Object.defineProperty()` 。

`Object.assign`属于`浅拷贝范畴`，因为 `Object.assign`拷贝的是属性值。假如源对象的属性值是一个对象的引用，那么它也只指向那个引用。

---

## Object.create()

`Object.create()` 使用指定的原型对象和属性创建一个新对象。(创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`。)

利用`Object.create()`实现继承

```js
// Shape - 父类(superclass)
function Shape() {
  this.x = 0
  this.y = 0
}

// 父类的方法
Shape.prototype.move = function (x, y) {
  this.x += x
  this.y += y
  console.info('Shape moved.')
}

// Rectangle - 子类(subclass)
function Rectangle() {
  Shape.call(this) // call super constructor.
}

// 子类续承父类
Rectangle.prototype = Object.create(Shape.prototype)
Rectangle.prototype.constructor = Rectangle

var rect = new Rectangle()

console.log('Is rect an instance of Rectangle?', rect instanceof Rectangle) // true
console.log('Is rect an instance of Shape?', rect instanceof Shape) // true
rect.move(1, 1) // Outputs, 'Shape moved.'
```

`Rectangle.prototype` 重定义为`Shape.prototype` ,然后需要改变原型中`constructor`的指向为自己`Rectangle`

---

## Object.defineProperty()

`Object.defineProperty()` 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性并指定该属性的配置，并返回此对象。

> 备注：应当直接在 `Object` 构造器对象上调用此方法，而不是在任意一个 `Object` 类型的实例上调用。12

```js
Object.defineProperty(obj, prop, descriptor)
```

- **obj** 要定义属性的对象。
- **prop** 要定义或修改的属性的名称或 Symbol 。
- **descriptor** 要定义或修改的属性描述符。

该方法允许精确地添加或修改对象的属性。通过赋值操作添加的普通属性是可枚举的，在枚举对象属性时会被枚举到（`for...in` 或 `Object.keys` 方法），可以改变这些属性的值，也可以删除这些属性。这个方法允许修改默认的额外选项（或配置）。默认情况下，使用 `Object.defineProperty()` 添加的属性值是不可修改（`immutable`）的。

对象里目前存在的属性描述符有两种主要形式：

- **数据描述符**
  数据描述符是一个具有值的属性，该值可以是可写的，也可以是不可写的
- **存取描述符**
  存取描述符是由 getter 函数和 setter 函数所描述的属性

  **一个描述符只能是这两者其中之一；不能同时是两者。（因为 get 和 set 设置的时候 js 会忽略 value 和 writable 的特性）**

这两种描述符都是对象。它们共享以下可选键值（默认值是指在使用 `Object.defineProperty()` 定义属性时的默认值）:

- **configurable 默认为 false。**
  当且仅当该属性的 `configurable` 为 `true` 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。
- **enumerable 默认为 false**
  当且仅当该属性的 `enumerable` 键值为 `true` 时，该属性才会出现在对象的枚举属性中(**是否可枚举 for...in**)。

---

### 数据描述符

- **value 默认为 undefined**
  该属性对应的值。
- **writable 默认为 false**
  当且仅当该属性的 `writable` 键值为 `true` 时，属性的值，也就是上面的 `value`，才能被赋值运算符改变。

---

### 存取描述符

- **get 默认为 undefined**
  属性的 `getter` 函数，如果没有 `getter`，则为 `undefined`。**当访问该属性时，会调用此函数**。执行时不传入任何参数，但是会传入 `this` 对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。**该函数的返回值会被用作属性的值**。
- **set 默认为 undefined**
  属性的 `setter` 函数，如果没有 `setter`，则为 `undefined`。**当属性值被修改时，会调用此函数**。该方法接受一个参数（也就是被赋予的新值），会传入赋值时的 `this` 对象。

---

### 描述符可拥有的键值

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520145353203.png)
如果一个描述符不具有 `value、writable、get 和 set` 中的任意一个键，那么它将被认为是一个**数据描述符**。
如果一个描述符同时拥有 `value 或 writable 和 get 或 set` 键，则会产生一个**异常**。

==记住，这些选项不一定是自身属性，也要考虑继承来的属性。为了确认保留这些默认值，在设置之前，可能要冻结 Object.prototype，明确指定所有的选项，或者通过 Object.create(null) 将 **proto** 属性指向 null。==

---

### 描述符默认值汇总

- 拥有布尔值的键 `configurable、enumerable 和 writable` 的默认值都是 `false`。
- 属性值和函数的键 `value、get 和 set` 字段的默认值为 `undefined`。

---

### 案例-数据描述符

```js
var o = {} // 创建一个新对象

// 在对象中添加一个属性与数据描述符的示例
Object.defineProperty(o, 'a', {
  value: 37,
  writable: true,
  enumerable: true,
  configurable: true,
})
console.log(o.a) // 37
```

### 案例-存取描述符

```js
var o = {} // 创建一个新对象

var value
Object.defineProperty(o, 'a', {
  enumerable: true,
  configurable: true,
  get() {
    return value
  },
  set(newvalue) {
    console.log('newvalue', newvalue)
    value = newvalue
  },
})
console.log(o.a) // undefined
o.a = 1 // 触发set特性 打印 newvalue 1
console.log(o.a) // 1
```

---

## Object.defineProperties()

给对象添加多个属性并分别指定它们的配置。

```javascript
Object.defineProperties(obj, props)
```

- **obj**
  在其上定义或修改属性的对象。
- props
  要定义其可枚举属性或修改的属性描述符的对象。

```javascript
var obj = {}
Object.defineProperties(obj, {
  property1: {
    value: true,
    writable: true,
  },
  property2: {
    value: 'Hello',
    writable: false,
  },
  // etc. etc.
})
```

`Object.defineProperties`是`Object.defineProperty`的复数形式，`Object.defineProperty`对单个属性进行配置，`Object.defineProperties`对于多个属性同时进行配置，用法跟`Object.defineProperty`一样。

---

## Object.keys()

`Object.keys()` 方法会返回一个由一个给定对象的**自身可枚举**属性名组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致。

- obj 要返回其枚举自身属性的对象。
- 返回值 一个表示给定对象的所有可枚举属性名的字符串数组。

```javascript
var arr = ['a', 'b', 'c']
console.log(Object.keys(arr)) // console: ['0', '1', '2']
var o = {
  name: 'qfl',
  age: 26,
}
console.log(Object.keys(o)) // ["name", "age"];

function Fn() {
  this.a = 'a'
  this.b = 'b'
}
Fn.prototype.c = 'c'
var fn = new Fn()
console.log(Object.keys(fn)) // ["a", "b"]
```

- 对于数组而言，属性名对应的就是索引值
- **只返回自身可枚举的属性**，原型上的不会获取 ,如果你想获取一个对象的所有属性,，甚至包括不可枚举的，请查看`Object.getOwnPropertyNames`。

---

## Object.values()

方法返回一个给定对象**自身的所有可枚举属性值**的数组，值的顺序与使用 for...in 循环的顺序相同 ( 区别在于 for-in 循环枚举原型链中的属性 )。

```javascript
Object.values(obj)
```

```javascript
var obj = { foo: 'bar', baz: 42 }
console.log(Object.values(obj)) // ['bar', 42]
var obj1 = { 0: 'a', 1: 'b', 2: 'c' }
console.log(Object.values(obj1)) // ['a', 'b', 'c']
var obj2 = [1, 2, 3, 4]
console.log(Object.values(obj2)) // [1, 2, 3, 4]
```

---

## hasOwnProperty()

`hasOwnProperty() `方法会返回一个**布尔值**，指示对象自身属性中是否具有指定的属性（也就是，是否有指定的键 **不可枚举的属性也可以查到**），无法检查原型链上是否具有此属性名。

```javascript
obj.hasOwnProperty(prop)
```

即使属性的值是 `null` 或 `undefined`，只要属性存在，`hasOwnProperty` 依旧会返回 true。

```javascript
o = new Object()
o.propOne = null
o.hasOwnProperty('propOne') // 返回 true
o.propTwo = undefined
o.hasOwnProperty('propTwo') // 返回 true
```

```javascript
o = new Object()
o.hasOwnProperty('prop') // 返回 false
o.prop = 'exists'
o.hasOwnProperty('prop') // 返回 true
delete o.prop
o.hasOwnProperty('prop') // 返回 false
```

遍历一个对象上的所有属性(自身属性与继承属性)

```javascript
function Fn(){
this.name='qfl';
this.age=26
}
Fn.prototype.hobby = 'web';

var fn = new Fn();
Object.defineProperty(fn,'name',{
enumerable:false
})

for(var key in fn){
console.log(key);
if(fn.hasOwnProperty(key)){
console.log(`自有属性含有:${key}`); //自有属性含有: age
}else{
console.log(`原型继承属性有:${key}`);//原型继承属性有:hobby
}
```

`for...in`会遍历自身和继承的所有属性，`hasOwnProperty`判断是否是自身属性，这样两个都可以区分开来

---

## Object.getOwnPropertyNames()

`Object.getOwnPropertyNames()`方法返回一个由指定**对象的所有自身属性的属性名**(不包括原型链上的属性)（包括不可枚举属性但不包括 Symbol 值作为名称的属性）组成的数组。

```javascript
Object.getOwnPropertyNames(obj)
```

```javascript
var arr = ['a', 'b', 'c']
console.log(Object.getOwnPropertyNames(arr).sort()) // ["0", "1", "2", "length"]

// 类数组对象
var obj = { 0: 'a', 1: 'b', 2: 'c' }
console.log(Object.getOwnPropertyNames(obj).sort()) // ["0", "1", "2"]

//不可枚举属性
var my_obj = Object.create(
  {},
  {
    getFoo: {
      value: function () {
        return this.foo
      },
      enumerable: false,
    },
  }
)
my_obj.foo = 1

console.log(Object.getOwnPropertyNames(my_obj).sort()) // ["foo", "getFoo"]
```

如果你只要获取到可枚举属性，查看[Object.keys](#5)或用 for...in 循环（还会获取到原型链上的可枚举属性，不过可以使用 hasOwnProperty()方法过滤掉）。

下面的例子演示了该方法不会获取到原型链上的属性：

```javascript
function ParentClass() {}
ParentClass.prototype.inheritedMethod = function () {}

function ChildClass() {
  this.prop = 5
  this.method = function () {}
}

ChildClass.prototype = new ParentClass()
ChildClass.prototype.prototypeMethod = function () {}

console.log(
  Object.getOwnPropertyNames(
    new ChildClass() // ["prop", "method"]
  )
)
```

---

## Object.getPrototypeOf()

`Object.getPrototypeOf() `方法返回指定对象的原型（内部[[Prototype]]属性的值）。

```javascript
Object.getPrototypeOf(object)
```

```javascript
var proto = {}
var obj = Object.create(proto)
console.log(Object.getPrototypeOf(obj) === proto) // true

var reg = /a/
console.log(Object.getPrototypeOf(reg) === RegExp.prototype) // true
```

---

## Object.is()

`Object.is()` 方法判断两个值是否是相同的值。

```javascript
Object.is(value1, value2)
```

Object.is() 判断两个值是否相同。如果下列任何一项成立，则两个值相同：

- 两个值都是 undefined
- 两个值都是 null
- 两个值都是 true 或者都是 false
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且
  - 都是正零 +0
  - 都是负零 -0
  - 都是 NaN
  - 都是除零和 NaN 外的其它同一个数字

这种相等性判断逻辑和传统的 == 运算不同，== 运算符会对它两边的操作数做隐式类型转换（如果它们类型不同），然后才进行相等性比较，（所以才会有类似 "" == false 等于 true 的现象），但 Object.is 不会做这种类型转换。

```javascript
Object.is('foo', 'foo') // true
Object.is(window, window) // true

Object.is('foo', 'bar') // false
Object.is([], []) // false

var foo = { a: 1 }
var bar = { a: 1 }
Object.is(foo, foo) // true
Object.is(foo, bar) // false

Object.is(null, null) // true

// 特例
Object.is(0, -0) // false
Object.is(0, +0) // true
Object.is(-0, -0) // true
Object.is(NaN, 0 / 0) // true
```

---

## Object.entries()

方法返回一个给定对象**自身可枚举属性的键值对数组**，其排列与使用`for...in`循环遍历该对象时返回的顺序一致（区别在于 for-in 循环还会枚举原型链中的属性）。

```javascript
Object.entries(obj)
```

```javascript
const obj = { foo: 'bar', baz: 42 }
console.log(Object.entries(obj)) // [ ['foo', 'bar'], ['baz', 42] ]

const obj1 = { 0: 'a', 1: 'b', 2: 'c' }
console.log(Object.entries(obj1)) // [ ['0', 'a'], ['1', 'b'], ['2', 'c'] ]

const obj2 = ['a', 'b', 'c']
console.log(Object.entries(obj2))
;[
  ['0', 'a'],
  ['1', 'b'],
  ['2', 'c'],
]

const obj3 = { a: 5, b: 7, c: 9 }
for (const [key, value] of Object.entries(obj3)) {
  console.log(`${key} ${value}`) // "a 5", "b 7", "c 9"
}
```

---

## 总结

- `Object.assign(target,obj)` - - - 浅拷贝一个对象到目标对象，返回目标对象，将自身所有可枚举的属性都拷贝但不包括原型链上的。
- `Object.create(obj,objProps)` - - - 以目标对象为原型对象创建一个新对象，常用于继承，第二个参数是添加属性对属性进行描述符配置。
- `Object.defineProperty()` - - - 对已有属性或创建一个新属性，对属性进行描述符（数据、存取）配置,返回此对象。
- `Object.defineProperties()` - - - Object.defineProperty()·的复数形式，可以对多个属性同时进行配置。
- `Object.getPrototypeOf()` - - - 返回指定对象的原型，常用于判断对象的原型,Object.getPrototypeOf(obj) === proto
- `Object.is()` - - - 判断两个值是否是相同的值。
- `Object.freeze()` - - - 冻结一个对象，不能更改,**数据属性的值不可更改**(字符串，数字和布尔总是不可变的),如果一个属性的值是个对象(函数、对象和数组)，则这个**对象中的属性是可以修改**的
- `Object.isFrozen()` - - - 判断一个对象是否被冻结，是否能扩展。

#### 对枚举属性相关的操作方法如下

- `for... in` - - - 对**自有属性**和**原型链**上的所有**可枚举**的属性都能返回。
- `Object.keys()`- - - 返回所有**自身可枚举的属性名**(不包括原型链上的)组成的数组。
- `Object.values()`- - - 返回所有**自身可枚举的属性值**(不包括原型链上的)组成的数组。
- `Object.entries()`- - - 返回所有**自身可枚举的属性名和属性值组成的数组组合**(不包括原型链上的)组成的数组。[ ['键','值']，['foo', 'bar'], ['baz', 42] ]
- `Object.getOwnPropertyNames` - - - 返回一个所有**可枚举和不可枚举的自身属性的属性名**(不包括原型链上的)。
- `xxx.hasOwnProperty(prop)` - - - 判断**是否是自有属性包括不可枚举的属性**(不包括原型链上的)，返回布尔值。

---

## 兼容性

下面按照浏览器兼容性分类哪些能使用，哪些不能使用，以 ie 浏览器为例：

- 不兼容 ie：`Object.assign()`、`Object.values()`、`Object.entries()`、`Object.is()`
- ie9 及以上：`Object.create()`、`Object.defineProperty()`、`Object.defineProperties()`、`Object.getOwnPropertyNames()`、`Object.getPrototypeOf()`、`Object.isFrozen()`、`Object.isFrozen()`

---

## for in 和 for of 的区别

> for in 更适合遍历对象 返回的是属性名,遍历包括自有属性和原型链上的属性

> for of 更适合遍历数组 或者说类数组（一个数据结构只要部署了 Symbol.iterator 属性，就被视为具有 iterator 接口，可以使用 for of。）都可以遍历

> for of 不同于 forEach，for of 是可以 break，continue，return 配合使用，for of 循环可以随时退出循环。 类似于 for 循环

```js
var obj = { name: 'saucxs', age: 21, sex: 1 }
for (key in obj) {
  console.log(key, obj[key])
  // name saucxs
  // age 21
  // sex 1
}
for (key of obj) {
  console.log(key, obj[key])
  // typeError :obj is not iterable报错
}
```

说明 obj 对象没有 iterable 属性报错，使用不了 for of。

我们现在再看一个数组遍历的例子

```js
var array = ['a', 'b', 'c']
for (var key in array) {
  console.log(key, array[key])
  // 0 a
  // 1 b
  // 2 c
}
for (var key of array) {
  console.log(key, array[key])
  // a undefined
  // b undefined
  // c undefined
}
```

```js
var array = ['a', 'b', 'c']
array.name = 'saucxs'
for (key in array) {
  console.log(key, array[key])
  // 0 a
  // 1 b
  // 2 c
  // name saucxs
}
```

### for in 的特点

for in 循环返回的值都是数据结构的**键名**。

**遍历对象返回的是对象的 key 值，遍历数组返回的是数组的下标。**

**还会遍历原型上的值和手动添加的值**

总的来说：for in 适合遍历对象。

---

### for of 的特点

for of 循环获取一对键值中的**键值。**

一个数据结构只要部署了 Symbol.iterator 属性，就被视为具有 iterator 接口(set Array map string arguments 类数组)，可以使用 for of。

**for of 不同于 forEach，for of 是可以 break，continue，return 配合使用，for of 循环可以随时退出循环。**

总的来说：for of 遍历所有数据结构的统一接口。

### for of 键值对形式

虽然 for of 获取的是键值对中的值，但我们可以处理下数组

```js
const fruits = ['apple', 'banana', 'orange', 'pear']

for (let [index, fruit] of fruits.entries()) {
  console.log(`${fruit} ranks ${index + 1} in my favorite fruits`)
}
```

`Object.entries()`可以将对象的键 值 组成数组的形式，这样通过结构的方式就可以获取到键值对了

---

## for 循环和 forEach 的区别

`for`循环在最开始执行循环的时候，会建立一个循环变量 i，之后每次循环都是操作这个变量，也就是说它是对一个循环变量在重复的赋值，因此 i 在最后只会存储一个值；（所以为什么 for 循环绑定事件的时候都需要用闭包）

而`forEach()`虽然变量名没变，但是实际上每次循环都会创建一个独立不同的变量，而存储的数值自然也是不同的数值，因此相互之间不会影响，

```js
var ele = document.querySelectAll('p')

for (var i = 0; i < ele.length; i++) {
  eles[i].onclick = function () {
    console.log(i) // 每次打印的i都是最后一个
  }
}
ele.forEach(function (v, i) {
  v.onclick = function () {
    console.log(i) // 每次打印都是不同的i
  }
})
```

其次,**for 循环实际上是可以使用 break 和 continue 去终止循环的，但是 forEach 不行；**

> forEach 包含一个立即执行函数 return 只是跳出这个 forEach 而已 所以无法跳出整个 forEach 循环 要想跳出可以使用 try cach 结合 return new Error（）
