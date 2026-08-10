<!--
 * @Author: qfuli
 * @Date: 2023-11-09 14:13:26
 * @LastEditors: qfuli
 * @LastEditTime: 2023-11-09 14:48:42
 * @Description: Do not edit
 * @FilePath: /web-notes/docs/vue/index3.md
-->
# 组件通信

## <font color="#e96900">1. props / $emit</font>

父组件通过`props`向子组件传递数据，子组件通过`$emit`和父组件通信。


注意：子组件无法直接修改父组件传递的`props`，否则会报错。

### 子组件如何修改父组件的值呢？
子组件直接使用父组件的数据，通过 `this.$parent.xxx` 来获取数据

注意：`这种方式子组件可以直接修改父组件的数据` 

--- 

## <font color="#e96900">2. 依赖注入（provide/ inject）</font>

这种方式就是`Vue`中的**依赖注入**，该方法用于父子组件之间的通信。当然这里所说的父子不一定是真正的父子，也可以是祖孙组件，在层数很深的情况下，可以使用这种方法来进行传值。就不用一层一层的传递了。

`provide / inject`是`Vue`提供的两个钩子，和`data、methods`是同级的。并且`provide`的书写形式和`data`一样。

- provide 钩子用来发送数据或方法
- inject 钩子用来接收数据或方法

```js
父组件
provide(){
  return {
    message: 'provided by father'
  }
      
  },

子组件
inject: [ "message" ],
```
<font color="#e96900">注意：是向下传递的 兄弟组件这种是无法使用的</font>

优势：父组件可以直接向某个后代组件传值,不需要一级一级传递

缺点：不知道是谁传过来的

---

## <font color="#e96900">3. ref</font>

父组件在使用子组件的时候设置ref

父组件通过设置子组件ref来获取数据

```js
<Children ref="foo" />  
  
this.$refs.foo  // 获取子组件实例，通过子组件实例我们就能拿到对应的数据  
```

## <font color="#e96900">4. EventBus</font>

- 使用场景：兄弟组件传值
- 创建一个中央事件总线EventBus
- 兄弟组件通过$emit触发自定义事件，$emit第二个参数为传递的数值
- 另一个兄弟组件通过$on监听自定义事件

```js

bus.js

// 创建一个中央时间总线类  
class Bus {  
  constructor() {  
    this.callbacks = {};   // 存放事件的名字  
  }  
  $on(name, fn) {  
    this.callbacks[name] = this.callbacks[name] || [];  
    this.callbacks[name].push(fn);  
  }  
  $emit(name, args) {  
    if (this.callbacks[name]) {  
      this.callbacks[name].forEach((cb) => cb(args));  
    }  
  }  
}  
  
// main.js  
Vue.prototype.$bus = new Bus() // 将$bus挂载到vue实例的原型上  
// 另一种方式  
Vue.prototype.$bus = new Vue() // Vue已经实现了Bus的功能  

```
```js
this.$bus.$emit('foo')  

this.$bus.$on('foo', this.handle)  
```

---

## <font color="#e96900">5. $parent 或$ root</font>

通过共同祖辈`$parent`或者`$root`搭建通信桥连

兄弟组件

`this.$parent.on('add',this.add)`

另一个兄弟组件

`this.$parent.emit('add')`

也可以直接获取父组件的数据

---

## <font color="#e96900">6. vuex</font>

适用场景: 复杂关系的组件数据传递

Vuex作用相当于一个用来存储共享变量的容器

---

## <font color="#e96900">总结</font>

### 父子组件通信
1. 子组件通过 `props` 属性来接受父组件的数据，然后父组件在子组件上注册监听事件，子组件通过 `emit` 触发事件来向父组件发送数据。
2. 通过 `ref` 属性给子组件设置一个名字。父组件通过 `$refs` 组件名来获得子组件，子组件通过 `$parent` 获得父组件，这样也可以实现通信。
3. 使用 `provide/inject`，在父组件中通过 `provide`提供变量，在子组件中通过 `inject` 来将变量注入到组件中。不论子组件有多深，只要调用了 `inject` 那么就可以注入 `provide` 中的数据。

### 兄弟组件通信
1. 使用 `eventBus` 的方法，它的本质是通过创建一个空的 Vue 实例来作为消息传递的对象，通信的组件引入这个实例，通信的组件通过在这个实例上监听和触发事件，来实现消息的传递
2. 通过` $parent/$root `来获取到兄弟组件，也可以进行通信。

### 任意组件之间
1. 使用 `eventBus `
2. 使用 `provide/inject`

---