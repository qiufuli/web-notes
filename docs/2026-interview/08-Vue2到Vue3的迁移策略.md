# Vue 2 到 Vue 3 的迁移策略

> 迁移题考察的是工程判断：为什么现在迁、迁哪些、如何验证、出问题怎么退，而不是会不会把 Options API 改成 `<script setup>`。

## P0：为什么不建议直接全量重写？

### 30 秒回答

框架迁移会同时触及依赖、构建、组件行为、路由、状态管理、浏览器兼容和测试。全量重写把这些风险集中到一个发布窗口，容易在功能对齐和业务持续迭代中失控。更稳妥的做法是先评估收益和前置条件，从独立、低耦合、可回归验证的模块开始渐进迁移，并保留回滚能力。

### 展开说明

需要迁移的理由可以是 Vue 2 生命周期结束、生态依赖不再支持、开发体验和类型能力不足，或者新模块长期增加。需要暂缓的理由也可能成立，例如关键业务窗口、旧浏览器约束、缺乏自动化测试或团队没有足够维护带宽。

### 项目表达

我会先输出一份风险清单：运行时依赖、UI 组件库、构建插件、浏览器范围、全局 API、路由守卫、Store、关键链路测试和监控。没有这些基线，迁移进度只能靠感觉判断。

### 可能追问

- 如何确定第一批模块？优先选依赖少、边界清楚、用户影响可控、回归路径明确的叶子页面或新功能。
- 可否 Vue 2、Vue 3 共存？可以采用独立子应用、渐进迁移构建或兼容构建等方案，但必须评估运行时体积、调试复杂度和团队维护成本。

## P0：Vue 2 和 Vue 3 响应式的主要区别是什么？

### 30 秒回答

Vue 2 基于 `Object.defineProperty` 转换已有属性，需要处理新增属性和数组变更边界；Vue 3 基于 Proxy 代理对象，能拦截更多操作并按需代理嵌套对象。Vue 3 还把响应式能力抽成 `ref`、`reactive`、`computed`、`watch` 等独立 API，更适合把同一业务关注点组合在一起。

### 展开说明

Proxy 不是没有边界。它只能代理对象，基本类型需要 `ref`；把 reactive 对象直接解构会取出普通值，可能失去响应式连接，需要 `toRefs` 或保持对象访问。面试时讲清这些边界比只说“Proxy 更好”更有说服力。

```js
import { reactive, toRefs } from 'vue'

const state = reactive({ page: 1, keyword: '' })

// const { page } = state // 这里得到普通值，不能保持对 state.page 的响应式连接
const { page, keyword } = toRefs(state)
```

### 项目表达

迁移时我不会把所有 `data` 机械替换为一个巨大 `reactive` 对象。简单基本值用 `ref`，相关字段可用 `reactive`，组件复杂逻辑则按业务关注点拆 composable，让状态、请求和副作用更易测试。

## P0：Options API 如何迁移到 `<script setup>`？

### 30 秒回答

迁移的重点不是语法压缩，而是把同一业务关注点的状态、计算、请求和清理放在一起。`<script setup>` 减少样板代码，`defineProps`、`defineEmits` 明确组件边界；对于复杂组件，可以先保留清晰的 Options API，再逐步抽取 composable，不强制一次改完。

### 对比代码

```vue
<!-- Vue 2 Options API -->
<script>
export default {
  props: { initialCount: { type: Number, default: 0 } },
  data() {
    return { count: this.initialCount }
  },
  computed: {
    label() {
      return `当前数量：${this.count}`
    },
  },
  methods: {
    increment() {
      this.count += 1
      this.$emit('change', this.count)
    },
  },
}
</script>
```

```vue
<!-- Vue 3 <script setup lang="ts"> -->
<script setup lang="ts">
import { computed, ref } from 'vue'

const props = withDefaults(defineProps<{ initialCount?: number }>(), {
  initialCount: 0,
})
const emit = defineEmits<{ change: [value: number] }>()

const count = ref(props.initialCount)
const label = computed(() => `当前数量：${count.value}`)

function increment() {
  count.value += 1
  emit('change', count.value)
}
</script>
```

### 项目表达

重构前我会保证行为等价，再改善结构。将逻辑抽成 composable 前，先明确输入、输出、副作用和清理责任；否则只是把一个大组件搬到一个大函数里。

### 可能追问

- Vue 3 还能用 Options API 吗？可以，Vue 3 支持它。迁移选择应考虑组件复杂度和团队可读性。
- `setup` 里为什么访问 ref 要 `.value`？JavaScript 中 ref 是包装对象；模板会自动解包，但脚本不会。

## P1：路由、状态和依赖应该怎么迁？

### 30 秒回答

把路由、状态、UI 库和构建依赖当成独立迁移工作流，不和组件语法改造混为一谈。Vue Router 4、Pinia、Vite 都有新的 API 和运行方式；每次迁移要有版本兼容矩阵、关键路径回归、灰度和回滚。

### 渐进路径

1. 盘点依赖与浏览器范围，删除无主或低价值依赖。
2. 为登录、权限、列表编辑、提交等关键路径补回归用例。
3. 选择一个新模块或低耦合页面试点 Vue 3、TypeScript 与 Pinia。
4. 固化组件规范、请求层和构建配置，再扩大范围。
5. 保留旧入口或开关，在监控异常时能快速退回。

### 可能追问

- 能否一边迁移一边继续交付？可以，但要明确模块边界和版本策略，避免同一功能双栈长期同时维护。
- 如何验收迁移？功能回归、性能基线、错误监控、打包体积、浏览器兼容和发布回滚演练都应纳入验收。
