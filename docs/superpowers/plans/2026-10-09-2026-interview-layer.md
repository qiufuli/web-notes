# 2026 面试更新层实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在不改动旧资料正文的前提下，新增一套独立、现代且可自学的 Vue 面试手册，并从 Docsify 导航访问。

**Architecture:** 新手册的全部正文放在 `docs/2026-interview/`，以入口页串联 14 个主题章节。各章节以 `P0/P1/P2`、30 秒回答、原理、最小代码、项目表达和追问组织；既有资料仅作为取材来源，不被引用为阅读前置条件。`docs/_navbar.md` 只追加一个新入口。

**Tech Stack:** Markdown、Docsify 4、JavaScript、Vue 2/3、TypeScript、Pinia、Vite。

**Spec:** `docs/superpowers/specs/2026-10-09-2026-interview-layer-design.md`

## Global Constraints

- 不修改旧资料正文：`docs/javascript/`、`docs/vue/`、`docs/typeScript/`、`docs/htmlcss/`、`docs/browerAndNetwork/` 与 `interview/`。
- 所有新的面试正文都创建在 `docs/2026-interview/`。
- 每个核心主题至少包含 P0 内容、30 秒回答、解释、最小代码示例，以及项目表达或追问。
- Vue 3、TypeScript、Pinia、Vite 章节必须可独立阅读，并从 Vue 2 经验递进。
- 不添加不受支持的框架依赖；Docsify 继续通过现有 CDN 加载。

---

### Task 1: 创建入口与站点导航

**Files:**
- Create: `docs/2026-interview/README.md`
- Modify: `docs/_navbar.md`
- Test: Docsify 本地页面可访问入口与所有章节链接

**Interfaces:**
- Consumes: 设计说明中定义的文件名与优先级格式。
- Produces: 新手册唯一入口和全站导航入口，供所有后续章节链接。

- [ ] **Step 1: 写入入口页的验收清单**

入口页先列出 14 个目标章节、每章预期的主题说明，以及 `P0/P1/P2` 的明确含义。该清单在正文还不存在时应包含失效链接，用作待实现页面的失败信号。

- [ ] **Step 2: 验证入口链接尚不完整**

Run: `rg -n '\]\([^)]*\.md\)' docs/2026-interview/README.md`

Expected: 入口页列出 14 个章节链接，但除入口页以外的章节文件尚不存在。

- [ ] **Step 3: 添加导航入口并完成入口页**

在 `docs/_navbar.md` 末尾追加 `2026 面试更新` 分组，并链接 `2026-interview/README.md` 与 14 个章节。入口页说明阅读方式：P0 是必答题，P1 是高频延伸，P2 是补充；明确不需要回查旧笔记。

- [ ] **Step 4: 验证入口结构**

Run: `rg -n '2026 面试更新|2026-interview' docs/_navbar.md docs/2026-interview/README.md`

Expected: 导航和入口页都包含唯一且一致的 2026 路径。

- [ ] **Step 5: 提交入口工作**

Run: `git add docs/_navbar.md docs/2026-interview/README.md && git commit -m "docs: add 2026 interview handbook entry"`

Expected: 只提交导航与新入口页。

### Task 2: 编写基础语言、异步和浏览器性能章节

**Files:**
- Create: `docs/2026-interview/02-JavaScript-语言核心.md`
- Create: `docs/2026-interview/03-异步并发与事件循环.md`
- Create: `docs/2026-interview/04-浏览器渲染性能与存储.md`
- Test: Markdown 标题、优先级块和代码块检查

**Interfaces:**
- Consumes: 入口页的章节链接与统一题目格式。
- Produces: 面试手册的语言、异步与浏览器基础，供 Vue 和工程化章节引用概念但不要求跳转阅读。

- [ ] **Step 1: 为三个章节分别写出 P0 问题清单**

JavaScript 包含类型转换、作用域/闭包、`this`、原型、模块、垃圾回收；异步包含 Promise、`async/await`、微任务、浏览器与 Node 差异、取消与并发控制；浏览器包含渲染流水线、重排/重绘、Core Web Vitals、Performance API、缓存层级和存储选型。

- [ ] **Step 2: 验证每章的 P0 条目覆盖**

Run: `rg -n '^## P0|^### P0' docs/2026-interview/02-JavaScript-语言核心.md docs/2026-interview/03-异步并发与事件循环.md docs/2026-interview/04-浏览器渲染性能与存储.md`

Expected: 三个文件各自有至少三个 P0 条目。

- [ ] **Step 3: 用统一格式完成内容和最小代码**

每章至少写三个完整题目，所有题目均给出 30 秒回答；JavaScript 使用 `Object.is`、闭包或原型等短例子，异步使用 `Promise`、`queueMicrotask`、`AbortController` 等短例子，浏览器使用 `PerformanceObserver` 或渲染批处理等短例子。明确区分浏览器与 Node API，不沿用旧资料的 Event Table 描述。

- [ ] **Step 4: 验证代码块与必需小节**

Run: `rg -n '30 秒回答|项目表达|可能追问|```' docs/2026-interview/02-JavaScript-语言核心.md docs/2026-interview/03-异步并发与事件循环.md docs/2026-interview/04-浏览器渲染性能与存储.md`

Expected: 每个文件同时出现回答、追问和代码块标记。

- [ ] **Step 5: 提交基础章节**

Run: `git add docs/2026-interview/02-JavaScript-语言核心.md docs/2026-interview/03-异步并发与事件循环.md docs/2026-interview/04-浏览器渲染性能与存储.md && git commit -m "docs: add 2026 javascript and browser interview notes"`

Expected: 只提交三个新增章节。

### Task 3: 编写网络安全、CSS 与项目表达章节

**Files:**
- Create: `docs/2026-interview/01-求职定位与项目表达.md`
- Create: `docs/2026-interview/05-网络缓存与前端安全.md`
- Create: `docs/2026-interview/06-CSS-布局与工程实践.md`
- Test: 高风险结论和项目回答结构检查

**Interfaces:**
- Consumes: 统一题目格式与既有项目经验定位。
- Produces: 可直接用于面试开场、项目深挖、网络安全与 CSS 场景题的回答材料。

- [ ] **Step 1: 写入章节提纲**

项目表达覆盖 1 分钟自我介绍、项目 STAR 拆解、遗留系统治理、技术取舍和反问；网络安全覆盖 HTTP 缓存、HTTP/2/3、CORS、Cookie、CSRF、XSS、CSP、`fetch`；CSS 覆盖布局、BFC、层叠上下文、响应式、可访问性和性能。

- [ ] **Step 2: 写入并校正高风险网络结论**

明确说明跨域请求可以发出但浏览器会限制脚本读取跨源响应；`SameSite` 不是 CSRF 的唯一措施；`HttpOnly` 不能阻止 CSRF；HTTP/2 多路复用不是 HTTP/1.1 管线化。

- [ ] **Step 3: 完成项目表达和 CSS 场景题**

每个章节至少三项 P0 内容，包含一个自我介绍模板、一个遗留项目治理案例框架、一个 CSS 布局或层叠上下文的代码例子和对应项目表达。

- [ ] **Step 4: 验证必需概念与例子**

Run: `rg -n 'SameSite|HttpOnly|CSP|30 秒回答|项目表达|```' docs/2026-interview/01-求职定位与项目表达.md docs/2026-interview/05-网络缓存与前端安全.md docs/2026-interview/06-CSS-布局与工程实践.md`

Expected: 网络安全文件覆盖四个安全关键词；三章都有统一结构或代码示例。

- [ ] **Step 5: 提交表达与 Web 平台章节**

Run: `git add docs/2026-interview/01-求职定位与项目表达.md docs/2026-interview/05-网络缓存与前端安全.md docs/2026-interview/06-CSS-布局与工程实践.md && git commit -m "docs: add 2026 project and web platform interview notes"`

Expected: 只提交三个新增章节。

### Task 4: 编写 Vue 2 经验与 Vue 3 迁移章节

**Files:**
- Create: `docs/2026-interview/07-Vue2-核心原理与遗留项目治理.md`
- Create: `docs/2026-interview/08-Vue2到Vue3的迁移策略.md`
- Test: Vue 2/3 对比与迁移风险检查

**Interfaces:**
- Consumes: 既有 Vue 2 项目背景与现代 Vue 3 目标。
- Produces: 将 Vue 2 深度转化为资深候选人优势的回答与迁移决策框架。

- [ ] **Step 1: 写入 Vue 2 P0 内容**

涵盖响应式边界、组件通信、`nextTick`、`key` 与 diff、Vuex、性能诊断，以及“为什么不立即迁移”的业务判断。避免将 Vue 2 的 `Object.defineProperty` 限制错误延伸到 Vue 3。

- [ ] **Step 2: 写入迁移决策和渐进迁移路径**

涵盖迁移前评估、依赖与浏览器兼容、测试与灰度、优先改造叶子页面、Options API 与 Composition API 共存、Vuex/Pinia 选择、回滚方案。包含一段同一组件从 Options API 到 `<script setup>` 的对比代码。

- [ ] **Step 3: 验证迁移可讲述性**

Run: `rg -n '30 秒回答|迁移|风险|回滚|Options API|script setup|项目表达|```' docs/2026-interview/07-Vue2-核心原理与遗留项目治理.md docs/2026-interview/08-Vue2到Vue3的迁移策略.md`

Expected: 两章都有 P0 回答，迁移章同时明确风险、回滚和代码对比。

- [ ] **Step 4: 提交 Vue 2 与迁移章节**

Run: `git add docs/2026-interview/07-Vue2-核心原理与遗留项目治理.md docs/2026-interview/08-Vue2到Vue3的迁移策略.md && git commit -m "docs: add vue migration interview notes"`

Expected: 只提交两个新增章节。

### Task 5: 编写 Vue 3 与 TypeScript 学习式章节

**Files:**
- Create: `docs/2026-interview/09-Vue3-核心机制与实践.md`
- Create: `docs/2026-interview/10-TypeScript-从基础到Vue业务建模.md`
- Test: 渐进示例与 Vue 场景类型检查

**Interfaces:**
- Consumes: Vue 2/3 对比章节的迁移概念。
- Produces: 可从基础跟读到业务实践的 Vue 3 和 TypeScript 面试材料。

- [ ] **Step 1: 写入 Vue 3 的递进提纲与 P0 内容**

按 `ref/reactive`、`computed/watch`、生命周期、`<script setup>`、Props/Emits、Composable、依赖注入、异步组件与性能组织。每个新概念要说明对应的 Vue 2 经验和推荐原因。

- [ ] **Step 2: 写入 TypeScript 的业务递进内容**

从对象、函数、联合类型和收窄开始，再写泛型、`keyof`、映射/条件类型和工具类型。示例必须包含类型化 Props/Emits、接口响应、表单状态或通用请求，不以纯类型谜题收尾。

- [ ] **Step 3: 增加并解释最小示例**

Vue 3 使用一个 `<script setup lang="ts">` 组件说明响应式、Props、Emits 与 `computed`；TypeScript 使用一个 API 响应泛型和可判别联合说明收窄。每段代码后解释关键类型或 API 语句与面试话术。

- [ ] **Step 4: 验证章节自包含性**

Run: `rg -n 'Vue 2|script setup|ref\(|defineProps|30 秒回答|泛型|keyof|类型收窄|```' docs/2026-interview/09-Vue3-核心机制与实践.md docs/2026-interview/10-TypeScript-从基础到Vue业务建模.md`

Expected: Vue 3 章包含 Vue 2 对照和新 API；TypeScript 章包含业务类型例子与关键概念。

- [ ] **Step 5: 提交 Vue 3 与 TypeScript 章节**

Run: `git add docs/2026-interview/09-Vue3-核心机制与实践.md docs/2026-interview/10-TypeScript-从基础到Vue业务建模.md && git commit -m "docs: add vue3 and typescript interview handbook"`

Expected: 只提交两个新增章节。

### Task 6: 编写 Pinia、Vite 与质量保障章节

**Files:**
- Create: `docs/2026-interview/11-路由组件设计与Pinia状态管理.md`
- Create: `docs/2026-interview/12-Vite工程化构建发布与质量保障.md`
- Test: Store 边界、Vite 机制与工程质量检查

**Interfaces:**
- Consumes: Vue 3 和 TypeScript 中的 `<script setup>`、类型化接口和组合式函数概念。
- Produces: 现代 Vue 项目的状态管理与工程化面试回答。

- [ ] **Step 1: 写入路由、组件边界与 Pinia 内容**

覆盖 Vue Router 4 导航守卫、权限路由、组件状态与全局状态边界、Option/Setup Store、`state/getters/actions`、异步 action、持久化风险、跨 Store 协作、Vuex 对比和旧项目迁移判断。用 TypeScript 编写一个小型 Setup Store 示例。

- [ ] **Step 2: 写入 Vite 与质量保障内容**

覆盖开发服务器与原生 ESM、依赖预构建、Rollup 生产构建、环境变量、别名、代理、代码分包、Source Map、webpack 比较、单元/组件/E2E 测试边界、监控、灰度与回滚。用最小 `vite.config.ts` 示例说明代理与别名。

- [ ] **Step 3: 验证标准技术栈覆盖**

Run: `rg -n 'Pinia|defineStore|Vuex|导航守卫|Vite|依赖预构建|Rollup|Source Map|回滚|```' docs/2026-interview/11-路由组件设计与Pinia状态管理.md docs/2026-interview/12-Vite工程化构建发布与质量保障.md`

Expected: 两章覆盖标准技术栈、旧技术对比和至少一个最小代码例子。

- [ ] **Step 4: 提交现代工程章节**

Run: `git add docs/2026-interview/11-路由组件设计与Pinia状态管理.md docs/2026-interview/12-Vite工程化构建发布与质量保障.md && git commit -m "docs: add pinia and vite interview handbook"`

Expected: 只提交两个新增章节。

### Task 7: 编写手写、排障和模拟追问章节

**Files:**
- Create: `docs/2026-interview/13-手写题场景题与排障题.md`
- Create: `docs/2026-interview/14-模拟面试追问库.md`
- Test: 题目覆盖与可执行回答框架检查

**Interfaces:**
- Consumes: 先前章节中的所有核心概念与项目表达模板。
- Produces: 面试演练使用的场景题和追问清单。

- [ ] **Step 1: 收敛手写与排障题范围**

保留防抖、节流、深拷贝边界、并发请求控制、数组去重、手写 Promise 机制解释等高频内容；每题说明使用场景和边界，避免将面试题误当成生产实现。排障部分覆盖白屏、接口异常、性能回退、内存泄漏和发布回滚。

- [ ] **Step 2: 写入模拟追问树**

以自我介绍、Vue 2 遗留系统、Vue 3 学习与迁移、TypeScript、Pinia/Vite、性能、安全和协作带人分组。每组包含主问题、至少三条追问和答题要点，强调不虚构项目数据或结果。

- [ ] **Step 3: 验证题库可演练性**

Run: `rg -n 'P0|30 秒回答|追问|白屏|并发|深拷贝|Vue 3|TypeScript|Pinia|Vite' docs/2026-interview/13-手写题场景题与排障题.md docs/2026-interview/14-模拟面试追问库.md`

Expected: 两章都包含明确题目和可继续追问的结构。

- [ ] **Step 4: 提交演练章节**

Run: `git add docs/2026-interview/13-手写题场景题与排障题.md docs/2026-interview/14-模拟面试追问库.md && git commit -m "docs: add interview practice questions"`

Expected: 只提交两个新增章节。

### Task 8: 全量链接、范围与内容质量验证

**Files:**
- Modify: `docs/2026-interview/README.md`（仅修正验证发现的问题）
- Modify: `docs/_navbar.md`（仅修正验证发现的问题）
- Test: 链接存在性、章节格式、旧资料未改动与 Docsify 本地浏览

**Interfaces:**
- Consumes: 14 个已完成章节与导航入口。
- Produces: 可交付的独立 2026 面试更新层。

- [ ] **Step 1: 验证所有入口链接指向真实文件**

Run: `rg -o '2026-interview/[^)]*\.md' docs/_navbar.md docs/2026-interview/README.md | Sort-Object -Unique`

Expected: 输出入口页和 14 个章节路径，且每个路径在 `docs/` 下存在。

- [ ] **Step 2: 验证章节结构和必需主题**

Run: `rg -l '30 秒回答' docs/2026-interview/*.md; rg -n 'Pinia|Vite|Vue 3|TypeScript' docs/2026-interview/*.md`

Expected: 除入口页外的每个章节均包含 30 秒回答；关键技术栈在对应章节出现。

- [ ] **Step 3: 验证旧资料正文未被修改**

Run: `git diff --name-only HEAD~7..HEAD -- docs/javascript docs/vue docs/typeScript docs/htmlcss docs/browerAndNetwork interview`

Expected: 无输出。

- [ ] **Step 4: 本地启动 Docsify 并抽查导航和至少四个章节**

Run: `npx --yes docsify-cli serve docs --port 3000`

Expected: 浏览器可访问 `http://localhost:3000`，`2026 面试更新` 导航与入口、Vue 3、TypeScript、Pinia/Vite 页面均可加载。

- [ ] **Step 5: 提交最终修正**

Run: `git add docs/_navbar.md docs/2026-interview && git commit -m "docs: verify 2026 interview handbook"`

Expected: 提交仅包含验证发现的修正；若没有改动则不创建空提交。
