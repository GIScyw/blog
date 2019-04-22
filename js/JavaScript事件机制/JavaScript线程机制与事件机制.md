## 一、进程与线程

### 1.进程

**进程是指程序的一次执行,它占有一片独有的内存空间,可以通过windows任务管理器查看进程**(如下图)。同一个时间里，同一个计算机系统中允许两个或两个以上的进程处于并行状态，这是多进程。比如电脑同时运行微信，QQ，以及各种浏览器等。**浏览器运行是有些是单进程，如firefox和老版IE，有些是多进程，如chrome和新版IE**。



![img](https://user-gold-cdn.xitu.io/2018/10/6/16648431acbbd129?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 2.线程

有些进程还不止同时干一件事，比如Word，它可以同时进行打字、拼写检查、打印等事情。在一个进程内部，要同时干多件事，就需要同时运行多个“子任务”，我们把进程内的这些“子任务”称为线程（Thread）。 **线程是指CPU的基本调度单位,是程序执行的一个完整流程，是进程内的一个独立执行单元**。多线程是指在一个进程内, 同时有多个线程运行。**浏览器运行是多线程**。比如用浏览器一边下载，一边听歌，一边看视频。另外我们需要知道**JavaScript语言的一大特点就是单线程**，为了利用多核CPU的计算能力，**HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质**。 

![img](https://user-gold-cdn.xitu.io/2018/10/6/166482074c742c88?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



由于每个进程至少要干一件事，所以，一个进程至少有一个线程。当然，像Word这种复杂的进程可以有多个线程，多个线程可以同时执行，多线程的执行方式和多进程是一样的，也是由操作系统在多个线程之间快速切换，让每个线程都短暂地交替运行，看起来就像同时执行一样。当然，真正地同时执行多线程需要多核CPU才可能实现。

### 3.进程与线程

- 应用程序必须运行在某个进程的某个线程上
- 一个进程中至少有一个运行的线程: 主线程,  进程启动后自动创建
- 一个进程中如果同时运行多个线程, 那这个程序是多线程运行的
- 一个进程的内存空间是共享的，每个线程都可以使用这些共享内存。
- 多个进程之间的数据是不能直接共享的

### 4.单线程与多线程的优缺点?

**单线程的优点**:顺序编程简单易懂

**单线程的缺点**:效率低

**多线程的优点**:能有效提升CPU的利用率

**多线程的缺点**:

- 创建多线程开销
- 线程间切换开销
- 死锁与状态同步问题

## 二、浏览器内核

浏览器的内核是指支持浏览器运行的最核心的程序，分为两个部分的，一是渲染引擎，另一个是JS引擎。现在JS引擎比较独立，内核更加倾向于说渲染引擎。

### 1.不同的浏览器可能不太一样

- Chrome, Safari: webkit
- firefox: Gecko
- IE: Trident
- 360,搜狗等国内浏览器: Trident + webkit

### 2.内核由很多模块组成

- html,css文档解析模块 : 负责页面文本的解析
- dom/css模块 : 负责dom/css在内存中的相关处理
- 布局和渲染模块 : 负责页面的布局和效果的绘制
- 定时器模块 : 负责定时器的管理
- 网络请求模块 : 负责服务器请求(常规/Ajax)
- 事件响应模块 : 负责事件的管理

## 三、定时器引发的思考

### 1. 定时器真是定时执行的吗?

我们先来看个例子，试问定时器会保证200ms后执行吗？

```javascript
 document.getElementById('btn').onclick = function () {
      var start = Date.now()
      console.log('启动定时器前...')
      setTimeout(function () {
        console.log('定时器执行了', Date.now() - start)
      }, 200)
      console.log('启动定时器后...')
      // 做一个长时间的工作
      for (var i = 0; i < 1000000000; i++) {
      }
    }
```



![img](https://user-gold-cdn.xitu.io/2018/10/9/166569cbd5e0175a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 事实上，经过了625ms后定时器才执行。定时器并不能保证真正定时执行，一般会延迟一丁点,也有可能延迟很长时间(比如上面的例子)



### 2.定时器回调函数是在分线程执行的吗?

**定时器回调函数在主线程执行的**, 具体实现方式下文会介绍。

## 四、浏览器的事件循环(轮询)模型

### 1. 为什么JavaScript是单线程

**JavaScript语言的一大特点就是单线程，也就是说，同一个时间只能做一件事**。那么，为什么JavaScript不能有多个线程呢？这样能提高效率啊。

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。 为了利用多核CPU的计算能力，**HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM**。所以，这个新标准并没有改变JavaScript单线程的本质。

### 2.Event Loop

JavaScript中所有任务可以分成两种，一种是同步任务，另一种是异步任务(如各种浏览器事件、定时器和Ajax等)。**同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行**。

具体来说，异步执行的运行机制如下。（同步执行也是如此，因为它可以被视为没有异步任务的异步执行。）

（1）所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。

（2）主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。

（3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。

（4）主线程不断重复上面的第三步

**主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）**



![img](https://user-gold-cdn.xitu.io/2018/10/9/1665863c875fc3cd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 下面这个例子很好阐释事件循环：



```javascript
    setTimeout(function () {
      console.log('timeout 2222')
      alert('22222222')
    }, 2000)
    setTimeout(function () {
      console.log('timeout 1111')
      alert('1111111')
    }, 1000)
    setTimeout(function () {
      console.log('timeout() 00000')
    }, 0)//当指定的值小于 4 毫秒，则增加到 4ms（4ms 是 HTML5 标准指定的，对于 2010 年及之前的浏览器则是 10ms）
    function fn() {
      console.log('fn()')
    }
    fn()
    console.log('alert()之前')
    alert('------') //暂停当前主线程的执行, 同时暂停计时, 点击确定后, 恢复程序执行和计时
    console.log('alert()之后')
```



![img](https://user-gold-cdn.xitu.io/2018/10/9/166570b5707028af?imageslim)



有两点我们需要注意下：

- 定时器零延迟(setTimeout(func, 0))并不是意味着回调函数立刻执行。至少4ms,才会执行回调函数。它取决于主线程当前是否空闲与“任务队列”里其前面正在等待的任务。
- 只有在到达指定时间时，定时器就会将相应回调函数插入“任务队列”尾部

**总结:异步任务（各种浏览器事件、定时器和Ajax等）都是先添加到“任务队列”（定时器则到达其指定参数时）。当 Stack 栈（JavaScript 主线程）为空时，就会读取 Queue 队列（任务队列）的第一个任务（队首），最后执行**。

## 五、H5 Web Workers(多线程)

### 1. Web Workers的作用

正如上面所提到，JavaScript是单线程。当一个页面加载一个复杂运算的 js 文件时，用户界面可能会短暂地“冻结”，不能再做其他操作。比如下面这个例子：

```javascript
<input type="text" placeholder="数值" id="number">
<button id="btn">计算</button>
<script type="text/javascript">
  // 1 1 2 3 5 8    f(n) = f(n-1) + f(n-2)
  function fibonacci(n) {
    return n<=2 ? 1 : fibonacci(n-1) + fibonacci(n-2)  //递归调用
  }
  var input = document.getElementById('number')
  document.getElementById('btn').onclick = function () {
    var number = input.value
    var result = fibonacci(number)
    alert(result)
  }
</script>
```



![img](https://user-gold-cdn.xitu.io/2018/10/11/1666195643c54dfa?imageslim)

 很显然遇到这种页面堵塞情况，很影响用户体验的，有没有啥办法可以改进这种情形？----Web Worker就应运而生了！



**Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程**。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。其原理图如下：



![img](https://user-gold-cdn.xitu.io/2018/10/11/16661d8e4a4d0cfd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 2. Web Workers的基本使用

**主线程**

- 首先主线程采用new命令，调用Worker()构造函数，新建一个 Worker 线程

```javascript
var worker = new Worker('work.js');
```

- 然后主线程调用worker.postMessage()方法，向 Worker 发消息。
- 接着，主线程通过worker.onmessage指定监听函数，接收子线程发回来的消息。

```javascript
  var input = document.getElementById('number')
  document.getElementById('btn').onclick = function () {
    var number = input.value
    //创建一个Worker对象
    var worker = new Worker('worker.js')
    // 绑定接收消息的监听
    worker.onmessage = function (event) {
      console.log('主线程接收分线程返回的数据: '+event.data)
      alert(event.data)
    }
    // 向分线程发送消息
    worker.postMessage(number)
    console.log('主线程向分线程发送数据: '+number)
  }
    console.log(this) // window
```

**Worker 线程**

- Worker 线程内部需要有一个监听函数，监听message事件。
- 通过 postMessage(data) 方法来向主线程发送数据。

```javascript
//worker.js文件
function fibonacci(n) {
  return n<=2 ? 1 : fibonacci(n-1) + fibonacci(n-2)  //递归调用
}
console.log(this)//[object DedicatedWorkerGlobalScope]
this.onmessage = function (event) {
  var number = event.data
  console.log('分线程接收到主线程发送的数据: '+number)
  //计算
  var result = fibonacci(number)
  postMessage(result)
  console.log('分线程向主线程返回数据: '+result)
  // alert(result)  alert是window的方法, 在分线程不能调用
  // 分线程中的全局对象不再是window, 所以在分线程中不可能更新界面
}
```

这样当分线程在计算时，用户界面还可以操作，而且更早拿到计算后数据，响应速度更快了。 

![img](https://user-gold-cdn.xitu.io/2018/10/11/166619796b5b3515?imageslim)



### 3. Web Workers的缺点

- 不能跨域加载JS
- worker内代码不能访问DOM(更新UI)
- 不是每个浏览器都支持这个新特性(**本文例子只能在Firefox浏览器上运行，chrome不支持**)



