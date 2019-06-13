

组件的生命周期分为3个阶段：挂载阶段、更新阶段、卸载阶段，每个阶段都包含相应的生命周期方法,对于生命周期，我们需要注意以下几点：

- 每个阶段执行了哪些生命周期以及它们的执行顺序（包括初始化、组件props改变引起的更新，组件state改变引起的更新，组件销毁四种情况下执行的是哪些生命周期）

- 每个生命周期内可以干哪些事情

- 哪些生命周期可以发起数据请求

- 哪些生命周期可以setState，哪些不可以

  

### 概念

![img](https://user-gold-cdn.xitu.io/2018/5/25/1639548946e168e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- `constructor·是ES6对类的默认方法，主要用来初始化`state`

- `componentWillMount()`用于组件挂之前，全局只调用一次，如果在这个钩子里可以`setState`，`render`后可以看到更新后的`state`，不会触发重复渲染。该生命周期可以发起请求，并`setState`；这是服务端渲染中唯一调用的钩子； 通常情况下，推荐用`constructor()`方法代替;

- `render()`用于渲染组件，`render`必须定义，用来渲染`dom`,不要在`render`里面修改`state`，会触发死循环导致栈溢出。

- `componentDidMount`用于组件挂载完成后，全局只调用一次，可以在这里使用`refs`获取真实的`dom`元素。该钩子内也可以发起异步请求，并在异步请求后可以进行`setState`。

- `componentWillReceiveProps(nextProps)`用于在`props`即将变化之前，`props`发生变化以及父组件重新渲染时都会触发该生命周期，在该生命周期可以通过参数`nextProps`获取变化后的`props`参数，通过this.props访问之前的`props`。该生命周期内可以进行`setState`。

- `shouldComponentUpdate(nextProps,nextState)`用来确定是否重新渲染，组件挂载之后，每次调用`setState`后都会调用`shouldComponentUpdate`判断是否需要重新渲染组件。默认返回`true`，需要重新`render`，返回`false`则不触发渲染。在一些比较复杂的应用中，有一些数据的改变并不影响界面展示，可以在这里做判断，优化渲染效率。

- `componentWillupdate(nextProps,nextState)`，`shouldComponentUpdate(nextProps,nextState)`返回`true`或者调用`forceUpdate`之后，`componentWillUpdate`会被调用，不能在钩子中`setState`，会触发重复渲染。

- `componentDidUpdate()`用于完成组件渲染，除了首次调用`componentDidMount`，其他`render`结束之后都是调用`componentDidupdate`。该钩子内`setState`有可能会触发重复渲染，需要自行判断，否则会进入死循环。

- `componentWillUnmount()`用于组件即将被卸载，一般`componentDidMount`里面注册的事件需要在这里删除

###  阶段

####  组件初始化

  `componentWillMount` -> `render` ->` componentDidMount`

####  组件更新 – props change

 ` componentWillReceiveProps` -> `shouldComponentUpdate` -> `componentWillUpdate` ->   `componentDidUpdate`

#### 组件更新 – state change

` shoudlComponentUpdate` -> `componentWillUpdate` -> `componentDidUpdate`

#### 组件卸载或销毁

` componentWillUnmount`

### 应用场景

#### 数据请求的时机

对于组件所需的初始数据，最合适的地方，是在`componentDidMount`方法中，进行数据请求，这个时候，组件完成挂载，其代表的DOM已经挂载到页面的DOM树上，即使获取到的数据需要直接操作`DOM`节点，这个时候也是绝对安全的。有些人还习惯在`constructor`或者`componentWillMount`中，进行数据请求，认为这样可以更快的获取到数据，但它们相比`componentDidMount`的执行时间，提前的时间实在是太微乎其微了。另外，当进行服务器渲染时，`componentWillMount`是会被调用两次的，一次在服务器端，一次在客户端，这时候就会导致额外的请求发生。

组件进行数据请求的另一种场景：由父组件的更新导致组件的props发生变化，如果组件的数据请求依赖props，组件就需要重新进行数据请求。例如，新闻详情组件NewsDetail，在获取新闻详情数据时，需要传递新闻的id作为参数给服务器端，当NewsDetail已经处于挂载状态时，如果点击其他新闻，NewsDetail的componentDidMount并不会重新调用，因而componentDidMount中进行新闻详情数据请求的方法也不会再次执行。这时候，应该在componentWillReceiveProps中，进行数据请求：

```javascript
componentWillReceiveProps(nextProps) {
  if(this.props.newId !== nextProps.newsId) {
    fetchNewsDetailById(nextProps.newsId)  // 根据最新的新闻id，请求新闻详情数据
  }
}
```

如果进行数据请求的时机是由页面上的交互行为触发的，例如，点击查询按钮后，查询数据，这时只需要在查询按钮的事件监听函数中，执行数据请求即可，这种情况一般是不会有疑问的。

#### setState的时机

组件的生命周期方法众多，哪些方法中可以调用setState更新组件状态？哪些方法中不可以呢？

- **可以的方法**

  componentWillMount、componentDidMount、componentWillReceiveProps、componentDidUpdate

  这里有几个注意点：

  1. componentWillMount 中**同步**调用setState不会导致组件进行额外的渲染，组件经历的生命周期方法依次是componentWillMount -> render -> componentDidMount，组件并不会因为componentWillMount中的setState调用再次进行更新操作。如果是**异步**调用setState，组件是会进行额外的更新操作。不过实际场景中很少在componentWillMount中调用setState，一般可以通过直接在constructor中定义state的方式代替。
  2. 一般情况下，当调用setState后，组件会执行一次更新过程，componentWillReceiveProps等更新阶段的方法会再次被调用，但如果在componentWillReceiveProps中调用setState，并不会额外导致一次新的更新过程，也就是说，当前的更新过程结束后，componentWillReceiveProps等更新阶段的方法**不会**再被调用一次。（注意，这里仍然指**同步**调用setState，如果是异步调用，则会导致组件再次进行渲染）
  3. componentDidUpdate中调用setState要格外小心，在setState前必须有条件判断，只有满足了相应条件，才setState，否组组件会不断执行更新过程，进入死循环。因为setState会导致新一次的组件更新，组件更新完成后，componentDidUpdate被调用，又继续setState，死循环就产生了。

- **不可以的方法**

  其他生命周期方法都不能调用setState，主要原因有两个：

  1. 产生死循环。例如，shouldComponentUpdate、componentWillUpdate 和 render 中调用setState，组件本次的更新还没有执行完成，又会进入新一轮的更新，导致不断循环更新，进入死循环。
  2. 无意义。componentWillUnmount 调用时，组件即将被卸载，setState是为了更新组件，在一个即将卸载的组件上更新state显然是无意义的。实际上，在componentWillUnmount中调用setState也是会抛出异常的。

### 新版本的生命周期

#### react v16.3删掉以下三个生命周期

- componentWillMount
- componentWillReceiveProps
- componentWillUpdate

#### 新增两个生命周期

- static getDerivedStateFromProps
- getSnapshotBeforeUpdate

#### static getDerivedStateFromProps

- 触发时间：在组件构建之后(虚拟dom之后，实际dom挂载之前) ，以及每次获取新的props之后。
- 每次接收新的props之后都会返回一个对象作为新的state，返回null则说明不需要更新state.
- 配合componentDidUpdate，可以覆盖componentWillReceiveProps的所有用法

```javascript
class Example extends React.Component {
  static getDerivedStateFromProps(nextProps, prevState) {
    // 没错，这是一个static
  }
}
```

#### getSnapshotBeforeUpdate

- 触发时间: update发生的时候，在render之后，在组件dom渲染之前。
- 返回一个值，作为componentDidUpdate的第三个参数。
- 配合componentDidUpdate, 可以覆盖componentWillUpdate的所有用法。

```javascript
class Example extends React.Component {
	getSnapshotBeforeUpdate(prevProps, prevState) {
	// ...
	}
}
```





**参考文章：**

深入React技术栈

[React 深入系列４：组件的生命周期](https://juejin.im/post/5addb07af265da0b8e7f032a)

[React组件生命周期详解]()