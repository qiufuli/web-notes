# TypeScript：从基础到 Vue 业务建模

> TypeScript 的价值是让组件边界、接口数据和状态变化更早暴露错误，不是把所有代码写成复杂类型。

## 知识链

从值的基础类型开始，进入对象和函数契约、联合类型收窄、泛型抽象，再进入映射/条件类型、声明文件和工程配置。每一层都先解决业务建模问题，只有确实需要推导关系时才进入高级类型。

## P0：`any`、`unknown` 和 `never` 有什么区别？

### 30 秒回答

`any` 会绕过类型检查，适合作为临时边界但会把风险扩散；`unknown` 可以接收任意值，但使用前必须缩小类型，适合外部输入和错误处理；`never` 表示不可能出现的值或不会正常返回的函数，常用于穷尽检查。业务代码应优先用 `unknown` 接住不可信数据，再校验或收窄。

### 展开说明

接口响应、`catch` 捕获的异常、第三方回调都不应该被默认信任。TypeScript 只在编译期工作，运行时数据校验仍需要后端约束、校验库或手工判断。

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

### 项目表达

我不会为了消除报错直接写 `as any`。先确认数据来源：内部模型可以补完整类型，外部接口需做校验或转换，历史库则可以在适配层集中处理，避免 `any` 进入组件树。

### 可能追问

- `unknown` 为什么不能直接赋给具体类型？因为它代表尚未验证的值，必须先做类型守卫。
- `never` 和 `void` 的区别？`void` 表示调用方不使用返回值，函数仍可能正常结束；`never` 表示函数不会正常返回。

## P0：基础类型、字面量类型、可选属性和只读属性如何用于模型？

### 30 秒回答

基础类型描述值的种类，字面量类型把可选值限制为固定集合；`?` 表示属性可能缺失，`readonly` 限制通过当前类型写入。它们适合表达接口的真实约束，例如角色、订单状态、只读 ID，而不是把所有字段写成 `string`。

```ts
type OrderStatus = 'pending' | 'paid' | 'cancelled'

interface Order {
  readonly id: string
  status: OrderStatus
  remark?: string
}
```

### 可能追问

- 可选属性和 `T | undefined` 一样吗？不完全一样。前者属性可以不存在，后者属性存在但值可能是 undefined；序列化和 `in` 判断时有差异。
- `readonly` 会在运行时冻结对象吗？不会，它是编译期约束；运行时不可变需要 `Object.freeze` 或设计上的不可变更新。

## P0：联合类型和类型收窄怎样用于业务状态？

### 30 秒回答

联合类型表示一个值可能是多个明确形态，类型收窄通过 `typeof`、`in`、`instanceof` 或可判别字段把范围缩小。加载状态、接口结果和表单步骤等业务状态特别适合可判别联合，避免用多个容易矛盾的布尔值表示状态。

```ts
type LoadState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; message: string }

function getTitle(state: LoadState<{ name: string }>) {
  switch (state.status) {
    case 'idle':
      return '尚未加载'
    case 'loading':
      return '加载中'
    case 'success':
      return state.data.name
    case 'error':
      return state.message
    default:
      return assertNever(state)
  }
}

function assertNever(value: never): never {
  throw new Error(`unexpected state: ${String(value)}`)
}
```

### 项目表达

相比 `isLoading`、`hasError`、`data` 同时存在的散乱状态，可判别联合能让非法组合更难出现。新增一个状态时，`switch` 的穷尽检查会提醒所有展示分支同步更新。

### 可能追问

- `type` 和 `interface` 怎么选？对象扩展、声明合并等场景常用 interface；联合、映射、条件类型通常必须用 type。团队应保持一致，没必要把选择上升为绝对规则。

## P0：函数类型、重载和 `this` 参数怎么使用？

### 30 秒回答

函数类型应描述参数、返回值和可选/默认参数，而不是只让 TypeScript 推断。重载适合“同一个函数根据不同输入有不同返回类型”的公开 API；实现签名要覆盖所有重载。`this` 参数是仅用于检查的伪参数，可防止回调误用上下文。

```ts
function parse(value: string): Date
function parse(value: number): Date
function parse(value: string | number) {
  return new Date(value)
}

function visit(this: void, callback: () => void) {
  callback()
}
```

### 项目表达

普通业务函数优先保持一个清晰签名。只有调用方确实需要不同输入输出关系时才使用重载，否则联合类型往往更简单。

## P0：`type`、`interface`、交叉类型和索引签名如何选择？

### 30 秒回答

`interface` 适合可扩展对象契约和声明合并；`type` 更适合联合、元组、映射和条件类型。交叉类型表示同时满足多个约束，但属性冲突可能得到 `never`；索引签名适合键未知的字典，但不能因此放弃已知字段的精确类型。

```ts
interface PageInfo {
  page: number
  pageSize: number
}

type ListResponse<T> = PageInfo & {
  list: T[]
  total: number
}

type FeatureFlags = Record<string, boolean>
```

### 可能追问

- interface 为什么能合并？同名 interface 声明会合并成员，这是声明扩展能力；type 同名会报重复声明。

## P0：泛型如何用于请求与数据模型？

### 30 秒回答

泛型让函数在保持类型关系的同时适配不同数据类型。请求层可用泛型表达“传入的接口类型决定返回 data 类型”，但不能因为写了泛型就认为运行时响应一定安全；服务端异常或字段变更仍要校验和兜底。

```ts
type ApiResponse<T> = {
  code: number
  message: string
  data: T
}

async function request<T>(path: string): Promise<T> {
  const response = await fetch(`/api${path}`)
  const body = (await response.json()) as ApiResponse<T>

  if (body.code !== 0) {
    throw new Error(body.message)
  }
  return body.data
}

type User = { id: string; name: string }
const user = await request<User>('/users/current')
```

### 展开说明

泛型参数的名字应表达角色，例如 `TData`、`TItem`，复杂场景可通过 `extends` 限制最小能力。不要在没有复用和类型关系时为了“显得高级”加入泛型。

### 项目表达

我会在请求适配层把后端 DTO 转为前端领域模型。例如后端日期字符串转换为 Date 或展示模型，枚举转为组件选项，不让每个页面都重复判断字段是否存在。

## P0：`keyof`、`typeof` 和索引访问类型如何减少字段错误？

### 30 秒回答

`keyof T` 得到类型的属性键联合，`T[K]` 取得某个属性类型，`typeof value` 把已有变量的静态形状用于类型位置。它们适合让字段名和字段值保持关联，例如通用表格、筛选器和表单工具，不需要把字段名重复写成宽泛的 string。

```ts
type User = { id: string; name: string; enabled: boolean }

function getField<T, K extends keyof T>(item: T, key: K): T[K] {
  return item[key]
}

getField<User, 'name'>({ id: '1', name: 'Qiu', enabled: true }, 'name')
```

### 可能追问

- `Object.keys` 为什么不能自然返回 `(keyof T)[]`？运行时对象可能有额外属性，TypeScript 不承诺返回键完全等于静态类型；需要在受控边界谨慎断言。

## P1：映射类型、条件类型和 `infer` 解决什么问题？

### 30 秒回答

映射类型按已有键批量变换属性，条件类型根据类型关系选择结果，`infer` 在条件类型中提取局部类型。它们适合封装通用库类型或复杂表单/接口关系；普通业务模型优先使用直观 interface 和 type，避免类型系统本身成为维护成本。

```ts
type Nullable<T> = { [K in keyof T]: T[K] | null }
type ElementOf<T> = T extends Array<infer Item> ? Item : never

type NullableUser = Nullable<{ id: string; name: string }>
type UserItem = ElementOf<Array<{ id: string }>>
```

### 可能追问

- `extends` 在泛型约束和条件类型中意思一样吗？语法相同但语境不同：前者限制可传入类型，后者进行类型关系判断。

## P0：Vue 组件如何用 TypeScript 定义清晰边界？

### 30 秒回答

在 Vue 3 中，我会给 Props、Emits、组件内部状态和接口数据定义类型，让父子组件契约在编译期可检查。类型应描述业务语义，例如 `UserSummary`、`QueryParams`，而不只写匿名对象；可选 prop 需要同时处理默认值与运行时空值边界。

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

### 项目表达

组件抽象时，我先定义输入、输出和是否受控，再定义类型。不要让组件直接知道整个页面 Store 的结构，否则它难复用、也难测试。类型化 Props/Emits 能迫使这种边界更明确。

### 可能追问

- 为什么 props 不能直接修改？它由父组件拥有；子组件要通过 emit 表达更新意图，避免数据流不清晰。
- Vue 模板类型检查怎么做？使用 Volar 等语言工具和项目的类型检查脚本，把模板也纳入检查。

## P1：常用工具类型在业务中怎么用？

### 30 秒回答

`Partial` 适合部分更新，`Pick` 选择公开字段，`Omit` 去掉不该暴露的字段，`Record` 表示键值映射。它们应服务于模型边界，不能代替领域建模；过度嵌套会让报错难读。

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

### 可能追问

- `keyof` 有什么用途？从对象类型得到属性键的联合，可用于限制字段名，避免字符串拼错。
- 条件类型什么时候需要？在库封装或需要根据输入推导输出时使用；普通业务代码优先保持直观。

## P1：枚举、`as const` 和字面量联合如何取舍？

### 30 秒回答

字符串字面量联合加 `as const` 通常足够表达前端常量，并且更容易与接口值互操作；enum 会生成运行时代码，使用前要确认构建和跨端协议需要。不要把数字 enum 的反向映射等特性当作业务模型的默认选择。

```ts
const Roles = ['admin', 'member'] as const
type Role = (typeof Roles)[number]

const roleLabels: Record<Role, string> = {
  admin: '管理员',
  member: '成员',
}
```

## P1：声明文件 `.d.ts` 解决什么问题？

### 30 秒回答

声明文件描述 JavaScript 模块、全局变量或第三方库的类型，让 TypeScript 能检查调用方式而不需要实现源码。优先安装官方或 DefinitelyTyped 类型；没有类型时，在项目适配层写最小、准确的声明，避免用一个全局 `declare module '*'` 抹掉所有检查。

```ts
// env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

### 项目表达

我会把第三方无类型库包在一个适配模块后面，再为这个模块写类型。这样未来替换库或补全类型时，影响范围可控。

## P1：`tsconfig` 中哪些配置值得理解？

### 30 秒回答

`strict` 是类型安全的总开关；`noImplicitAny` 防止隐式 any，`strictNullChecks` 强迫处理 null/undefined，`noUncheckedIndexedAccess` 让索引访问更谨慎。迁移老项目时可以分阶段打开，但不能永久用关闭严格选项来掩盖模型问题。

### 可能追问

- `paths` 配置为什么还要同步 Vite？TypeScript 只负责类型解析，运行时构建工具也必须知道相同别名。
- 类型检查和 Vite build 一样吗？不完全一样。Vite 的转译强调速度，通常需要单独的 `vue-tsc` 或 `tsc --noEmit` 纳入 CI。

## P1：类型断言和运行时校验的边界是什么？

### 30 秒回答

`as SomeType` 只告诉编译器如何看待值，不会在运行时转换或验证数据。对接口、localStorage、URL 参数等外部输入，先使用类型守卫或校验，再转换为内部可信模型；断言只适合编译器无法推断但开发者有明确事实依据的场景。

```ts
function isUser(value: unknown): value is { id: string; name: string } {
  return typeof value === 'object'
    && value !== null
    && typeof (value as Record<string, unknown>).id === 'string'
    && typeof (value as Record<string, unknown>).name === 'string'
}
```

## P1：怎样逐步给 Vue 2 项目引入 TypeScript？

### 30 秒回答

不建议一次把全部 JS 改成 TS。先从工具函数、接口模型、请求层和新模块开始，开启合理的检查但允许渐进过渡；为复杂旧组件建立类型边界或适配层，避免在全局用 `any` 关闭问题。

### 项目表达

引入前要确认构建链路、Vue 版本、编辑器支持和第三方类型。可以先统计 `any` 和编译错误的类别，按收益逐步收敛，而不是把“文件后缀改成 ts”当作完成。
