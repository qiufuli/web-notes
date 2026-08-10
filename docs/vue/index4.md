# 路由

##  <font color="#e96900">Vue-Router 的懒加载如何实现</font>

1. 使用箭头函数+import动态加载
2. 使用箭头函数+require动态加载

---

## <font color="#e96900">history和hash的区别？</font>

```javascript
1. hash 带#号 history更美观

2. hash访问不会向服务器发送页面请求，history会向服务器发送一次请求，没有返回的话就会404
所以一般后端设置会每次访问index.html/xxx 都重定向到index.html 这样才会显示出来页面
所以有的时候本地history打包访问页面就会404找不到 需要后端配置或者nginx配置

3.hash是监听location对象hash值变化事件来实现，history是用的浏览历史记录栈的API pushState、replaceState来实现的
```

```javascript
history和hash的差异主要有以下点：
1.history和hash都是利用浏览器的两种特性实现前端路由，history是利用浏览历史记录栈的API实现，hash是监听location对象hash值变化事件来实现
2.history的url没有'#'号，hash反之
3.history修改的url可以是同域的任意url，hash是同文档的url
4.相同的url，history会触发添加到浏览器历史记录栈中，hash不会触发。
5.history模式往往需要后端支持，如果后端nginx没有覆盖路由地址，就会返回404，hash因为是同文档的url，即使后端没有覆盖路由地址，也不会返回404
6.hash模式下，如果把url作为参数传后端，那么后端会直接从'#'号截断，只处理'#'号前的url，因此会存在#后的参数内容丢失的问题，不过这个问题hash模式下也有解决的方法。
```

---

## <font color="#e96900">$route 和$router 的区别</font>
- $route 是“路由信息对象”，包括 path，params，hash，query，fullPath，matched，name 等路由信息参数
- $router 是“路由实例”对象包括了路由的跳转方法，钩子函数等。

---

## <font color="#e96900">如何定义动态路由？</font>

### param方式
- 配置路由格式：/router/:id
- 传递的方式：在path后面跟上对应的值
- 传递后形成的路径：/router/123

### 响应路由参数的变化
```js
相同路由跳转比如 /users/johnny跳转/users/jolyne

相同的组件实例将被重复使用。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。不过，这也意味着组件的生命周期钩子不会被调用。
要对同一个组件中参数的变化做出响应的话
1. watch监听$route
2. beforeRouteUpdate导航守卫
这两种方式都可以监听到params变化，然后做相应的逻辑处理
```
路由匹配是不区分大小写和尾部斜杠的，例如，路由 `/users` 将匹配 `/users、/users/、甚至 /Users/`。这种行为可以通过 `strict（大小写）` 和 `sensitive（尾部斜杠）` 选项来修改，它们可以既可以应用在整个全局路由上，又可以应用于当前路由上：

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes: [
    // 将匹配 /users/posva 而非：
    // - /users/posva/ 当 strict: true
    // - /Users/posva 当 sensitive: true
    { path: '/users/:id', sensitive: true },
    // 将匹配 /users, /Users, 以及 /users/42 而非 /users/ 或 /users/42/
    { path: '/users/:id?' },
  ],
  strict: true, // applies to all routes
})
```

重定向`redirect`可以写path也可以写成函数返回，注意，beforeEnter拦截的是最终的路径地址而不是重定向之前的

```javascript
const routes = [{ path: '/home', redirect: '/' }]
const routes = [
  {
    // /search/screens -> /search?q=screens
    path: '/search/:searchText',
    redirect: to => {
      // 方法接收目标路由作为参数
      // return 重定向的字符串路径/路径对象
      return { path: '/search', query: { q: to.params.searchText } }
    },
  },
```

--- 

## <font color="#e96900">说一下路由守卫</font>

 1. 全局导航守卫
 	-  beforeEach
 	- beforeResolve
 	- afterEach
 2. 路由导航守卫
 	- beforeEnter 
 3. 组件导航守卫
     -  beforeRouteEnter
     - beforeRouteUpdate
     - beforeRouteLeave

### 全局导航守卫

```javascript
当一个导航触发时，全局前置守卫按照创建顺序调用。
守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于等待中。

参数说明：
to: Route， 即将要进入的目标 路由对象；
from: Route，当前导航正要离开的路由；
next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。
next(false): 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。
next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。
 

全局前置守卫
router.beforeEach((to,from,next)=>{
	if(xxxxx){
	next('/login')
	}else{
	next()
	}
})

全局解析守卫
注意:解析守卫刚好会在导航被确认之前、所有组件内守卫和异步路由组件被解析之后调用
router.beforeResolve(()=>{})

全局后置钩子
守卫不同的是，这些钩子不会接受 next 函数也不会改变导航本身
它们对于分析、更改页面标题、声明页面等辅助功能以及许多其他事情都很有用
router.afterEach((to, from) => {
  sendToAnalytics(to.fullPath)
})
```

### 路由导航守卫

```javascript
const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: (to, from) => {
      // reject the navigation
      return false
    },
  },
]
```

注意，beforeEnter拦截的是最终的路径地址而不是重定向之前的

### 组件导航守卫

```javascript
const UserDetails = {
  template: `...`,
  beforeRouteEnter(to, from) {
    在渲染该组件的对应路由被验证前调用
    不能获取组件实例 `this` ！
     因为当守卫执行时，组件实例还没被创建！
  },
  beforeRouteUpdate(to, from) {
    在当前路由改变，但是该组件被复用时调用
     举例来说，对于一个带有动态参数的路径 `/users/:id`，在 `/users/1` 和 `/users/2` 之间跳转的时候，
     由于会渲染同样的 `UserDetails` 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
     因为在这种情况发生的时候，组件已经挂载好了，导航守卫可以访问组件实例 `this`
  },
  beforeRouteLeave(to, from) {
     在导航离开渲染该组件的对应路由时调用
     与 `beforeRouteUpdate` 一样，它可以访问组件实例 `this`
  },
}
```

### 完整的导航解析流程
1. 导航被触发
2. 在之前失活的组件触发`beforeRouterLeave`
3. 调用全局前置守卫`beforeEach`
4. 如果是重用的组件`/user/:id`，调用`beforeRouterUpdate`更新组件
5. 然后执行路由守卫`beforeEnter`
6. 解析异步路由组件
7. 然后执行组件内的`beforeRouterEnter`
8. 然后会调用全局解析守卫`beforeResolve`
9. 这时候导航已经被确认了
10. 调用全局后置钩子`afterEach`
11. 触发 DOM 更新
12. 调用` beforeRouteEnter` 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。 

---

## <font color="#e96900">params和query的区别</font>

1. query要用path来引入，params要用name来引入
2. 接收参数都是类似的，分别是 this.$route.query.name 和 this.$route.params.name 。
3. query更加类似于ajax中get传参，params则类似于post，说的再简单一点，前者在浏览器地址栏中显示参数，后者则不显示
4. query刷新不会丢失query里面的数据 params刷新会丢失params里面的数据，因为params存在内存中。

---