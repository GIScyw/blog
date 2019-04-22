## 引言

这篇文章主要介绍事件流的三个阶段，事件处理程序的三种方式的不同（`DOM0`，`DOM2`，`IE`），考虑到`IE`中的事件处理和事件对象的差异如何做兼容性处理，事件对象中的属性如何运用到实际应用中以及它们之间的差异，以及事件捕获与冒泡的先后顺序问题。   

文章开头先简短介绍下本文的几个重要知识点：

- `DOM`事件处理程序有三种方式，`DOM0`的`onType`，`IE9`以下的`attachEvent`与`detachEvent`，`DOM2`的`addEventListener`与`removeEventListener`。
- `DOM2`级的优点是可以通过`addEventListener`的第三个参数来指定是捕获还是冒泡，并且可以为同一个`DOM`元素注册多个同类型的事件处理程序；而`DOM0`对每个事件只支持一个事件处理程序
- `DOM0`和`DOM2`的事件处理程序都会自动传入`event`对象;IE中的`event`对象取决于指定的事件处理程序的方法,所以在IE中会有`window.event`、`event`两种情况；`event`对象里有一些很有用 处的属性，比如target、`currentTarget`，`preventDefault`，`stopPropagation`，`stopImmediatePropagation`等
- 对于`DOM0`的 `ontype`，给元素的事件行为绑定方法都是在当前元素事件行为的冒泡阶段(或者目标阶段)执行的。对于`DOM2`的`addEventListener`，为了最大限度的兼容，大多是情况下都是将事件处理程序添加到事件冒泡阶段。不是特别需要，不建议在事件捕获阶段注册事件处理程序。
- 事件处理函数的兼容性处理要考虑到`DOM0`和`IE9`以下的事件处理方式，事件对象与事件对象属性的兼容性处理要考虑到`IE`中的不同
- `event.stopPropagation()` 方法阻止事件冒泡到父元素，阻止任何父事件处理程序被执行（一般我们认为`stopPropagation`是用来阻止事件冒泡的，其实该函数也可以阻止捕获事件）
- `event.target`指向引起触发事件的元素，而`event.currentTarget`则是事件绑定的元素，只有被点击的那个目标元素的`event.target`才会等于`event.currentTarget`

> 多数支持DOM事件流的浏览器都实现了一种特定的行为；即使“DOM2级事件”规范明确要求捕获阶段不会涉及事件目标，但IE9、Safari、Chrome、Firefox和Opera9.5及更高版本都会在捕获阶段触发事件对象上的事件。结果，就是有两个机会在目标对象上操作事件。

## 事件流

事件流描述的是从页面中接受事件的顺序。但有意思的是，`IE`和`Netscape`开发团队居然提出了两个截然相反的事件流概念。IE的事件流是事件冒泡流，标准的浏览器事件流是事件捕获流。不过，`W3C`为了制定标准，采取了折中的方式：先捕获再冒泡（通过`addEventListene`r给出的第三个参数同时支持冒泡与捕获）。具体地，同一个`DOM`元素可以注册多个同类型的事件，通过`addEventListener`来注册事件，`removeEventListener`来解除事件。

> 注意要想注册过的事件能够被解除，必须将回调函数保存起来，否则无法解除。

`DOM `事件流分为三个阶段：`捕获阶段`、`目标阶段`、`冒泡阶段`。先调用捕获阶段的处理函数，其次调用目标阶段的处理函数，最后调用冒泡阶段的处理函数。（下面的图中没有标html标签）

![img](https://user-gold-cdn.xitu.io/2019/2/24/1691f3e556cd038b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

（1）捕获阶段：事件从`window`对象自上而下向目标节点传播的阶段；

（2）目标阶段：真正的目标节点正在处理事件的阶段；

（3）冒泡阶段：事件从目标节点自下而上向`window`对象传播的阶段。

捕获是从上到下，事件先从`window`对象，然后再到`document`（对象），然后是`html`标签（通过`document.documentElement`获取`html`标签），然后是`body`标签（通过`document.body`获取`body`标签），然后按照普通的`html`结构一层一层往下传，最后到达目标元素。

而事件冒泡的流程刚好是事件捕获的逆过程。 接下来我们看个事件冒泡的例子：

```javascript
// 例3
<div id="outer">
    <div id="inner"></div>
</div>
......
window.onclick = function() {
    console.log('window');
};
document.onclick = function() {
    console.log('document');
};
document.documentElement.onclick = function() {
    console.log('html');
};
document.body.onclick = function() {
    console.log('body');
}
outer.onclick = function(ev) {
    console.log('outer');
};
inner.onclick = function(ev) {
    console.log('inner');
};
```



![img](https://user-gold-cdn.xitu.io/2018/12/1/16767dfc84974d41?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

正如我们下面提到的`onclick`给元素的事件行为绑定方法都是在当前元素事件行为的冒泡阶段(或者目标阶段)执行的。

## DOM事件级别

`DOM`级别一共可以分为四个级别：`DOM0级`、`DOM1级`、`DOM2级`和`DOM3级`。而**`DOM`事件分为3个级别：`DOM 0`级事件处理，`DOM 2`级事件处理和`DOM 3`级事件处理**。由于`DOM 1`级中没有事件的相关内容，所以没有`DOM 1`级事件。又因为`IE`和其他浏览器在`DOM2`级别上事件处理又不一样，因此一般可以将事件处理方式分为三类，即`DOM0`，`DOM2`，`IE`。下面是从`DOM`级别上来划分

#### DOM 0级事件

**el.onclick=function(){}**

```javascript
var btn = document.getElementById('btn');
 btn.onclick = function(){
     alert(this.innerHTML);
 }
```

**当希望为同一个元素/标签绑定多个同类型事件的时候（如给上面的这个btn元素绑定3个点击事件），是不被允许的**。DOM0事件绑定，给元素的事件行为绑定方法，**这些方法都是在当前元素事件行为的冒泡阶段(或者目标阶段)执行的**。

#### DOM 2级事件

**el.addEventListener(event-name, callback, useCapture)**

- event-name: 事件名称，可以是标准的DOM事件
- callback: 回调函数，当事件触发时，函数会被注入一个参数为当前的事件对象 event
- **useCapture: 默认是false，代表事件句柄在冒泡阶段执行（或者说注册的是冒泡事件）,true表示事件句柄在捕获阶段执行 （或者说注册的是捕获事件）**

```javascript
var btn = document.getElementById('btn');
btn.addEventListener("click", test, false);
function test(e){
	e = e || window.event;
    alert((e.target || e.srcElement).innerHTML);
    btn.removeEventListener("click", test)
}
//IE9-:attachEvent()与detachEvent()。
//IE9+/chrom/FF:addEventListener()和removeEventListener()
```

IE9以下的IE浏览器不支持 addEventListener()和removeEventListener()，使用 attachEvent()与detachEvent() 代替，因为IE9以下是不支持事件捕获的，所以也没有第三个参数，第一个事件名称前要加on。可以对此做个兼容性处理：

#### DOM 3级事件

在DOM 2级事件的基础上添加了更多的事件类型。

- UI事件，当用户与页面上的元素交互时触发，如：load、scroll

- 焦点事件，当元素获得或失去焦点时触发，如：blur、focus

- 鼠标事件，当用户通过鼠标在页面执行操作时触发如：dblclick、mouseup

- 滚轮事件，当使用鼠标滚轮或类似设备时触发，如：mousewheel

- 文本事件，当在文档中输入文本时触发，如：textInput

- 键盘事件，当用户通过键盘在页面上执行操作时触发，如：keydown、keypress

- 合成事件，当为IME（输入法编辑器）输入字符时触发，如：compositionstart

- 变动事件，当底层DOM结构发生变化时触发，如：DOMsubtreeModified

- 同时DOM3级事件也允许使用者自定义一些事件。


**总结：**

- DOM2级的好处是可以添加多个事件处理程序；DOM0对每个事件只支持一个事件处理程序；

- 通过DOM2添加的匿名函数无法移除，`addEventListener`和`removeEventListener`的`handler`必须同名

- 作用域：DOM0的`handler`会在所属元素的作用域内运行，IE的`handler`会在全局作用域运行，`this === window`

- 触发顺序：添加多个事件时，DOM2会按照添加顺序执行，IE会以相反的顺序执行，请谨记



#### 跨浏览器的事件处理程序

兼容`ie9`以下的浏览器和`DOM0`

```javascript
var EventUtil = {
  // element是当前元素，可以通过getElementById(id)获取
  // type 是事件类型，一般是click ,也有可能是鼠标、焦点、滚轮事件等等
  // handle 事件处理函数
  addHandler: (element, type, handler) => {
    // 先检测是否存在DOM2级方法,再检测IE的方法，最后是DOM0级方法（一般不会到这）
    if (element.addEventListener) {
      // 第三个参数false表示冒泡阶段
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
      element.attachEvent(`on${type}`, handler)
    } else {
      element[`on${type}`] = handler;
    }
  },

  removeHandler: (element, type, handler) => {
    if (element.removeEventListener) {
      // 第三个参数false表示冒泡阶段
      element.removeEventListener(type, handler, false);
    } else if (element.detachEvent) {
      element.detachEvent(`on${type}`, handler)
    } else {
      element[`on${type}`] = null;
    }
  }
}

// 获取元素
var btn = document.getElementById('btn');
// 定义handler
var handler = function(e) {
  console.log('我被点击了');
}
// 监听事件
EventUtil.addHandler(btn, 'click', handler);
// 移除事件监听
// EventUtil.removeHandler(button1, 'click', clickEvent);
```

## 事件代理

由于事件会在冒泡阶段向上传播到父节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的代理（delegation）,也叫事件委托。事件代理有以下两个优点：

- 减少内存消耗，提高性能

假设有一个列表，列表之中有大量的列表项，我们需要在点击每个列表项的时候响应一个事件

```javascript
// 例4
<ul id="list">
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
  ......
  <li>item n</li>
</ul>
```

如果给每个列表项一一都绑定一个函数，那对于内存消耗是非常大的，效率上需要消耗很多性能。借助事件代理，我们只需要给父容器ul绑定方法即可，这样不管点击的是哪一个后代元素，都会根据冒泡传播的传递机制，把容器的click行为触发，然后把对应的方法执行，根据事件源，我们可以知道点击的是谁，从而完成不同的事。

- 动态绑定事件

在很多时候，我们需要通过用户操作动态的增删列表项元素，如果一开始给每个子元素绑定事件，那么在列表发生变化时，就需要重新给新增的元素绑定事件，给即将删去的元素解绑事件，如果用事件代理就会省去很多这样麻烦。

接下来我们来实现上例中父层元素 #list 下的 li 元素的事件委托到它的父层元素上：

```javascript
// 给父层元素绑定事件
document.getElementById('list').addEventListener('click', function (e) {
  // 兼容性处理
  var event = e || window.event;
  var target = event.target || event.srcElement;
  // 判断是否匹配目标元素
  if (target.nodeName.toLocaleLowerCase === 'li') {
    console.log('the content is: ', target.innerHTML);
  }
});
```

## 事件对象

`DOM0`和`DOM2`的事件处理程序都会自动传入`event`对象,即触发`DOM`上的某个事件时，会产生一个事件对象，里面包含着所有和事件有关的信息。`IE`中的`event`对象取决于指定的事件处理程序的方法

> IE的`handler`会在全局作用域运行，`this === window`
> 所以在IE中会有`window.event`、`event`两种情况

另外在`IE`中，事件对象的属性也不一样，对应关系如下：

`srcElement` => `target`
`returnValue` => `preventDefault()`
`cancelBubble` => `stopPropagation()`
`IE` 不支持事件捕获，因而只能取消事件冒泡，但`stopPropagation`可以同时取消事件捕获和冒泡

> 只有在事件处理程序期间，`event`对象才会存在，一旦事件处理程序执行完成，`event`对象就会被销毁

#### 1. event. preventDefault()

**如果调用这个方法，默认事件行为将不再触发**。什么是默认事件呢？例如表单一点击提交按钮(submit)跳转页面、a标签默认页面跳转或是锚点定位等。

很多时候我们使用a标签仅仅是想当做一个普通的按钮，点击实现一个功能，不想页面跳转，也不想锚点定位。

```javascript
//方法一：
<a href="javascript:;">链接</a>
```

也可以通过JS方法来阻止，给其click事件绑定方法，当我们点击A标签的时候，先触发click事件，其次才会执行自己的默认行为

```javascript
//方法二:
<a id="test" href="http://www.cnblogs.com">链接</a>
<script>
test.onclick = function(e){
    e = e || window.event;
    return false;
}
</script>

//方法三：
<a id="test" href="http://www.cnblogs.com">链接</a>
<script>
test.onclick = function(e){
    e = e || window.event;
    e.preventDefault();
}
</script>
```

接下来我们看个例子：输入框最多只能输入六个字符，如何实现？

```javascript
// 例5
 <input type="text" id='tempInp'>
 <script>
    tempInp.onkeydown = function(ev) {
        ev = ev || window.event;
        let val = this.value.trim() //trim去除字符串首位空格（不兼容）
        // this.value=this.value.replace(/^ +| +$/g,'') 兼容写法
        let len = val.length
        if (len >= 6) {
            this.value = val.substr(0, 6);
            //阻止默认行为去除特殊按键（DELETE\BACK-SPACE\方向键...）
            let code = ev.which || ev.keyCode;
            if (!/^(46|8|37|38|39|40)$/.test(code)) {
                ev.preventDefault()
            }
        }
    }
 </script>
```

#### 2. event.stopPropagation() & event.stopImmediatePropagation()

**`event.stopPropagation()` 方法阻止事件冒泡到父元素，阻止任何父事件处理程序被执行（一般我们认为`stopPropagation`是用来阻止事件冒泡的，其实该函数也可以阻止捕获事件）**。上面提到事件冒泡阶段是指事件从目标节点自下而上向`window`对象传播的阶段。 我们在上面例子中的`inner`元素`click`事件上，添加	`event.stopPropagation()`这句话后，就阻止了父事件的执行，最后只打印了`'inner'`。

```javascript
 inner.onclick = function(ev) {
    console.log('inner');
    ev.stopPropagation();
};
```

**`stopImmediatePropagation` 既能阻止事件向父元素冒泡，也能阻止元素同事件类型的其它监听器被触发。而 `stopPropagation` 只能实现前者的效果**。我们来看个例子：

```javascript
<body>
  <button id="btn">click me to stop propagation</button>
</body>
......
const btn = document.querySelector('#btn');
btn.addEventListener('click', event => {
  console.log('btn click 1');
  event.stopImmediatePropagation();
});
btn.addEventListener('click', event => {
  console.log('btn click 2');
});
document.body.addEventListener('click', () => {
  console.log('body click');
});
// btn click 1
```

如上所示，使用` stopImmediatePropagation`后，点击按钮时，不仅`body`绑定事件不会触发，与此同时按钮的另一个点击事件也不触发。

#### 3. event.target & event.currentTarget

老实说这两者的区别，并不好用文字描述，我们先来看个例子：

```javascript
<div id="a">
  <div id="b">
    <div id="c">
      <div id="d"></div>
    </div>
  </div>
</div>
<script>
  document.getElementById('a').addEventListener('click', function(e) {
    console.log(
      'target:' + e.target.id + '&currentTarget:' + e.currentTarget.id
    )
  })
  document.getElementById('b').addEventListener('click', function(e) {
    console.log(
      'target:' + e.target.id + '&currentTarget:' + e.currentTarget.id
    )
  })
  document.getElementById('c').addEventListener('click', function(e) {
    console.log(
      'target:' + e.target.id + '&currentTarget:' + e.currentTarget.id
    )
  })
  document.getElementById('d').addEventListener('click', function(e) {
    console.log(
      'target:' + e.target.id + '&currentTarget:' + e.currentTarget.id
    )
  })
</script>
```



![img](https://user-gold-cdn.xitu.io/2018/12/4/1677974dad275fb7?imageslim)



当我们点击最里层的元素d的时候，会依次输出:

```javascript
target:d&currentTarget:d
target:d&currentTarget:c
target:d&currentTarget:b
target:d&currentTarget:a
```

从输出中我们可以看到，`event.target`指向引起触发事件的元素，而`event.currentTarget`则是事件绑定的元素，只有被点击的那个目标元素的`event.target`才会等于`event.currentTarget`。**也就是说，event.currentTarget始终是监听事件者，而event.target是事件的真正发出者**。



#### 4. 跨浏览器的事件对象

```javascript
var EventUtil = {
    addHandler: function (el, type, handler) {
        if (el.addEventListener) {
            el.addEventListener(type, handler, false);
        } else if (el.attachEvent) {
            el.attachEvent('on' + type, handler);
        } else {
            el['on' + type] = handler;
        }
    },
    removeHandler: function (el, type, handler) {
        if (el.removeEventListener) {
            el.removeEventListerner(type, handler, false);
        } else if (el.detachEvent) {
            el.detachEvent('on' + type, handler);
        } else {
            el['on' + type] = null;
        }
    },
    getEvent: function (e) {
        return e ? e : window.event;
    },
    getTarget: function (e) {
        return e.target ? e.target : e.srcElement;
    },
    preventDefault: function (e) {
        if (e.preventDefault) {
            e.preventDefault();
        } else {
            e.returnValue = false;
        }
    },
    stopPropagation: function (e) {
        if (e.stopPropagation) {
            e.stopPropagation();
        } else {
            e.cancelBubble = true;
        }
    }
};
```



## 捕获与冒泡的顺序问题

当有多层交互嵌套时，事件捕获和冒泡的先后顺序，似乎不是那么好理解。接下来，将分 5 种情况讨论它们的顺序，以及如何规避意外情况的发生。

**1.在外层 div 注册事件，点击内层 div 来触发事件时，捕获事件总是要比冒泡事件先触发(与代码顺序无关)**

假设，有这样的 html 结构：

```javascript
<div id="test" class="test">
   <div id="testInner" class="test-inner"></div>
</div>
```

然后，我们在外层 div 上注册两个 click 事件，分别是捕获事件和冒泡事件，代码如下：

```javascript
const btn = document.getElementById("test");
 
//捕获事件
btn.addEventListener("click", function(e){
    alert("capture is ok");
}, true);
 
//冒泡事件
btn.addEventListener("click", function(e){
    alert("bubble is ok");
}, false);
```

点击内层的 div，先弹出 capture is ok，后弹出 bubble is ok。只有当真正触发事件的 DOM 元素是内层的时候，外层 DOM 元素才有机会模拟捕获事件和冒泡事件。

**2.当在触发事件的 DOM 元素上注册事件时，哪个先注册，就先执行哪个**

html 结构同上，js 代码如下：

```javascript
const btnInner = document.getElementById("testInner");

//冒泡事件
btnInner.addEventListener("click", function(e){
    alert("bubble is ok");
}, false);
 
//捕获事件
btnInner.addEventListener("click", function(e){
    alert("capture is ok");
}, true);
```

本例中，冒泡事件先注册，所以先执行。所以，点击内层 div，先弹出 `bubble is ok`，再弹出 `capture is ok`。

**3.当外层 div 和内层 div 同时注册了捕获事件时，点击内层 div 时，外层 div 的事件一定会先触发**

```javascript
const btn = document.getElementById("test");
const btnInner = document.getElementById("testInner");

btnInner.addEventListener("click", function(e){
    alert("inner capture is ok");
}, true);

btn.addEventListener("click", function(e){
    alert("outer capture is ok");
}, true);
```

虽然外层 div 的事件注册在后面，但会先触发。所以，结果是先弹出 `outer capture is ok`，再弹出 `inner capture is ok`。

**4.同理，当外层 div 和内层 div 都同时注册了冒泡事件，点击内层 div 时，一定是内层 div 事件先触发。**

```javascript
const btn = document.getElementById("test");
const btnInner = document.getElementById("testInner");

btn.addEventListener("click", function(e){
    alert("outer bubble is ok");
}, false);

btnInner.addEventListener("click", function(e){
    alert("inner bubble is ok");
}, false);
```

先弹出 `inner bubble is ok`，再弹出 `outer bubble is ok`。

**5.阻止事件的派发**

通常情况下，我们都希望点击某个`div` 时，就只触发自己的事件回调。比如，明明点击的是内层 `div`，但是外层` div `的事件也触发了，这是就不是我们想要的了。这时，就需要阻止事件的派发。

事件触发时，会默认传入一个 `event` 对象，这个 `event` 对象上有一个方法：`stopPropagation`。MDN 上的解释是：**阻止 捕获 和 冒泡 阶段中，当前事件的进一步传播**。所以，通过此方法，让外层 `div` 接收不到事件，自然也就不会触发了。

```javascript
btnInner.addEventListener("click", function(e){
    //阻止冒泡
    e.stopPropagation();
    alert("inner bubble is ok");
}, false);
```





**参考文章**

1. [event.target和event.currentTarget的区别](https://link.juejin.im?target=http%3A%2F%2Fwww.calledt.com%2Ftarget-and-currenttarget%2F)
2. [JS事件：捕获与冒泡、事件处理程序、事件对象、跨浏览器、事件委托](https://github.com/amandakelake/blog/issues/38) 
3. [javascript事件流](https://www.cnblogs.com/xianyulaodi/p/5544312.html)
4. [「前端面试题系列7」JavaScript 中的事件机制（从原生到框架）](https://juejin.im/post/5c727abde51d457fc564cd77#heading-1)
5. [DOM事件机制](https://juejin.im/post/5bd2e5f8e51d4524640e1304)