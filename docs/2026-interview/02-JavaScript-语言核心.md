# JavaScript 语言核心

> 不要把 JavaScript 面试复习成“术语表”。每个问题都回到三个判断：值是什么、代码在哪里执行、对象和函数之间通过什么关系连接。

## 一、先建立语言运行模型

```text
输入值 -> 类型与转换
代码执行 -> 作用域、执行上下文、调用栈
函数调用 -> this、闭包、参数传递
对象关系 -> 原型链、class、集合
工程边界 -> 模块、错误、内存
```

## 二、值与类型：为什么“看起来一样”结果不同

### P0：`==`、`===` 和 `Object.is` 有什么区别？

`===` 不做隐式类型转换，是业务默认选择；`==` 有完整转换规则，只有明确想同时判断 `null` 和 `undefined` 时才有少数可读用法；`Object.is` 基本类似严格相等，但认为 `NaN` 等于自身，并区分 `+0` 和 `-0`。

```js
0 === -0 // true
Object.is(0, -0) // false
NaN === NaN // false
Object.is(NaN, NaN) // true

function isMissing(value) {
  return value == null // 仅匹配 null / undefined
}
```

不要用 `if (!value)` 判断接口字段是否缺失，因为 `0`、`false` 和空字符串可能都是合法值。对象比较的是引用，不是结构内容。

### P0：`typeof`、`instanceof` 和类型判断的边界

```js
typeof null // 'object'，历史遗留
typeof (() => {}) // 'function'
Array.isArray([]) // 更适合判断数组，跨 iframe 也可靠
Object.prototype.toString.call(new Date()) // '[object Date]'
```

`instanceof` 检查原型链，跨 iframe/realm 时可能失效；业务更应该围绕输入边界做校验，而不是只追求“判断出一个类型字符串”。

### P0：隐式转换为什么容易制造线上问题

运算、比较和字符串拼接会触发转换。对象转原始值会先尝试 `Symbol.toPrimitive`，再按 hint 使用 `valueOf` 或 `toString`：

```js
const money = {
  [Symbol.toPrimitive](hint) {
    return hint === 'string' ? '¥10' : 10
  },
}

String(money) // '¥10'
money + 2 // 12
```

金额、日期和枚举值应在边界显式转换，不要让 `+`、`==` 或空值转换替你决定业务含义。

## 三、代码如何执行：提升、作用域和闭包

### P0：执行上下文、提升和暂时性死区

执行一段代码时，JavaScript 会建立全局、函数或模块执行上下文，其中包含词法环境、变量环境和 `this` 绑定。声明在创建阶段被处理，但不同声明的初始化状态不同：

```js
console.log(a) // undefined
var a = 1

// console.log(b) // ReferenceError：处于暂时性死区
let b = 2

say() // 可以调用
function say() { return 'hello' }
```

“提升”不是代码真的被搬到顶部；准确理解创建阶段和初始化时机，才能解释 `var`、函数声明与 `let/const` 的差异。ESM 还有独立模块作用域，顶层变量不会自动成为 `window` 属性。

### P0：作用域、作用域链和闭包是什么

JavaScript 使用词法作用域，函数定义的位置决定它能访问哪些外层变量。函数与它仍然需要的外层环境一起形成闭包：

```js
function createCounter() {
  let count = 0
  return () => ++count
}

const next = createCounter()
next() // 1
next() // 2
```

闭包不是内存泄漏。只有不再需要的闭包仍被定时器、事件监听、全局缓存或第三方对象引用，相关数据才无法回收。组件卸载时要清理这些外部引用。

### P0：参数传递是值传递还是引用传递

JavaScript 一律按值传递。对象变量保存的值是对象引用，所以通过形参修改对象属性会影响外部对象；但形参重新指向新对象不会改变外部变量：

```js
function update(user) {
  user.name = 'new name'
  user = { name: 'another user' }
}

const user = { name: 'old name' }
update(user)
user.name // 'new name'
```

这也是组件 prop 不能直接改的底层原因之一：即使引用能改到父对象，也破坏了数据所有权和单向数据流。

## 四、函数调用：`this` 为什么会变

### P0：普通函数和箭头函数的 this

普通函数的 `this` 由调用方式决定：对象调用、`call/apply/bind`、`new` 和独立调用各有规则；箭头函数没有自己的 `this`，从定义位置的外层词法作用域捕获，不能作为构造函数。

```js
const user = {
  name: 'Qiu',
  say() { return this.name },
}

user.say() // 'Qiu'
const say = user.say
say() // 严格模式下 this 为 undefined
```

箭头函数适合回调中保留外层上下文，不适合需要动态接收者的对象方法。`bind` 返回新函数，`call/apply` 立即调用；手写时核心是临时把函数作为目标对象属性调用，但生产代码应先确认是否真的需要动态 this。

## 五、对象关系：原型链和 class

### P0：原型链和 class 的关系

对象找属性时先看自身，再沿原型链向上查找。`class` 是建立在原型机制上的更清晰语法，实例方法通常放在 `prototype` 上共享：

```js
class User {
  constructor(name) { this.name = name }
  greet() { return `hello, ${this.name}` }
}

const user = new User('Qiu')
user.hasOwnProperty('name') // true
user.hasOwnProperty('greet') // false
Object.getPrototypeOf(user) === User.prototype // true
```

`instanceof` 本质是检查构造函数的 `prototype` 是否出现在对象原型链上。业务中不需要为了“面向对象”强行使用深层继承，组合和纯函数通常更容易维护。

### P1：`new` 做了什么

```js
function createInstance(Constructor, ...args) {
  const instance = Object.create(Constructor.prototype)
  const result = Constructor.apply(instance, args)
  return result !== null && (typeof result === 'object' || typeof result === 'function')
    ? result
    : instance
}
```

`new` 创建对象、连接原型、执行构造函数并决定返回值；若构造函数显式返回对象，则使用该对象，否则返回新实例。

## 六、数组、集合和错误边界

### P1：数组方法怎样避免无意修改共享状态

`push/pop/shift/unshift/splice/sort/reverse` 修改原数组；`map/filter/slice/concat/flat` 返回新数组。排序共享状态前先复制：

```js
const sorted = [...users].sort((a, b) => a.id - b.id)
```

`forEach` 不能用 `break`，也不会等待 async 回调；需要中断或逐项 await 时用 `for...of`。

### P1：Set、Map、WeakMap 如何选择

Set 表示唯一值集合，Map 允许任意类型键并有明确遍历语义；WeakMap 以对象为键且不阻止键对象被回收，适合保存实例附加元数据。键类型、是否需要枚举和生命周期决定选择。

### P1：错误处理应该放在哪一层

请求层把网络、状态码和业务错误转换为可识别错误；页面决定提示、重试或降级；全局监控记录未捕获异常和版本信息。不要无差别 `catch` 后吞掉错误：

```js
async function loadProfile() {
  try {
    return await request('/profile')
  } catch (error) {
    reportError(error, { feature: 'profile' })
    throw error
  }
}
```

## 七、模块化和内存

### P1：ESM 与 CommonJS

ESM 的静态 `import/export` 便于依赖分析和 Tree Shaking；CommonJS 的 `require` 运行时执行，老项目和 Node 环境中仍常见。动态 `import()` 返回 Promise，适合路由和低频功能按需加载。

### P1：垃圾回收与内存泄漏

引擎按可达性判断对象是否可回收。变量离开函数不等于立即释放，只要仍能从全局、闭包、定时器、DOM 监听或缓存找到它，就仍然可达：

```js
function watchResize(onResize) {
  window.addEventListener('resize', onResize)
  return () => window.removeEventListener('resize', onResize)
}
```

组件或页面销毁时调用返回的清理函数，才能断开外部引用。

## 八、面试官想听到的话

### 30 秒版本

> 我会按值、执行和对象关系理解 JavaScript。业务默认使用严格相等，显式处理类型和空值；执行层要理解作用域、执行上下文、闭包和 this，闭包本身不是泄漏，长生命周期引用才可能造成泄漏。JavaScript 一律按值传参，对象值是引用。原型链是 class 的底层机制，函数调用方式决定普通函数的 this，箭头函数继承词法 this。工程上再结合 ESM、错误边界和监听器清理，避免用 any 式的隐式转换和全局引用掩盖问题。

### 2 分钟版本

> 遇到语言题我先说明问题来源。`==` 会隐式转换，`===` 通常更可读，Object.is 处理 NaN 和零符号边界；类型判断还要考虑 null、跨 realm 和输入校验。执行时创建执行上下文，词法作用域决定闭包能访问的环境，普通函数的 this 看调用点，箭头函数看定义位置。参数一律值传递，对象只是传递了引用这个值。对象属性查找沿原型链，class 只是更清晰的原型语法。最后把这些边界落到 Vue：清理监听和定时器、不要直接改 prop、不要依赖隐式转换。这样回答不是背 API，而是在解释代码为什么会产生结果。

## 九、自测与复习卡

1. 为什么 `Object.is(NaN, NaN)` 是 true？
2. 闭包什么时候会变成内存问题？
3. JavaScript 为什么说一律按值传递？
4. 普通函数和箭头函数的 this 差异是什么？
5. class 和原型链是什么关系？
6. 哪些数组方法会修改原数组？

```text
值：显式转换，避免隐式边界
执行：上下文 -> 作用域链 -> 调用栈
函数：普通 this 看调用，箭头 this 看定义
对象：自身属性 -> 原型链；class 建立在原型上
内存：看可达引用，组件卸载要清理外部资源
工程：ESM、错误边界、集合选择服务于真实业务
```
