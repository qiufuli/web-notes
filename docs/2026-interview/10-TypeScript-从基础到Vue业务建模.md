# TypeScript：从基础到 Vue 业务建模

> TypeScript 的价值是让组件边界、接口数据和状态变化更早暴露错误，不是把所有代码写成复杂类型。

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

## P1：怎样逐步给 Vue 2 项目引入 TypeScript？

### 30 秒回答

不建议一次把全部 JS 改成 TS。先从工具函数、接口模型、请求层和新模块开始，开启合理的检查但允许渐进过渡；为复杂旧组件建立类型边界或适配层，避免在全局用 `any` 关闭问题。

### 项目表达

引入前要确认构建链路、Vue 版本、编辑器支持和第三方类型。可以先统计 `any` 和编译错误的类别，按收益逐步收敛，而不是把“文件后缀改成 ts”当作完成。
