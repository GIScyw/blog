

## Promise的基本用法

#### 什么是Promise?

> Promise是JS异步编程中的重要概念，异步抽象处理对象，是目前比较流行Javascript异步编程解决方案之一

#### 常见异步编程方案

- 回调函数
- 事件监听
- 发布/订阅
- Promise对象

对于回调函数，当我们需要发送多个异步请求，并且每个请求之间需要相互依赖，那这时以嵌套的形式来解决会形成“回调地狱”

```javascript
$.get(url,data=>{
    console.log(data1);
    $.get(data1.url,data2=>{
        console.log(data1)
    })
})
```

而我们可以用Promise更直观地解决“回调地狱”

```javascript
const request = url => { 
    return new Promise((resolve, reject) => {
        $.get(url, data => {
            resolve(data)
        });
    })
};

// 请求data1
request(url).then(data1 => {
    return request(data1.url);   
}).then(data2 => {
    return request(data2.url);
}).then(data3 => {
    console.log(data3);
}).catch(err => throw new Error(err));
```

事件监听的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"，有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。阅读代码的时候，很难看出主流程；发布/订阅性质与“事件监听”类似，但是明显优于后者。因为可以通过查看“消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

#### Promise的使用

Promise 是一个构造函数， new Promise 返回一个 promise对象 接收一个excutor执行函数作为参数, excutor有两个函数类型形参resolve reject

```javascript
const promise = new Promise((resolve, reject) => {
       // 异步处理
       // 处理结束后、调用resolve 或 reject
});
```

> 实际上，可以有两种方式来初始化一个Promise对象，这两种方式都会返回一个Promise对象
>
> - new Promise(fn)
> - Promise.resolve(fn)

promise相当于一个状态机

- pending
- fulfilled
- rejected

(1) promise 对象初始化状态为 pending

(2) 当调用resolve(成功)，会由pending => fulfilled

(3) 当调用reject(失败)，会由pending => rejected

> 注意promsie状态 只能由 pending => fulfilled/rejected, 一旦修改就不能再变

#### Promise实例方法

(1) Promise.prototype.then方法注册 当resolve(成功)/reject(失败)的回调函数

```javascript
// onFulfilled 是用来接收promise成功的值
// onRejected 是用来接收promise失败的原因
promise.then(onFulfilled, onRejected);
```

> 注意：then方法是异步执行的,而Promise构造函数时同步进行的

(4) Promise.prototype.catch

在链式写法中可以捕获前面then中发送的异常

```javascript
promise.catch(onRejected)
相当于
promise.then(null, onRrejected);

// 注意
// onRejected 不能捕获当前onFulfilled中的异常
promise.then(onFulfilled, onRrejected); 

// 可以写成：
promise.then(onFulfilled)
       .catch(onRrejected); 
```

#### Promise chain

promise.then方法每次调用 都返回一个新的promise对象 所以可以链式写法

```javascript
function taskA() {
    console.log("Task A");
}
function taskB() {
    console.log("Task B");
}
function onRejected(error) {
    console.log("Catch Error: A or B", error);
}

var promise = Promise.resolve();
promise
    .then(taskA)
    .then(taskB)
    .catch(onRejected) // 捕获前面then方法中的异常
```

#### Promise的静态方法

(1) Promise.resolve 返回一个fulfilled状态的promise对象

```javascript
Promise.resolve('hello').then(function(value){
    console.log(value);
});

Promise.resolve('hello');
// 相当于
const promise = new Promise(resolve => {
   resolve('hello');
});
```

(2) Promise.reject 返回一个rejected状态的promise对象

```javascript
Promise.reject(24);
//相当于
new Promise((resolve, reject) => {
   reject(24);
});
```

(3) Promise.all 接收一个promise对象数组为参数

只有全部为resolve才会调用,通常会用来处理 多个并行异步操作

```javascript
const p1 = new Promise((resolve, reject) => {
    resolve(1);
});

const p2 = new Promise((resolve, reject) => {
    resolve(2);
});

const p3 = new Promise((resolve, reject) => {
    resolve(3);
});

Promise.all([p1, p2, p3]).then(data => { 
    console.log(data); // [1, 2, 3] 结果顺序和promise实例数组顺序是一致的
}, err => {
    console.log(err);
});
```

(4) Promise.race 接收一个promise对象数组为参数

只要有一个Promise实例率先发生变化（无论状态是变成resolved还是rejected）都会触发then中的回调，返回值将传递给回调

```javascript
function timerPromisefy(delay) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve(delay);
        }, delay);
    });
}
var startDate = Date.now();

Promise.race([
    timerPromisefy(10),
    timerPromisefy(20),
    timerPromisefy(30)
]).then(function (values) {
    console.log(values); // 10
});
```

#### Promise与事件循环

Promise在初始化时，传入的函数是同步执行的，然后注册 then 回调。注册完之后，继续往下执行同步代码，在这之前，then 中回调不会执行。**同步代码块执行完毕后，才会在事件循环中检测是否有可用的 promise 回调**，如果有，那么执行，如果没有，继续下一个事件循环。例如下面的代码

```javascript
const promise = new Promise((resolve, reject) => {
console.log(1)
resolve()
console.log(2)
})
promise.then(() => {
console.log(3)
})
console.log(4)

//运行结果
1
2
3
4
```

## Promise的注意点

#### Promise的立即执行性

这指的是Promise的构造函数时立即执行的，从源码中看出，在构造函数里面就已经执行executor(resolve,reject)函数，因此Promise是立即执行的，即当构造一个Promise实例是就已经执行了。

```javascript
var p = new Promise(function(resolve, reject){
  console.log("create a promise");
  resolve("success");
});

console.log("after new Promise");

p.then(function(value){
  console.log(value);
});
```

控制台输出：

```css
"create a promise"
"after new Promise"
"success"
```

#### Promise then()的回调异步性

这是因为在源码中，不管是同步方法还是异步方法 ，最后都会利用setTimeout封装成异步方法。因此then方法里面的函数时异步执行的。

```javascript
var p = new Promise(function(resolve, reject){
  resolve("success");
});

p.then(function(value){
  console.log(value);
});

console.log("which one is called first ?");
```

控制台输出：

```css
"which one is called first ?"
"success"
```

Promise接收的函数参数是同步执行的，但`then`方法中的回调函数执行则是异步的，因此，"success"会在后面输出。

#### Promise状态的不可逆性

这是因为我们在构造函数中resolve和reject只会执行其中一个，而且后面也不会再执行了。

```javascript
var p1 = new Promise(function(resolve, reject){
  resolve("success1");
  resolve("success2");
});

var p2 = new Promise(function(resolve, reject){
  resolve("success");
  reject("reject");
});

p1.then(function(value){
  console.log(value);
});

p2.then(function(value){
  console.log(value);
});

```

控制台输出：

```css
"success1"
"success"
```

Promise状态的一旦变成resolved或rejected时，Promise的状态和值就固定下来了，不论你后续再怎么调用resolve或reject方法，都不能改变它的状态和值。

#### Promise可以链式调用

这时因为then方法返回的是一个新的Promise对象（实例），每个Promise实例是有then方法的。

```javascript
var p = new Promise(function(resolve, reject){
  resolve(1);
});
p.then(function(value){               //第一个then
  console.log(value);
  return value*2;
}).then(function(value){              //第二个then
  console.log(value);
}).then(function(value){              //第三个then
  console.log(value);
  return Promise.resolve('resolve'); 
}).then(function(value){              //第四个then
  console.log(value);
  return Promise.reject('reject');
}).then(function(value){              //第五个then
  console.log('resolve: '+ value);
}, function(err){
  console.log('reject: ' + err);
})
```

控制台输出：

```css
1
2
undefined
"resolve"
"reject: reject"
```

#### Promise中异常的非阻塞性

```javascript
var p1 = new Promise( function(resolve,reject){
  foo.bar();
  resolve( 1 );	  
});

p1.then(
  function(value){
    console.log('p1 then value: ' + value);
  },
  function(err){
    console.log('p1 then err: ' + err);
  }
).then(
  function(value){
    console.log('p1 then then value: '+value);
  },
  function(err){
    console.log('p1 then then err: ' + err);
  }
);

var p2 = new Promise(function(resolve,reject){
  resolve( 2 );	
});

p2.then(
  function(value){
    console.log('p2 then value: ' + value);
    foo.bar();
  }, 
  function(err){
    console.log('p2 then err: ' + err);
  }
).then(
  function(value){
    console.log('p2 then then value: ' + value);
  },
  function(err){
    console.log('p2 then then err: ' + err);
    return 1;
  }
).then(
  function(value){
    console.log('p2 then then then value: ' + value);
  },
  function(err){
    console.log('p2 then then then err: ' + err);
  }
);
```

控制台输出：

```javascript
p1 then err: ReferenceError: foo is not defined
p2 then value: 2
p1 then then value: undefined
p2 then then err: ReferenceError: foo is not defined
p2 then then then value: 1
```

Promise中的异常由`then`参数中第二个回调函数（Promise执行失败的回调）处理，异常信息将作为Promise的值。异常一旦得到处理，`then`返回的后续Promise对象将恢复正常，并会被Promise执行成功的回调函数处理。另外，需要注意p1、p2 多级`then`的回调函数是交替执行的 ，这正是由Promise `then`回调的异步性决定的。

#### Promise.race()异步操作的非中断性

也就是说Promise.race()中最快的异步操作完成后，其他没有执行完毕的操作仍然会继续执行，而不是停止。从源码中看出，Promise.race()会遍历promise数组中的每个任务并执行，而不是哪个任务完成了就中断遍历操作了。



## Promise的使用场景

#### 执行单个的异步任务

**利用Promise封装一个异步操作，然后利用then来获取这个异步操作的结果数据，并进行相关的处理。表现为整个then链中只有一个第一个构造函数返回一个Promise，后面then函数每次返回的都是第一个Promise返回的值**

下面的创建图片要素并加载的操作其实是一个异步操作，异步操作的结果数据时img这个要素，在then里面就可以处理或者获取这个img的一些信息了，比如img.width等等

```javascript
<script src="https://cdn.bootcss.com/bluebird/3.5.1/bluebird.min.js"></script>//如果低版本浏览器不支持Promise，通过cdn这种方式
      <script type="text/javascript">
        function loadImg(src) {
            var promise = new Promise(function (resolve, reject) {
                var img = document.createElement('img')
                img.onload = function () {
                    resolve(img)
                }
                img.onerror = function () {
                    reject('图片加载失败')
                }
                img.src = src
            })
            return promise
        }
        var src = 'https://www.imooc.com/static/img/index/logo_new.png'
        var result = loadImg(src)
        result.then(function (img) {
            console.log(1, img.width)
            return img
        }, function () {
            console.log('error 1')
        }).then(function (img) {
            console.log(2, img.height)
        })
     </script>
```

#### 串行执行多个异步任务，每个异步任务有先后关系

**有若干个异步任务，需要先做任务1，如果成功后再做任务2，任何任务失败则不再继续并执行错误处理函数。表现为每个then函数都会执行一个异步的Promise操作**

下面我们想实现第一个图片加载完成后，再加载第二个图片，如果其中有一个执行失败，就执行错误函数

```javascript
        var src1 = 'https://www.imooc.com/static/img/index/logo_new.png'
        var result1 = loadImg(src1) //result1是Promise对象
        var src2 = 'https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg'
        var result2 = loadImg(src2) //result2是Promise对象
        result1.then(function (img1) {
            console.log('第一个图片加载完成', img1.width)
            return result2  // 链式操作
        }).then(function (img2) {
            console.log('第二个图片加载完成', img2.width)
        }).catch(function (ex) {
            console.log(ex)
        })
```

#### 并行执行多个异步任务,最慢（所有）的异步操作执行完毕后执行回调

**Promise.all接受一个promise对象的数组，待全部完成之后，统一执行success**，比如说一个页面需要等两个或多个ajax的数据回来以后才正常显示，在此之前只显示loading图标

```javascript
     var src1 = 'https://www.imooc.com/static/img/index/logo_new.png'
     var result1 = loadImg(src1)
     var src2 = 'https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg'
     var result2 = loadImg(src2)
     Promise.all([result1, result2]).then(function (datas) {
         console.log('all', datas[0])//<img src="https://www.imooc.com/static/img/index/logo_new.png">
         console.log('all', datas[1])//<img src="https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg">
     })
```

#### 并行执行多个异步任务，最快的异步操作执行完毕后执行回调

**Promise.race接受一个包含多个promise对象的数组，只要有一个完成，就执行success**，有些时候，多个异步任务是为了容错。比如，同时向两个URL读取用户的个人信息，只需要获得先返回的结果即可。这种情况下，用Promise.race()实现

```javascript
     var src1 = 'https://www.imooc.com/static/img/index/logo_new.png'
     var result1 = loadImg(src1)
     var src2 = 'https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg'
     var result2 = loadImg(src2)
     Promise.race([result1, result2]).then(function (data) {
         console.log('race', data)//<img src="https://img1.mukewang.com/545862fe00017c2602200220-100-100.jpg">
     })
```

#### 给某个异步请求设置超时时间，并且在超时后执行相应的操作

面代码 requestImg 函数异步请求一张图片，timeout 函数是一个延时 5 秒的异步操作。我们将它们一起放在 race 中赛跑。如果 5 秒内图片请求成功那么便进入 then 方法，执行正常的流程。如果 5 秒钟图片还未成功返回，那么则进入 catch，报“图片请求超时”的信息。**注意：其他没有执行完毕的操作仍然会继续执行，而不是停止。**

```javascript
//请求某个图片资源
function requestImg(){
    var p = new Promise(function(resolve, reject){
	    var img = new Image();
	    img.onload = function(){
	       resolve(img);
	    }
	    img.src = "http://moxiaofei.com/wp-content/uploads/2018/04/javascript.png";
    });
    return p;
}
 
//延时函数，用于给请求计时
function timeout(){
    var p = new Promise(function(resolve, reject){
        setTimeout(function(){
            reject('图片请求超时');
        }, 5000);
    });
    return p;
}
 
Promise
.race([requestImg(), timeout()])
.then(function(results){
    console.log(results);
})
.catch(function(reason){
    console.log(reason);
});
```

## Promise的源码实现

源码实现可以参照这篇文章（[BAT前端经典面试问题：史上最最最详细的手写Promise教程](https://juejin.im/post/5b2f02cd5188252b937548ab#heading-6)），主要要注意的是then函数的实现，以及all与race方法实现上的区别，下面直接贴出最后的源码

```javascript
class Promise{
  constructor(executor){
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallbacks.forEach(fn=>fn());
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn=>fn());
      }
    };
    try{
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled,onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
    let promise2 = new Promise((resolve, reject) => {
      if (this.state === 'fulfilled') {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'rejected') {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0)
        });
      };
    });
    return promise2;
  }
  catch(fn){
    return this.then(null,fn);
  }
}
function resolvePromise(promise2, x, resolve, reject){
  if(x === promise2){
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  let called;
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      let then = x.then;
      if (typeof then === 'function') { 
        then.call(x, y => {
          if(called)return;
          called = true;
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          if(called)return;
          called = true;
          reject(err);
        })
      } else {
        resolve(x);
      }
    } catch (e) {
      if(called)return;
      called = true;
      reject(e); 
    }
  } else {
    resolve(x);
  }
}
//resolve方法
Promise.resolve = function(val){
  return new Promise((resolve,reject)=>{
    resolve(val)
  });
}
//reject方法
Promise.reject = function(val){
  return new Promise((resolve,reject)=>{
    reject(val)
  });
}
//race方法 
Promise.race = function(promises){
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(resolve,reject)
    };
  })
}
//all方法(获取所有的promise，都执行then，把结果放到数组，一起返回)
Promise.all = function(promises){
  let arr = [];
  let i = 0;
  function processData(index,data){
    arr[index] = data;
    i++;
    if(i == promises.length){
      resolve(arr);
    };
  };
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(data=>{
        processData(i,data);
      },reject);
    };
  });
}
```

## Promise的相关面试题

1、了解 Promise 吗？

2、Promise 解决的痛点是什么？

3、Promise 解决的痛点还有其他方法可以解决吗？如果有，请列举。

4、Promise 如何使用？

5、Promise 常用的方法有哪些？它们的作用是什么？

6、Promise 在事件循环中的执行过程是怎样的？

7、Promise 的业界实现都有哪些？

8、能不能手写一个 Promise 的 polyfill？





参考文章：

1. [八段代码彻底掌握 Promise](https://juejin.im/post/597724c26fb9a06bb75260e8#heading-4)
2. [BAT前端经典面试问题：史上最最最详细的手写Promise教程](https://juejin.im/post/5b2f02cd5188252b937548ab#heading-6)
3. [异步解决方案----Promise与Await](https://juejin.im/post/5b1962616fb9a01e7c2783a8)
4. [Promise 必知必会（十道题）](https://juejin.im/post/5a04066351882517c416715d)