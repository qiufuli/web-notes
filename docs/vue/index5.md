<!--
 * @Author: qfuli
 * @Date: 2023-11-09 15:18:20
 * @LastEditors: qfuli
 * @LastEditTime: 2023-11-09 15:23:44
 * @Description: Do not edit
 * @FilePath: /web-notes/docs/vue/index5.md
-->
# vuex

## 简单说一下对vuex的了解

- `state`∶ 页面状态管理容器对象。集中存储Vuecomponents中data对象的零散数据，全局唯一，以进行统一的状态管理。页面显示所需的数据从该对象中进行读取，利用Vue的细粒度数据响应机制来进行高效的状态更新。
- `getters`∶ state对象读取方法。图中没有单独列出该模块，应该被包含在了render中，Vue Components通过该方法读取全局state对象。
- `mutations`∶状态改变操作方法。是Vuex修改state的唯一推荐方法，其他修改方式在严格模式下将会报错。该方法只能进行同步操作，且方法名只能全局唯一。操作之中会有一些hook暴露出来，以进行state的监控等。
-` actions`∶ 操作行为处理模块。负责处理Vue Components接收到的所有交互行为。包含同步/异步操作，支持多个同名方法，按照注册的顺序依次触发。向后台API请求的操作就在这个模块中进行，包括触发其他action以及提交mutation的操作。该模块提供了Promise的封装，以支持action的链式触发。
- `modules`∶ 模块管理器。用于模块化的状态管理，需要安装插件。
- `commit`∶状态改变提交操作方法。对mutation进行提交，是唯一能执行mutation的方法。
- `dispatch`∶操作行为触发方法，是唯一能执行action的方法。

总结：
Vuex 实现了一个单向数据流，在全局拥有一个 State 存放数据，当组件要更改 State 中的数据时，必须通过 Mutation 提交修改信息， Mutation 同时提供了订阅者模式供外部插件调用获取 State 数据的更新。而当所有异步操作(常见于调用后端接口异步获取更新数据)或批量的同步操作需要走 Action ，但 Action 也是无法直接修改 State 的，还是需要通过Mutation 来修改State的数据。最后，根据 State 的变化，渲染到视图上。

##  Vuex中action和mutation的区别

1. mutation中的操作是一系列的同步函数，用于修改state中的变量的的状态
2. Action 可以包含任意异步操作,Action 提交的是 mutation，而不是直接变更状态。