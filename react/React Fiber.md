

### 引言

- React可以分为reconcilation和commit两个阶段

- React调和器主要作用就是在组件状态改变时，调用组件树各组件的render方法，渲染，卸载组件。在React V16之前的版本中，计算组件树变更时将会阻塞整个进程，整个渲染过程是连续不中断完成的，大量的组件渲染会导致主进程长时间被占用，而这时的其他任务都会被阻塞，如一些动画或高频操作出现卡顿和掉帧的情况。解决同步阻塞的方法，通常有两种，异步与任务分割。而Fiber Reconcile便是为了实现任务分割而诞生的。React V16将调度算法进行了重构，将之前的stack reconcile重构成新版的fiber reconcile，它允许渲染进程分段完成，而不是一次性完成，中间可以返回至主线程控制执行其他任务。而这是通过计算部分组件树的变更，并暂停渲染更新，询问主进程是否有更高需求的绘制或者更新任务需要执行，这些高需求的任务完成后才开始渲染
- 由于React新提出的fiber，导致React 官方对生命周期有了新的 变动建议，用`getDerivedStateFromProps` 替换`componentWillMount`；使用`getSnapshotBeforeUpdate`替换`componentWillUpdate`；避免使用`componentWillReceiveProps`。具体的，生命周期变动是因为在 Fiber 中，reconciliation 阶段进行了任务分割，涉及到 暂停 和 重启，因此可能会导致 reconciliation 中的生命周期函数在一次更新渲染循环中被 **多次调用** 的情况，产生一些意外错误。

### React核心流程

React的核心流程可以分为两个部分：

- reconciliation(调度算法，也可以称为render)
  - 更新state和props
  - 调用生命周期钩子
  - 生成vitual dom
  - 通过新旧vdom进行diff算法，获取vdom change
  - 确定是否需要重新渲染

- commit

  如果需要，则操作dom节点更新

### React 调和

React核心是定义组件，渲染组件方式由环境决定，定义组件，组件状态管理，生命周期方法管理，组件更新等应该跨平台一致处理，不受渲染环境影响，这部分内容统一由调和(Reconciler)处理，不同渲染器都会使用该模块。**调和器主要作用就是在组件状态变更时，调用组件树各组件的render方法，渲染，卸载组件**

#### Stack Reconcile

我们知道浏览器渲染引擎是单线程的，在React 15.x版本及之前版本，计算组件树变更时将会阻塞整个线程，整个渲染过程是连续不中断完成的，大量的组件渲染会导致主进程长时间被占用，而这时的其他任务都会被阻塞，如一些动画或高频操作出现卡顿和掉帧的情况，比如当你在访问某一网站时，输入某个搜索关键字，更优先的应该是交互反馈或动画效果，如果交互反馈延迟200ms，用户则会感觉较明显的卡顿，而数据响应晚200毫秒并没太大问题。这个版本的调和器可以称为栈调和器（Stack Reconciler），其调和算法大致过程见[React Diff算法](https://link.juejin.im?target=http%3A%2F%2Fblog.codingplayboy.com%2F2016%2F10%2F27%2Freact_diff%2F) 和[React Stack Reconciler实现](https://link.juejin.im?target=https%3A%2F%2Freactjs.org%2Fdocs%2Fimplementation-notes.html)。

Stack Reconcilier的主要缺陷就是不能暂停渲染任务，也不能切分任务，无法有效平衡组件更新渲染与动画相关任务间的执行顺序，即不能划分任务优先级，有可能导致重要任务卡顿，动画掉帧等问题。

#### Fiber Reconcile

React 16版本提出了一个更先进的调和器，它允许渲染进程分段完成，而不必须一次性完成，中间可以返回至主进程控制执行其他任务。而这是通过计算部分组件树的变更，并暂停渲染更新，询问主进程是否有更高需求的绘制或者更新任务需要执行，这些高需求的任务完成后才开始渲染。这一切的实现是在代码层引入了一个新的数据结构-Fiber对象，每一个组件实例对应有一个fiber实例，此fiber实例负责管理组件实例的更新，渲染任务及与其他fiber实例的联系。

这个新推出的调和器就叫做纤维调和器（Fiber Reconciler），它提供的新功能主要有：

1. 可切分，可中断任务；
2. 可重用各分阶段任务，且可以设置优先级；
3. 可以在父子组件任务间前进后退切换任务；
4. `render`方法可以返回多元素（即可以返回数组）；
5. 支持异常边界处理异常；

### Fiber Reconcile核心实现

在 React V16 将调度算法进行了重构， 将之前的 stack reconciler 重构成新版的 fiber reconciler，变成了具有链表和指针的 **单链表树遍历算法**。通过指针映射，每个单元都记录着遍历当下的上一步与下一步，从而使遍历变得可以被暂停和重启。这里我理解为是一种 **任务分割调度算法**，主要是 将原先同步更新渲染的任务分割成一个个独立的 **小任务单位**，根据不同的优先级，将小任务分散到浏览器的空闲时间执行，充分利用主进程的事件循环机制。

- Fiber 这里可以抽象为一个 **数据结构**:

```javascript
class Fiber {
	constructor(instance) {
		this.instance = instance
		// 指向第一个 child 节点
		this.child = child
		// 指向父节点
		this.return = parent
		// 指向第一个兄弟节点
		this.sibling = previous
	}	
}
```

- **链表树遍历算法**: 通过 **节点保存与映射**，便能够随时地进行 停止和重启，这样便能达到实现任务分割的基本前提；
  - 1、首先通过不断遍历子节点，到树末尾；
  - 2、开始通过 sibling 遍历兄弟节点；,
  - 3、return 返回父节点，继续执行2；
  - 4、直到 root 节点后，跳出遍历；

- **任务分割**:React 中的渲染更新可以分成两个阶段:
  - **reconciliation 阶段**: vdom 的数据对比，是个适合拆分的阶段，比如对比一部分树后，先暂停执行个动画调用，待完成后再回来继续比对。
  - **Commit 阶段**: 将 change list 更新到 dom 上，不适合拆分，因为使用 vdom 的意义就是为了节省传说中最耗时的 dom 操作，把所有操作一次性更新，如果在这里又拆分，那不是又懵了么。

- **分散执行**: 任务分割后，就可以把小任务单元分散到浏览器的空闲期间去排队执行，而实现的关键是两个新API: `requestIdleCallback` 与 `requestAnimationFrame`

  - 低优先级的任务交给`requestIdleCallback`处理，这是个浏览器提供的事件循环空闲期的回调函数，需要 pollyfill，而且拥有 deadline 参数，限制执行事件，以继续切分任务；
  - 高优先级的任务交给`requestAnimationFrame`处理；

  ```javascript
  // 类似于这样的方式
  requestIdleCallback((deadline) => {
      // 当有空闲时间时，我们执行一个组件渲染；
      // 把任务塞到一个个碎片时间中去；
      while ((deadline.timeRemaining() > 0 || deadline.didTimeout) && nextComponent) {
          nextComponent = performWork(nextComponent);
      }
  });
  ```

- **优先级策略**: 文本框输入 > 本次调度结束需完成的任务 > 动画过渡 > 交互反馈 > 数据更新 > 不会显示但以防将来会显示的任务

> Fiber 其实可以算是一种编程思想，在其它语言中也有许多应用(Ruby Fiber)。当遇到进程阻塞的问题时，**任务分割**、**异步调用** 和 **缓存策略** 是三个显著的解决思路。

### 生命周期

在新版本中，React 官方对生命周期有了新的 **变动建议**:

- 使用`getDerivedStateFromProps` 替换`componentWillMount`；
- 使用`getSnapshotBeforeUpdate`替换`componentWillUpdate`；
- 避免使用`componentWillReceiveProps`；

其实该变动的原因，正是由于上述提到的 Fiber。首先，从上面我们知道 React 可以分成 reconciliation 与 commit 两个阶段，对应的生命周期如下:

- **reconciliation**:
  - `componentWillMount`
  - `componentWillReceiveProps`
  - `shouldComponentUpdate`
  - `componentWillUpdate`
- **commit**:
  - `componentDidMount`
  - `componentDidUpdate`
  - `componentWillUnmount`

在 Fiber 中，reconciliation 阶段进行了任务分割，涉及到 暂停 和 重启，因此可能会导致 reconciliation 中的生命周期函数在一次更新渲染循环中被 **多次调用** 的情况，产生一些意外错误。

新版的建议生命周期如下:

```javascript
class Component extends React.Component {
  // 替换 `componentWillReceiveProps` ，
  // 初始化和 update 时被调用
  // 静态函数，无法使用 this
  static getDerivedStateFromProps(nextProps, prevState) {}
  
  // 判断是否需要更新组件
  // 可以用于组件性能优化
  shouldComponentUpdate(nextProps, nextState) {}
  
  // 组件被挂载后触发
  componentDidMount() {}
  
  // 替换 componentWillUpdate
  // 可以在更新之前获取最新 dom 数据
  getSnapshotBeforeUpdate() {}
  
  // 组件更新后调用
  componentDidUpdate() {}
  
  // 组件即将销毁
  componentWillUnmount() {}
  
  // 组件已销毁
  componentDidUnMount() {}
}
```

- **使用建议**:

  - 在`constructor`初始化 state；
  - 在`componentDidMount`中进行事件监听，并在`componentWillUnmount`中解绑事件；
  - 在`componentDidMount`中进行数据的请求，而不是在`componentWillMount`；
  - 需要根据 props 更新 state 时，使用`getDerivedStateFromProps(nextProps, prevState)` 
    - 旧 props 需要自己存储，以便比较；

  ```javascript
  public static getDerivedStateFromProps(nextProps, prevState) {
  	// 当新 props 中的 data 发生变化时，同步更新到 state 上
  	if (nextProps.data !== prevState.data) {
  		return {
  			data: nextProps.data
  		}
  	} else {
  		return null1
  	}
  }
  ```

  - 可以在`componentDidUpdate`监听 props 或者 state 的变化，例如:

  ```javascript
  componentDidUpdate(prevProps) {
  	// 当 id 发生变化时，重新获取数据
  	if (this.props.id !== prevProps.id) {
  		this.fetchData(this.props.id);
  	}
  }
  ```

  - 在`componentDidUpdate`使用`setState`时，必须加条件，否则将进入死循环；
  - `getSnapshotBeforeUpdate(prevProps, prevState)`可以在更新之前获取最新的渲染数据，它的调用是在 render 之后， update 之前；
  - `shouldComponentUpdate`: 默认每次调用`setState`，一定会最终走到 diff 阶段，但可以通过`shouldComponentUpdate`的生命钩子返回`false`来直接阻止后面的逻辑执行，通常是用于做条件渲染，优化渲染的性能。





参考资料：

[(中篇)中高级前端大厂面试秘籍，寒冬中为您保驾护航，直通大厂](https://juejin.im/post/5c92f499f265da612647b754)

[React Fiber初探](https://juejin.im/post/5a2276d5518825619a027f57)