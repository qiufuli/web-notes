# TypeScript：从基础到 Vue 业务建模

> TypeScript 的核心不是把 JavaScript 写得更复杂，而是把“数据是什么、状态有哪些、组件怎样交互”提前写成可检查的契约。它只能在编译期帮助你，外部数据仍必须经过运行时验证。

## 一、为什么前端需要类型，而不是只靠测试

后台系统里常见一个接口同时被列表、详情、编辑表单和权限判断使用。只写 `string` 或 `any` 时，字段重命名、状态遗漏、接口返回不完整，往往要等到用户点击某条路径才暴露。

```text
后端 DTO / URL / localStorage / 用户输入
  -> 不可信外部数据
  -> 校验与转换
  -> 前端内部领域模型
  -> Props / Store / 组件事件
```

TypeScript 最有价值的地方是在“内部可信边界”上维持关系：订单状态决定可执行动作，组件 prop 决定 emit 参数，分页请求的 item 类型决定列表展示类型。

## 二、`any`、`unknown`、`never`：不可信数据从哪里进来

### P0：三者分别解决什么问题

```ts
function getErrorMessage(error: unknown): string {
  if (error instanceof Error) return error.message
  if (typeof error === 'string') return error
  return '未知错误'
}

function assertNever(value: never): never {
  throw new Error(`未处理的分支: ${String(value)}`)
}
```

- `any`：关闭检查，短期适配可以用，但风险会继续扩散。
- `unknown`：允许接收任意值，但使用前必须收窄，适合接口响应和异常边界。
- `never`：表示不可能出现或不会正常返回，用于穷尽检查。

`void` 只是调用方不使用返回值，函数仍可能正常结束；`never` 则表示函数不会正常返回。

## 三、从“多个布尔值”升级为可判别状态

### P0：联合类型和类型收窄怎样减少非法状态

下面这种状态容易出现矛盾：`loading = false`、`error = null`，但 `data` 也是 null，到底是未加载还是加载失败？

```ts
type LoadState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; message: string }

function getTitle(state: LoadState<{ name: string }>) {
  switch (state.status) {
    case 'idle': return '尚未加载'
    case 'loading': return '加载中'
    case 'success': return state.data.name
    case 'error': return state.message
    default: return assertNever(state)
  }
}
```

`status` 是可判别字段，进入 `success` 分支后 TypeScript 知道有 `data`。新增状态时，`assertNever` 会提醒所有分支需要补齐。这比散落的 `isLoading/hasError/hasData` 更能表达业务状态。

## 四、可选、只读、字面量和引用边界

```ts
type OrderStatus = 'pending' | 'paid' | 'cancelled'

interface Order {
  readonly id: string
  status: OrderStatus
  remark?: string
}
```

`remark?: string` 表示属性可以不存在；`remark: string | undefined` 表示属性存在但值可能是 undefined，序列化和 `in` 判断上并不完全相同。`readonly` 是编译期约束，不会在运行时冻结对象。

对象展开也只是浅拷贝：

```ts
const next = { ...order }
```

嵌套对象仍可能共享引用，类型系统不会自动替你实现不可变数据。

## 五、函数、泛型和请求层：保持输入输出关系

### P0：泛型为什么比 `any` 更有价值

```ts
type ApiResponse<T> = {
  code: number
  message: string
  data: T
}

async function request<T>(path: string): Promise<T> {
  const response = await fetch(`/api${path}`)
  const body = (await response.json()) as ApiResponse<T>
  if (body.code !== 0) throw new Error(body.message)
  return body.data
}

type User = { id: string; name: string }
const user = await request<User>('/users/current')
```

泛型表达的是“输入类型和输出类型之间的关系”，不是运行时验证。上面的 `as ApiResponse<T>` 只是告诉编译器如何看待 JSON，服务端字段变化仍可能造成运行时错误，因此请求适配层要配合校验或转换。

普通业务函数优先一个清晰签名；重载适合公开 API 中同一函数根据输入返回不同类型，不能为了显得高级而增加复杂度。

## 六、`keyof`、索引访问和工具类型怎样防止字段写错

```ts
type User = { id: string; name: string; enabled: boolean }

function getField<T, K extends keyof T>(item: T, key: K): T[K] {
  return item[key]
}

getField<User>({ id: '1', name: 'Qiu', enabled: true }, 'name')
```

`keyof T` 得到属性键联合，`T[K]` 保持字段和值的对应关系。常用工具类型服务于边界：

```ts
type User = {
  id: string
  name: string
  role: 'admin' | 'member'
  passwordHash: string
}

type UpdateUserInput = Partial<Pick<User, 'name' | 'role'>>
type UserView = Omit<User, 'passwordHash'>
type UserById = Record<string, UserView>
```

`Partial` 适合部分更新，`Pick` 和 `Omit` 表达公开字段边界，`Record` 表示字典。工具类型过度嵌套会让错误难读，领域模型应优先直观。

## 七、Vue 组件为什么要把输入输出类型化

### P0：Props、Emits 和模型边界

```vue
<script setup lang="ts">
type QueryParams = {
  keyword: string
  page: number
}

const props = withDefaults(defineProps<{
  modelValue: QueryParams
  pageSize?: number
}>(), {
  pageSize: 20,
})

const emit = defineEmits<{
  'update:modelValue': [value: QueryParams]
  search: [params: QueryParams]
}>()

function submit() {
  emit('search', props.modelValue)
}
</script>
```

类型化 Props/Emits 的价值不是让代码更长，而是把父子组件契约固定下来：名称拼错、事件参数不对、可选 prop 未处理，会在开发阶段暴露。组件不要直接依赖整个页面 Store；输入、输出和受控关系越明确，越容易复用和测试。

## 八、外部数据为什么还要运行时校验

```ts
function isUser(value: unknown): value is { id: string; name: string } {
  return typeof value === 'object'
    && value !== null
    && typeof (value as Record<string, unknown>).id === 'string'
    && typeof (value as Record<string, unknown>).name === 'string'
}
```

接口响应、URL 参数、localStorage 和第三方 SDK 都是在 TypeScript 编译后才到达的值，编译器无法替你检查。项目可以使用校验库，也可以在适配层手写守卫，然后将不可信 DTO 转为内部领域模型。

## 九、高级类型什么时候值得用

映射类型按已有键批量变换，条件类型根据关系选择结果，`infer` 提取局部类型：

```ts
type Nullable<T> = { [K in keyof T]: T[K] | null }
type ElementOf<T> = T extends Array<infer Item> ? Item : never
```

它们适合组件库、请求库和复杂表单工具。普通业务模型如果用一个直观 interface 就能表达，不要为了展示类型技巧而引入高阶类型。

## 十、`type`、`interface`、enum 与 `as const`

`interface` 适合可扩展对象契约和声明合并；`type` 更适合联合、元组、映射和条件类型。两者在普通对象建模上都能用，团队统一即可。

前端常量通常可以用 `as const` 加字面量联合：

```ts
const Roles = ['admin', 'member'] as const
type Role = (typeof Roles)[number]

const roleLabels: Record<Role, string> = {
  admin: '管理员',
  member: '成员',
}
```

它不产生额外运行时代码，和接口字符串更容易互操作；enum 是否使用要看项目编译目标和团队约定。

## 十一、tsconfig 和 Vue 2 渐进引入

`strict`、`strictNullChecks`、`noImplicitAny` 能把真实边界暴露出来；`noUncheckedIndexedAccess` 会让索引访问更谨慎。`paths` 只影响 TypeScript 解析，Vite、测试工具和 ESLint 也必须配置相同别名。

给 Vue 2 项目引入 TS 不建议一次改完：先从工具函数、接口模型、请求适配层和新模块开始，统计 `any` 与编译错误的类型，逐步建立适配边界。Vite 转译成功不等于类型检查通过，应在 CI 中单独运行 `vue-tsc` 或 `tsc --noEmit`。

## 十二、面试官想听到的话

### 30 秒版本

> TypeScript 的价值是把数据、状态和组件边界提前变成可检查的契约。`unknown` 适合接住外部不可信数据，经过收窄后才能使用；`any` 会绕过检查，`never` 常用于穷尽分支。联合类型和可判别字段适合表达加载、错误和业务状态，泛型用于保持输入输出关系，`keyof`、工具类型和 Props/Emits 类型可以减少字段和组件契约错误。类型断言只影响编译器，接口和 localStorage 等外部数据仍需要运行时校验。Vue 2 项目应从请求层和新模块渐进引入，而不是全局用 any。

### 2 分钟版本

> 我会先区分不可信边界和内部模型。接口、URL 和 localStorage 先以 unknown 接收并校验，再转换为前端领域模型；内部状态用可判别联合表达互斥状态，避免多个布尔值组合出非法情况。泛型用于保持请求 data 类型和调用方之间的关系，keyof、索引访问和 Pick/Omit/Partial 等工具类型用于让字段与更新边界保持一致。Vue 组件中重点给 Props、Emits、v-model 和 Store 接口类型化，组件不要直接依赖整个页面状态。最后要说明 TypeScript 只做编译期检查，类型断言不等于运行时验证；遗留 Vue 2 项目会按收益从适配层和新模块渐进收敛。

## 十三、自测与复习卡

1. 为什么外部 JSON 不能直接断言成接口类型？
2. `unknown` 和 `any` 的风险差异是什么？
3. 可判别联合怎样减少非法状态？
4. 泛型和 `any` 的核心差异是什么？
5. `keyof T` 与 `T[K]` 如何保持字段关系？
6. `readonly` 和运行时冻结有什么区别？

```text
边界：外部 unknown -> 校验/转换 -> 内部可信模型
状态：联合类型 + 判别字段 + 穷尽检查
关系：泛型、keyof、T[K] 保持输入输出与字段对应
组件：Props / Emits / v-model 类型化
高级：映射/条件类型服务于真实复用
工程：strict + vue-tsc，渐进引入，不用 any 掩盖问题
```
