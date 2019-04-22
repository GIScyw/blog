理解生成器Generator首先要了解遍历器Iterator，因为Generator函数返回的是一个遍历器对象，它只是封装了一下从而简化了操作，相比较Iterator，它可以控制程序的执行。

### 基本用法

```javascript
function* gen1() {
    yield 1;
    yield 'hello';
    return true;
}
let g1 = gen1();
g1.next();  // Object {value: 1, done: false}
g1.next();  // Object {value: "hello", done: false}
g1.next();  // Object {value: true, done: true}
g1.next();  // Object {value: undefined, done: true}
```

上面的代码就定义了一个Generator函数，Generator函数的定义跟普通函数差不多，只是在function关键字后面加了一个星号。调用Generator函数后和普通函数不同的是，该函数并不立即执行，也不返回函数执行结果，而是返回一个指向内部状态的generator对象，也可以看作是一个遍历器对象。然后必须调用该对象的next方法，让函数继续走下去，是指针移向下一个状态。每当碰到yield语句,内部指针就停下来，直到下一次调用next()才开始执行。
上面代码调用了四次next方法，遍历才结束。next方法会返回一个有两个属性的对象，value属性的值为当前yield语句的值，done属性的值表示遍历是否结束，即最后一次调用next方法时，再也碰不到yield或者return语句了。

> function关键字和函数名之间的星号写在哪都可以，只要在两者之间即可，但是一般都采取我上面代码的那种写法

### Generator函数本质

Generator函数的理解有多种：

1. Generator函数可以被理解成一个状态机，里面封装了多种状态，有兴趣的同学可以去了解一下状态机，操作系统的书里都会讲到。
2. Generator函数还可以被理解成一个遍历器对象生成器，它返回的遍历器对象可以依次遍历Generator函数内部的每一个状态。这就是为什么之前说Generator函数不仅是为了解决回调函数嵌套问题。Generator函数是生成一个对象，但是调用的时候前面**不能加new命令**。

### yield语句

yield语句是Generator函数内部可以暂停执行程序的语句，yield语句后面的值可以是各种数据类型，字符串，整数，布尔值等等都可以。这里主要想说说Generator函数中yield语句和return语句的区别。

```javascript
function* gen2() {
    return true;
    yield 1;
    yield 'hello';
}
let g2 = gen2();
g2.next();  // Object {value: true, done: true}
g2.next();  // Object {value: undefined, done: true}
```

从上面例子可以看出，当碰到return语句时，返回对象的done属性值就为true，遍历结束，不管后面是否还有yield或者return语句。**这种区别本质上是因为yield语句具备位置记忆功能而return语句则没有该功能。**

### next()方法

next()是可以带参数的，不仅next方法可以传参数，Generator函数也是可以传参数的

```javascript
function* gen4(a) {
    let b = yield (a + 1);
    return b * 2;
}
let g4 = gen4(1);
g4.next();  //  Object {value: 2, done: false}
g4.next();  //  Object {value: NaN, done: true}
let g5 = gen4(1);
g5.next();  //  Object {value: 2, done: false}
g5.next(3);  //  Object {value: 6, done: true}
```

上面例子中，Generator函数需要接收一个参数a，表面上变量b是用yield语句赋值了，但是遗憾的是这个赋值好像并没有成功，当第二次调用next方法（没有传参数）时，返回的对象value值居然为NaN，而不是我们想的 2 *（1+1）= 4。但是如果第二次调用next方法时，传入一个参数3，返回对象的value值就为6。这可以说明两点：

1. yield语句没有返回值，或者总是返回undefined；
2. next方法如果带上一个参数，这个参数就是作为上一个yield语句的返回值。

> 因为next方法表示上一个yield语句的返回值，所以必须有上一个yield语句的存在，那么第一次调用next方法时就不能传参数。第一个next只是用来启动Generator函数内部的遍历器，传参也没有多大意义

### Generator中的this

大家都知道普通函数都会有一个this对象，那么Generator的this对象怎么用呢？还是例子更直观：

```javascript
function* gen6() {
    this.a = 1;
}
let g7 = gen6();
g7.a;  //  undefined
```

上面代码中，Generator函数在this对象上添加了一个属性a，g7实例并不能取到这个属性。那么怎么让Generator函数返回一个可以正常使用this对象的实例呢？阮一峰老师提供了一种方法，首先，生成一个空对象，使用call方法绑定Generator函数内部的this。这样，构造函数调用以后，这个空对象就是Generator函数的实例对象了。参考代码在这：<http://es6.ruanyifeng.com/#docs/generator>



参考文章：

1. [ES6学习笔记之Generator函数](https://hieeyh.github.io/2016/12/06/ES6-Generator/)