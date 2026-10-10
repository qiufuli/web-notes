# Vue 双向绑定与响应式原理：数据为什么能自己更新到页面

> 这一题最容易背成“Vue 2 是 `defineProperty`，Vue 3 是 Proxy”。这只回答了“用什么拦截”，没有回答“为什么读一次数据就知道以后该更新谁”。要从页面如何随着状态变化开始讲。

## 一、这道题真正要解决什么问题

写页面时，我们希望表达的是业务关系：

```vue
<p>{{ user.name }}</p>
```

当 `user.name` 改了，文字应该更新。如果没有框架，开发者需要在每个修改数据的地方手动找元素并更新 DOM：

```js
state.user.name = '李雷'
document.querySelector('#name').textContent = state.user.name
```

小页面可以这样做；状态、组件和派生值一多，就会出现两个问题：

1. 一个状态变化会影响哪些界面，容易漏更新。
2. 多处状态变化时，重复操作 DOM，难以维护也浪费性能。

Vue 响应式要解决的就是：**让声明“我依赖什么”的渲染逻辑，能在它依赖的数据变化后被准确地重新执行，并把 DOM 更新合并起来。**

## 二、先分清三个概念：单向数据流、双向绑定、响应式

这三个词经常被混着说。

### 1. 单向数据流：谁拥有数据，谁负责改数据

组件间通常是父组件向子组件传入状态，子组件通过事件表达“我希望改变它”：

```vue
<UserEditor :name="name" @save="name = $event" />
```

这样数据来源清楚，子组件不会偷偷改父组件的状态。

### 2. 双向绑定：把“传值”和“回传事件”写短

原生输入框的 `v-model` 近似等价于：

```vue
<input :value="keyword" @input="keyword = $event.target.value" />
```

组件上的 `v-model` 在 Vue 3 中近似等价于：

```vue
<SearchBox
  :model-value="keyword"
  @update:model-value="keyword = $event"
/>
```

子组件实现的是：

```vue
<script setup>
defineProps({ modelValue: String })
const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)" />
</template>
```

所以双向绑定不是“子组件直接改 prop”，仍然是明确的单向数据流加一个反向事件。

### 3. 响应式：状态改后，依赖它的逻辑能被通知

双向绑定只是一个常见使用场景。`computed`、`watchEffect`、组件渲染都依赖同一件事：读取状态时建立关系，修改状态时通知相关逻辑。

---

## 三、先用最小模型理解依赖收集

假设有一个渲染函数：

```js
function render() {
  titleElement.textContent = state.title
}
```

问题是：`state.title` 改变后，框架怎么知道应该重新执行 `render`，而不是刷新整个应用？

答案是两步：

```text
读取 title 时：记录“render 依赖 title”
修改 title 时：找到依赖 title 的 render，再让它重新运行
```

用极简伪代码看这个过程：

```js
let activeEffect
const subscribers = new Set()

function effect(fn) {
  activeEffect = fn
  fn()                 // 运行时会读取 state.title
  activeEffect = null
}

const state = {
  _title: '首页',
  get title() {
    if (activeEffect) subscribers.add(activeEffect)
    return this._title
  },
  set title(value) {
    this._title = value
    subscribers.forEach(fn => fn())
  },
}

effect(() => {
  titleElement.textContent = state.title
})

state.title = '订单页'
```

真实 Vue 当然复杂得多：它要按“对象 + key”分别保存依赖，要处理嵌套 effect、条件分支、清理旧依赖、组件调度和异常。但核心因果关系就是这一句：**读时收集，写时触发。**

## 四、为什么不能“数据一变就刷新整个页面”

若任何状态变化都从根节点重新渲染，页面越大，做的无效工作越多。例如筛选条件变化并不意味着侧边栏头像也要重新计算。

依赖收集让 Vue 知道：

```text
搜索结果组件读取了 keyword 和 page
用户信息组件读取了 user.name
总价计算读取了 cart.items
```

当 `keyword` 改变，只需要让依赖 `keyword` 的渲染 effect 或计算 effect 失效、重新调度；而不是盲目刷新所有组件。这是响应式系统的价值，不是“自动更新”四个字本身。

## 五、Vue 2：为什么是 `Object.defineProperty`

Vue 2 需要在初始化时把已有的 `data` 属性变成带 getter/setter 的属性：

```js
let value = 0

Object.defineProperty(state, 'count', {
  get() {
    // 当前 watcher 读取 count，建立依赖
    return value
  },
  set(next) {
    value = next
    // 通知依赖 count 的 watcher 更新
  },
})
```

其中可以这样理解：

```text
Watcher：一段会重新执行的工作，例如组件渲染、computed、watch
Dep：某个响应式属性的“订阅者集合”
getter：当前 Watcher 读该属性时订阅
setter：属性变化时通知这些 Watcher
```

### Vue 2 的边界从哪里来

`defineProperty` 是给“某个已经存在的属性”装 getter/setter。因此它天然看不见以后才加上的普通属性：

```js
this.user.age = 18 // Vue 2 中如果 age 初始化时不存在，无法自动侦测
```

数组的 `arr[1] = value`、直接修改 `length` 也不会经过预先安装的普通属性 setter，所以 Vue 2 对数组变更方法做了额外处理，并提供 `$set` / `Vue.set` 作为补充。不要把它背成“Vue 2 数组不响应式”；准确说法是：**它无法侦测某些新增属性和直接按索引赋值的写法。**

## 六、Vue 3：为什么 Proxy 改善了这些问题

Vue 3 不再给每个已知属性单独装访问器，而是代理整个对象：

```js
const state = new Proxy({ count: 0 }, {
  get(target, key, receiver) {
    track(target, key)
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    const oldValue = target[key]
    const result = Reflect.set(target, key, value, receiver)
    if (oldValue !== value) trigger(target, key)
    return result
  },
})
```

Proxy 可以拦截对象级别的读取、设置、新增、删除、`in`、遍历等操作，因此新增属性和数组索引变化也能被追踪。Vue 3 内部可以概念化为：

```text
targetMap
  某个原始对象
    某个 key
      依赖这个 key 的多个 effect
```

当模板读取 `state.count`，执行 `track(state, 'count')`；赋值时执行 `trigger(state, 'count')`。只会通知真正读过这个 key 的 effect。

### `reactive` 和 `ref` 为什么都存在

Proxy 只能代理对象，不能代理 `1`、`'hello'` 这样的基本值。因此 Vue 用 `ref` 把基本值放进一个带 `.value` 的对象里：

```ts
const count = ref(0)
const user = reactive({ name: '小王' })

count.value += 1
user.name = '小李'
```

模板会自动解包部分 ref，但 JavaScript 代码里要理解 `.value` 是真实的依赖访问点。选择上不必教条：单个值常用 `ref`，有内聚字段的对象常用 `reactive`，关键是不要因为解构而丢失响应式连接。

## 七、为什么解构会“丢响应式”

看这一段：

```ts
const state = reactive({ count: 0 })
const { count } = state

console.log(count) // 只是此刻取出的普通 number
```

解构时调用了一次代理的 getter，拿到的 `count` 是基本值本身。此后读取局部变量，不会再经过 `state.count` 的 Proxy getter，自然无法 track。

需要保留响应式引用时使用：

```ts
const state = reactive({ count: 0 })
const { count } = toRefs(state)

count.value += 1
```

它不是“Proxy 失效”，而是你绕开了 Proxy 属性访问。

## 八、`computed`、`watch`、组件更新各在做什么

### `computed`：有缓存的派生状态

```ts
const firstName = ref('张')
const lastName = ref('三')
const fullName = computed(() => `${firstName.value}${lastName.value}`)
```

它的 getter 有自己的 effect。依赖变化时，Vue 先把 computed 标为“脏”；真正下次有人读取 `fullName.value` 时才重新计算。多次读取但依赖未变，会复用缓存。

### `watch`：状态变化后做副作用

```ts
watch(keyword, async value => {
  await loadUsers(value)
})
```

请求、写本地存储、埋点、同步第三方组件都是副作用。它们不是模板中应直接计算出来的值，因此用 `watch` 表达更合适。异步请求还应处理取消或版本校验，避免旧响应覆盖新输入。

### 组件渲染：一种可调度的 effect

组件模板会编译为渲染函数。渲染时读到了哪些响应式数据，就依赖哪些数据；变更后 Vue 调度这次组件更新，再通过虚拟 DOM 比较和 patch 更新真实 DOM。

## 九、为什么 DOM 不会在赋值后一行立刻更新

看下面代码：

```ts
count.value++
count.value++
console.log(element.textContent)
```

如果每次赋值都同步 patch DOM，同一段同步代码会渲染两次。Vue 通常把组件更新放入队列，同一轮只执行一次；这就是批量更新。

```ts
count.value++
await nextTick()
console.log(element.textContent) // 读取这一轮更新后的 DOM
```

`nextTick` 的目的不是“让代码晚一点”，而是等待 Vue 当前这轮调度完成。它和浏览器事件循环有关，但面试时不要武断说成“Vue 的 nextTick 永远就是 Promise”；核心是 Vue 管理自己的更新队列。

## 十、常见错误回答

### 错误一：“双向绑定就是数据双向流动”

不准确。`v-model` 是 prop 下传与事件上报的语法糖，数据所有权仍然清晰。

### 错误二：“Proxy 可以监听一切变化”

它能改善对象操作的侦测范围，但解构、原始对象引用、第三方对象、深层遍历成本和副作用清理仍要开发者处理。

### 错误三：“computed 和 watch 都是监听数据变化”

computed 的目标是得到派生值并缓存；watch 的目标是响应变化做副作用。它们解决的问题不同。

## 十一、面试官想听到的话

### 30 秒版本

> Vue 的响应式核心是读时收集依赖、写时触发依赖。组件渲染、computed 和 watch 都可以看作 effect：它们读取某个状态时订阅该状态，状态变更时 Vue 只调度相关 effect。Vue 2 用 `Object.defineProperty` 劫持已有属性，所以新增属性和部分数组写法有边界；Vue 3 用 Proxy 从对象层面拦截 get/set 等操作，能更自然地处理新增、删除和数组索引。`v-model` 本质是值通过 prop 下传、更新通过事件回传，Vue 还会批量更新 DOM，需要读更新后 DOM 时使用 nextTick。

### 2 分钟版本

> 我会先区分双向绑定和响应式。双向绑定只是 `v-model` 把“传值 + 更新事件”组合起来，子组件不会直接修改父组件 prop。响应式真正解决的是：模板或计算逻辑读过哪些状态，状态变化后就重新运行哪些逻辑。Vue 2 在 getter 里收集 Watcher、setter 里通过 Dep 通知 Watcher；由于它只能给初始化时已有属性装访问器，新增属性和直接数组索引赋值有边界。Vue 3 用 Proxy，在 get 时 track、set 时 trigger，并按对象和 key 精确保存 effect。组件更新不是立即改 DOM，而是先入队合并，再 patch，所以连续赋值通常只更新一次。项目里我还会根据目的区分 computed 的缓存派生值和 watch 的请求、埋点等副作用。

## 十二、自测与复习卡

1. 为什么只说“Proxy 劫持数据”还不足以解释 Vue 更新？
2. `v-model` 为什么不违反单向数据流？
3. Vue 2 新增属性为什么有边界？
4. 解构 `reactive` 为什么可能失去响应式？
5. `computed` 缓存的依据是什么？
6. `nextTick` 等待的到底是什么？

```text
根问题：状态变了，谁需要重新执行？
核心：读时 track / 订阅，写时 trigger / 通知
Vue 2：defineProperty，已有属性为主
Vue 3：Proxy，按对象 + key 追踪
v-model：prop 下传 + update 事件回传
computed：派生值 + 缓存
watch：状态变化后的副作用
DOM：更新入队合并，nextTick 等待本轮刷新
```
