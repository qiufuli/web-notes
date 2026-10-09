# JavaScript 语言核心

> 目标不是把语言特性背成词条，而是能说清楚 JavaScript 在真实业务里最容易出错的边界。

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
