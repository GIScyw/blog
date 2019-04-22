### 引言

在平时我们会经常遇到这样一个情况：给定的几行代码，我们需要知道其输入内容和顺序，这就需要我们了解JavaScript的执行机制，即JavaScript处理同步任务和异步任务的机制。

下面先列出本文主要的几个结论

- JavaScript不是一行一行执行的，这是因为JavaScript中引入了异步的概念，我们是需要理解JavaScript的执行机制才能知道先执行谁后执行谁，先输出谁后输出谁。
- javascript是一门单线程语言，不管是什么新框架新语法糖实现的所谓异步，其实都是用同步的方法去模拟的
- JavaScript中的同步和异步指的是会进入到不同执行场所的任务
- 事件循环Event Loop是JavaScript的执行机制，事件循环可以理解为任务循环

### JavaScript中的异步

JavaScript是一门单线程语言，如果对于拿到的程序，一行一行的执行，上面的没有执行完成下面是不会执行的，只能慢慢的等待，特别是对于JavaScript最初使用的环境即浏览器就更不一样了，因为对于浏览器端运行的js,可能会有大量的网络请求，而一个网络资源啥时候返回，这个时间是无法预估的，这种情况如果只是慢慢的等，卡顿着，什么都不做是不行的。因此，JavaScript对于这种场景设计了异步，即发起一个网络请求时先发起再说，先干其他事情，网络请求啥时候返回结果，到时候再说，这样就能保证一个网页的流程运行。我们这里要注意的是，JavaScript中的异步操作导致代码的执行不是一行一行执行的，我们是需要理解JavaScript的执行机制才能知道先执行谁后执行谁，先输出谁后输出谁。

**JavaScript的同步：**同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。实际上同步任务可以视为没有异步任务的同步执行，而异步任务可以理解为有异步任务的同步执行。

**JavaScript的异步：**异步任务指的是，不进入主线程，而进入任务队列的任务，只有任务队列通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。宏任务和微任务的执行会有一些区别，具体地下面会阐述。

**常见的异步操作：**

JavaScript中一般占用资源大耗时久的操作都应该写成异步的形式

- 网络请求，如ajax，http.get
- IO操作，如readFile，readdir
- 定时函数，如setTimeout,setInterval

> 这里要注意，事件绑定到底属不属于异步操作，反正我是把它理解成异步操作，这样可以简化很多问题

### JavaScript事件循环Event Loop

JavaScript是单线程，又因为在经常处理一些耗时的操作，因此JavaScript中才将任务分为两类，即同步任务与异步任务，当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染，而像加载图片音乐等占用资源大耗时久的任务，就是异步任务，那么JavaScript中会一套机制来处理这两类任务：


![img](https://user-gold-cdn.xitu.io/2017/11/21/15fdd88994142347?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


- 同步和异步任务分别进入不同的执行场所，同步的进入主线程，异步的进入Event Table并注册函数
- 当指定的事情完成时，Event Table会将这个函数移入Event Queue
- 主线程内的任务执行完毕为空时，会去Event Queue读取对应的函数，进入主线程执行
- 上述过程会不断重复，也就是常说的Event Loop(事件循环)

```javascript
let data = [];
$.ajax({
    url:www.javascript.com,
    data:data,
    success:() => {
        console.log('发送成功!');
    }
})
console.log('代码执行结束');
```

上面是一段简易的`ajax`请求代码：

- ajax进入Event Table，注册回调函数`success`。
- 执行`console.log('代码执行结束')`。
- ajax事件完成，回调函数`success`进入Event Queue。
- 主线程从Event Queue读取回调函数`success`并执行。

#### setTimeout的事件循环

**setTimeout（task, k)函数，是经过指定时间后，把要执行的任务加入到Event Queue中，而不是立即执行，又因为单线程任务要一个一个的执行，所以并不是延时了k秒，而实际上是大于k秒才执行的**。

```javascript
setTimeout(() => {
    task()
},3000)

sleep(10000000)
```

我们把这段代码在chrome执行一下，会发现控制台执行`task()`需要的时间远远超过3秒，说好的延时三秒，为啥现在需要这么长时间啊？

这时候我们需要重新理解`setTimeout`的定义。我们先说上述代码是怎么执行的：

- `task()`进入Event Table并注册,计时开始。
- 执行`sleep`函数，很慢，非常慢，计时仍在继续。
- 3秒到了，计时事件`timeout`完成，`task()`进入Event Queue，但是`sleep`也太慢了吧，还没执行完，只好等着。
- `sleep`终于执行完了，`task()`终于从Event Queue进入了主线程执行。

我们还经常遇到`setTimeout(fn,0)`这样的代码，0秒后执行又是什么意思呢？是不是可以立即执行呢？

答案是不会的，**`setTimeout(fn,0)`的含义是，指定某个任务在主线程最早可得的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行**。关于`setTimeout`要补充的是，即便主线程为空，0毫秒实际上也是达不到的。根据HTML的标准，最低是4毫秒。

#### setInterval的事件循环

setInterval与setTimeout差不多，只不过后者是循环的执行，对于执行顺序来说，setInterval会每隔指定的时间将注册的函数加入到Event Queue中，如果前面的任务耗时太久，那么同样需要等待。

唯一需要注意的一点是，对于`setInterval(fn,ms)`来说，我们已经知道不是每过`ms`秒会执行一次`fn`，而是每过`ms`秒，会有`fn`进入Event Queue。一旦**setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了**。

#### Promise和process.nextTick()的事件循环

除了广义的同步任务和异步任务，我们对任务有更精细的定义：

- macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
- micro-task(微任务)：Promise，process.nextTick

不同类型的任务会进入对应的Event Queue，比如`setTimeout`和`setInterval`会进入相同的Event Queue。**事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。**

```javascript
setTimeout(function() {
    console.log('setTimeout');
})

new Promise(function(resolve) {
    console.log('promise');
}).then(function() {
    console.log('then');
})

console.log('console');
```

- 这段代码作为宏任务，进入主线程。
- 先遇到`setTimeout`，那么将其回调函数注册后分发到宏任务Event Queue。(注册过程与上同，下文不再描述)
- 接下来遇到了`Promise`，`new Promise`立即执行，`then`函数分发到微任务Event Queue。
- 遇到`console.log()`，立即执行。
- 好啦，整体代码script作为第一个宏任务执行结束，看看有哪些微任务？我们发现了`then`在微任务Event Queue里面，执行。
- ok，第一轮事件循环结束了，我们开始第二轮循环，当然要从宏任务Event Queue开始。我们发现了宏任务Event Queue中`setTimeout`对应的回调函数，立即执行。
- 结束。

事件循环，宏任务，微任务的关系如图所示：



![img](https://user-gold-cdn.xitu.io/2017/11/21/15fdcea13361a1ec?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





**附加：**

我们来分析一段较复杂的代码，看看你是否真的掌握了js的执行机制：

```javascript
console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})
```

第一轮事件循环流程分析如下：

- 整体script作为第一个宏任务进入主线程，遇到`console.log`，输出1。
- 遇到`setTimeout`，其回调函数被分发到宏任务Event Queue中。我们暂且记为`setTimeout1`。
- 遇到`process.nextTick()`，其回调函数被分发到微任务Event Queue中。我们记为`process1`。
- 遇到`Promise`，`new Promise`直接执行，输出7。`then`被分发到微任务Event Queue中。我们记为`then1`。
- 又遇到了`setTimeout`，其回调函数被分发到宏任务Event Queue中，我们记为`setTimeout2`。

| 宏任务Event Queue | 微任务Event Queue |
| ----------------- | ----------------- |
| setTimeout1       | process1          |
| setTimeout2       | then1             |

- 上表是第一轮事件循环宏任务结束时各Event Queue的情况，此时已经输出了1和7。
- 我们发现了`process1`和`then1`两个微任务。
- 执行`process1`,输出6。
- 执行`then1`，输出8。

好了，第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。那么第二轮时间循环从`setTimeout1`宏任务开始：

- 首先输出2。接下来遇到了`process.nextTick()`，同样将其分发到微任务Event Queue中，记为`process2`。`new Promise`立即执行输出4，`then`也分发到微任务Event Queue中，记为`then2`。

| 宏任务Event Queue | 微任务Event Queue |
| ----------------- | ----------------- |
| setTimeout2       | process2          |
|                   | then2             |

- 第二轮事件循环宏任务结束，我们发现有`process2`和`then2`两个微任务可以执行。
- 输出3。
- 输出5。
- 第二轮事件循环结束，第二轮输出2，4，3，5。
- 第三轮事件循环开始，此时只剩setTimeout2了，执行。
- 直接输出9。
- 将`process.nextTick()`分发到微任务Event Queue中。记为`process3`。
- 直接执行`new Promise`，输出11。
- 将`then`分发到微任务Event Queue中，记为`then3`。

| 宏任务Event Queue | 微任务Event Queue |
| ----------------- | ----------------- |
|                   | process3          |
|                   | then3             |

- 第三轮事件循环宏任务执行结束，执行两个微任务`process3`和`then3`。
- 输出10。
- 输出12。
- 第三轮事件循环结束，第三轮输出9，11，10，12。

整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12。
(请注意，node环境下的事件监听依赖libuv与前端环境不完全相同，输出顺序可能会有误差)



**参考文章：**

1. [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89#heading-1)
2. [总是一知半解的Event Loop](https://juejin.im/post/5927ca63a0bb9f0057d3608e)
3. [JavaScript线程机制与事件机制](https://juejin.im/post/5bb05494e51d450e7428da59#heading-14)