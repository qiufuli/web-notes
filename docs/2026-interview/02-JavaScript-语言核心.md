# JavaScript 语言核心

> 目标不是把语言特性背成词条，而是能说清楚 JavaScript 在真实业务里最容易出错的边界。

## 知识链

建议按“值如何产生和转换 -> 代码如何执行 -> 函数如何确定上下文 -> 对象如何继承 -> 集合与模块如何组织”复习。面试官通常会从一道类型题追到作用域、闭包、`this`、原型，再落到业务中的状态与内存问题。

## P0：`==`、`===` 和 `Object.is` 有什么区别？

### 30 秒回答

日常业务默认使用 `===`，因为它不做隐式类型转换。`==` 会按一套转换规则比较，只有在我明确要同时匹配 `null` 和 `undefined` 时才会有意识地使用。`Object.is` 和 `===` 基本相同，但它把 `NaN` 视为等于自身，并区分 `+0` 与 `-0`，适合需要精确判断边界值的场景。

### 展开说明

`===` 同时比较类型和值；对象则比较引用。`==` 的隐式转换规则复杂，读代码的人很难一眼确认意图。`Object.is` 是规范意义上的 SameValue 比较，React 等库内部会用它避免 `NaN` 和零符号的边界问题。

```js
0 === -0 // true
Object.is(0, -0) // false

NaN === NaN // false
Object.is(NaN, NaN) // true

// 这是少数可读性较好的 == 用法：同时判空
function isMissing(value) {
  return value == null // true: null 或 undefined
}
```

### 项目表达

接口字段是否存在时，我会区分“未返回”与“返回空字符串、0、false”。不能使用 `if (!value)` 一概判断，否则金额为 `0`、开关为 `false` 都会被误判为缺失。

### 可能追问

- `NaN` 应如何判断？使用 `Number.isNaN`，不要用会先转换类型的全局 `isNaN`。
- 两个内容相同的对象为什么不相等？对象比较的是内存引用，不是结构内容。

## P0：如何判断数据类型？`typeof`、`instanceof` 和 `Object.prototype.toString` 各有什么边界？

### 30 秒回答

基本类型优先用 `typeof`，但 `typeof null` 是历史遗留的 `'object'`；引用类型可用 `instanceof`，但它依赖原型链，跨 iframe/realm 时可能失效；需要更稳定地识别内建对象时可使用 `Object.prototype.toString.call(value)`。实际业务不应只为“判断类型”而判断，应围绕输入边界做校验。

```js
typeof null // 'object'
typeof (() => {}) // 'function'

[] instanceof Array // true
Object.prototype.toString.call(new Date()) // '[object Date]'
Object.prototype.toString.call([]) // '[object Array]'
```

### 可能追问

- `Array.isArray` 为什么比 `instanceof Array` 更合适？它能正确处理跨 realm 的数组。
- `typeof function` 为什么是 `'function'`？函数在 JavaScript 中本质也是对象，但规范对可调用对象给出了特殊的 `typeof` 结果。

## P0：隐式类型转换和对象转原始值的规则是什么？

### 30 秒回答

JavaScript 在运算、比较和字符串拼接时会发生转换。对象转原始值会优先调用 `Symbol.toPrimitive`，没有时再按 hint 尝试 `valueOf` 与 `toString`。面试中不必背完所有表格，但要避免依赖隐式转换写业务逻辑，尤其不能把 `+`、`==` 与对象混用。

```js
const money = {
  [Symbol.toPrimitive](hint) {
    return hint === 'string' ? '¥10' : 10
  },
}

String(money) // '¥10'
money + 2 // 12
```

### 项目表达

接口金额、日期和枚举值先显式转换，再参与计算或展示。表单中把 `'0'` 当成 false、把空字符串当成数值 0，都是常见线上问题来源。

## P0：执行上下文、提升和暂时性死区如何解释？

### 30 秒回答

代码执行时会创建全局、函数或模块执行上下文，其中包含变量环境、词法环境和 `this` 绑定。`var` 声明会在创建阶段初始化为 `undefined`；函数声明可直接调用；`let`、`const` 虽然也会被创建，但在声明前处于暂时性死区，访问会抛错。不要把“提升”理解成代码真的被移动。

```js
console.log(a) // undefined
var a = 1

// console.log(b) // ReferenceError
let b = 2

say() // 'hello'
function say() {
  return 'hello'
}
```

### 可能追问

- `let` 能否重复声明？同一作用域不可以；它避免了 `var` 的意外覆盖。
- 为什么模块顶层和普通 script 不一样？ESM 有自己的模块作用域，顶层声明不会自动成为 `window` 属性。

## P0：参数传递是值传递还是引用传递？

### 30 秒回答

JavaScript 一律按值传递。对象变量中保存的值是对象引用，因此在函数内通过该引用修改对象属性会影响外部对象；但把形参重新赋值为新对象不会改变外部变量本身。用“引用传递”概括会掩盖这个关键区别。

```js
function update(user) {
  user.name = 'new name'
  user = { name: 'another user' }
}

const user = { name: 'old name' }
update(user)
user.name // 'new name'
```

### 项目表达

组件收到对象 prop 时，直接修改嵌套属性会绕开单向数据流。即使技术上能改变引用对象，也应通过事件把更新意图交还父组件，或复制出编辑草稿。

## P0：什么是作用域、作用域链和闭包？闭包会导致内存泄漏吗？

### 30 秒回答

作用域决定标识符在哪里可访问，JavaScript 使用词法作用域，函数定义的位置决定它能访问哪些外层变量。函数连同它仍然需要的外层变量形成闭包。闭包本身不是内存泄漏，只有不再需要的闭包仍被长生命周期对象引用时，相关内存才无法回收。

### 展开说明

闭包常用于封装状态、回调和函数工厂。排查问题时要关注引用链：定时器、事件监听、全局缓存、未结束的请求回调都可能让闭包长期存活。

```js
function createCounter() {
  let count = 0

  return {
    increment() {
      count += 1
      return count
    },
  }
}

const counter = createCounter()
counter.increment() // 1
counter.increment() // 2
```

这里的 `count` 没有被外部直接访问，但返回的方法仍引用它，因此不会在 `createCounter` 执行结束后消失。

### 项目表达

在组件卸载时，我会清理事件监听、定时器、订阅和手工创建的缓存。Vue 的响应式副作用通常会随组件卸载停止，但第三方 SDK 的监听和原生 `window.addEventListener` 仍要自行清理。

### 可能追问

- `var` 在循环里配合异步回调为什么常出问题？它是函数作用域，所有回调可能共享同一个变量；用 `let` 或函数参数创建每轮独立绑定。
- 怎么定位泄漏？用 Chrome Memory 面板对比堆快照，重点看 Detached DOM、监听器和意外的全局引用。

## P0：`this` 的绑定规则是什么？箭头函数为什么不同？

### 30 秒回答

普通函数的 `this` 主要由调用方式决定：对象方法调用指向该对象，`call/apply/bind` 可以显式指定，`new` 调用指向新实例，独立调用在严格模式下是 `undefined`。箭头函数没有自己的 `this`，它在定义时从外层词法作用域继承，因此不能用作构造函数，也不能靠 `call` 改变 `this`。

### 展开说明

不要根据函数“写在哪”判断普通函数的 `this`。要看调用点。将对象方法赋给变量再调用，会丢失原来的接收者。

```js
const user = {
  name: 'Qiu',
  say() {
    return this.name
  },
}

user.say() // 'Qiu'

const say = user.say
say() // 严格模式下 this 为 undefined，不能读取 name

const later = () => user.say()
setTimeout(later, 0) // 箭头函数保留定义处的 user 引用
```

### 项目表达

组件方法、类方法传给第三方回调时，我会明确是否需要绑定上下文。Vue 组件中更常见的选择是把逻辑写成组合式函数，减少依赖隐式 `this` 的代码。

### 可能追问

- `bind` 与 `call/apply` 的区别？`bind` 返回新函数，后两者立即调用。
- `new` 与 `bind` 一起使用时？构造调用的优先级更高，绑定的普通 `this` 会被忽略。

## P1：`call`、`apply`、`bind` 应如何解释，手写时要考虑什么？

### 30 秒回答

三者都能显式指定普通函数的 `this`：`call` 逐个传参并立即调用，`apply` 以数组传参并立即调用，`bind` 返回一个可复用的新函数。手写题的核心是临时把函数作为目标对象属性调用；生产中更应关注是否真的需要依赖动态 `this`。

```js
function callLike(fn, context, ...args) {
  const target = context == null ? globalThis : Object(context)
  const key = Symbol('fn')
  target[key] = fn
  const result = target[key](...args)
  delete target[key]
  return result
}
```

### 可能追问

- 手写 `bind` 为什么更复杂？要处理预置参数、作为构造函数调用时的原型关系和 `this` 优先级。
- 箭头函数能用 `bind` 改 `this` 吗？不能，它没有自身的动态 `this`。

## P0：原型链和 class 的关系是什么？

### 30 秒回答

JavaScript 的继承底层是原型链：对象查找属性时，先查自身，再沿着原型一直向上查。`class` 是建立在原型之上的语法糖，实例方法通常放在类的 `prototype` 上供实例共享，而不是复制到每个实例。

### 展开说明

理解原型链的关键是区分“实例自己的属性”和“原型共享的方法”，也要避免直接修改内置原型造成全局副作用。

```js
class User {
  constructor(name) {
    this.name = name
  }

  greet() {
    return `hello, ${this.name}`
  }
}

const user = new User('Qiu')
user.hasOwnProperty('name') // true
user.hasOwnProperty('greet') // false
Object.getPrototypeOf(user) === User.prototype // true
```

### 项目表达

在业务代码中，我不会为了“面向对象”强行使用继承。组件组合、纯函数和组合式函数通常比深层类继承更易维护。理解原型更重要的价值是排查 `instanceof`、方法覆盖和第三方库行为。

### 可能追问

- `instanceof` 原理？检查构造函数的 `prototype` 是否出现在对象的原型链上。
- `Object.create(null)` 有什么特点？它没有 `Object.prototype`，适合做纯字典，但也没有 `hasOwnProperty` 等方法。

## P1：`new` 做了什么？构造函数显式返回对象会怎样？

### 30 秒回答

`new` 会创建一个新对象、把它的原型关联到构造函数的 `prototype`、以新对象作为 `this` 执行构造函数，并在构造函数没有显式返回对象或函数时返回这个新对象。它解释了 class 实例、原型方法和 `instanceof` 的关系。

```js
function createInstance(Constructor, ...args) {
  const instance = Object.create(Constructor.prototype)
  const result = Constructor.apply(instance, args)
  return result !== null && (typeof result === 'object' || typeof result === 'function')
    ? result
    : instance
}
```

### 可能追问

- `constructor` 属性可靠吗？它通常来自原型，但可被修改，不能作为严格类型判断依据。
- class 是否完全等于构造函数？底层共享原型思想相近，但 class 有严格模式、不可直接调用等语义差异。

## P1：数组常用方法如何区分“改变原数组”和“返回新数组”？

### 30 秒回答

`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse` 会修改原数组；`map`、`filter`、`slice`、`concat`、`flat` 等返回新数组。面试更关心你能否避免无意修改共享状态，而不是死背表格。

```js
const users = [{ id: 2 }, { id: 1 }]

// sort 会直接改变 users，先复制再排序更安全
const sorted = [...users].sort((a, b) => a.id - b.id)
```

### 可能追问

- `map` 为什么会跳过稀疏数组空位？数组迭代方法会按规范跳过不存在的索引，不能把稀疏数组当普通列表。
- `forEach` 与 `for...of`？`forEach` 无法用 `break`/`await` 控制串行流程；`for...of` 更适合可中断或逐项 await 的逻辑。

## P1：Set、Map、WeakMap 如何选择？

### 30 秒回答

Set 表示唯一值集合，Map 允许任意类型键并保持插入顺序，适合缓存或映射；WeakMap 的键只能是对象且不阻止键对象被回收，适合保存对象附加元数据。不要用普通对象硬凑所有字典需求，键类型和枚举需求决定选择。

### 项目表达

简单数组去重使用 `new Set(array)`；根据组件实例保存第三方对象或清理函数时，WeakMap 能避免因附加元数据阻止实例回收。

## P1：错误处理应该如何组织？

### 30 秒回答

错误要在能恢复或补充上下文的边界处理。底层请求层把网络、状态码和业务错误转换为可识别错误；页面决定提示、重试或降级；全局监控记录未捕获异常和版本信息。`try/catch` 不应无差别吞掉错误。

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

## P1：模块化与动态导入在工程里如何使用？

### 30 秒回答

ESM 的静态 `import/export` 便于构建工具分析依赖和 Tree Shaking；`import()` 返回 Promise，适合按路由或低频功能延迟加载。动态导入不能替代合理的模块边界，加载失败也要有错误提示和重试策略。

## P1：ESM 与 CommonJS 如何区分？

### 30 秒回答

ESM 是静态结构，`import/export` 在编译阶段可分析，因此更利于 Tree Shaking；CommonJS 的 `require` 在运行时执行，导出通常是对象引用。现代 Vue/Vite 项目使用 ESM，服务端或老项目可能仍会遇到 CommonJS。

```js
// ESM：静态导入，构建工具可分析依赖图
import { formatDate } from './date.js'

// CommonJS：运行时加载
const { formatDate } = require('./date.cjs')
```

### 项目表达

排查构建包体时，我会先确认依赖是否提供 ESM 版本，是否存在具备副作用的入口文件，再判断 Tree Shaking 为什么没有生效，而不是只盲目配置压缩选项。

## P1：垃圾回收如何工作，业务代码如何避免不必要的内存占用？

### 30 秒回答

JavaScript 引擎以可达性为核心判断对象是否可回收。V8 会分代管理新生代和老生代，并采用标记、清理、压缩等策略；业务开发不需要手动回收，但要主动断开不需要的引用。

### 展开说明

不要把“变量离开函数作用域”简单等同于立即回收。只要仍能从根对象、闭包、定时器、DOM 监听或缓存找到它，就仍然可达。

```js
function watchResize(onResize) {
  window.addEventListener('resize', onResize)

  return () => window.removeEventListener('resize', onResize)
}

const stop = watchResize(() => console.log('resize'))
// 页面或组件销毁时执行，释放外部监听引用
stop()
```

### 可能追问

- `WeakMap` 适合什么场景？以对象为键保存附加信息，键对象不可达后条目可被回收；它不可枚举，不能当普通缓存遍历。
- 为什么大型列表容易占内存？节点、事件和响应式数据都可能持续存在，需要虚拟列表、分页和卸载策略。
