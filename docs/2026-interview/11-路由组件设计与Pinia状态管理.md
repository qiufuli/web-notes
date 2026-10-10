# 路由、组件设计与 Pinia 状态管理

> Pinia 是 Vue 3 项目的常见标配，但状态放进 Store 并不会自动变得合理。先判断状态的所有者、生命周期和复用范围，再选择组件、composable、URL 或 Store。

## 一、先解决“状态应该放在哪里”

一个后台列表页通常同时有搜索条件、分页、弹窗、接口数据、登录用户和权限。全部放组件会透传，全部放 Store 又会让页面互相污染。可以先问四个问题：

```text
谁需要读取？
生命周期有多长？
刷新或分享链接后是否要恢复？
它是服务端数据、用户输入，还是纯 UI 状态？
```

```text
弹窗开关、当前 tab       -> 组件状态
请求、分页、取消逻辑     -> 页面 composable
可分享的筛选与分页       -> route.query
用户、权限、应用配置     -> Pinia
后端列表数据             -> 按复用范围选择页面缓存、Store 或专用数据层
```

状态放得过高会增加同步、清理和调试成本，放得过低会导致层层传参。URL 里不要放密码、Token 等敏感数据，因为它会进入历史、日志和分享链接。

## 二、Pinia 的工作模型

### P0：Pinia 的核心概念是什么？

Pinia Store 可以看作一个有名字、应用级生命周期和 Devtools 支持的共享状态容器：

```text
state：事实数据
getters：由 state 派生的只读结果
actions：修改状态与业务编排，可包含异步
```

```ts
// stores/auth.ts
import { computed, ref } from 'vue'
import { defineStore } from 'pinia'

type Profile = { id: string; name: string; roles: string[] }

export const useAuthStore = defineStore('auth', () => {
  const profile = ref<Profile | null>(null)
  const loading = ref(false)
  const isAdmin = computed(() => profile.value?.roles.includes('admin') ?? false)

  async function loadProfile() {
    loading.value = true
    try {
      profile.value = await request<Profile>('/profile')
    } finally {
      loading.value = false
    }
  }

  function clear() {
    profile.value = null
  }

  return { profile, loading, isAdmin, loadProfile, clear }
})
```

Setup Store 直接使用 `ref`、`computed` 和函数，适合 Vue 3 组合式逻辑；Option Store 的 `state/getters/actions` 更接近 Vuex，迁移团队更容易理解。项目内应形成统一约定，不要无规则混用。

## 三、Store 和 composable 的边界

Store 和 composable 都能封装状态，但默认语义不同：

```text
composable：通常每次调用创建独立状态，适合页面逻辑复用
Pinia Store：应用内按 id 共享，适合跨页面、稳定生命周期的状态
```

请求列表、分页和取消逻辑只被一个页面使用时，composable 往往更轻；当前用户、权限、租户和全局配置才适合 Store。服务端数据还要考虑缓存、失效和重复请求，不能因为“全局共享”就全部塞进 Store。

## 四、Pinia 解构、批量更新与持久化

### P0：Pinia 中如何保持解构后的响应式

直接解构 Store 的 state/getter 会失去响应式连接：

```ts
const store = useCounterStore()
const { count } = store // 可能只是取出当前值
```

使用 `storeToRefs`：

```ts
const { count, double } = storeToRefs(store)
const { increment } = store // action 是稳定函数，可直接解构
```

批量更新可以使用 `$patch`，让变更意图集中，也方便 Devtools 追踪：

```ts
store.$patch({ count: store.count + 1 })
```

持久化不是 Pinia 自动能力。需要明确白名单、版本、过期、迁移和退出清除；登录 Token、权限和用户偏好不能用同一套策略处理，敏感数据也不应无条件写进 localStorage。

## 五、路由守卫为什么不能等于安全

### P0：Vue Router 4 的权限控制怎样设计

路由守卫适合处理进入页面前的认证、页面级权限和必要的前置准备：

```ts
router.beforeEach(async (to) => {
  const auth = useAuthStore()

  if (to.meta.requiresAuth && !auth.profile) {
    try {
      await auth.loadProfile()
    } catch {
      return { name: 'login', query: { redirect: to.fullPath } }
    }
  }

  const role = to.meta.role
  if (typeof role === 'string' && !auth.profile?.roles.includes(role)) {
    return { name: 'forbidden' }
  }

  return true
})
```

认证、路由权限、按钮权限和服务端资源授权是四层问题：

```text
守卫：能否进入页面
组件/指令：能否看到或操作某个 UI
服务端：最终是否允许访问资源和执行动作
```

守卫要排除公开路由，避免未登录跳登录页又被自己拦截；动态路由应由可信权限数据驱动，不要把后端任意字符串直接当作可执行组件路径。

## 六、URL 参数、Store 和页面数据如何协作

```text
/users/:id       资源身份
?page=2&status=x 可分享、可恢复的查询条件
Pinia            跨页面共享且不适合放 URL 的状态
组件/composable  当前页面的短生命周期数据
```

进入页面时要定义唯一数据源。如果 URL 和 Store 都维护一份筛选条件，就必须明确初始化、回退和更新方向，否则刷新、前进后退与手动修改会互相覆盖。

## 七、异步状态、错误和取消要一起设计

一个 Store 不应该只有一个全局 `loading`：

```ts
type Resource<T> = {
  data: T | null
  loading: boolean
  error: string | null
}
```

不同资源最好有独立状态，并设计成功、失败、取消、重试和重复调用行为。页面只需要一个列表时，把这套状态放 composable 通常比 Store 更容易清理。异步请求还要防止旧响应覆盖新筛选，可用 AbortController 或请求版本号。

## 八、组件 API 怎样避免过度通用

组件应暴露最小、语义化的 Props、Events、slot 和样式扩展点，并明确受控/非受控行为。不要为了“通用”堆几十个布尔 prop：变体增加时可以拆组件、使用 slot 或提供一个清晰配置对象。`provide/inject` 适合表单、主题等跨层稳定上下文，不应替代所有全局状态。

## 九、面试官想听到的话

### 30 秒版本

> Pinia 不是所有状态的垃圾桶。我会先按读取范围、生命周期和是否需要刷新/分享来划分：组件负责局部 UI，composable 负责页面逻辑复用，URL 负责可恢复的查询条件，Pinia 负责跨页面共享的用户、权限和应用配置，服务端数据再单独设计缓存和失效。Pinia 由 state、getters 和 actions 组成，直接解构 state/getter 要用 storeToRefs。路由守卫负责体验层的认证和页面权限，服务端仍是最终授权者，还要处理刷新恢复、重定向循环、退出清理和异步错误。

### 2 分钟版本

> 我会先问状态谁读、活多久、是否要分享和是否属于服务端数据。一个列表页的筛选和请求取消通常留在页面 composable；当前用户、权限和租户才进入 Pinia。Store 可以用 Setup 或 Option 形式，actions 负责有边界的业务编排，getters 负责派生结果。使用时不能直接解构 state，否则会丢响应式，要用 storeToRefs；持久化还要有白名单、版本、过期和退出清理。路由层把认证、页面权限和按钮权限分开，守卫不是安全边界，服务端必须再次校验。URL、Store 和组件不要维护同一份真相，进入和离开页面时要明确同步方向。

## 十、自测与复习卡

1. 什么状态应该放 Pinia，什么状态不应该放？
2. Store 和 composable 的默认生命周期有何不同？
3. Pinia 为什么需要 storeToRefs？
4. 路由守卫为什么不能代替服务端鉴权？
5. URL、Store、组件同时保存筛选条件会产生什么问题？
6. 持久化 Store 需要哪些失效和清理策略？

```text
划分依据：读取范围 + 生命周期 + 可分享性 + 数据来源
Pinia：state / getters / actions，共享状态容器
解构：state/getter 用 storeToRefs，action 可直接解构
路由：认证 -> 页面权限 -> UI 权限 -> 服务端授权
异步：data / loading / error / cancel / retry
持久化：白名单、版本、过期、退出清理
```
