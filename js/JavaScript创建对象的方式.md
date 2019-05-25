### 引言

创建自定义对象最简单的方式就是创建一个Object的实例，然后再为它添加属性和方法，早期的开发人员经常使用这种模式来创建对象，后来对象字面量的方法成了创建对象的首选模式。虽然object构造函数或者对象字面量的方法都可以用来创建对象，但是这些方法使用同一个接口创建很多对象，会产生大量的重复代码。为了解决这个问题，人们开始使用各种模式来创建对象，在这些模式中，一般推荐使用四种方式，包括构造函数模式、原型模式、构造函数和原型组合模式，动态原型模式，其他的方式，包括工厂模式、寄生构造函数模式、稳妥构造函数模式不推荐使用。而这些方式中，用的最多最推荐的应是**组合模式和动态原型模式**。下面先介绍下各个模式的优缺点：

- **工厂模式**的优点就是确实是解决了创建多个相似对象的问题，但是却没有解决对象识别的问题，即不知道创建实例的是哪一个对象（构造函数），因为所有的实例都是object创造的；
- **构造函数模式**最大的优点就是可以通过contructor或者instanceof可以识别对象实例的类别（即它的构造函数），但是每次创建实例时，每个方法都要被创建一次；

- **原型模式**的优点是实例的方法是共享的，所有的实例的同一个方法都指向同一个，这样使得每次创建实例时方法不会重新创建，但是这也是缺点，原型模式中所有的属性和方法都共享，这样造成两个问题一是没有办法创建实例自己的属性和方法，二是实例改变属性里的数据时其他实例也会跟着改变，另外它也不能像创建构造函数那样可以传递参数；

- **构造函数和原型组合模式**优点就是该共享的共享，该私有的私有，即用构造函数定义对象的所有非函数属性，用原型方式定义方法和共享的属性，结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。另外，这种混成模式还支持向构造函数传递参数；这种构造函数和原型混成的模式，是目前在ECMAScript中使用最广泛、认同度最高的一种创建定义类型的方法。

  注意组合模式解决了原型模式没有办法传递参数的缺点，也解决了构造函数模式不能共享方法的缺点。

- **动态原型模式**是目前极为推荐的一种形式,但是在使用时要注意不能用对象字面量重写原型

- **寄生构造函数模式**和工厂模式基本一样，除了多了个new操作符，因此它也不能识别对象，或者说不能识别实例的类别，该方法不推荐使用

- **稳妥构造函数模式**适合在一些安全的环境中，稳妥对象是指没有公共属性，而且其方法也不引用this的对象，它也无法识别对象的所属类型，该方法不推荐使用

### 工厂模式

```javascript
function Person(name) {
  var o = new Object();
  o.name = name;
  o.say = function() {
    alert(this.name);
  }
  return o;
}
var person1 = Person("yawei");
```

**缺点：**

1. 对象无法识别，所有实例都指向一个原型；无法通过constructor识别对象，因为都是来自Object。
2. 每次通过Person创建对象的时候，所有的say方法都是一样的，但是却存储了多次，浪费资源。

### 构造函数模式

```javascript
function Person() {
  this.name = 'hanmeimei';
  this.say = function() {
    alert(this.name)
  }
}

var person1 = new Person();
```

**优点：**

1. 通过constructor或者instanceof可以识别对象实例的类别
2. 可以通过new 关键字来创建对象实例，更像OO语言中创建对象实例

**缺点：**

1. 多个实例的say方法都是实现一样的效果，但是却存储了很多次（两个对象实例的say方法是不同的，因为存放的地址不同）

**注意：**

1. 构造函数模式隐试的在最后返回`return this` 所以在缺少new的情况下，会将属性和方法添加给全局对象，浏览器端就会添加给window对象。
2. 也可以根据`return this` 的特性调用call或者apply指定this。这一点在后面的继承有很大帮助。

### 原型模式

```javascript
function Person() {}
Person.prototype.name = 'hanmeimei';
Person.prototype.say = function() {
  alert(this.name);
}
Person.prototype.friends = ['lilei'];

var person1 = new Person();
```

**优点：**

1. say方法是共享的了，所有的实例的say方法都指向同一个。

2. 可以动态的添加原型对象的方法和属性，并直接反映在对象实例上。

   ```javascript
   var person1 = new Person()
   Person.prototype.showFriends = function() {
     console.log(this.friends)
   }
   person1.showFriends()  //['lilei']
   ```

**缺点：**

1. 出现引用的情况下会出现问题具体见下面代码：

   ```javascript
   var person1 = new Person();
   var person2 = new Person();
   person1.friends.push('xiaoming');
   console.log(person2.friends)  //['lilei', 'xiaoming']
   ```

   因为js对引用类型的赋值都是将地址存储在变量中，所以person1和person2的friends属性指向的是同一块存储区域。

2. 第一次调用say方法或者name属性的时候会搜索两次，第一次是在实例上寻找say方法，没有找到就去原型对象(Person.prototype)上找say方法，找到后就会在实力上添加这些方法or属性。

3. 所有的方法都是共享的，没有办法创建实例自己的属性和方法，也没有办法像构造函数那样传递参数。

**注意：**

1. 优点②中存在一个问题就是直接通过对象字面量给`Person.prototype`进行赋值的时候会导致`constructor`改变，所以需要手动设置，其次就是通过对象字面量给`Person.prototype`进行赋值，会无法作用在之前创建的对象实例上

   ```javascript
   var person1 = new Person()
   Person.prototype = {
   	name: 'hanmeimei2',
     	setName: function(name){
         this.name = name
     	}
   }
   
   person1.setName()   //Uncaught TypeError: person1.set is not a function(…)
   ```

   这是因为对象实例和对象原型直接是通过一个指针链接的，这个指针是一个内部属性[[Prototype]]，可以通过`__proto__`访问。我们通过对象字面量修改了Person.prototype指向的地址，然而对象实例的`__proto__`，并没有跟着一起更新，所以这就导致，实例还访问着原来的`Person.prototype`，所以建议不要通过这种方式去改变`Person.prototype`属性

### 构造函数和原型组合模式

```javascript
function Person(name) {
  this.name = name
  this.friends = ['lilei']
}
Person.prototype.say = function() {
  console.log(this.name)
}

var person1 = new Person('hanmeimei')
person1.say() //hanmeimei
```

**优点：**

1. 解决了原型模式对于引用对象的缺点
2. 解决了原型模式没有办法传递参数的缺点
3. 解决了构造函数模式不能共享方法的缺点

**缺点：**

1. 和原型模式中注意①一样

### 动态原型模式

```javascript
function Person(name) {
  this.name = name
    // 检测say 是不是一个函数
    // 实际上只在当前第一次时候没有创建的时候在原型上添加sayName方法
    //因为构造函数执行时，里面的代码都会执行一遍，而原型有一个就行，不用每次都重复，所以仅在第一执行时生成一个原型，后面执行就不必在生成，所以就不会执行if包裹的函数，
//其次为什么不能再使用字面量的写法，我们都知道，使用构造函数其实是把new出来的对象作用域绑定在构造函数上，而字面量的写法，会重新生成一个新对象，就切断了两者的联系！
  if(typeof this.say != 'function') {
    Person.prototype.say = function(
    alert(this.name)
  }
}
```

**优点：**

1. 可以在初次调用构造函数的时候就完成原型对象的修改
2. 修改能体现在所有的实例中

**缺点：**红宝书都说这个方案完美了。。。。

**注意：**

1. 只用检查一个在执行后应该存在的方法或者属性就行了
   假设除了:say方法外，你还定义了很多其他方法，比如:sayBye、cry、smile等等。此时你只需要把它们都放到对sayName判断的if块里面就可以了。

   ```javascript
   if (typeof this.sayName != "function") {
       Person.prototype.sayName = function() {...};
       Person.prototype.sayBye = function() {...};
       Person.prototype.cry = function() {...};
       ...
   }
   ```

   

2. 不能用对象字面量修改原型对象

### 寄生构造函数模式

```javascript
function Person(name) {
  var o = new Object()
  o.name = name
  o.say = function() {
    alert(this.name)
  }
  return o
}

var peron1 = new Person('hanmeimei')
```

**优点：**

1. 和工厂模式基本一样，除了多了个new操作符

**缺点：**

1. 和工厂模式一样，不能区分实例的类别

### 稳妥构造模式

```javascript
function Person(name) {
  var o = new Object()
  o.say = function() {
    alert(name)
  }
}

var person1 = new Person('hanmeimei');
person1.name  // undefined
person1.say() //hanmeimei
```

**优点：**

1. 安全，那么好像成为了私有变量，只能通过say方法去访问。所谓稳妥对象，指的是没有公共睡醒，而且其方法也不引用this的对象。

**缺点：**

1. 跟工厂模式一样，不能区分实例的类别



参考文章：

[JavaScript中的对象，如何创建对象，创建对象的7种模式](https://blog.csdn.net/u014346301/article/details/52204967)

[一个人就需要对象之js中八种创建对象方式](https://juejin.im/post/5cb93841f265da03612ee3df#heading-9)

[读书笔记之创建对象](http://alvinyuxt.github.io/2016/11/11/%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0%E4%B9%8B%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1/)