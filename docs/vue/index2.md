<!--
 * @Author: qfuli
 * @Date: 2023-11-06 15:39:29
 * @LastEditors: qfuli
 * @LastEditTime: 2023-11-30 15:28:07
 * @Description: Do not edit
 * @FilePath: /web-notes/docs/vue/index2.md
-->

# 生命周期

## <font color="#e96900">说一下你对 vue 生命周期的理解</font>

Vue 实例有⼀个完整的⽣命周期，也就是从开始创建、初始化数据、编译模版、挂载 Dom -> 渲染、更新 -> 渲染、卸载 等⼀系列过程，称这是 Vue 的⽣命周期。

1. beforeCreate（创建前）：数据观测和初始化事件还未开始，此时 `data `的响应式追踪、`event/watcher` 都**还没有被设置**，也就是说**不能**访问到`data、computed、watch、methods`上的方法和数据。
2. created（创建后） ：实例创建完成，实例上配置的 options 包括 `data、computed、watch、methods `等都配置完成，但是此时渲染得节点还**未挂载到 DOM**，所以不能访问到 `$el` 属性。
3. beforeMount（挂载前）：在挂载开始之前被调用，相关的`render`函数首次被调用。实例已完成以下的配置：编译模板，把`data`里面的数据和模板生成`html`。此时还没有挂载`html`到页面上。
4. mounted（挂载后）：在`el`被新创建的 `vm.$el` 替换，并挂载到实例上去之后调用。实例已完成以下的配置：用上面编译好的`html`内容替换`el`属性指向的`DOM`对象。完成模板中的`html`渲染到`html` 页面中。此过程中进行`ajax`交互。
5. beforeUpdate（更新前）：响应式数据更新时调用，此时虽然响应式数据更新了，但是对应的真实 DOM 还没有被渲染。
6. updated（更新后） ：在由于数据更改导致的虚拟 DOM 重新渲染和打补丁之后调用。此时 DOM 已经根据响应式数据的变化更新了。调用时，组件 DOM 已经更新，所以可以执行依赖于 DOM 的操作。然而在大多数情况下，应该避免在此期间更改状态，因为这可能会导致更新无限循环。该钩子在服务器端渲染期间不被调用。
7. beforeDestroy（销毁前）：实例销毁之前调用。这一步，实例仍然完全可用，`this `仍能获取到实例。
8. destroyed（销毁后）：实例销毁后调用，调用后，`Vue `实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务端渲染期间不被调用。

另外还有 `keep-alive` 独有的生命周期，分别为 `activated` 和 `deactivated` 。

用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `activated` 钩子函数。

---

注意：在`Vue`生命周期钩子会自动绑定 `this` 上下文到实例中，因此你可以访问数据，对 `property` 和方法进行运算这意味着你**不能使用箭头函数来定义一个生命周期方法 (例如 `created: () => this.fetchTodos()`)**

---

## <font color="#e96900">说一下 Vue 父子组件生命周期的执行顺序</font>

加载渲染过程：

1. 父组件 beforeCreate
2. 父组件 created
3. 父组件 beforeMount
   - 子组件 beforeCreate
   - 子组件 created
   - 子组件 beforeMount
   - 子组件 mounted
4. 父组件 mounted

更新过程：

1. 父组件 beforeUpdate
   - 子组件 beforeUpdate
   - 子组件 updated
2. 父组件 updated

销毁过程：

1. 父组件 beforeDestroy
   - 子组件 beforeDestroy
   - 子组件 destroyed
2. 父组件 destoryed

## <font color="#e96900">切换、刷新页面都执行哪些生命周期？</font>

### 页面切换执行的生命周期

#### 1.从 A 页面到 B 页面

      - 先执行B页面的beforeCreate、created、beforeMount
      - 再执行A页面的beforeDestroy、destroyed
      - 最后执行B页面的mounted

#### 2.从 B 页面返回到 A 页面

      - 先执行A页面的beforeCreate、created、beforeMount
      - 再执行B页面的beforeDestroy、destroyed
      - 最后执行A页面的mounted

### 页面刷新和关闭不执行任何生命周期

无法通过 beforeDestroy、destroyed 来捕获刷新/关闭的状态

可以通过监听浏览器的`onbeforeunload`事件来做处理

```js
mounted() {
  // 存一份this
  let _this = this;
  window.onbeforeunload = function(e) {
    // 那个路由页面需要，就把path的名字修改成那个，比如我当前页面的path是/vue
    if (_this.$route.path == "/vue") {
      // 兼容IE8和Firefox 4之前的事件对象写法（不加也行，现在少有项目兼容老版本浏览器了）
      e = e || window.event;
      if (e) {
        e.returnValue = "returnValue属性值的文字不能自定义，写不写都行的";
      }
      // Chrome支持, Safari支持, Firefox 4版本以后支持, Opera 12版本以后支持 , IE 9版本以后支持
      return "returnValue属性值的文字不能自定义，写不写都行的";
    }
  };
},
beforeDestroy() {
  // 离开页面时候再清除
  window.onbeforeunload = () => {};
}
```

## <font color="#e96900">一般在哪个生命周期请求异步数据</font>

这个需要看实际的应用场景，如果需要获取$ref 来操作最早只能在 mounted 里面获取。

如果只是比较 created 和 mouted，那么 created 最好。

讨论这个问题本质就是**触发的时机** ，放在`mounted`中的请求有可能导致页面闪动（因为此时页面`dom`结构已经生成），但如果在页面加载前完成请求，则不会出现此情况。建议对页面内容的改动放在`created`生命周期当中。

## <font color="#e96900">在 created 中如何获取 dom</font>

1. **只要写成异步**，在回调中获取`dom`就可以，因为生命周期是同步任务，等同步任务执行完才会执行异步任务，这个时候`dom`已经挂载完成了
   所以说理论上在前 4 个生命周期中只要用异步的方式都可以获取到`dom`
   比如 `setTimeout promise().then()...`
2. 使用`vue`提供的`this.$nextTick()`

## <font color="#e96900">keep-alive 中的生命周期哪些</font>

`keep-alive`是 `Vue` 提供的一个内置组件，用来对组件进行缓存——在组件切换过程中将状态保留在内存中，防止重复渲染`DOM`。

如果为一个组件包裹了 `keep-alive`，那么它会多出两个生命周期：`deactivated、activated`。同时，`beforeDestroy` 和 `destroyed` 就不会再被触发了，因为**组件不会被真正销毁**。

```js
beforeCreate
created
beforeMount
mounted
activated
```

第二次或第 n 次进入`keep-alive`组件会执行哪些生命周期

```js
只会执行activated，页面已经缓存了
```

离开页面的时候会执行`deactivated`

```js
deactivated
```

### keep-alive 如何使用？

keep-alive 可以设置以下 props 属性：

- include - 字符串或正则表达式。只有名称匹配的组件会被缓存
- exclude - 字符串或正则表达式。任何名称匹配的组件都不会被缓存
- max - 数字。最多可以缓存多少组件实例（超出 max 的组件会将长时间不用的组件剔除再插入新的）

使用 includes 和 exclude：

```js
<keep-alive include="a,b">
  <component :is="view"></component>
</keep-alive>

<!-- 正则表达式 (使用 `v-bind`) -->
<keep-alive :include="/a|b/">
  <component :is="view"></component>
</keep-alive>

<!-- 数组 (使用 `v-bind`) -->
<keep-alive :include="['a', 'b']">
  <component :is="view"></component>
</keep-alive>
```

匹配首先检查组件自身的 name 选项，如果 name 选项不可用，则匹配它的局部注册名称 (父组件 components 选项的键值)，匿名组件不能被匹配
