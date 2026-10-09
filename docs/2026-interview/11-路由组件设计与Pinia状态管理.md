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
