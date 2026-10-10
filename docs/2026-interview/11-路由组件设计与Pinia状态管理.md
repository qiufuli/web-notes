# 路由、组件设计与 Pinia 状态管理

> Pinia 是 Vue 3 项目的常见标配，但“所有状态都放 Store”依然是错误设计。先定义状态边界，再选择组件、composable、URL 或 Store。

## P0：组件状态、URL 状态和全局 Store 状态如何划分？

### 30 秒回答

只影响单个组件的临时 UI 状态放组件内；可在同一页面复用的逻辑放 composable；用户需要分享、刷新或前进后退保留的筛选条件可放 URL query；跨路由、跨组件共享且需要稳定生命周期的状态才放 Pinia。服务端数据还要考虑缓存、失效和请求状态，不应和纯 UI 状态混在一起。

### 展开说明

状态放得过高会增加同步、清理和调试成本，放得过低则产生层层透传。判断时可问：谁读它、生命周期多长、刷新后是否要恢复、是否可由其他状态推导、谁有权修改它。

```text
输入框是否展开：组件 state
列表请求、分页、取消逻辑：页面 composable
筛选条件需要可复制链接：route.query
当前用户、权限、全局应用配置：Pinia
后端列表数据：根据复用范围放页面缓存、Store 或专用数据请求层
```

### 项目表达

我会避免把表单每个字段和每个弹窗开关都放全局 Store。状态边界清楚以后，页面跳转残留、刷新不同步和多个模块相互改状态的问题会明显减少。

### 可能追问

- Store 和 composable 都能共享状态，区别是什么？composable 默认每次调用创建独立状态；Store 是有命名、Devtools 和应用级生命周期的共享状态容器。
- URL 参数是否适合保存敏感数据？不适合。URL 会出现在历史、日志和分享链接中。

## P0：Pinia 的核心概念是什么？与 Vuex 有哪些差异？

### 30 秒回答

Pinia 用 `defineStore` 定义 Store，核心是 state、getters 和 actions；actions 可以直接写异步逻辑。它更贴近 Composition API，模块按 Store 自然拆分，TypeScript 推导更好，也不再要求通过 mutation 修改状态。Vuex 在 Vue 2 存量项目中仍可稳定使用，迁移应看项目边界和收益，不是强制替换。

### Setup Store 示例

```ts
// stores/auth.ts
import { computed, ref } from 'vue'
import { defineStore } from 'pinia'

type Profile = {
  id: string
  name: string
  roles: string[]
}

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

### 展开说明

Option Store 写法接近 Vuex，Setup Store 更接近 composable。二者都可以使用，团队应统一约定。Store 内可直接改 state，但仍要保持 action 命名和副作用边界，不能让任何组件随意修改深层对象。

### 项目表达

我的 Store 会按业务领域拆分，例如 auth、permission、app-settings，而不是一个巨大的 `useGlobalStore`。组件读取多个字段时可用 `storeToRefs` 保持响应式；涉及登录退出时由 Store 统一清理 profile、权限和相关缓存。

### 可能追问

- Pinia 数据如何持久化？Pinia 本身不自动持久化，可用插件或手工订阅；要区分非敏感偏好和登录安全数据，并处理版本升级、过期和清理。
- 能否在 Store 里使用 router？可以，但要避免 Store 与路由形成循环依赖；导航意图通常由页面或服务层协调更清晰。

## P0：Vue Router 4 的导航守卫如何设计权限控制？

### 30 秒回答

路由守卫适合处理进入路由前的认证、权限和必要数据准备。前端权限控制主要是体验层，服务端仍必须校验权限。守卫要避免每次跳转都重复请求用户信息、避免重定向循环，并对动态路由加载失败有降级路径。

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

  const requiredRole = to.meta.role
  if (typeof requiredRole === 'string' && !auth.profile?.roles.includes(requiredRole)) {
    return { name: 'forbidden' }
  }

  return true
})
```

### 展开说明

动态路由应由可信的权限数据驱动，并避免把任意后端字符串直接映射为可执行组件路径。首次加载权限、刷新恢复、退出后清除动态路由和 404 兜底都需要明确流程。

### 项目表达

我会把“是否登录”“页面权限”“按钮权限”分层：守卫控制路由进入，组件根据权限标识控制可见或可操作状态，服务端作为最终授权者。这样不会把一个前端 `if` 误当成安全边界。

### 可能追问

- `beforeEach` 为什么容易死循环？未登录跳到登录页时，登录页也被同一条件拦截；需要排除公开路由并验证 redirect 参数。
- 数据请求都放守卫吗？不一定。影响是否能进入页面的必要数据可在守卫处理，页面主体数据通常由组件或路由加载策略处理，便于显示 skeleton 和错误态。

## P1：Store 中的异步与错误状态怎么组织？

### 30 秒回答

Store action 可以处理异步，但要把 loading、data、error 与取消/重试策略设计清楚，不能只写一个 `loading = true`。不同资源最好有独立状态，避免一个全局 loading 让无关页面互相影响。

```ts
type Resource<T> = {
  data: T | null
  loading: boolean
  error: string | null
}

const notices = ref<Resource<Notice[]>>({ data: null, loading: false, error: null })
```

### 项目表达

对于服务端数据，我会根据复用范围评估是否需要 Store；若只属于一个页面，页面 composable 通常更轻。无论放在哪里，都要让错误可展示、可重试，避免静默失败。

## P1：怎样设计可维护的组件 API？

### 30 秒回答

组件应暴露最小、语义化的 Props 和 Events，并把样式覆盖点、插槽和受控/非受控行为设计清楚。不要为了通用性一次暴露几十个布尔 prop；当变体变多时，应拆组件、使用 slot 或引入清晰的配置对象。

### 可能追问

- 什么时候用 provide/inject？适合跨多层组件传递稳定上下文，例如表单或主题；不适合替代所有全局状态，且要防止隐式依赖难追踪。

## P0：Pinia 的 Option Store 和 Setup Store 怎么选？

### 30 秒回答

Option Store 用 `state/getters/actions` 组织，迁移 Vuex 的团队更容易接受；Setup Store 直接使用 ref、computed 和函数，能复用 Composition API 逻辑，TypeScript 推导也自然。选择应以团队约定、测试方式和业务复杂度为准，不要同一项目无规则混用。

```ts
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count += 1
    },
  },
})
```

## P0：Pinia 中如何保持解构后的响应式？

### 30 秒回答

直接解构 Store 会丢失 state/getter 的响应式连接；使用 `storeToRefs` 解构 state 和 getters，actions 可以直接解构，因为它们是稳定函数。需要整体替换或批量更新时，可使用 `$patch`，并保持变更意图集中。

```ts
const store = useCounterStore()
const { count, double } = storeToRefs(store)
const { increment } = store
```

## P0：Pinia 的插件、持久化和 SSR 注意什么？

### 30 秒回答

插件可以添加持久化、审计或通用能力，但持久化必须选择白名单、版本和过期策略，不能把所有 Store 序列化到 localStorage。SSR 场景还要避免把一个用户的 Store 状态泄漏给另一个请求，服务端和客户端需要正确 hydrate。

### 项目表达

登录态、权限和用户偏好要分开处理。退出时清除敏感 Store，持久化插件升级时做版本迁移；如果没有明确需求，宁可不持久化。

## P1：路由参数、query 和 Store 如何协作？

### 30 秒回答

路径参数通常标识资源身份，例如 `/users/:id`；query 适合可分享、可恢复的筛选和分页；Store 适合跨页面共享且不适合放进 URL 的状态。进入页面时定义唯一数据源，避免 URL、Store 和组件各自维护一份互相覆盖。

## P1：如何设计权限模型而不是只做路由拦截？

### 30 秒回答

认证、路由权限、页面元素权限和服务端资源授权是四层问题。路由守卫决定能否进入，按钮指令或组件能力控制可见性，API 服务端校验最终权限和资源归属。前端权限数据还要考虑刷新恢复、过期、租户切换和退出清理。

## P1：Store 如何测试？

### 30 秒回答

纯状态转换和 getter 可直接测试，异步 action 通过请求层边界验证成功、失败、取消和重复调用。测试重点是对外行为和状态结果，不要把测试绑死在 Pinia 内部实现或具体调用次数上。
