# Webpack 打包流程：它怎样从入口文件变成可发布资源

> 不要只背“loader 转换、plugin 扩展”。先明确 Webpack 面对的问题：浏览器不能直接理解 TypeScript、Sass 或模块依赖图，也不该由浏览器逐个猜测生产资源、缓存文件名和异步分包关系。

## 一、Webpack 到底解决什么问题

开发时的代码是许多相互引用的模块：

```text
main.ts
  -> App.vue
  -> router.ts
  -> pages/OrderList.vue
  -> api/order.ts
  -> styles/index.scss
```

其中有 TS、Vue SFC、CSS、图片、字体、动态 import，还可能有环境变量和兼容性要求。发布时浏览器需要一组确定的 JS/CSS/资源文件，以及一份知道如何加载异步块的运行时代码。

Webpack 就是把“源代码的模块关系”编译为“浏览器可加载的产物关系”。主线是：

```text
配置 -> 入口 -> 递归构建模块图 -> 转换模块 -> 组织 chunk -> 优化 -> 输出文件
```

## 二、从入口出发，为什么能找到所有代码

```js
// src/main.ts
import { createApp } from 'vue'
import App from './App.vue'
import './styles/index.scss'

createApp(App).mount('#app')
```

Webpack 从 `entry` 读取该文件，解析其中的静态 `import`、`require` 和动态 `import()`，继续读取依赖，再递归下去。最终得到**模块图**，而不是简单按文件夹打包。

```text
entry
  -> module A
      -> module C
  -> module B
      -> module C（同一模块只记录一次）
```

这也是它能做 Tree Shaking、重复模块去重、代码分割的前提：先知道模块之间真实的依赖关系。

## 三、loader 为什么是“转换链”

Webpack 原生重点理解 JavaScript 模块。遇到 `.ts`、`.vue`、`.scss`、图片时，需要规则决定怎样把它们转换为 Webpack 能继续处理的模块：

```js
module.exports = {
  module: {
    rules: [
      { test: /\.ts$/, use: 'ts-loader', exclude: /node_modules/ },
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },
    ],
  },
}
```

可以把 loader 理解成“从一种模块内容到另一种模块内容的编译器”。多个 loader 串联时，通常从配置数组右侧先执行：

```text
scss 源码 -> sass-loader -> css-loader -> style-loader -> JS 模块
```

例如 `css-loader` 让 CSS 中的 `@import`、`url()` 进入模块依赖体系；`style-loader` 再把样式注入页面。生产环境常用提取 CSS 的方案，而不是简单注入，这说明 loader 的选择应服务于构建目标。

## 四、plugin 为什么比 loader 的范围大

loader 聚焦“一个模块怎样转”；plugin 可以通过 Webpack 的 hooks 参与构建生命周期，对整体构建施加影响。

```text
构建前：读取/调整配置
编译中：访问模块、chunk、资源、依赖关系
产物阶段：生成 HTML、提取 CSS、压缩、注入变量、输出分析报告
```

常见例子：

- `HtmlWebpackPlugin`：生成或处理 HTML，并注入产物资源。
- `DefinePlugin`：在构建时替换已知常量。
- CSS 提取类插件：将样式输出为独立文件。
- 分析类插件：查看哪个依赖撑大了包体。

所以更完整的区别是：**loader 参与模块内容转换；plugin 借由生命周期扩展整次构建的能力。**

## 五、模块、chunk、bundle 到底是什么

这三个词不要混用：

```text
module：源码层面的一个依赖单元，例如某个 TS/Vue/CSS 文件
chunk：Webpack 按入口、动态 import 和优化规则组织的一组模块
bundle / asset：最终输出的一个或多个文件，例如 app.abc.js
```

动态 import 是生成异步 chunk 的常见来源：

```js
const OrderPage = () => import('./pages/OrderPage.vue')
```

用户进入订单页时，运行时代码再按映射加载对应 chunk。这就是路由懒加载的实际基础。

## 六、优化发生在什么时候

模块图完成后，Webpack 会根据 mode、optimization 和插件做优化，例如：

- Tree Shaking：删除可证明未使用的 ESM 导出。
- SplitChunks：提取可复用模块或第三方依赖。
- 压缩、最小化：减小传输体积。
- contenthash：内容变化才改文件名，帮助长期缓存。
- runtime chunk：拆出模块加载映射，减少业务代码变化对公共缓存的影响。

Tree Shaking 并不是“配置 production 就必然万事大吉”。它需要静态 ESM 结构，并受模块副作用影响。例如一个模块顶层就注册全局行为，构建工具不能随意删除它；`package.json` 的 `sideEffects` 若标错，也可能删掉本应保留的 CSS 或初始化代码。

## 七、开发构建与生产构建为什么不同

开发阶段优先考虑可调试和增量更新，通常使用 source map、开发服务器和 HMR；生产阶段优先考虑体积、缓存与运行性能，通常做压缩、hash、资源提取、分包和安全的环境注入。

因此不要把生产配置原样搬进开发环境，也不要只因“能打出来”就认为构建配置合理。

## 八、排查包体大时的正确顺序

1. 先生成 stats 或用可视化分析报告，找真实大模块。
2. 判断是重复依赖、全量引入、图片/字体，还是业务代码本身。
3. 决定是替换库、按需引入、路由分包、外置资源还是删除无用代码。
4. 重新测量首屏关键 chunk、缓存命中和构建耗时。

不要只看总包大小，也要看首屏到底下载和解析了哪些资源。

## 九、面试官想听到的话

### 30 秒版本

> Webpack 从 entry 出发递归解析 import、require 和动态 import，建立模块依赖图。loader 把 TS、Vue、CSS、图片等资源转换为可继续处理的模块，plugin 通过 compiler 和 compilation 的生命周期处理整体构建，例如生成 HTML、注入变量、提取 CSS 和分析产物。之后 Webpack 根据入口和动态 import 组织 chunk，做 Tree Shaking、分包、压缩和 hash，最后 emit 到 output。module 是源码单元，chunk 是模块组合，最终 asset 才是浏览器下载的文件。

### 追问：loader 和 plugin 的区别

> loader 是模块转换链，处理“这个文件怎样变成模块”；plugin 是构建生命周期扩展，处理“这次构建整体还要做什么”。例如 TS 转 JS 是 loader，生成 HTML、提取 CSS、压缩或分析包体更适合 plugin。

## 十、自测与复习卡

1. Webpack 为什么能从一个 entry 找到全部依赖？
2. CSS 为什么能进入模块图？
3. module、chunk、最终输出文件有何区别？
4. Tree Shaking 为什么依赖 ESM 与副作用信息？
5. 为什么动态 import 能支持路由懒加载？

```text
根问题：源码模块关系 -> 浏览器可发布资源
入口：递归解析依赖，建立模块图
loader：单模块内容转换
plugin：构建生命周期扩展
chunk：模块组织单位；asset：最终输出文件
优化：分包、Tree Shaking、压缩、hash、runtime
排查：先分析真实产物，再决定优化手段
```
