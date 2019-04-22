数组的方法有数组原型方法，也有从object对象继承来的方法，这里我们只介绍数组的原型方法，数组原型方法主要有以下这些：

### 创建方法

创建一个数组可以用字面量、构造器，Array.of()、Array.from

```javascript
 // 字面量方式:
    // 这个方法也是我们最常用的，在初始化数组的时候 相当方便
    var a = [3, 11, 8];  // [3,11,8];
    // 构造器:
    // 实际上 new Array === Array,加不加new 一点影响都没有。
    var a = Array(); // [] 
    var a = Array(3); // [,,] 
    var a = Array(3,11,8); // [ 3,11,8 ]
```

#### ES6 Array.of() 返回由所有参数值组成的数组

定义：返回由所有参数值组成的数组，如果没有参数，就返回一个空数组。

目的：Array.of() 出现的目的是为了解决上述构造器因参数个数不同，导致的行为有差异的问题

```javascript
    let a = Array.of(3, 11, 8); // [3,11,8]
    let a = Array.of(3); // [3]
```

#### ES6 Arrary.from() 将两类对象转为真正的数组

定义：用于将两类对象转为真正的数组（不改变原对象，返回新的数组）。

参数：

第一个参数(必需):要转化为真正数组的对象。

第二个参数(可选): 类似数组的map方法，对每个元素进行处理，将处理后的值放入返回的数组。

第三个参数(可选): 用来绑定this。

```javascript
    // 1. 对象拥有length属性
    let obj = {0: 'a', 1: 'b', 2:'c', length: 3};
    let arr = Array.from(obj); // ['a','b','c'];
    // 2. 部署了 Iterator接口的数据结构 比如:字符串、Set、NodeList对象
    let arr = Array.from('hello'); // ['h','e','l','l','o']
    let arr = Array.from(new Set(['a','b'])); // ['a','b']
```

### 常规方法

#### join()

标准：join(separator)。

参数：separator（接收一个参数，即分隔符）

返回：字符串。

解释：将数组以separator为分隔符，组成一个字符串；separator省略则默认逗号为分隔符。

```javascript
var arr = [1,2,3];

console.log(arr.join()); // 1,2,3

console.log(arr.join("-")); // 1-2-3

console.log(arr); // [1, 2, 3]（原数组不变）
```

应用场景：

```javascript
通过join()方法可以实现重复字符串，只需传入字符串以及重复的次数，就能返回重复后的字符串，函数如下：
		
function repeatString(str, n) {
return new Array(n + 1).join(str);
}
console.log(repeatString("abc", 3)); // abcabcabc
console.log(repeatString("Hi", 5)); // HiHiHiHiHi
```

#### push()和pop()

标准：

push(): 将任意数量的参数逐个添加到数组末尾，并返回修改后数组的长度；

pop()：数组末尾移除最后一项，减少数组的 length 值，然后返回移除的项。

参数：任意数量的参数。

返回：数组。

```javascript
var arr = ["Lily","lucy","Tom"];

var count = arr.push("Jack","Sean");

console.log(count); // 5

console.log(arr); // ["Lily", "lucy", "Tom", "Jack", "Sean"]

var item = arr.pop();

console.log(item); // Sean

console.log(arr); // ["Lily", "lucy", "Tom", "Jack"]
```

#### shift() 和 unshift()

标准：

shift()：删除原数组第一项，并返回删除元素的值；如果数组为空则返回undefined 。

unshift:将参数添加到原数组开头，并返回数组的长度 。

解释：这组方法和上面的push()和pop()方法正好对应，一个是操作数组的开头，一个是操作数组的结尾。

```javascript
var arr = ["Lily","lucy","Tom"];

var count = arr.unshift("Jack","Sean");

console.log(count); // 5

console.log(arr); //["Jack", "Sean", "Lily", "lucy", "Tom"]

var item = arr.shift();

console.log(item); // Jack

console.log(arr); // ["Sean", "Lily", "lucy", "Tom"]
```

#### sort()

标准：

sort()：按升序排列数组项——即最小的值位于最前面，最大的值排在最后面。

解释：在排序时，sort()方法会调用每个数组项的 toString()转型方法，然后比较得到的字符串，以确定如何排序。即使数组中的每一项都是数值， sort()方法比较的也是字符串，因此会出现以下的这种情况：

```javascript
var arr1 = ["a", "d", "c", "b"];

console.log(arr1.sort()); // ["a", "b", "c", "d"]

arr2 = [13, 24, 51, 3];

console.log(arr2.sort()); // [13, 24, 3, 51]

console.log(arr2); // [13, 24, 3, 51](元数组被改变)
```

为了解决上述问题，sort()方法可以接收一个比较函数作为参数，以便我们指定哪个值位于哪个值的前面。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回 0，如果第一个参数应该位于第二个之后则返回一个正数。以下就是一个简单的比较函数：

```javascript
function compare(value1, value2) {

if (value1 < value2) {

return -1;

} else if (value1 > value2) {

return 1;

} else {

return 0;

}

}
arr2 = [13, 24, 51, 3];

console.log(arr2.sort(compare)); // [3, 13, 24, 51]
```

如果需要通过比较函数产生降序排序的结果，只要交换比较函数返回的值即可：

```javascript
function compare(value1, value2) {

if (value1 < value2) {

return 1;

} else if (value1 > value2) {

return -1;

} else {

return 0;

}

}

arr2 = [13, 24, 51, 3];

console.log(arr2.sort(compare)); // [51, 24, 13, 3]
```

**sort排序常见用法**：

1. 数组元素为数字的升序、降序:

   ```javascript
    var array =  [10, 1, 3, 4,20,4,25,8];
    // 升序 a-b < 0   a将排到b的前面，按照a的大小来排序的 
    // 比如被减数a是10，减数是20  10-20 < 0   被减数a(10)在减数b(20)前面   
    array.sort(function(a,b){
      return a-b;
    });
    console.log(array); // [1,3,4,4,8,10,20,25];
    // 降序 被减数和减数调换了  20-10>0 被减数b(20)在减数a(10)的前面
    array.sort(function(a,b){
      return b-a;
    });
    console.log(array); // [25,20,10,8,4,4,3,1];
   复制代码
   ```

2. 数组多条件排序

   ```javascript
    var array = [{id:10,age:2},{id:5,age:4},{id:6,age:10},{id:9,age:6},{id:2,age:8},{id:10,age:9}];
        array.sort(function(a,b){
            if(a.id === b.id){// 如果id的值相等，按照age的值降序
                return b.age - a.age
            }else{ // 如果id的值不相等，按照id的值升序
                return a.id - b.id
            }
        })
     // [{"id":2,"age":8},{"id":5,"age":4},{"id":6,"age":10},{"id":9,"age":6},{"id":10,"age":9},{"id":10,"age":2}] 
   复制代码
   ```

3. 自定义比较函数，天空才是你的极限

类似的：**运用好返回值，我们可以写出任意符合自己需求的比较函数**

```javascript
    var array = [{name:'Koro1'},{name:'Koro1'},{name:'OB'},{name:'Koro1'},{name:'OB'},{name:'OB'}];
    array.sort(function(a,b){
        if(a.name === 'Koro1'){// 如果name是'Koro1' 返回-1 ，-1<0 a排在b的前面
            return -1
        }else{ // 如果不是的话，a排在b的后面
          return 1
        }
    })
    // [{"name":"Koro1"},{"name":"Koro1"},{"name":"Koro1"},{"name":"OB"},{"name":"OB"},{"name":"OB"
```



#### reverse()

reverse()：反转数组项的顺序。

```javascript
var arr = [13, 24, 51, 3];
console.log(arr.reverse()); //[3, 51, 24, 13]
console.log(arr); //[3, 51, 24, 13](原数组改变)
```

#### concat()

concat() ：将参数添加到原数组中。这个方法会先创建当前数组一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。在没有给 concat()方法传递参数的情况下，它只是复制当前数组并返回副本。

```javascript
var arr = [1,3,5,7];

var arrCopy = arr.concat(9,[11,13]);

console.log(arrCopy); //[1, 3, 5, 7, 9, 11, 13]

console.log(arr); // [1, 3, 5, 7](原数组未被修改)
```

从上面测试结果可以发现：传入的不是数组，则直接把参数添加到数组后面，如果传入的是数组，则将数组中的各个项添加到数组中。但是如果传入的是一个二维数组呢？

```JavaScript
var arrCopy2 = arr.concat([9,[11,13]]);

console.log(arrCopy2); //[1, 3, 5, 7, 9, Array[2]]

console.log(arrCopy2[5]); //[11, 13]
```

上述代码中，arrCopy2数组的第五项是一个包含两项的数组，也就是说concat方法只能将传入数组中的每一项添加到数组中，如果传入数组中有些项是数组，那么也会把这一数组项当作一项添加到arrCopy2中。

#### slice()

slice()：返回从原数组中指定开始下标到结束下标之间的项组成的新数组。slice()方法可以接受一或两个参数，即要返回项的起始和结束位置。在只有一个参数的情况下， slice()方法返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项——但不包括结束位置的项。

```javascript
var arr = [1,3,5,7,9,11];

var arrCopy = arr.slice(1);

var arrCopy2 = arr.slice(1,4);

var arrCopy3 = arr.slice(1,-2);

var arrCopy4 = arr.slice(-4,-1);

console.log(arr); //[1, 3, 5, 7, 9, 11](原数组没变)

console.log(arrCopy); //[3, 5, 7, 9, 11]

console.log(arrCopy2); //[3, 5, 7]

console.log(arrCopy3); //[3, 5, 7]

console.log(arrCopy4); //[5, 7, 9]
```

arrCopy只设置了一个参数，也就是起始下标为1，所以返回的数组为下标1（包括下标1）开始到数组最后。 arrCopy2设置了两个参数，返回起始下标（包括1）开始到终止下标（不包括4）的子数组。 arrCopy3设置了两个参数，**终止下标为负数，当出现负数时，将负数加上数组长度的值（6）来替换该位置的数**，因此就是从1开始到4（不包括）的子数组。 arrCopy4中两个参数都是负数，所以都加上数组长度6转换成正数，因此相当于slice(2,5)。

#### splice()

解释： splice()：很强大的数组方法，它有很多种用法，可以实现删除、插入和替换。 删除：可以删除任意数量的项，只需指定 2 个参数：要删除的第一项的位置和要删除的项数。例如， splice(0,2)会删除数组中的前两项。 插入：可以向指定位置插入任意数量的项，只需提供 3 个参数：起始位置、 0（要删除的项数）和要插入的项。例如，splice(2,0,4,6)会从当前数组的位置 2 开始插入4和6。 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定 3 个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice (2,1,4,6)会删除当前数组位置 2 的项，然后再从位置 2 开始插入4和6。 splice()方法始终都会返回一个数组，该数组中包含从原始数组中删除的项，如果没有删除任何项，则返回一个空数组。

```javascript
var arr = [1,3,5,7,9,11];

var arrRemoved = arr.splice(0,2);

console.log(arr); //[5, 7, 9, 11]

console.log(arrRemoved); //[1, 3]

var arrRemoved2 = arr.splice(2,0,4,6);

console.log(arr); // [5, 7, 4, 6, 9, 11]

console.log(arrRemoved2); // []

var arrRemoved3 = arr.splice(1,1,2,4);

console.log(arr); // [5, 2, 4, 4, 6, 9, 11]

console.log(arrRemoved3); //[7]
```

#### indexOf()和 lastIndexOf()

解释： indexOf()：接收两个参数：要查找的项和（可选的）表示查找起点位置的索引。其中， 从数组的开头（位置 0）开始向后查找。 lastIndexOf：接收两个参数：要查找的项和（可选的）表示查找起点位置的索引。其中， 从数组的末尾开始向前查找。 这两个方法都返回要查找的项在数组中的位置，或者在没找到的情况下返回1。在比较第一个参数与数组中的每一项时，会使用全等操作符。

```javascript
var arr = [1,3,5,7,7,5,3,1];

console.log(arr.indexOf(5)); //2

console.log(arr.lastIndexOf(5)); //5

console.log(arr.indexOf(5,2)); //2

console.log(arr.lastIndexOf(5,4)); //2

console.log(arr.indexOf("5")); //-1
```

### 遍历方法

#### 引言

首先需要知道对于数组和可迭代对象的遍历方法，我们需要从不同的维度进行对比，方法的功能性，方法的应用场景，方法的兼容性，方法的效率，方法的返回值以及是否改变原始数组。深层次的我们可以思考如何实现这些方法，并且考虑到低版本浏览器的兼容性。如果要分group的话，可以这么分：`forEach()与map()`，`every()与some()`，`filter()与find()findIndex()`,`keys()、values()与entries()`,`reduce()与reduceRight()`。当然对于的常规的遍历方法，比如`for`，`while`与`do-while`,`$.each`,`$(selector).each`这里就不赘述了，原生的方法当然是主角，但也要看应用场景，项目中使用新增的遍历方法确实能提高代码效率，jquery的方法当然也有借鉴的地方，但是在现在的前端项目中用到的较少。
![](https://user-gold-cdn.xitu.io/2019/3/30/169cd09f2e6e07c5?w=1833&h=1969&f=png&s=361878)

#### forEach ()

指定数组的每一项元素都执行了一次传入的函数，返回值为undefined，thisArg可选，用来当做fn函数内的this对象；forEach不能在低版本IE（6~8）中使用，兼容写法请参考 [Polyfill](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2FforEach%23%E5%85%BC%E5%AE%B9%E6%97%A7%E7%8E%AF%E5%A2%83%EF%BC%88Polyfill%EF%BC%89)。得益于鸭式辨型，虽然forEach不能直接遍历对象，但它可以通过call方式遍历类数组对象

```javascript
var o = {0:1, 1:3, 2:5, length:3};
Array.prototype.forEach.call(o,function(value, index, obj){
  console.log(value,index,obj);
  obj[index] = value * value;
},o);
// 1 0 Object {0: 1, 1: 3, 2: 5, length: 3}
// 3 1 Object {0: 1, 1: 3, 2: 5, length: 3}
// 5 2 Object {0: 1, 1: 9, 2: 5, length: 3}
console.log(o); // Object {0: 1, 1: 9, 2: 25, length: 3}
```

可以说这么多遍历函数中，forEach()要注意的地方最多，有以下要注意的：

- this都是指向调用方法的数组；

  ```javascript
  var array = [1, 3, 5];
  var obj = {name:'cc'};
  var sReturn = array.forEach(function(value, index, array){
    array[index] = value * value;
    console.log(this.name); // cc被打印了三次
  },obj);
  console.log(array); // [1, 9, 25], 可见原数组改变了
  console.log(sReturn); // undefined, 可见返回值为undefined
  ```

- 没有返回值,它总是返回undefined值，即使你return了一个值

  ```javascript
  var solt = arr1.forEach((v,i,t) => {
    console.log(v)
  })
  
  console.log(solt)	// undefined
  ```

- 不能 中途退出循环，不能用break，会报错的，只能用return退出本次回调，进行下一次回调

  ```javascript
  var arr1 = [1, 2, 3, 4, 5]
  
  // 使用break会报错
  arr1.forEach((v,i,arr) => {
    console.log(v)
    if(v === 3) {
      break
    }
  })
  //SyntaxError: Illegal break statement
  
  
  
  // return false 也无效
  arr1.forEach((v,i,arr) => {
    console.log(v)
    if(v === 3) {
      return false
      console.log('-----')//不会执行
    }
  })
  /*
  1
  2
  3
  4
  5
  */
  ```

- 使用箭头函数，`thisArg` 参数会被忽略

  ```javascript
  var arr1 = [1, 2, 3]
  var arr2 = [7, 8, 9]
  
  arr1.forEach((v, i, arr) => {
    console.log(this)
  })
  // window
  // window
  // window
  
  arr1.forEach((v, i, arr) => {
    console.log(this)
  }, arr2)
  // window
  // window
  // window
  ```

#### map()

map() 方法遍历数组，使用传入函数处理每个元素，并返回函数的返回值组成的新数组。其在低版本IE（6~8）的兼容写法请参考 [Polyfill](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Fmap%23Compatibility)。

#### every()与some()

every() 方法使用传入的函数测试所有元素，只要其中有一个函数返回值为 false，那么该方法的结果为 false；如果全部返回 true，那么该方法的结果才为 true;every 一样不能在低版本IE(6~8)中使用，兼容写法请参考 [Polyfill](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Fevery%23Compatibility)。

以下是鸭式辨型的写法：

```javascript
var o = {0:10, 1:8, 2:25, length:3};
var bool = Array.prototype.every.call(o,function(value, index, obj){
  return value >= 8;
},o);
console.log(bool); // true
```

some()方法 测试数组元素时，只要有一个函数返回值为 true，则该方法返回 true，若全部返回 false，则该方法返回 false；some 的鸭式辨型写法可以参照every，同样它也不能在低版本IE（6~8）中使用，兼容写法请参考 [Polyfill](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Fsome%23Compatibility)。

#### filter()

filter() 方法使用传入的函数测试所有元素，并返回所有通过测试的元素组成的新数组。它就好比一个过滤器，筛掉不符合条件的元素。filter一样支持鸭式辨型，具体请参考every方法鸭式辨型写法。其在低版本IE（6~8）的兼容写法请参考 [Polyfill](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FArray%2Ffilter%23Compatibility)。

```javascript
var array = [18, 9, 10, 35, 80];
var array2 = array.filter(function(value, index, array){
  return value > 20;
});
console.log(array2); // [35, 80]
```

#### for-in 语句

一般会使用`for-in`来遍历对象的属性的,不过属性需要 **enumerable**,才能被读取到. `for-in` 循环只遍历可枚举属性。一般常用来遍历对象，包括非整数类型的名称和继承的那些原型链上面的属性也能被遍历。像 Array和 Object使用内置构造函数所创建的对象都会继承自Object.prototype和String.prototype的不可枚举属性就不能遍历了

```javascript
var obj = {
    name: 'test',
    color: 'red',
    day: 'sunday',
    number: 5
}
for (var key in obj) {
    console.log(obj[key])
}
```

#### for-of 语句

`for-of`语句在可迭代对象（包括 Array，Map，Set，String，TypedArray，arguments 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句。只要是一个iterable的对象,就可以通过`for-of`来迭代.

```javascript
var arr = [{name:'bb'},5,'test']
for (item of arr) {
    console.log(item)
}
```

#### find()与findIndex()

find() 方法基于**ECMAScript 2015（ES6）规范**，返回数组中第一个满足条件的元素（如果有的话）， 如果没有，则返回undefined。

findIndex() 方法也基于**ECMAScript 2015（ES6）规范**，它返回数组中第一个满足条件的元素的索引（如果有的话）否则返回-1。

```javascript
var array = [1, 3, 5, 7, 8, 9, 10];
function f(value, index, array){
  return value%2==0; // 返回偶数
}
function f2(value, index, array){
  return value > 20; // 返回大于20的数
}
console.log(array.find(f)); // 8
console.log(array.find(f2)); // undefined
console.log(array.findIndex(f)); // 4
console.log(array.findIndex(f2)); // -1
```

#### includes()

判断一个数组是否包含一个指定的值，如果包含则返回 true，否则返回false。

```javascript
var array1 = [1, 2, 3];

console.log(array1.includes(2));
// expected output: true

var pets = ['cat', 'dog', 'bat'];

console.log(pets.includes('cat'));
// expected output: true

console.log(pets.includes('at'));
// expected output: false
```

#### reduce()与reduceRight()

reduce() 方法接收一个方法作为累加器，数组中的每个值(从左至右) 开始合并，最终为一个值;initialValue 指定第一次调用 fn 的第一个参数。;当 fn 第一次执行时：

- 如果 initialValue 在调用 reduce 时被提供，那么第一个 previousValue 将等于 initialValue，此时 item 等于数组中的第一个值；
- 如果 initialValue 未被提供，那么 previousVaule 等于数组中的第一个值，item 等于数组中的第二个值。此时如果数组为空，那么将抛出 TypeError。
- 如果数组仅有一个元素，并且没有提供 initialValue，或提供了 initialValue 但数组为空，那么fn不会被执行，数组的唯一值将被返回。

```javascript
var array = [1, 2, 3, 4];
var s = array.reduce(function(previousValue, value, index, array){
  return previousValue * value;
},1);
console.log(s); // 24
// ES6写法更加简洁
array.reduce((p, v) => p * v); // 24
```

reduceRight() 方法接收一个方法作为累加器，数组中的每个值（从右至左）开始合并，最终为一个值。除了与reduce执行方向相反外，其他完全与其一致.

#### entries()

entries() 方法基于**ECMAScript 2015（ES6）规范**，返回一个数组迭代器对象，该对象包含数组中每个索引的键值对。

```
var array = ["a", "b", "c"];
var iterator = array.entries();
console.log(iterator.next().value); // [0, "a"]
console.log(iterator.next().value); // [1, "b"]
console.log(iterator.next().value); // [2, "c"]
console.log(iterator.next().value); // undefined, 迭代器处于数组末尾时, 再迭代就会返回undefined
```

entries 也受益于鸭式辨型，如下：

```javascript
var o = {0:"a", 1:"b", 2:"c", length:3};
var iterator = Array.prototype.entries.call(o);
console.log(iterator.next().value); // [0, "a"]
console.log(iterator.next().value); // [1, "b"]
console.log(iterator.next().value); // [2, "c"]
```

#### keys()

keys() 方法基于**ECMAScript 2015（ES6）规范**，返回一个数组索引的迭代器。（浏览器实际实现可能会有调整）

```javascript
var array = ["abc", "xyz"];
var iterator = array.keys();
console.log(iterator.next()); // Object {value: 0, done: false}
console.log(iterator.next()); // Object {value: 1, done: false}
console.log(iterator.next()); // Object {value: undefined, done: false}
```

要注意的是索引迭代器会包含那些没有对应元素的索引

```javascript
var array = ["abc", , "xyz"];
var sparseKeys = Object.keys(array);
var denseKeys = [...array.keys()];
console.log(sparseKeys); // ["0", "2"]
console.log(denseKeys);  // [0, 1, 2]
```

#### values()

values() 方法基于**ECMAScript 2015（ES6）规范**，返回一个数组迭代器对象，该对象包含数组中每个索引的值。其用法基本与上述 entries 方法一致。兼容性方面，目前没有浏览器实现了该方法

```javascript
var array = ["abc", "xyz"];
var iterator = array.values();
console.log(iterator.next().value);//abc
console.log(iterator.next().value);//xyz
```

#### Symbol.iterator()

该方法基于**ECMAScript 2015（ES6）规范**，同 values 方法功能相同。

```javascript
var array = ["abc", "xyz"];
var iterator = array[Symbol.iterator]();
console.log(iterator.next().value); // abc
console.log(iterator.next().value); // xyz      
```

#### 总结

**数组遍历的方法中**

- 返回值有的是数组，有的是一个值，有的是布尔值，有的是一个数组迭代器对象；

- every，some是用来判断的，返回一个布尔值；filter，find，findIndex都是用来筛选的，并且返回满足条件的元素或元素组成的数组，不同的是，filter返回所有满足条件的数组成的数组，find与findIndex返回第一个满足条件的元素或元素的索引；

- Array.prototype的所有方法均具有鸭式辨型这种神奇的特性，它们不仅可以用来处理数组对象，还可以处理类数组对象；

- ES6中的新增方法应用对象更广泛，可以用于所有的具有Iterator接口的可迭代对象，比如数组，类数组对象，Map和Set结构；

- 对于各种遍历方法的效率问题，这篇文章（[详解JS遍历](http://louiszhai.github.io/2015/12/18/traverse/#%E6%B5%8B%E8%AF%95%E5%90%84%E6%96%B9%E6%B3%95%E6%95%88%E7%8E%87)）对JS中遍历方法进行了测试比较，得出的结论就是JavaScript原生的方法的效率高于各种封装的方法；

- forEach()与map()的不同点：

  - `map()`创建了新数组，不改变原数组；`forEach()`可以改变原数组。
  - 遇到空缺的时候`map()`虽然会跳过，但保留空缺；`forEach()`遍历时跳过空缺，不保留空缺。
  - `map()`按照原始数组元素顺序依次处理元素；`forEach()`遍历数组的每个元素，将元素传给回调函数。

  **用`forEach()`为数组中的每个元素添加属性**

  ```javascript
    var arr = [
        {name : 'kiwi', age : 12},
        {name : 'sasa', age : 22},
        {name : 'alice', age : 32},
        {name : 'joe', age : 42}
    ]
    arr.forEach(function(ele, index){
        if(index > 2){
            ele.sex = 'boy';
        }else{
            ele.sex = 'girl';
        }
        return arr1
    })
    console.log(arr)//元素组发生改变
    //[{name: "kiwi", age: 12, sex: "girl"},{name: "sasa", age: 22, sex: "girl"},{name: "alice", age: 32, sex: "gi
  ```

   ** 遇到空缺比较**

  ```javascript
    ['a', , 'b'].forEach(function(ele,index){
        console.log(ele + '. ' + index);
    })
    //0.a
    //2.b
    ['a', , 'b'].map(function(ele,index){
        console.log(ele + '. ' + index);
    })
    //['0.a', , '2.b']
  ```

- for-of与for-in的不同点

  - `for...of`循环不会循环对象的key，只会循环出数组的value，因此`for...of`不能循环遍历**普通对象**,对普通对象的属性遍历推荐使用`for...in`。如果实在想用`for...of`来遍历普通对象的属性的话，可以通过和`Object.keys()`搭配使用，先获取对象的所有key的数组然后遍历：

  ```javascript
  var student={
      name:'wujunchuan',
      age:22,
      locate:{
      country:'china',
      city:'xiamen',
      school:'XMUT'
      }
  }
  for(var key of Object.keys(student)){
      //使用Object.keys()方法获取对象key的数组
      console.log(key+": "+student[key]);
  }
  /*
  //如果是下面这样，会报错提示 “TypeError: student is not iterable”
  for(var key of student){
      //使用Object.keys()方法获取对象key的数组
      console.log(key+": "+student[key]);
  }
  */
  ```

  - `for...in`循环出的是key，`for...of`循环出的是value;推荐在循环对象属性的时候，使用`for...in`,在遍历数组的时候的时候使用`for...of`。注意，`for...of`是ES6新引入的特性。修复了ES5引入的`for...in`的不足

**参考：**

1. [【深度长文】JavaScript数组所有API全解密](https://juejin.im/post/5902d56e1b69e60058c634d6#heading-44)
2. [【干货】js 数组详细操作方法及解析合集](https://juejin.im/post/5b0903b26fb9a07a9d70c7e0#heading-36)
3. [ 详解JS遍历](http://louiszhai.github.io/2015/12/18/traverse/#%E6%B5%8B%E8%AF%95%E5%90%84%E6%96%B9%E6%B3%95%E6%95%88%E7%8E%87)

