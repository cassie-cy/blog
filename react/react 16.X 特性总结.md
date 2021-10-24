# React 16+

## 总结：

主要解决大型 React 项目的性能问题。

顺手解决痛点：

1. 组件不能返回数组，最见的场合是 UL 元素下只能使用 LI，用 div 包裹破坏语义性。 => Fragment <></>
2. 弹窗问题，之前一直使用不稳定的 unstable_renderSubtreeIntoContainer。 => createPortal
3. 异常处理，太深的组件树查找困难。 => componentDidCatch ErrorBoundary
4. HOC 的流行带来两个问题，ref 与 context 的向下传递。
5. 组件性能优化靠人肉。主要集中在 shouldComponentUpdate、memo、pureComponent => 批量更新及基于时间分片的限量更新。

## 生命周期：

### React17 将移除 componentWillReceiveProps、componentWillMount、componentWillUpdate 生命周期，并新增 getDerivedStateFromProps 和 componentWillReceiveProps

![新生命周期图](/assets/flex.jpeg)

新的生命周期中，将移除 componentWillReceiveProps、componentWillMount、componentWillUpdate 生命周期。

废弃 componentWillMount 出于以下考虑：

1. 这些生命周期函数经常被滥用。componentWillMount 生命周期中不应该出现副作用。比如在 componentWillMount 中不能执行 setState 的操作，因为 componentWillMount 之后会立即执行 render()，所以在 componentWillMount 执行 setState 不会触发重新渲染。（？）与其约束开发者，不如直接干掉！

2. 服务端渲染会执行到 render 之前的生命周期，所以 componentWillMount 生命周期会在服务端执行一次、客户端再执行一次。（具体会引发什么问题？）（getDerivedStateFromProps 也会？）除此之外，服务端不会执行 componentWillUnMount 生命周期（服务端到底会执行哪些生命周期，以及 why），在 componentWillMount 中订阅事件可能会造成内存泄漏。

3. Fiber 架构中，任务会被随时打断（也是因为 componentWillMount 在 render 前执行），下次续上的时候 componentWillMount 可能会再次执行。

废弃 componentWillReceiveProps 出于以下考虑：

不是功能问题、是性能问题。componentWillReceiveProps 的常用方式是，对比新旧 props 是否变化，从而去 setState()。当 props 在短时间内多次变化时，componentWillReceiveProps 会多次调用，不会被合并。如果里面做异步请求，会导致多个异步请求阻塞。

废弃 componentWillUpdate 出于以下考虑：

同 componentWillReceiveProps 会被调用多次。且 componentWillUpdate 本来没有什么必用的用处，只是为了对称好看。

迁移指北：

- componentWillMount
  数据请求用 componentDidMount() 替换。在 componentDidMount 中 setState 会在浏览器绘制首屏前出发重新 render。

- componentWillReceiveProps
  使用 getDerivedStateFromProps 代替，静态方法（避免再次被滥用），不可访问 this，只能从 props 中获取 prevState，作用更纯碎。getDerivedStateFromProps 只要父级重新渲染就会被重新调用，不管 props 是否改变。

- componentWillUpdate
  使用 componentDidUpdate 代替。一般会在这里干什么呢？

> 篇外：看官方文档时 get 了一个重置组件 state 的方法新思路。修改组件的 key 值，整个组件都会重新渲染，无需再手动清空状态！（TODO：给 Demo）另外，重置组件的其他思路：1. 组件初始化的开销太大导致 key 不起作用，在 getDerivedStateFromProps 观察 userID 的变化： return null 重置 state。 2. 使用实例方法传给父级，重置非受控组件。

```js
<EmailInput defaultEmail={this.props.user.email} key={this.props.user.id} />
```

### Context API

数据存储解决方案。但是有性能问题。(TODO 性能对比)

### React hook

解决了什么问题？

1. 非 UI 的逻辑组件复用困难。只能使用 HOC 高阶组件或者 renderProps 复用带有状态或副作用的逻辑组件。（TODO demo 高阶、renderProps 以及 hooks 复用写法对比）。嵌套的高阶组件 props 容易丢失或覆盖，且容易产生嵌套地狱。

### Portals

解决了什么问题？

将子组件渲染到父组件以外的 DOM 节点。用于弹窗、悬浮卡、加载动画。（TODO demo）在此之前一直使用的是 unstable_renderSubtreeIntoContainer。

```js
ReactDOM.createPortal(child, container);
```

### ErrorBoundary 和 componentDidCatch(error, errorInfo)

### 服务端 renderToString() 和 renderToStream()

### fiber 架构

标签化天然套嵌的结构 -> 最终编译成递归执行的代码 ->栈调度器->不能随意的 break -> 链表结构可以 break

React15, 每次更新从根组件或 setState 后的组件开始，更新整个子树，我们用 scu 来断开某一部分的更新。
React16, 虚拟 DOM 转化成 Fiber -> Fiber 转换成组件实例或真实 DOM(耗时, 会执行 render and before)

### setImmediate，requestAnimationFrame 定时器 TODO

requestAnimationFrame：做动画时经常用到，jQuery 新版本都使用它
requestIdleCallback：React
web worker：angular2
IntersectionObserver：ListView

[React Fiber 架构](https://zhuanlan.zhihu.com/p/37095662)

参考：
(三张图对比 React 组件生命周期)[https://zhuanlan.zhihu.com/p/60168527]
(componentWillReceiveProps 替代方案)[https://caoss.github.io/2020/12/14/css.html]
(你可能不需要使用派生 state)[https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization]
(React 为什么需要 Hook)[https://zhuanlan.zhihu.com/p/137183261]
