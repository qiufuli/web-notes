# eventloop
>涉及面试题：进程与线程区别？JS 单线程带来的好处？

### 进程与线程
相信大家经常会听到 JS 是**单线程**执行的，但是你是否疑惑过什么是线程？

讲到线程，那么肯定也得说一下进程。本质上来说，两个名词都是 CPU **工作时间片**的一个描述。

进程描述了 CPU 在**运行指令及加载和保存上下文所需的时间**，放在应用上来说就代表了一个程序。线程是进程中的更小单位，描述了执行一段指令所需的时间。

**(进程相当于一个程序，线程是更小的单位，描述了执行一段指令所需要的时间)**

把这些概念拿到浏览器中来说，当你打开一个 Tab 页时，其实就是创建了一个进程，一个进程中可以有多个线程，比如渲染线程、JS 引擎线程、HTTP 请求线程等等。当你发起一个请求时，其实就是创建了一个线程，当请求结束后，该线程可能就会被销毁。

**（好处是：单线程无需考虑 你在读取数据的时候其他线程对该数据的修改所造成的错误）**

### 单线程
作为浏览器脚本语言，`JavaScript`的主要用途是与用户互动，以及操作`DOM`。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定`JavaScript`同时有两个线程，一个线程在某个`DOM`节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？
所以，为了避免复杂性，从一诞生，`JavaScript`就是单线程

### 执行栈
>涉及面试题：什么是执行栈？

重要！！！！！ **可以把执行栈认为是一个存储函数调用的`栈结构`，遵循`先进后出`的原则。**

![](https://user-gold-cdn.xitu.io/2018/11/13/1670d2d20ead32ec?w=1211&h=623&f=gif&s=140580)

当开始执行` JS `代码时，首先会执行一个 `main `函数，然后执行我们的代码。根据**先进后出**的原则，后执行的函数会先弹出栈，在图中我们也可以发现，`foo `函数后执行，当执行完毕后就从栈中弹出了。

### 浏览器中的 Event Loop
>涉及面试题：异步代码执行顺序？解释一下什么是 Event Loop ？

由于js是单线程的,只有当上一个任务完成之后才会继续完成下一个任务,如果前一个任务耗时很长，后一个任务就不得不一直等着。

那么问题来了，假如我们想浏览新闻，但是新闻包含的超清图片加载很慢，难道我们的网页要一直卡着直到图片完全显示出来？因此聪明的程序员将任务分为两类：
- **同步任务** ------ 在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务
- **异步任务** ------ 不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。关于这部分有严格的文字定义，但本文的目的是用最小的学习成本彻底弄懂执行机制，所以我们用导图来说明：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512173136421.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjI0NQ==,size_16,color_FFFFFF,t_70)

导图要表达的内容用文字来表述的话：

- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
- 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
- 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的Event Loop(事件循环)。


其实当遇到**异步**的代码时，会被**挂起**并在需要执行的时候加入到 `Task`（有多种 `Task`） 队列中。一旦执行栈为空，`Event Loop` 就会从` Task` 队列中拿出需要执行的代码并放入执行栈中执行，**所以本质上来说 `JS `中的异步还是同步行为**。

![](https://user-gold-cdn.xitu.io/2018/11/23/16740fa4cd9c6937?w=3161&h=1274&f=png&s=202906)


>那么 什么样的任务算是同步任务放到主线程 什么样的算是异步任务加入事件队列中呢?

除了广义的**同步任务**和**异步任务**，我们对任务有更精细的定义：

**macro-task(宏任务)：**
- 包括整体代码`script`，`setTimeout`，`setInterval`
- 加入主线程 同步执行,每次执行栈执行的代码就是一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行,每一个宏任务会从头到尾将这个任务执行完毕，不会执行其它）
**micro-task(微任务)：**
- `Promise.then()`，`process.nextTick`
- 加入事件队列，注册回调函数后添加到Task queue，等待当前宏任务执行完立即执行,可以理解是在当前 `task` 执行结束后立即执行的任务

> 每创建一个宏任务，需要当前的宏任务+当前的微任务执行完，算执行完一次宏任务，然后去执行下一个宏任务<br>
（所以说为什么不管`setTimeout`宏任务不管放到哪里执行，都需要等到当前宏任务执行完之后才能执行）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512173446952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzMjI0NQ==,size_16,color_FFFFFF,t_70)

---

### 代码讲解

```js
setTimeout(function() {
    console.log('1');
})

new Promise(function(resolve) {
    console.log('2');
}).then(function() {
    console.log('3');
})

console.log('4');

//打印顺序 2 4 3 1
```

首先整体代码是一个宏任务,遇到`setTimeout`,会创建另一个宏任务,接着执行当前的宏任务,`Promise` 新建后就会立即执行。所以会首先打印`2`，`then`方法是一个微任务，遇到`then`，添加到微任务队列，代码接着执行会打印`4`。此时宏任务执行完毕，接着就会检查当前微任务队列是否有微任务，如果有，立即执行当前的微任务（也就是`then` 打印`3`），当前微任务执行完毕之后，开始执行下一轮的宏任务`setTimeout`，会打印`1`。

```js
console.log('script start')

async function async1() {
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 end')
}
async1()

setTimeout(function() {
  console.log('setTimeout')
}, 0)

new Promise(resolve => {
  console.log('Promise')
  resolve()
})
  .then(function() {
    console.log('promise1')
  })
  .then(function() {
    console.log('promise2')
  })

console.log('script end')
// script start => async2 end => Promise => script end => async1 end => promise1 => promise2 => setTimeout
```

首先会打印出 `script start` ，然后遇到`async`函数（`promsie`的语法糖，内容先执行，`then`放入队列）,执行`async2 end`，遇到`setTimeout`再创建一个宏任务，遇到`Promise`执行`Promise` `then`的内容放入事件队列，然后执行`script end`，这时当前的同步任务已经执行完，去执行事件队列中的回调，这时打印出`async1 end`这是第一个假如task队列的回调，然后执行`promise1，promise2`这是`Promise`的回调，这时当前的宏任务里面所有的同步任务和异步任务都已经执行完成，那么下面会去执行下个宏任务，这时打印的是`setTimeout`，按照上面的代码结构，这时所有的任务都已经执行完成了。