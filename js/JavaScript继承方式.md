

### 原型链继承

```JavaScript
// 声明父类
function Animal() {
  this.name = 'animal';
  this.type = ['pig', 'cat'];
}

// 为父类添加共有方法
Animal.prototype.greet = function(sound) {
  console.log(sound);
}

// 声明子类
function Dog() {
  this.name = 'dog';
}

// 继承父类
Dog.prototype = new Animal();

var dog = new Dog();
dog.greet('汪汪');  //  "汪汪"
console.log(dog.type); // ["pig", "cat"]
```

在上面的代码中，我们创建了两个类Animal和Dog，而且给`Animal.prototype`原型上添加了一个greet共有方法，然后通过`new`命令实例化一个Animal，并且赋值给`Dog.prototype`原型。

原理说明：在实例化一个类时，新创建的对象复制了父类的构造函数内的属性与方法并且将原型`__proto__`指向了父类的原型对象，这样就拥有了父类的原型对象上的属性与方法。

不过，通过类式继承方式，有两个缺点。

第一个是引用缺陷：

父类的所有属性被共享，只要一个实例修改了属性，其他所有的子类实例都会被影响

```javascript
dog.type.push('dog');
var dog2 = new Dog();console.log(dog2.type);  // ["dog", "cat", "dog"]
```

通过上面的执行结果，我们看到当通过dog实例对象修改继承自Animal中的数组type(引用类型)时，另外一个新创建的实例dog2也会受到影响。

第二个是无法向父类构造函数传参，我们无法为不同的实例初始化继承来的属性，我们可以修改一下上面的例子：

```javascript
function Animal(color) {
  this.color = color;
}
...
Dog.prototype = new Animal('白色');
...
console.log(dog.color); // "白色"
console.log(do2.color); // "白色"
```

通过上面的代码可以看到，我们无法为不同dog赋值不同的颜色，所有dog只能同一种颜色。

### 构造函数继承

造函数继承方式可以避免类式继承的缺陷：可以为分类传参，避免了共享属性

```JavaScript
// 声明父类
function Animal(color) {
  this.name = 'animal';
  this.type = ['pig','cat'];
  this.color = color;
}

// 添加共有方法
Animal.prototype.greet = function(sound) {
  console.log(sound);
}

// 声明子类
function Dog(color) {
  Animal.apply(this, arguments);
}

var dog = new Dog('白色');
var dog2 = new Dog('黑色');

dog.type.push('dog');
console.log(dog.color);  // "白色"
console.log(dog.type);  // ["pig", "cat", "dog"]
console.log(dog2.type);  // ["pig", "cat"]
console.log(dog2.color);  // "黑色"
```

首先要知道`apply`方法的运用，它是可以更改函数的作用域，所以在上面的例子中，我们在Dog子类中调用这个方法也就是将Dog子类的变量在父类中执行一遍，这样子类就拥有了父类中的共有属性和方法。

但是，构造函数继承也是有缺陷的，那就是我们无法获取到父类的共有方法，也就是通过原型`prototype`绑定的方法，因为只是子类的实例，不是父类的实例：

```javascript
dog.greet();  // Uncaught TypeError: dog.greet is not a function
```

另外方法都在构造函数中定义，每次创建实例都会创建一遍方法

### 组合式继承

组合继承其实就是将类式继承和构造函数继承组合在一起：

```
// 声明父类   
function Animal(color) {    
  this.name = 'animal';    
  this.type = ['pig','cat'];    
  this.color = color;   
}     

// 添加共有方法  
Animal.prototype.greet = function(sound) {    
  console.log(sound);   
}     

// 声明子类   
function Dog(color) { 
  // 构造函数继承    
  Animal.apply(this, arguments);   
}   
// 类式继承
Dog.prototype = new Animal();   

var dog = new Dog('白色');   
var dog2 = new Dog('黑色');     

dog.type.push('dog');   
console.log(dog.color); // "白色"
console.log(dog.type);  // ["pig", "cat", "dog"]
console.log(dog2.type); // ["pig", "cat"]
console.log(dog2.color);  // "黑色"
dog.greet('汪汪');  // "汪汪"
```

在上面的例子中，我们在子类构造函数中执行父类构造函数，在子类原型上实例化父类，这就是组合继承了，可以看到它综合了类式继承和构造函数继承的优点，同时去除了缺陷。

可能你会奇怪为什么组合式继承可以去除类式继承中的引用缺陷？其实这是由于`原型链`来决定的，由于JavaScript引擎在访问对象的属性时，会先在对象本身中查找，如果没有找到，才会去原型链中查找，如果找到，则返回值，如果整个原型链中都没有找到这个属性，则返回undefined。

也就是说，我们访问到的引用类型(比如上面的type)其实是通过`apply`复制到子类中的，所以不会发生共享。

这种组合继承也是有点小缺陷的，那就是它调用了两次父类的构造函数。

### 原型式继承

```javascript
function create(o) {
  function F() {}
  F.prototype = o
  return new F()
}

let obj = {
  gift: ['a', 'b']
}

let jon = create(obj)
let xiaoming = create(obj)

jon.gift.push('c')
xiaoming.gift // ['a', 'b', 'c']
```

缺点：共享了属性和方法

### 寄生式继承

创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象

```javascript
function createObj (o) {
  var clone = Object.create(o)
  clone.sayName = function () {
    console.log('hi')
  }
  return clone
}
```

缺点：跟借用构造函数模式一样，每次创建对象都会创建一遍方法

### 寄生组合式继承

寄生组合式继承强化的部分就是在组合继承的基础上减少一次多余的调用父类的构造函数：

```javascript
function Animal(color) {
  this.color = color;
  this.name = 'animal';
  this.type = ['pig', 'cat'];
}


Animal.prototype.greet = function(sound) {
  console.log(sound);
}

function Dog(color) {
  Animal.apply(this, arguments);
  this.name = 'dog';
}

/* 注意下面两行 */
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.getName = function() {
  console.log(this.name);
}

var dog = new Dog('白色');   
var dog2 = new Dog('黑色');     

dog.type.push('dog');   
console.log(dog.color);   // "白色"
console.log(dog.type);   // ["pig", "cat", "dog"]
console.log(dog2.type);  // ["pig", "cat"]
console.log(dog2.color);  // "黑色"
dog.greet('汪汪');  //  "汪汪"
```

在上面的例子中，我们并不像构造函数继承一样直接将父类Animal的一个实例赋值给Dog.prototype，而是使用`Object.create()`进行一次浅拷贝，将父类原型上的方法拷贝后赋给Dog.prototype，这样子类上就能拥有了父类的共有方法，而且少了一次调用父类的构造函数。

Object.create()的浅拷贝的作用类式下面的函数：

```javascript
function create(obj) {
  function F() {};
  F.prototype = obj;
  return new F();
}
```

这里还需注意一点，由于对Animal的原型进行了拷贝后赋给Dog.prototype，因此Dog.prototype上的`constructor`属性也被重写了，所以我们要修复这一个问题：

```javascript
Dog.prototype.constructor = Dog;
```

### extends继承

Class和`extends`是在ES6中新增的，`Class`用来创建一个类，`extends`用来实现继承：

```javascript
class Animal {   
  constructor(color) {   
   this.color = color;   
  }   

  greet(sound) {   
    console.log(sound);   
  }  
}   

class Dog extends Animal {   
  constructor(color) {   
    super(color);   
    this.color = color;   
  }  
}   


let dog = new Dog('黑色');  
dog.greet('汪汪');  // "汪汪"
console.log(dog.color); // "黑色"
```

在上面的代码中，创建了父类Animal，然后Dog子类继承父类，两个类中都有一个constructor构造方法，实质就是构造函数Animal和Dog。

不知道你有没有注意到一点，我在子类的构造方法中调用了super方法，它表示父类的构造函数，用来新建父类的this对象。

注意：子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。

