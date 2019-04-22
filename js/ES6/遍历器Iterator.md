

- 具有iterator接口的都有Symbol.iterator属性，`for...of`循环本质上就是调用这个接口产生的遍历器

- 具有Iterator接口的都可以使用for of，for of解决的是每次手动调用next()的问题

### 什么是iterator

遍历器（iterator）是这样一种机制，它是一种接口，它为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署iterator接口，就可以完成遍历操作。

遍历器有next方法，该方法返回一个包含value和done两个属性的对象（下称result）。前者是迭代的值，后者是表示迭代是否完成的标志（true表示迭代完成，false表示没有完成）。iterator内部有指向迭代位置的指针，每次调用next，自动移动指针并返回相应的result。

### Iterator接口的实现

一种数据结构只要部署了Iterator接口，也就是有next方法，返回的是value和done属性，我们就称这种数据结构是“可遍历的”（iterable).ES6规定，默认的Iterator接口部署在数据结构的Symbol.iterator属性，或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是“可遍历的”。Symbol.iterator属性本身是一个函数，就是当前数据结构默认的遍历器生成函数。执行这个函数，就会返回一个遍历器。至于属性名Symbol.iterator，它是一个表达式，返回Symbol对象的iterator属性，这是一个预定义好的、类型为 Symbol 的特殊值，所以要放在方括号内。

```javascript
const obj = {
  [Symbol.iterator] : function () {  
    //Symbol.iterator是指Symbol对象的iterator属性，所以使用其作为obj的属性名需要使用[]抱起来，
    return {
      next: function () {
        return {
          value: 1,
          done: true
        };
      }
    };
  }
};
```

上面代码中，对象obj是可遍历的（iterable），因为具有Symbol.iterator属性。执行这个属性，会返回一个遍历器对象。该对象的根本特征就是具有next方法。每次调用next方法，都会返回一个代表当前成员的信息对象，具有value和done两个属性。

**或者**

```javascript
function createIterator(items) {
  var i = 0;
  return {
    next: function () {
      var done = (i >= items.length);
      var value = !done ? items[i++] : undefined; return {
        done: done,
        value: value
      };
    }
  };
}

var iterator = createIterator([1, 2, 3]);
console.log(iterator.next()); // "{ value: 1, done: false }"
console.log(iterator.next());  // "{ value: 2, done: false }"
console.log(iterator.next());  // "{ value: 3, done: false }"
console.log(iterator.next());  // "{ value: undefined, done: true }"
// 后续的所有调用返回的结果都一样
console.log(iterator.next());  // "{ value: undefined, done: true }"
```



### 具有Iterator接口的数据结构

**原生具有Iterator接口的数据结构**：

- Array
- Map
- Set
- String
- TypedArray
- 函数的arguments对象
- NodeList对象

#### 集合的iterator

ES6里的集合有三类：

- 数组
- set
- map

它们有下面的iterator：

- entries：返回迭代结果为键值对的iterator
- values：返回迭代结果为集合里的值的iterator
- keys：返回迭代结果为集合里的keys的iterator

下面以map为例：

```javascript
let tracking = new Map([
  ['name', 'jeyvie'],
  ['pro', 'fd'],
  ['hobby', 'programming']
]);

for (let entry of tracking.entries()) {
  console.log(entry);
}
// ["name", "jeyvie"]
// ["pro", "fd"]
// ["hobby", "programming"]

for (let key of tracking.keys()) {
  console.log(key);
}
// name
// pro
// hobby

for (let value of tracking.values()) {
  console.log(value);
}
// jeyvie
// fd
// programming
```

其中 `set` 里的 `key` 和 `value` 相等，都是 `set` 里的值。数组的 `key` 是其下标 `index`, `value`是里面的值。

另外，各集合都有默认的 `iterator` 供 **for-of** 调用。 其执行结果，反应的是集合如何被初始化的。

```javascript
for (let value of tracking) {
  console.log(value);
}
// ["name", "jeyvie"]
// ["pro", "fd"]
// ["hobby", "programming"]
```

对应这 `Map` 实例化是出入的子项 -- 有两个元素的数组。其他的`set`、 数组与之类似，不赘述了。

**注意Object中没有内置迭代器，因此`JavaScript` 不支持用 `for of` 来遍历对象，但是提供了一个 `for in` 方法来遍历所有非 `Symbol` 类型并且是可枚举的属性**。标准不支持，但是我们可以利用Generator函来实现：

```javascript
Object.prototype[Symbol.iterator] = function*() {
  for (const [key, value] of Object.entries(this)) {
    yield { key, value };
  }
};


for (const { key, value } of { a: 1, b: 2, c: 3 }) {
  console.log(key, value);
}
```

#### 字符串的iterator

字符串里 `iterator`，理解一句话就可以: **是基于 code point(可以理解为基于字符) 而不是code unit 迭代的**。

```javascript
var message = "A 𠮷 B";
for (let c of message) {
  console.log(c);
}

// A
// (空)
// 𠮷 
// (空)
// B
```

```javascript
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```

#### NodeList的iterator

```javascript
for (let el of NodeList) {
	// el 就是集合里的每个元素
}
```

### 调用Iterator接口的场合

有一些场合会默认调用Iterator接口（即`Symbol.iterator`方法），除了上文会介绍的`for...of`循环，还有几个别的场合

#### 解构赋值

对数组和Set结构进行解构赋值时，会默认调用`Symbol.iterator`方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

#### 扩展运算符

扩展运算符（...）也会调用默认的iterator接口。

```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

实际上，这提供了一种简便机制，可以将任何部署了Iterator接口的数据结构，转为数组。也就是说，只要某个数据结构部署了Iterator接口，就可以对它使用扩展运算符，将其转为数组。

```javascript
let arr = [...iterable];
```

#### yield

yield*后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。

```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

#### 其他场合

- Array.from()
- Map(), Set(), WeakMap(), WeakSet()（比如`new Map([['a',1],['b',2]])`）
- Promise.all()
- Promise.race()

### Generator与Iterator

Generator可以实例化出一个iterator，并且里面的yield语句可以用来中断代码的执行，也就是说，配合next方法，每次只会执行一个yield语句。Generator还有一个有意思的特性，yield后面可以跟上另一个Generator并且他们会按照次序执行

```javascript
function* gen() {
  yield 1;
  yield* gen2();
  return;
}

function* gen2() {
  yield 4;
  yield 5;
}

let iterator = gen();
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

![img](https://user-gold-cdn.xitu.io/2019/1/17/1685b1a5e631b7fe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**参考文章：**

1. [面试：你知道为什么会有 Generator 吗](https://juejin.im/post/5adae8246fb9a07aa541e150#heading-2)
2. [[前端漫谈_1] 从 for of 聊到 Generator](https://juejin.im/post/5c40484bf265da61171cfb4d#heading-4)





