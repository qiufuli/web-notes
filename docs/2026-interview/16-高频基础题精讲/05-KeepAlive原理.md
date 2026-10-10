# KeepAlive 原理：组件切走后，到底保留了什么

> `KeepAlive` 不是“缓存页面”这么简单。要先回答一个实际问题：用户从列表进详情，再返回列表时，为什么筛选条件、滚动位置和局部状态总是丢失？

## 一、先看没有 KeepAlive 时发生了什么

动态组件或路由组件正常切换时，Vue 会把旧组件卸载，再创建新组件：

```vue
<component :is="currentView" />
```

假设用户在“订单列表”输入了筛选词、滚动到第 5 屏，然后进入详情页。切走时列表组件被卸载：组件实例、局部 `ref`、子组件和 DOM 都会随之销毁。回来时创建的是一个新实例，因此需要重新请求、重新初始化，滚动位置也回到初始值。

这并非 Vue 做错了，而是普通组件生命周期本来就应该如此：**不再渲染的组件可以释放资源。**

但有些页面切换非常频繁，用户期待“回去还是刚才那个现场”。`KeepAlive` 就是在“保留现场”和“释放资源”之间提供一个可控的选择。

## 二、KeepAlive 到底解决什么问题

```vue
<KeepAlive>
  <component :is="currentView" />
</KeepAlive>
```

它让组件离开当前活动视图时，不立即销毁实例；下次切回同一个组件身份时，复用之前保存的实例和子树，而不是重新创建。

你可以先把它理解成下面的状态转换：

```text
普通组件：创建 -> 挂载 -> 卸载 -> 销毁 -> 下次重新创建
KeepAlive：创建 -> 挂载 -> 失活（缓存） -> 激活（复用） -> 最终销毁
```

它适合保留的通常是“用户在这个页面已经形成的局部现场”：

- 表单已输入但未提交的内容
- 列表筛选、分页、展开项
- 页面内滚动位置
- 局部组件状态与已建立的组件树

它不应该被当作通用数据缓存或路由权限方案。

## 三、它缓存的是什么，不缓存什么

Vue 内部会缓存对应的组件 VNode 和组件实例。切出时，组件没有按普通路径卸载；它从活动渲染树中移走或停用，实例仍可在缓存中等待复用。实现细节会因 Vue 版本而异，但面试中抓住“实例可复用、不是简单保存 HTML 字符串”即可。

因此要刻意分开三件事：

| 问题 | KeepAlive 是否天然解决 | 正确做法 |
| --- | --- | --- |
| 返回时表单、局部状态还在 | 是 | 选择性缓存组件 |
| 接口数据是否仍然新鲜 | 否 | 设计过期时间、版本或主动刷新 |
| 定时器、订阅是否继续工作 | 可能继续 | 在失活时明确暂停或清理 |
| 浏览器 HTTP 缓存是否命中 | 否 | 看 HTTP 缓存策略 |

最常见的误解是：“页面没重新请求，所以 KeepAlive 缓存了接口”。更准确的解释是：组件实例和内存中的数据还在；如果你要保证数据新鲜，仍需定义刷新策略。

## 四、生命周期为什么会变

普通组件离开时走 `unmounted`（Vue 2 中是 `destroyed`）路径。被 KeepAlive 管理的组件切出时，会进入 deactivated；再次进入时是 activated。

```text
首次进入：setup / created -> mounted -> activated
切出：deactivated
再次回来：activated
缓存被淘汰或父级销毁：unmounted
```

在 Vue 3 组合式 API 中：

```ts
import { onActivated, onDeactivated, onUnmounted } from 'vue'

onActivated(() => {
  // 页面恢复可见；可判断数据是否过期后刷新
})

onDeactivated(() => {
  // 页面不再可见；暂停轮询、视频、地图等高成本资源
})

onUnmounted(() => {
  // 真正销毁时做最终清理
})
```

这里的直觉是：`deactivated` 不是“组件死了”，只是“先从当前画面退场，实例仍在”。所以不能只把资源清理写在 `onUnmounted`，否则缓存期间轮询、事件监听或第三方实例可能持续占用资源。

## 五、缓存命中靠什么：组件身份和 key

缓存不是按“看起来长得一样”判断，而是根据组件身份和 key 区分。可以把它理解成：

```text
缓存键 = 组件类型 + key（概念化理解）
```

因此：

```vue
<RouterView v-slot="{ Component, route }">
  <KeepAlive>
    <component :is="Component" :key="route.name" />
  </KeepAlive>
</RouterView>
```

是否使用这个 `key`、该用什么 key，要由产品行为决定。

- key 不变：同类路由可能复用同一份组件状态。
- key 变了：Vue 会认为是不同身份，可能产生新的缓存实例。
- key 包含随时变化的参数：可能让缓存不断膨胀，或让“保留现场”失效。

例如详情页 `/order/1` 和 `/order/2` 是否要共享组件现场，是业务选择。不要为了“强制刷新”随手加随机 key；那会绕过缓存，也会让问题更难排查。

## 六、`include`、`exclude` 和 `max` 分别控制什么

```vue
<KeepAlive include="OrderList,UserList" :max="8">
  <RouterView />
</KeepAlive>
```

- `include`：只缓存名字匹配的组件。
- `exclude`：名字匹配的组件不缓存。
- `max`：缓存实例数上限。超过上限会淘汰较久未使用的缓存项，行为可以理解为 LRU 风格。

这里匹配的是**组件名称**，不是路由的 `meta.title`。使用 `script setup` 或路由懒加载时，要确认组件实际 name 与规则能对应上。排查“为什么没缓存”时，优先检查组件名、插槽结构、key 和 `include/exclude`，不要只盯着路由地址。

## 七、一个更接近业务的写法

订单列表需要保留筛选与滚动，但接口数据超过 60 秒要刷新：

```ts
const lastLoadedAt = ref(0)
let timer: ReturnType<typeof setInterval> | undefined

async function loadOrders() {
  await api.getOrders()
  lastLoadedAt.value = Date.now()
}

onActivated(async () => {
  if (Date.now() - lastLoadedAt.value > 60_000) {
    await loadOrders()
  }
  timer = setInterval(loadOrders, 30_000)
})

onDeactivated(() => {
  if (timer) clearInterval(timer)
  timer = undefined
})
```

示例要表达的不是“所有页面都轮询”，而是职责分离：

```text
KeepAlive：保留组件现场
activated：恢复可见时决定是否刷新
deactivated：停止不该在后台继续工作的资源
业务策略：决定数据何时过期
```

全局 Store 的订阅、WebSocket、地图 SDK、视频和 document 事件，也要按相同思路审视：页面失活时它们是否还应该继续工作？

## 八、什么时候不该用 KeepAlive

- 页面很少返回，重新创建成本低。
- 页面含有大量 DOM、图表或第三方实例，长期保存内存代价高。
- 数据必须每次进入都重新校验，且保留局部状态没有业务价值。
- 多个身份的详情页会无限打开，但没有明确上限和淘汰策略。

缓存不是免费的。一个缓存组件仍然持有响应式数据、子组件、可能的 DOM 关联和外部资源。正确问题不是“能不能缓存”，而是“保存哪些现场、保存多久、何时失效、最多保留多少”。

## 九、常见错误回答

### 错误一：“KeepAlive 缓存 DOM，所以接口不再请求”

它核心缓存的是组件实例和 VNode 子树。接口是否请求由你的生命周期逻辑和数据策略决定。

### 错误二：“切走后 `onUnmounted` 会执行”

被缓存的组件通常先触发 `onDeactivated`，并没有真正卸载。只有被淘汰、父级销毁等情况下才会最终 unmount。

### 错误三：“缓存越多，体验越好”

体验和内存、旧数据、后台任务之间有取舍。应当精确选择页面，并设置限制和清理逻辑。

## 十、面试官想听到的话

### 30 秒版本

> KeepAlive 用于缓存动态组件或路由组件的实例与子树。组件切出时不立即销毁，而是进入 deactivated；再次进入触发 activated 并复用原实例，所以筛选条件、表单草稿和局部状态可以保留。它不等于接口缓存，我会在 activated 中按过期策略决定是否刷新数据，在 deactivated 中暂停轮询或第三方资源，并通过组件 name 的 include/exclude、max 和稳定 key 控制缓存范围与数量。

### 2 分钟版本

> 正常切换组件会卸载旧实例，返回时重新创建，所以列表筛选和滚动状态会丢。KeepAlive 把对应组件实例和 VNode 缓存起来，切出时从活动树中停用而不是销毁，回来时激活复用。关键是区分三件事：它能保留组件局部状态，但不会天然保证接口数据新鲜，也不会自动停止定时器和订阅。因此缓存组件应使用 activated/deactivated 管理刷新和暂停逻辑。缓存命中还依赖组件身份和 key，include/exclude 匹配组件 name，max 用来限制数量。实际使用时我会只缓存确实需要“返回现场”的列表或表单页面，而不是把所有路由都包起来。

## 十一、自测与复习卡

1. 普通组件切走与 KeepAlive 组件切走的生命周期有什么不同？
2. 为什么 KeepAlive 不能替代接口缓存策略？
3. 哪些资源应在 `onDeactivated` 处理？
4. key 不稳定为什么会导致缓存问题？
5. `include` 匹配的是路由名还是组件名？

```text
根问题：切页后是否要保留用户刚才的现场
普通：unmount，回来重新创建
KeepAlive：deactivated，回来 activated 复用
缓存：组件实例 + 子树，不是 HTTP / 接口缓存
控制：组件 name、include/exclude、max、稳定 key
工程重点：数据失效策略 + 后台资源暂停 + 内存上限
```
