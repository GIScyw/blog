## 引言

面向对象编程就是将你的需求抽象成一个对象，然后对这个对象进行分析，为其添加对应的特征（属性）与行为（方法），我们将这个对象称之为 **类**。 面向对象一个很重要的特点就是封装，虽然 `javascript` 这种解释性的弱类型语言没有像一些经典的强类型语言(例如`C++，JAVA`等)有专门的方式用来实现类的封装，但我们可以利用 `javascript` 语言灵活的特点，去模拟实现这些功能。而在许多大型`web`项目中国，`JavaScript`代码的数量已经非常多了，我们有必要将一些优秀的设计模式借鉴到`JavaScript`中。

![JavaScript设计模式](C:\Users\yawei.chen\Desktop\summry\设计模式\JavaScript设计模式.png)

## 装饰者模式

给对象动态地添加职责的方式称为装饰者模式。传统的面向对象语言中给对象添加功能常常使用继承的方式，但是继承的方式不灵活，而与之相比，装饰者模式更加灵活，“即用即付”。

**AOP 装饰函数**:

```javascript
Function.prototype.before = function(fn) {
  const self = this
  return function() {
    fn.apply(new(self), arguments)  // https://github.com/MuYunyun/blog/pull/30#event-1817065820
    return self.apply(new(self), arguments)
  }
}


Function.prototype.after = function(fn) {
  const self = this
  return function() {
    self.apply(new(self), arguments)
    return fn.apply(new(self), arguments)
  }
}
```

#### 场景：插件式的表单验证

```javascript

<html>
<body>
	用户名：<input id="username" type="text"/>

	密码： <input id="password" type="password"/>
	<input id="submitBtn" type="button" value="提交"></button>
</body>
<script>
	var username = document.getElementById( 'username' ),
	password = document.getElementById( 'password' ),
	submitBtn = document.getElementById( 'submitBtn' );
	var formSubmit = function(){
		if ( username.value === '' ){
			return alert ( '用户名不能为空' );
		}
		if ( password.value === '' ){
			return alert ( '密码不能为空' );
		}
		var param = {
			username: username.value,
			password: password.value
		}
		ajax( 'http:// xxx.com/login', param ); // ajax 具体实现略
	}

	submitBtn.onclick = function(){
		formSubmit();
	}


	var validata = function(){
		if ( username.value === '' ){
			alert ( '用户名不能为空' );
			return false;
		}
		if ( password.value === '' ){
			alert ( '密码不能为空' );
			return false;
		}
	}
</script>
</html>
```

上面formSubmit函数除了提交ajax请求之外，还要验证用户输入的合法，这样会造成函数臃肿，职责混乱，我们可以分离检验输入和ajax请求的代码，我们把校验输入的逻辑放在validata函数中，并且约定当validata函数返回false的时候，表示校验未通过

```javascript
Function.prototype.before = function( beforefn ){
		var __self = this;
		return function(){
			if ( beforefn.apply( this, arguments ) === false ){
	// beforefn 返回false 的情况直接return，不再执行后面的原函数
				return;
			}
			return __self.apply( this, arguments );
		}
	}


	var validata = function(){
		if ( username.value === '' ){
			alert ( '用户名不能为空' );
			return false;
		}
		if ( password.value === '' ){
			alert ( '密码不能为空' );
			return false;
		}
	}
	var formSubmit = function(){
		var param = {
			username: username.value,
			password: password.value
		}
		ajax( 'http:// xxx.com/login', param );
	}

	formSubmit = formSubmit.before( validata );

	submitBtn.onclick = function(){
		formSubmit();
	}
```

## 发布-订阅模式

发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。在JavaScript中，我们一般用事件模型来替代传统的发布-订阅模式。

#### 场景：售房信息通知

```javascript
var salesOffices = {}; // 定义售楼处
	salesOffices.clientList = []; // 缓存列表，存放订阅者的回调函数

	salesOffices.listen = function( key, fn ){
		if ( !this.clientList[ key ] ){ // 如果还没有订阅过此类消息，给该类消息创建一个缓存列表
			this.clientList[ key ] = [];
		}
		this.clientList[ key ].push( fn ); // 订阅的消息添加进消息缓存列表
	};

	salesOffices.trigger = function(){ // 发布消息
		var key = Array.prototype.shift.call( arguments ), // 取出消息类型
		fns = this.clientList[ key ]; // 取出该消息对应的回调函数集合
		if ( !fns || fns.length === 0 ){ // 如果没有订阅该消息，则返回
			return false;
		}
		for( var i = 0, fn; fn = fns[ i++ ]; ){
			fn.apply( this, arguments ); // (2) // arguments 是发布消息时附送的参数
		}
	};

	salesOffices.listen( 'squareMeter88', function( price ){ // 小明订阅88 平方米房子的消息
		console.log( '价格= ' + price ); // 输出： 2000000
	});

	salesOffices.listen( 'squareMeter110', function( price ){ // 小红订阅110 平方米房子的消息
		console.log( '价格= ' + price ); // 输出： 3000000
	});

	salesOffices.trigger( 'squareMeter88', 2000000 ); // 发布88 平方米房子的价格
	salesOffices.trigger( 'squareMeter110', 3000000 ); // 发布110 平方米房子的价格
```

## 状态模式

状态模式的关键在于区分事物内部的状态，事物内部状态的改变往往会带来事物的行为改变

- 状态模式定义了状态与行为之间的关系，并将它们封装在一个类中。
- context中的请求动作和状态类中封装的行为可以非常容易地独立变化而互不影响

#### 场景：电灯程序

电灯开关按钮被按下时，第一次按下打开弱光，第二次按下打开强光，第三次是关闭电灯。我们谈到封装，一般都会优先封装对象的行为，而不是对象的状态。但在状态模式中刚好相反，状态模式的关键是把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部，所以button被按下的时候，只需要在上下文中，把这个请求委托给当前的状态对象即可，该状态对象会负责渲染它自身的行为。

```javascript
var OffLightState = function( light ){
		this.light = light;
	};

	OffLightState.prototype.buttonWasPressed = function(){
		console.log( '弱光' ); // offLightState 对应的行为
		this.light.setState( this.light.weakLightState ); // 切换状态到weakLightState
	};
// WeakLightState：
var WeakLightState = function( light ){
	this.light = light;
};

WeakLightState.prototype.buttonWasPressed = function(){
		console.log( '强光' ); // weakLightState 对应的行为
		this.light.setState( this.light.strongLightState ); // 切换状态到strongLightState
	};
	// StrongLightState：
	var StrongLightState = function( light ){
		this.light = light;
	};

	StrongLightState.prototype.buttonWasPressed = function(){
		console.log( '关灯' ); // strongLightState 对应的行为
		this.light.setState( this.light.offLightState ); // 切换状态到offLightState
	};

//这个light称为上下文（context）
	var Light = function(){
       //在light的构造函数中，我们要创建每一个状态类的实例对象，context持有这些状态对象的引用，以便将请求委托给状态对象。用户的请求，即点击button的动作也是实现在Context中的
		this.offLightState = new OffLightState( this );
		this.weakLightState = new WeakLightState( this );
		this.strongLightState = new StrongLightState( this );
		this.button = null;
	};


	Light.prototype.init = function(){
		var button = document.createElement( 'button' ),
		self = this;
		this.button = document.body.appendChild( button );
		this.button.innerHTML = '开关';
		this.currState = this.offLightState; // 设置当前状态
		this.button.onclick = function(){
			self.currState.buttonWasPressed();
		}	
	};

	Light.prototype.setState = function( newState ){
		this.currState = newState;
	};

	var light = new Light();
	light.init();
```

**JavaScript状态机：**

```javascript
var delegate = function( client, delegation ){
		return {
			buttonWasPressed: function(){ // 将客户的操作委托给delegation 对象
				return delegation.buttonWasPressed.apply( client, arguments );
			}
		}
	};

	var FSM = {
		off: {
			buttonWasPressed: function(){
				console.log( '关灯' );
				this.button.innerHTML = '下一次按我是开灯';
				this.currState = this.onState;
			}
		},
		on: {
			buttonWasPressed: function(){
				console.log( '开灯' );
				this.button.innerHTML = '下一次按我是关灯';
				this.currState = this.offState;
			}
		}
	};

	var Light = function(){
		this.offState = delegate( this, FSM.off );
		this.onState = delegate( this, FSM.on );
		this.currState = this.offState; // 设置初始状态为关闭状态
		this.button = null;
	};

	Light.prototype.init = function(){
		var button = document.createElement( 'button' ),
		self = this;
		button.innerHTML = '已关灯';
		this.button = document.body.appendChild( button );
		this.button.onclick = function(){
			self.currState.buttonWasPressed();
		}
	};
	var light = new Light();
	light.init();
```

## 代理模式

代理有很多种类，比如**保护代理**会过滤请求，代理B可以拒绝将A发起的请求给传递给C；虚拟代理会将一些操作交给代理执行，代理B可以在A满足某些条件下再执行操作，即虚拟代理会把一些开销大的对象延迟到真正需要它的时候再去创建。保护代理用于控制不同权限的对象对目标对象的访问，但在JavaScript中并不太容易实现保护代理，因为我们无法判断谁访问了某个对象。

**代理**和**本体**的接口必须保持一致性，对于用户，他们并不清楚代理和本体的区别，在任何使用本体的地方都可以替换成使用代理。

#### 场景：图片预加载

```javascript
//只有本体
var myImage=(function(){
    var imgNode=document.createElement('img');
    document.body.appendChild(imgNode);
    
    return{
        setSrc:function(src){
            imgNode.src=src;
        }
    }
})
myImg.setSrc('./logo.lpg');

//有本体和代理的情况；引入代理对象proxyImage,通过这个代理对象，在图片被真正加载好之前，页面将出现一张占位的菊花图来提示用户图片正在被加载
var myImage=(function(){
    var imgNode=document.createElement('img');
    document.body.appendChild(imgNode);
    return{
        setSrc:function(src){
            imgNode.src=src;
        }
    }
})()

var proxyImage=(function(){
    var img=new Image;
    img.onload=function(){
        myImage.setSrc(this.src);
    }
    return{
        setSrc:function(src){
            myImage.setSrc('./loading.gif');
            img.src=src;
        }
    }
})()
proxyImage.setSrc('./logo.jpg')
```

## 策略模式

策略模式的目的就是将算法的使用与算法的实现分离开，一个策略模式的程序至少由两部分组成。第一个部分是策略类，策略类封装了具体的算法，并负责具体的计算过程。第二个部分是环境类Context，Context接受客户的请求，随后把请求委托给某一个策略类。

#### 传统面向对象语言实现

```javascript
/*strategy类*/
var performanceS=function(){};

performanceS.prototype.caculate=function(salary){
    return salary*4;
}

var performanceA=function(){};

performanceA.prototype.caculate=function(salary){
    return salary*3;
}

var performanceB=function(){};

performanceB.proptype.caculate=function(salary){
    return salary*2;
}

/*Context类*/
var Bonus=function(){
    this.salary=null;
    this.strategy=null;
}

Bonus.prototype.setSalary=function(salary){
    this.salary=salary;
}
Bonus.prototype.setStrategy=function(strategy){
    this.strategy=strategy;
}
Bonus.prototype.getBonus=function(){
    return this.strategy.caculate(this.salary);
}

var bonus=new Bonus();
bonus.setSalary(1000);
bonus.setStrategy(new performanceS());
console.log(bonus.getBonus());
```

#### Javascript中实现

在JavaScript中，Stratege我们可以把它定义为函数对象，Context负责接收用户的请求并委托给strategy对象

```JavaScript
var strategies={
    "S":function(salary){
        return salary*4;
    },
    "A":function(salary){
        return salary*3;
    },
    "B":function(salary){
        return salary*2;
    }
};

var calculateBonus=function(level,salary){
    return strategies[level](salary);
};

console.log(calculateBonus('S',2000));
```

#### 场景：表单检验

```javascript
/*Validator类的实现*/
var Validator = function(){
    this.cache = [];
}
Validator.prototype.add =function(dom,rule,errorMsg){
    var ary = rule.split(':');
    this.cache.push(function(){
        var strategy =ary.shift();
        ary.unshift(dom.value);
        ary.push(errorMsg);
        return strategies[strategy].apply(dom,ary);
    });
};

Validator.prototype.start = function(){
    for(var i=0,validatorFunc;validatorFunc = this.cache[i++]){
        var msg=validatorFunc(); 
        if(msg){
            return msg;
        }
    }
}

/*定义strategy*/
var strategies = {
    isNonEmpty:function(value,errorMsg){
        if(value=''){
            return errorMsg;
        }
    },
    minLength:function(value,length,errorMsg){
        if(value.length<length){
            return errorMsg
        }
    },
    isMobile:function(value,errorMsg){
        if(!/^1[3|5|8][0-9]$/.test(value)){
            return errorMsg;
        }
    }
}

/*定义Context*/
var validataFunc = function(){
    var validator = new Validator();
    validator.add(registerForm.userName,'isNonEmpty','用户名不能为空’);
    validator.add(registerForm.password,'inLength:6',密码长度不能少于6位'')
    validator.add(registerForm.phoneNumber,'isMobile','手机号码格式不正确’);

    var errorMsg = validator.start();
    return errorMsg;
}

var registerForm = document.getElementById("registerForm");
registerForm.onsubmit = function(){
    varerrorMsg = validataFunc(); 
    if(errorMsg){
        alert(errorMsg);
        return false;
    }
}
```

## 职责链模式

职责链模式: 类似多米诺骨牌, 通过请求第一个条件, 会持续执行后续的条件, 直到返回结果为止。

[![img](https://camo.githubusercontent.com/a1c3bdd9af66c0ac6ec7f37d5bd818f30f6ce9f0/687474703a2f2f776974682e6d7579756e79756e2e636e2f63626338633130626261653230326263643234336636623037303464653362612e6a70672d333030)](https://camo.githubusercontent.com/a1c3bdd9af66c0ac6ec7f37d5bd818f30f6ce9f0/687474703a2f2f776974682e6d7579756e79756e2e636e2f63626338633130626261653230326263643234336636623037303464653362612e6a70672d333030)

重要性: 4 星, 在项目中能对 if-else 语句进行优化

#### 场景 :手机售卖

场景: 某电商针对已付过定金的用户有优惠政策, 在正式购买后, 已经支付过 500 元定金的用户会收到 100 元的优惠券, 200 元定金的用户可以收到 50 元优惠券, 没有支付过定金的用户只能正常购买。

```javascript
// orderType: 表示订单类型, 1: 500 元定金用户；2: 200 元定金用户；3: 普通购买用户
// pay: 表示用户是否已经支付定金, true: 已支付；false: 未支付
// stock: 表示当前用于普通购买的手机库存数量, 已支付过定金的用户不受此限制

const order = function( orderType, pay, stock ) {
  if ( orderType === 1 ) {
    if ( pay === true ) {
      console.log('500 元定金预购, 得到 100 元优惠券')
    } else {
      if (stock > 0) {
        console.log('普通购买, 无优惠券')
      } else {
        console.log('库存不够, 无法购买')
      }
    }
  } else if ( orderType === 2 ) {
    if ( pay === true ) {
      console.log('200 元定金预购, 得到 50 元优惠券')
    } else {
      if (stock > 0) {
        console.log('普通购买, 无优惠券')
      } else {
        console.log('库存不够, 无法购买')
      }
    }
  } else if ( orderType === 3 ) {
    if (stock > 0) {
        console.log('普通购买, 无优惠券')
    } else {
      console.log('库存不够, 无法购买')
    }
  }
}

order( 3, true, 500 ) // 普通购买, 无优惠券
```

下面用职责链模式改造代码:

```javascript
const order500 = function(orderType, pay, stock) {
  if ( orderType === 1 && pay === true ) {
    console.log('500 元定金预购, 得到 100 元优惠券')
  } else {
    order200(orderType, pay, stock)
  }
}

const order200 = function(orderType, pay, stock) {
  if ( orderType === 2 && pay === true ) {
    console.log('200 元定金预购, 得到 50 元优惠券')
  } else {
    orderCommon(orderType, pay, stock)
  }
}

const orderCommon = function(orderType, pay, stock) {
  if (orderType === 3 && stock > 0) {
    console.log('普通购买, 无优惠券')
  } else {
    console.log('库存不够, 无法购买')
  }
}

order500( 3, true, 500 ) // 普通购买, 无优惠券
```

改造后可以发现代码相对清晰了, 但是链路代码和业务代码依然耦合在一起, 进一步优化:

```javascript
// 业务代码
const order500 = function(orderType, pay, stock) {
  if ( orderType === 1 && pay === true ) {
    console.log('500 元定金预购, 得到 100 元优惠券')
  } else {
    return 'nextSuccess'
  }
}

const order200 = function(orderType, pay, stock) {
  if ( orderType === 2 && pay === true ) {
    console.log('200 元定金预购, 得到 50 元优惠券')
  } else {
    return 'nextSuccess'
  }
}

const orderCommon = function(orderType, pay, stock) {
  if (orderType === 3 && stock > 0) {
    console.log('普通购买, 无优惠券')
  } else {
    console.log('库存不够, 无法购买')
  }
}

// 链路代码
const chain = function(fn) {
  this.fn = fn
  this.sucessor = null
}

chain.prototype.setNext = function(sucessor) {
  this.sucessor = sucessor
}

chain.prototype.init = function() {
  const result = this.fn.apply(this, arguments)
  if (result === 'nextSuccess') {
    this.sucessor.init.apply(this.sucessor, arguments)
  }
}

const order500New = new chain(order500)
const order200New = new chain(order200)
const orderCommonNew = new chain(orderCommon)

order500New.setNext(order200New)
order200New.setNext(orderCommonNew)

order500New.init( 3, true, 500 ) // 普通购买, 无优惠券
```

重构后, 链路代码和业务代码彻底地分离。假如未来需要新增 order300, 那只需新增与其相关的函数而不必改动原有业务代码。

另外结合 AOP 还能简化上述链路代码:

```javascript
// 业务代码
const order500 = function(orderType, pay, stock) {
  if ( orderType === 1 && pay === true ) {
    console.log('500 元定金预购, 得到 100 元优惠券')
  } else {
    return 'nextSuccess'
  }
}

const order200 = function(orderType, pay, stock) {
  if ( orderType === 2 && pay === true ) {
    console.log('200 元定金预购, 得到 50 元优惠券')
  } else {
    return 'nextSuccess'
  }
}

const orderCommon = function(orderType, pay, stock) {
  if (orderType === 3 && stock > 0) {
    console.log('普通购买, 无优惠券')
  } else {
    console.log('库存不够, 无法购买')
  }
}

// 链路代码
Function.prototype.after = function(fn) {
  const self = this
  return function() {
    const result = self.apply(self, arguments)
    if (result === 'nextSuccess') {
      return fn.apply(self, arguments) // 这里 return 别忘记了~
    }
  }
}

const order = order500.after(order200).after(orderCommon)

order( 3, true, 500 ) // 普通购买, 无优惠券
```

职责链模式比较重要, 项目中能用到它的地方会有很多, 用上它能解耦 1 个请求对象和 n 个目标对象的关系。





## 单例模式

单例模式的定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点

应用：线程池、全局缓存，浏览器中的window对象等

#### 传统面向对象语言中实现

```javascript
var SingleTon =function(name){
    this.name=name;
    this.instance=null;
}

SingleTon.proptype.getName=function(){
    alert(this.name);

}

SingleTon.getInstance=function(name){
    if(!this.instance){
        this.instance=new SingleTon(name);
    }
    return this.instance;
}

var a=SingleTon.getInstance('sven1');
var b=SingleTon.getInstance('sven2');
```

或者将this.intance改为局部变量的形式

```javascript
var SingleTon =function(name){
    this.name=name;
}

SingleTon.proptype.getName=function(){
    alert(this.name);

}

SingleTon.getInstance=function(name){
    var instance=null;
    return function(name) {
        if (!instance) {
            this.instance = new SingleTon(name);
        }
        return this.instance;
    }
}

var a=SingleTon.getInstance('sven1');
var b=SingleTon.getInstance('sven2');
```

更加透明的单例类，用户创建对象时，可以像使用其他任何普通类一样

```javascript
//这段代码既创建了对象，又返回了单例，跟其他创建对象方式无异
var CreateDiv=(function(){
    var instance;
    var CreateDiv=function(html){
        if(instance){
            return instance;
        }
        this.htm=html;
        this.init();
        return instance=this;
    };


    CreateDiv.proptype.init=function(){
        var div=document.createElement('div');
        div.innerHTML=this.html;
        document.body.appendChild(div);
    };
    return CreateDiv;
})（）

var a=new CreateDiv('sven1');
var b=new CreateDiv('sven2');
```

我们还需要将创建对象，执行初始化init方法与管理单例的逻辑分开，这里引入代理类来优化代码

```javascript
var CreateDiv=function(html) {
    this.html = html;
    this.init();
}
CreateDiv.proptype.init=function(){
    var div=document.createElement('div');
    div.innerHTML=this.html;
    document.body.appendChild(div);
};


var ProxySingleTonCreateDiv=(function() {
    var instance;
    return function (html) {
        if (!instance) {
            instance = new CreateDiv(html);
        }

        return instance;
    };
})()


var a=new CreateDiv('sven1');
var b=new CreateDiv('sven2');
```

#### Javascript中实现

JavaScript是无类的语言，在JavaScript中，我们通过全局变量来实现单例模式，但是全局变量经常带来命名污染的问题，解决方案如下：

**1.使用命名空间**

```javascript
var namespace1={
    a:function(){
        alert(1);
    },
    b:function(){
        laert(2);
    }
}
```

**2.使用闭包封装私有变量**

```javascript
var user=(function(){
    var _name='sven',_age=29;
    return{
        getUserInfo:function(){
            return _name+'-'+'_age';
        }
    }
})
```

## 中介者模式

中介者模式是指对象和对象之间借助第三方中介者进行通信.中介模式使各个对象之间得以解耦，以中介者和对象之间的一对多关系取代了对象之间的网状多对多关系。各个对象只需关注自身的实现，对象之间的交互关系交给了中介者来实现和维护。

一场测试结束后, 公布结果: 告知解答出题目的人挑战成功, 否则挑战失败。

```javascript
const player = function(name) {
  this.name = name
  playerMiddle.add(name)
}

player.prototype.win = function() {
  playerMiddle.win(this.name)
}

player.prototype.lose = function() {
  playerMiddle.lose(this.name)
}

//在这段代码中 A、B、C 之间没有直接发生关系, 而是通过另外的 playerMiddle 对象建立链接, 姑且将之当成是中介者模式了
const playerMiddle = (function() { 
  const players = []
  const winArr = []
  const loseArr = []
  return {
    add: function(name) {
      players.push(name)
    },
    win: function(name) {
      winArr.push(name)
      if (winArr.length + loseArr.length === players.length) {
        this.show()
      }
    },
    lose: function(name) {
      loseArr.push(name)
      if (winArr.length + loseArr.length === players.length) {
        this.show()
      }
    },
    show: function() {
      for (let winner of winArr) {
        console.log(winner + '挑战成功;')
      }
      for (let loser of loseArr) {
        console.log(loser + '挑战失败;')
      }
    },
  }
}())

const a = new player('A 选手')
const b = new player('B 选手')
const c = new player('C 选手')

a.win()
b.win()
c.lose()
```

## 适配器模式

适配器模式用来解决两个软件实体的接口不兼容的问题

```javascript
var googleMap = {
		show: function(){
			console.log( '开始渲染谷歌地图' );
		}
	};
	var baiduMap = {
		display: function(){
			console.log( '开始渲染百度地图' );
		}
	};

    var renderMap = function( map ){
		if ( map.show instanceof Function ){
			map.show();
		}
	}; 



  //解决百度地图中展示地图的方法名与谷歌地图中展示地图的方法名不一样的问题
	var baiduMapAdapter = {
		show: function(){
			return baiduMap.display();

		}
	};

	renderMap( googleMap ); // 输出：开始渲染谷歌地图
	renderMap( baiduMapAdapter ); // 输出：开始渲染百度地图
```

## 迭代器模式

迭代器是指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。迭代器模式可以把迭代的过程从业务逻辑中分离出来，在使用迭代器模式之后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

**迭代器的实现：**

```javascript
var each=function(arry,callback){
    for(var i=0,l=ary.length;i<l;i++) {
        callback.call(ary[i], i, ary[i]);
    }
}

each([1,2,3],function(i,n){
    alert([i,n]);
})
```

#### 内部迭代器

上面的each函数即使内部迭代器，由于内部迭代器的迭代规则已经被提前规定，上面的each函数就无法同时迭代2个数组了，不如现在如果我们要实现判断2个数组里面的元素的值是否完全相等，如果不能修改each函数本身的话就只能修改each的回调函数了

```javascript
var compare=function(ary1,ary2){
    if(ary1.length!=ary2.length){
        throw new Error("ary1和ary2不相等")
    }
    each(ary1,funtion(i,n){
        if(n!==ary2[i]){
            thorw new Error('ary1和ary2不相等')
        }
    })
    alert('ary1和ary2相等')
}

compare([1,2,3],[1,2,4]);
```

#### 外部迭代器

外部迭代器必须显式的请求迭代下一个元素，外部迭代器增加了一些调用的复杂度，但相对也增强了迭代器的灵活性，我们可以手工控制迭代的过程或顺序

```javascript
var Interator=function(obj){
    var current=0;
    var next=funtion(){
        current+=1;
    }
    var isDone=function(){
        return current>=obj.length;
    }
    var getCurrItem=function(){
        return obj[current];
    }
    return {
        next:next,
        isDone:isDone,
    getCurrItem:getCurrItem,
        length:obj.length
    }
}
```

## 组合模式

- 组合模式在对象间形成`树形结构`;
- 组合模式中基本对象和组合对象被`一致对待`;
- 无须关心对象有多少层, 调用时只需在根部进行调用;

  扫描文件夹时, 文件夹下面可以为另一个文件夹也可以为文件, 我们希望统一对待这些文件夹和文件, 这种情形适合使用组合模式。

```javascript
const Folder = function(folder) {
  this.folder = folder
  this.lists = []
}

Folder.prototype.add = function(resource) {
  this.lists.push(resource)
}

Folder.prototype.scan = function() {
  console.log('开始扫描文件夹: ', this.folder)
  for (let i = 0, folder; folder = this.lists[i++];) {
    folder.scan()
  }
}

const File = function(file) {
  this.file = file
}

File.prototype.add = function() {
  throw Error('文件下不能添加其它文件夹或文件')
}

File.prototype.scan = function() {
  console.log('开始扫描文件: ', this.file)
}

const folder = new Folder('根文件夹')
const folder1 = new Folder('JS')
const folder2 = new Folder('life')

const file1 = new File('深入React技术栈.pdf')
const file2 = new File('JavaScript权威指南.pdf')
const file3 = new File('小王子.pdf')

folder1.add(file1)
folder1.add(file2)

folder2.add(file3)

folder.add(folder1)
folder.add(folder2)

folder.scan()

// 开始扫描文件夹:  根文件夹
// 开始扫描文件夹:  JS
// 开始扫描文件:  深入React技术栈.pdf
// 开始扫描文件:  JavaScript权威指南.pdf
// 开始扫描文件夹:  life
// 开始扫描文件:  小王子.pdf
```

## 享元模式

享元模式是一种优化程序性能的模式, 本质为减少对象创建的个数。

以下情况可以使用享元模式:

1. 有大量相似的对象, 占用了大量内存
2. 对象中大部分状态可以抽离为外部状态

某商家有 50 种男款内衣和 50 种款女款内衣, 要展示它们

方案一: 造 50 个塑料男模和 50 个塑料女模, 让他们穿上展示, 代码如下:

```javascript
const Model = function(gender, underwear) {
  this.gender = gender
  this.underwear = underwear
}

Model.prototype.takephoto = function() {
  console.log(`${this.gender}穿着${this.underwear}`)
}

for (let i = 1; i < 51; i++) {
  const maleModel = new Model('male', `第${i}款衣服`)
  maleModel.takephoto()
}

for (let i = 1; i < 51; i++) {
  const female = new Model('female', `第${i}款衣服`)
  female.takephoto()
}
```

方案二: 造 1 个塑料男模特 1 个塑料女模特, 分别试穿 50 款内衣

```javascript
const Model = function(gender) {
  this.gender = gender
}

Model.prototype.takephoto = function() {
  console.log(`${this.sex}穿着${this.underwear}`)
}

const maleModel = new Model('male')
const femaleModel = new Model('female')

for (let i = 1; i < 51; i++) {
  maleModel.underwear = `第${i}款衣服`
  maleModel.takephoto()
}

for (let i = 1; i < 51; i++) {
  femaleModel.underwear = `第${i}款衣服`
  femaleModel.takephoto()
}
```

对比发现: 方案一创建了 100 个对象, 方案二只创建了 2 个对象, 在该 demo 中, gender(性别) 是内部对象, underwear(穿着) 是外部对象。

当然在方案二的 demo 中, 还可以进一步改善:

1. 一开始就通过构造函数显示地创建实例, 可用工场模式将其升级成可控生成
2. 在实例上手动添加 underwear 不是很优雅, 可以在外部单独在写个 manager 函数

```javascript
const Model = function(gender) {
  this.gender = gender
}

Model.prototype.takephoto = function() {
  console.log(`${this.gender}穿着${this.underwear}`)
}

const modelFactory = (function() { // 优化第一点
  const modelGender = {}
  return {
    createModel: function(gender) {
      if (modelGender[gender]) {
        return modelGender[gender]
      }
      return modelGender[gender] = new Model(gender)
    }
  }
}())

const modelManager = (function() {
  const modelObj = {}
  return {
    add: function(gender, i) {
      modelObj[i] = {
        underwear: `第${i}款衣服`
      }
      return modelFactory.createModel(gender)
    },
    copy: function(model, i) { // 优化第二点
      model.underwear = modelObj[i].underwear
    }
  }
}())

for (let i = 1; i < 51; i++) {
  const maleModel = modelManager.add('male', i)
  modelManager.copy(maleModel, i)
  maleModel.takephoto()
}

for (let i = 1; i < 51; i++) {
  const femaleModel = modelManager.add('female', i)
  modelManager.copy(femaleModel, i)
  femaleModel.takephoto()
}
```

## 模板方法模式

模板方法是一种基于继承的设计模式，模板方法模式有两个部分组成，第一个是抽象父类，第二个是具体的实现子类。但在JavaScript中，我们很多时候都不需要画瓢一样去实现一个模板方法模式，高阶函数是更好的选择。

```javascript
var Coffee = function(){};
	Coffee.prototype.boilWater = function(){
		console.log( '把水煮沸' );
	};
	Coffee.prototype.brewCoffeeGriends = function(){
		console.log( '用沸水冲泡咖啡' );
	};
	Coffee.prototype.pourInCup = function(){
		console.log( '把咖啡倒进杯子' );
	};
	Coffee.prototype.addSugarAndMilk = function(){
		console.log( '加糖和牛奶' );
	};
	Coffee.prototype.init = function(){
		this.boilWater();
		this.brewCoffeeGriends();
		this.pourInCup();
		this.addSugarAndMilk();
	};
	var coffee = new Coffee();
	coffee.init();

	var Tea = function(){};
	Tea.prototype.boilWater = function(){
		console.log( '把水煮沸' );
	};
	Tea.prototype.steepTeaBag = function(){
		console.log( '用沸水浸泡茶叶' );
	};
	Tea.prototype.pourInCup = function(){
		console.log( '把茶水倒进杯子' );
	};
	Tea.prototype.addLemon = function(){
		console.log( '加柠檬' );
	};
	Tea.prototype.init = function(){
		this.boilWater();
		this.steepTeaBag();
		this.pourInCup();
		this.addLemon();
	};
	var tea = new Tea();
	tea.init();
```

泡茶和泡咖啡的过程中有几个是一样的，我们可以定义一个Beverage类，定义泡茶和泡咖啡中的共同动作，而泡茶和泡咖啡中独有的动作可以直接重写Beverage类中的方法即可

```javascript
var Beverage = function( param ){
		var boilWater = function(){
			console.log( '把水煮沸' );
		};
		var brew = param.brew || function(){
			throw new Error( '必须传递brew 方法' );
		};
		var pourInCup = param.pourInCup || function(){
			throw new Error( '必须传递pourInCup 方法' );
		};
		var addCondiments = param.addCondiments || function(){
			throw new Error( '必须传递addCondiments 方法' );
		};
		var F = function(){};
		F.prototype.init = function(){
			boilWater();
			brew();
			pourInCup();
			addCondiments();
		};
		return F;
	};
	var Coffee = Beverage({
		brew: function(){
			console.log( '用沸水冲泡咖啡' );
		},
		pourInCup: function(){
			console.log( '把咖啡倒进杯子' );
		},
		addCondiments: function(){
			console.log( '加糖和牛奶' );
		}
	});

	var Tea = Beverage({
		brew: function(){
			console.log( '用沸水浸泡茶叶' );
		},
		pourInCup: function(){
			console.log( '把茶倒进杯子' );
		},
		addCondiments: function(){
			console.log( '加柠檬' );
		}
	});
	var coffee = new Coffee();
	coffee.init();
	var tea = new Tea();
	tea.init();
```

## 命令模式

我们把某次任务分成两个部分，一部分程序员实现绘制按钮，他们不知道按钮用来干什么，一部分程序员负责编写点击按钮后的具体行为。

```javascript
<body>

	<button id="button1">点击按钮1</button>
	<button id="button2">点击按钮2</button>
	<button id="button3">点击按钮3</button>

	<script>
		var button1 = document.getElementById( 'button1' ),
		var button2 = document.getElementById( 'button2' ),
		var button3 = document.getElementById( 'button3' );

		var setCommand = function( button, command ){
			button.onclick = function(){
				command.execute();
			}
		};

		var MenuBar = {
			refresh: function(){
				console.log( '刷新菜单目录' );
			}
		};
		var SubMenu = {
			add: function(){
				console.log( '增加子菜单' );
			},
			del: function(){
				console.log( '删除子菜单' );
			}
		};
		在让button 变得有用起来之前，我们要先把这些行为都封装在命令类中：
		var RefreshMenuBarCommand = function( receiver ){
			this.receiver = receiver;
		};
		RefreshMenuBarCommand.prototype.execute = function(){
			this.receiver.refresh();
		};
		var AddSubMenuCommand = function( receiver ){
			this.receiver = receiver;
		};

		AddSubMenuCommand.prototype.execute = function(){
			this.receiver.add();
		};
		var DelSubMenuCommand = function( receiver ){
			this.receiver = receiver;
		};
		DelSubMenuCommand.prototype.execute = function(){
			console.log( '删除子菜单' );
		};

		var refreshMenuBarCommand = new RefreshMenuBarCommand( MenuBar );
		var addSubMenuCommand = new AddSubMenuCommand( SubMenu );
		var delSubMenuCommand = new DelSubMenuCommand( SubMenu );
		setCommand( button1, refreshMenuBarCommand );
		setCommand( button2, addSubMenuCommand );
		setCommand( button3, delSubMenuCommand );
	</script>
</body>
```



**参考资料：**

《JavaScript设计模式与开发实践》