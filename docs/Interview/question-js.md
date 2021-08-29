### 概念题

#### 基本数据类型

> String Number  Boolean Null Undefined Symbol  BigInt
>
> Symbol 独一无二 且不可变的数据类型 ，主要为了解决全局变量冲突的问题
>
> BigInt 是数字类型的数据，表示任意精度格式的整数，可以安全的存储和操作大整数

#### js有几种类型的值

> 栈：原始数据类型 栈中存放引用数据类型的指针
>
> 堆：引用数据类型（对象、数组和函数）

#### 什么是堆？什么是栈？他们之前有什么区别和联系

> 栈：后进先出 
>
> 堆：是一个优先队列，按优先级进行排序
>
> 栈区内存由编译器自动分配释放，存放函数的参数值，局部变量的值
>
> 堆区内存一般由程序员分配

#### 内部属性[[Class]]是什么

> 所有typeof返回值为 object的对象（如数组）都包含一个内部属性[[class]] (我们可以把它看做是一个内部的分类，而非传统的面向对象意义上的类)。这个属性无法直接访问，一般通过 Object.prototype.toString.call()来查看
>
> Object.prototype.toString.call([1,2]) // [object Array]
>
> 可以自定义toStringTag
>
> class Class2{
>
> ​	get [Symbol.toStringTag]() {
>
> ​		return 'Class2'
>
> ​	}
>
> }
>
> Object.prototype.toString.call(new Class2()) // [object Class2]

#### js的内置对象

> js中的内置对象主要指的是在程序执行前存在全局作用域里的由js定义的一些全局值属性、函数和用来实例化其他对象的构造函数对象，一般我们经常用到的全局变量值NaN、undefined. 全局函数如parseInt()、parseFloat()。用来实例化对象的构造函数如 Date、Object等 还有Math

#### Undefined 和 undeclared的区别

> 已在作用域中声明但还没有赋值的变量 是 undefined
>
> 没有在作用域声明过的变量 是undeclared

#### Null 和 undefined的区别

> undefined代表未定义，null代表空对象，一般声明了但是没有定义的时候会返回undefined, null主要用于赋值一些可能会返回对象的变量，作为初始化
>
> undefined不是保留字，可以作为变量名，这样会影响对undefined的判断， 通过void 0来拿到undefined的值
>
> typeof null  = = 'object'   undefined == null // true   undefined === null // false

#### 原型和原型链

> 略

#### js获取原型的方法

> `p.__proto__`
>
> p.constructor.prototype
>
> Object.getPrototypeOf(p)

#### Typeof NaN的结果是什么

> Typeof  NaN // "number"
>
> NaN!=NaN  // true

#### isNaN和Number.isNaN函数的区别

> 函数isNaN接收参数后，会尝试将这个参数转换为数值， 任何不能被转换为数值的值都会返回true，因此非数字传入也会返回 true
>
> Number.isNaN会首先判断传入参数是否为数字，如果是数字 在继续判断是否为NaN。

#### 其他值到布尔值类型的值的转换规则

> +0 -0 '' null undefined NaN false 这几个值为false，其余都为true

#### {} 和 [] 的valueOf 和 toString的结果是什么

> {} 的valueOf结果为 {}, toString的结果是 '[object Object]'
>
> [] 的valueOf结果为 [], toString的结果为 ""

#### parseInt 和 Number转换为数字的区别

> parseInt 解析按从左到右的顺序，如果遇到非数字字符就停止
>
> Number不允许出现非数字字符，否则会失败并返回NaN

#### 继承

> 略

#### this

> 略

#### 事件是什么，如何阻止冒泡

> 事件是用户操作网页时发生的交互动作，比如click/mouse等 还有文档加载、窗口滚动和大小调整
>
> e.stopPropagation()   e.cancelBubble = true

#### 事件模型

> dom0级  onclick 
>
> Dom2级  addEventListener('click', handler, false) // false 默认冒泡
>
> 先捕获=> 目标阶段 => 冒泡

#### 事件委托

> 利用事件冒泡，把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素事件

#### 闭包

> 闭包是指有权访问另一个函数作用域变量的函数。创建闭包的最常见的方式就是在一个函数内创建另一个函数，创建的函数可以访问到当前函数的局部变量

#### 判断一个对象是否属于某个类

> instanceof
>
> Constructor 不安全 因为其可以修改
>
> Object.prototype.toString.call

#### instanceof

```
function myInstance(left, right) {
	let proto = left.__proto__,
	prototype = right.prototype
	while(true) {
		if(!proto) return false
		if(proto == prototype) return true
		proto = proto.__proto__
	}
	
}
```

#### New 操作符

```js
function myNew() {
	let constructor = Array.prototype.shift.call(arguments)
	
	let newObj = Object.create(constructor.prototype)
	let result = constructor.apply(newObj, arguments)
	let flag = result && (typeof result == 'object' || typeof result == 'function')
	return flag ? result : newObj
}
```

#### js延迟加载的方式

> js脚本放到文档的底部，来使最后加载执行
>
> defer属性，脚本加载和文档解析同步，会等到文档解析完成后顺序执行
>
> async属性，异步加载不会阻塞文档的加载，脚本加载完成后立即执行js脚本，如果文档没有解析完成会阻塞。不会按照顺序执行
>
> 动态创建dom标签的方式

#### 同源策略

> 协议、端口、域名

#### 跨域

> jsonp
>
> Document.domain + iframe
>
> Location.hash + iframe
>
> Window.name + iframe
>
> postMessage
>
> cors
>
> nginx代理
>
> node中间件代理
>
> Websocket 

#### cookie

> 用于维护绘画状态信息的数据，通过服务器发送到浏览器，浏览器保存在本地，当下一次有同源的请求时，将保存的cookie值添加到请求头部，发送给服务器

#### 哪些操作会造成内存泄漏

> 意外的全局变量
>
> 被遗忘的计时器或回调函数
>
> 脱离DOM的引用
>
> 闭包

### 手写题

#### Call、apply、bind

```
// call
Function.prototype.myCall = function(context) {
  if(typeof this!=='function') {
    console.error('type error')
  }
  let args = [...arguments].slice(1),
      result = null
  context = context || window
  context.fn = this
  result = context.fn(...args)
  delete context.fn
  return result
}

// apply
Function.prototype.myApply = function(context) {
  if(typeof this!=='function') {
    console.error('type error')
  }
  let result = null
  context = context || window
  context.fn = this
  if(arguments[1]) {
    result = context.fn(...arguments[1])
  }else{
    result = context.fn()
  }
  delete context.fn
  return result
}
// bind
Function.prototype.myBind = function(context) {
  if(typeof this!=='function') {
    console.error('type error')
  }
  let args = [...arguments].slice(1),
      fn = this
  
  return function Fn() {
    return fn.apply(
    	this instanceof Fn ? this : context,
      args.concat(...arguments)
    )
  }
  
}
```

#### 深浅拷贝

**浅拷贝**

```js
const shallowClone = (target) => {
	if(typeof target == 'object' && target!==null) {
		const cloneObj = Array.isArray(target) ? [] : {}
		for(let key in target) {
			if(target.hasOwnProperty(key)) {
				cloneObj[key] = target[key]
			}
		}
		return cloneObj
	}else{
		return target
	}
}
```

**深拷贝**

```js
const isComplexDataType = obj => (typeof obj == 'object' || typeof obj == 'function ') && (obj!==null)

const deepClone = (obj, hash = new WeakMap()) => {
  // 特殊类型判断
	if(obj.constructor == Date) {
		return new Date(obj)
	}
	if(obj.constructor == RegExp) {
		return new RegExp(obj)
	}
	// 循环引用
	if(hash.has(obj)) {
		return hash.get(obj)
	}
  // 创建新的拷贝对象 / 数组
	let allDesc = Object.getOwnPropertyDescriptors(obj)
	
	let cloneObj = Object.create(Object.getPrototypeOf(obj), allDesc)
	
	hash.set(obj, cloneObj)
	
  // for in ownKeys可以获取不可枚举的类型
	for(let key in Reflect.ownKeys(obj)) {
		cloneObj[key] = (isComplexDataType(obj[key]) && typeof obj[key]!=='function') ? deepClone(obj[key], hash) : obj[key]
	}
	
	return cloneObj
}
```

#### aop实现函数的before和after

> 保证执行的顺序
>
> this指向如何保证
>
> 参数保证

```js
Function.prototype.before = function(beforeFn) {
  var _this = this
  
  return function() {
  	beforeFn.apply(this, arguments)
  	_this.apply(this, arguments)
  }
}

Function.prototype.after = function(afterFn) {
  var _this = this
  
  return function() {
  	_this.apply(this, arguments)
  	afterFn.apply(this, arguments)
  }
}

var obj = {
	test:test
}

function test() {
	console.log(this, ...arguments)
	console.log(2)
}

test = obj.test
.before(function() {
	console.log(this, ...arguments)
	console.log(1)
}).after(function() {
	console.log(this, ...arguments)
	console.log(3)
})

test.call(obj, 1,2,3)
```

#### 柯里化和反柯里化

**柯里化**

```js
function curry(func) {
 return function curried(...args) {
 		if(args.length >=func.length) {
 			return func.apply(this, args)
 		}else{
 			return function(...args2) {
 				return curried.apply(this, args.concat(args2))
 			}
 		}
 }
}
```

**反柯里化**

```js
Function.prototype.uncurry = function() {
	let fn = this
	return function() {
		let _this = Array.prototype.shift.call(arguments)
		return fn.apply(_this, arguments)
	}
}

// 使用
let push = Array.prototype.push.uncurry()
let obj = {}
push(obj, 'first')
```

#### 防抖和节流

**防抖**

```js
function debounce(func, wait, immediate) {
	let timeout;
  let result
  let decanceled = function () {
    if(timeout) clearTimeout(timeout)
	  if(immediate) {
	    // 第一次 立即执行 多次触发的时候 timeout有值导致callNow为false 所以函数不会在执行
	  	let callNow = !timeout
	  	timeout = setTimeout(() => {
	  		timeout = null
	  	},wait)
	  	if(callNow) result = func.apply(this, arguments)
	  }else{
	  	timeout = setTimeout(() => {
        // this指向和arguments
        func.apply(this, arguments)
      },wait)
	  }
    // 返回结果
		return result
  }
	// 取消
  decanceled.cancel = function() {
    clearTimeout(timeout)
    timeout = null
  }
  return decanceled
}
```

**节流**

```js
function throttle(func, wait, options) {
	// options => {leading, trailing} 分别表示头和尾是否执行
	let timeout
	let old = 0
	if(!options) options = {}
	
	return function() {
		let now = Date.now()
		if(options.leading === false && !old) {
			old = now
		}
		if(now - old > wait) {
      // 头执行 尾不执行
			if(timeout) {
				clearTimeout(timeout)
				timeout = null
			}
			func.apply(this, arguments)
			old = now
		}else if(!timeout && options.trailing!==false) {
      // 头不执行 尾执行
			timeout = setTimeout(() => {
				old = Date.now()
				timeout = null
				func.apply(this, arguments)
			},wait)
		}
	}
}
```

#### 原型相关

```js
function Foo() {
	getName = function() {
		console.log(1)
	}
	return this
}

Foo.getName = function() {
	console.log(2)
}
Foo.prototype.getName = function() {
	console.log(3)
}

var getName = function() {
	console.log(4)
}

function getName() {
	console.log(5)
}

Foo.getName() // 2

getName() // 4

Foo().getName() // 1

getName() // 1

new Foo.getName() // 2   优先执行. (new 后面的函数是否带参)

new Foo().getName() // 3

new new Foo().getName() // 3

```

#### 数据结构前端的应用场景

- Vdom diff
- Hooks
- Fiber
- Git （最小编辑距离算法）
- webpack
- AST
- Browser history

#### copyString

> aaaccaad  => a3c3a2d1   leetcode 443

```js
function copyString(str) {
	if(!str) {
	 return null
	}
	const arr = []
	let char = str[0]
	let count = 1
	
	for(let i = 1;i<str.length;i++) {
		if(str[i] == char) {
			count++
		}else{
			arr.push(char)
			arr.push(count)
			char = str[i]
		}
	}
	return arr.join("") + ""
}
```

#### dealCard

> 发牌 每次随机发指定数目的牌

```js
function sendColor(name) {
	let pokers = ['A','2','3','4','5','6','7','8','9','10','J','Q','K']
	let res= []
	pokers.forEach(item => res.push(name + item + ''))
	return res
}

function sendCard() {
	let heart = sendColor('heart')
	let club = sendColor('club')
	let diamond = sendColor('diamond')
	let spade = sendColor('spade')
	
	let arr = heart.concat(club).concat(diamand).concat(spade)
	
	arr.sort((a, b) => Math.random() - 0.5)
	
	return arr
}

const card = sendCard()

function dealCard(count) {
	if(count > 52) {
		return null
	}
	
	let result = []
	if(card.length <=count) {
		return card
	}else{
		for(let i = 0;i<count;i++) {
			result.push(card.pop())
		}
	}
	return result
}
```

#### 数据类型判断

```js
function typeOf(obj) {
    let res = Object.prototype.toString.call(obj).split(' ')[1]
    res = res.substring(0, res.length - 1).toLowerCase()
    return res
}
typeOf([])        // 'array'
typeOf({})        // 'object'
typeOf(new Date)  // 'date'
```

#### 继承

**原型链继承**

```
function Animal() {
    this.colors = ['black', 'white']
}
Animal.prototype.getColor = function() {
    return this.colors
}
function Dog() {}
Dog.prototype =  new Animal()

let dog1 = new Dog()
dog1.colors.push('brown')
let dog2 = new Dog()
console.log(dog2.colors)  // ['black', 'white', 'brown']
```

**借用构造函数继承**

```
function Animal(name) {
    this.name = name
    this.getName = function() {
        return this.name
    }
}
function Dog(name) {
    Animal.call(this, name)
}
Dog.prototype =  new Animal()
```

**组合继承**

```
function Animal(name) {
    this.name = name
    this.colors = ['black', 'white']
}
Animal.prototype.getName = function() {
    return this.name
}
function Dog(name, age) {
    Animal.call(this, name)
    this.age = age
}
Dog.prototype =  new Animal()
Dog.prototype.constructor = Dog

let dog1 = new Dog('奶昔', 2)
dog1.colors.push('brown')
let dog2 = new Dog('哈赤', 1)
console.log(dog2) 
// { name: "哈赤", colors: ["black", "white"], age: 1 }
```

**寄生组合式继承**

```
 // 组合式的代码。。。
 Dog.prototype =  Object.create(Animal.prototype)
 Dog.prototype.constructor = Dog
```

**class 实现继承**

```

class Animal {
    constructor(name) {
        this.name = name
    } 
    getName() {
        return this.name
    }
}
class Dog extends Animal {
    constructor(name, age) {
        super(name)
        this.age = age
    }
}
```

#### 数组去重

```
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var arrry= [];
    var  obj = {};
    for (var i = 0; i < arr.length; i++) {
        if (!obj[arr[i]]) {
            arrry.push(arr[i])
            obj[arr[i]] = 1
        } else {
            obj[arr[i]]++
        }
    }
    return arrry;
}

function unique(arr) {
	var res = arr.filter((item, index, array) => {
		return array.indexOf(item) === index
	})
	return res
}

function unique(arr) {
	return [...new Set(arr)]
}
```

#### 数组扁平化

```
[1,[2,[3]]].flat(2)

function flatten(arr) {
	var result = []
	for(let i = 0,len=arr.length;i<len;i++) {
		if(Array.isArray(arr[i])) {
			result = result.concat(flatten(arr[i]))
		}else{
			result.push(arr[i])
		}
	}
	return result
}

function flatten(arr) {
	while(arr.some(item=>Array.isArray(item))) {
		arr = [].concat(...arr)
	}
	return arr
}
```

#### 事件总线

```js
class EventEmitter {
  constructor() {
    this.events = {};
  }
  on(type, listener, isUnshift) {
    // 因为其他的类可能继承自EventEmitter，子类的events可能为空，保证子类必须存在此实例属性
    if(!this.events) {
      this.events = {};
    }
    if(this.events[type]) {
      if(isUnshift) {
        this.events[type].unshift(listener);
      } else {
        this.events[type].push(listener);
      }
    } else {
      this.events[type] = [listener]
    }

    if(type !== 'newListener') {
      // node的EventEmitter模块自带的特殊事件，该事件在添加新事件监听器的时候触发
      this.emit('newListener', type);
    }
  }
  emit(type, ...args) {
    if(this.events[type]) {
      this.events[type].forEach(fn => fn.call(this, ...args));
    }
  }
  // 只绑定一次，然后解绑
  once(type, listener) {
    const me = this;
    function oneTime(...args) {
      listener.call(this, ...args);
      me.off(type, oneTime);
    }
    me.on(type, oneTime)
  }
  off(type, listener) {
    if(this.events[type]) {
      const index = this.events[type].indexOf(listener);
      this.events[type].splice(index, 1);
    }
  }
}

// 运行示例
let event = new EventEmitter();

event.on('say',function(str) {
  console.log(str);
});

event.once('say', function(str) {
  console.log('这是once:' + str)
})

event.emit('say','visa');
event.emit('say','visa222');
event.emit('say','visa333');

```

#### 解析URL参数为对象

```js
// /([^&=]+)=([^&=]+)/g

function parseParam(url) {
    const paramsStr = /.+\?(.+)$/.exec(url)[1]; // 将 ? 后面的字符串取出来
    const paramsArr = paramsStr.split('&'); // 将字符串以 & 分割后存到数组中
    let paramsObj = {};
    // 将 params 存到对象中
    paramsArr.forEach(param => {
        if (/=/.test(param)) { // 处理有 value 的参数
            let [key, val] = param.split('='); // 分割 key 和 value
            val = decodeURIComponent(val); // 解码
            val = /^\d+$/.test(val) ? parseFloat(val) : val; // 判断是否转为数字
    
            if (paramsObj.hasOwnProperty(key)) { // 如果对象有 key，则添加一个值
                paramsObj[key] = [].concat(paramsObj[key], val);
            } else { // 如果对象没有这个 key，创建 key 并设置值
                paramsObj[key] = val;
            }
        } else { // 处理没有 value 的参数
            paramsObj[param] = true;
        }
    })
    
    return paramsObj;
}

```

#### 字符串模板

```js
function render(template, data) {
    const reg = /\{\{(\w+)\}\}/; // 模板字符串正则
    if (reg.test(template)) { // 判断模板里是否有模板字符串
        const name = reg.exec(template)[1]; // 查找当前模板里第一个模板字符串的字段
        template = template.replace(reg, data[name]); // 将第一个模板字符串渲染
        return render(template, data); // 递归的渲染并返回渲染后的结构
    }
    return template; // 如果模板没有模板字符串直接返回
}

let template = '我是{{name}}，年龄{{age}}，性别{{sex}}';
let person = {
    name: '布兰',
    age: 12
}
render(template, person); // 我是布兰，年龄12，性别undefined

```

#### 图片懒加载

```
let imgList = [...document.querySelectorAll('img')]
let length = imgList.length

const imgLazyLoad = function() {
    let count = 0
    return (function() {
        let deleteIndexList = []
        imgList.forEach((img, index) => {
            let rect = img.getBoundingClientRect()
            if (rect.top < window.innerHeight) {
                img.src = img.dataset.src
                deleteIndexList.push(index)
                count++
                if (count === length) {
                    document.removeEventListener('scroll', imgLazyLoad)
                }
            }
        })
        imgList = imgList.filter((img, index) => !deleteIndexList.includes(index))
    })()
}

// 这里最好加上防抖处理
document.addEventListener('scroll', imgLazyLoad)
```

#### JSONP

```js
const jsonp = ({ url, params, callbackName }) => {
    const generateUrl = () => {
        let dataSrc = ''
        for (let key in params) {
            if (params.hasOwnProperty(key)) {
                dataSrc += `${key}=${params[key]}&`
            }
        }
        dataSrc += `callback=${callbackName}`
        return `${url}?${dataSrc}`
    }
    return new Promise((resolve, reject) => {
        const scriptEle = document.createElement('script')
        scriptEle.src = generateUrl()
        document.body.appendChild(scriptEle)
        window[callbackName] = data => {
            resolve(data)
            document.removeChild(scriptEle)
        }
    })
}
```

#### AJAX

```js
const getJSON = function(url) {
    return new Promise((resolve, reject) => {
        const xhr = XMLHttpRequest ? new XMLHttpRequest() : new ActiveXObject('Mscrosoft.XMLHttp');
        xhr.open('GET', url, false); // 同步
        xhr.setRequestHeader('Accept', 'application/json');
        xhr.onreadystatechange = function() {
            if (xhr.readyState !== 4) return;
            if (xhr.status === 200 || xhr.status === 304) {
                resolve(xhr.responseText);
            } else {
                reject(new Error(xhr.responseText));
            }
        }
        xhr.send();
    })
}
```

#### 实现数组方法

**forEach**

```js
Array.prototype.forEach2 = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or not defined')
    }
    if (typeof callback !== "function") {
        throw new TypeError(callback + ' is not a function')
    }
    const O = Object(this)  // this 就是当前的数组
    const len = O.length >>> 0  
    let k = 0
    while (k < len) {
        if (k in O) {
            callback.call(thisArg, O[k], k, O);
        }
        k++;
    }
}
```

**map**

```js
- Array.prototype.forEach2 = function(callback, thisArg) {
+ Array.prototype.map2 = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or not defined')
    }
    if (typeof callback !== "function") {
        throw new TypeError(callback + ' is not a function')
    }
    const O = Object(this)
    const len = O.length >>> 0
-   let k = 0
+   let k = 0, res = []
    while (k < len) {
        if (k in O) {
-           callback.call(thisArg, O[k], k, O);
+           res[k] = callback.call(thisArg, O[k], k, O);
        }
        k++;
    }
+   return res
}
```

**filter**

```js
- Array.prototype.forEach2 = function(callback, thisArg) {
+ Array.prototype.filter2 = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or not defined')
    }
    if (typeof callback !== "function") {
        throw new TypeError(callback + ' is not a function')
    }
    const O = Object(this)
    const len = O.length >>> 0
-   let k = 0
+   let k = 0, res = []
    while (k < len) {
        if (k in O) {
-           callback.call(thisArg, O[k], k, O);
+           if (callback.call(thisArg, O[k], k, O)) {
+               res.push(O[k])                
+           }
        }
        k++;
    }
+   return res
}
```

**some**

```js
- Array.prototype.forEach2 = function(callback, thisArg) {
+ Array.prototype.some2 = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or not defined')
    }
    if (typeof callback !== "function") {
        throw new TypeError(callback + ' is not a function')
    }
    const O = Object(this)
    const len = O.length >>> 0
    let k = 0
    while (k < len) {
        if (k in O) {
-           callback.call(thisArg, O[k], k, O);
+           if (callback.call(thisArg, O[k], k, O)) {
+               return true
+           }
        }
        k++;
    }
+   return false
}
```

**reduce**

```js
Array.prototype.myReduce = function(...args) {
	let arr = this
	let fn = args[0]
	let initVal, index
	if(args.length > 1) {
		initVal = args[1]
		index = 0
	}else{
		initVal = arr[0]
		index = 1
	}
	
	let value = initVal
	for(let i = index,len=arr.length;i<len;i++) {
		value = fn(value, arr[i])
	}
	return value
}
```

#### Object.create

```js
Object.create2 = function(proto, propertyObject = undefined) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
        throw new TypeError('Object prototype may only be an Object or null.')
    if (propertyObject == null) {
        new TypeError('Cannot convert undefined or null to object')
    }
    function F() {}
    F.prototype = proto
    const obj = new F()
    if (propertyObject != undefined) {
        Object.defineProperties(obj, propertyObject)
    }
    if (proto === null) {
        // 创建一个没有原型对象的对象，Object.create(null)
        obj.__proto__ = null
    }
    return obj
}
```

#### Object.assign

```js
Object.assign2 = function(target, ...source) {
    if (target == null) {
        throw new TypeError('Cannot convert undefined or null to object')
    }
    let ret = Object(target) 
    source.forEach(function(obj) {
        if (obj != null) {
            for (let key in obj) {
                if (obj.hasOwnProperty(key)) {
                    ret[key] = obj[key]
                }
            }
        }
    })
    return ret
}
```

#### JSON.stringify

```
function jsonStringify(data) {
    let dataType = typeof data;
    
    if (dataType !== 'object') {
        let result = data;
        //data 可能是 string/number/null/undefined/boolean
        if (Number.isNaN(data) || data === Infinity) {
            //NaN 和 Infinity 序列化返回 "null"
            result = "null";
        } else if (dataType === 'function' || dataType === 'undefined' || dataType === 'symbol') {
            //function 、undefined 、symbol 序列化返回 undefined
            return undefined;
        } else if (dataType === 'string') {
            result = '"' + data + '"';
        }
        //boolean 返回 String()
        return String(result);
    } else if (dataType === 'object') {
        if (data === null) {
            return "null"
        } else if (data.toJSON && typeof data.toJSON === 'function') {
            return jsonStringify(data.toJSON());
        } else if (data instanceof Array) {
            let result = [];
            //如果是数组
            //toJSON 方法可以存在于原型链中
            data.forEach((item, index) => {
                if (typeof item === 'undefined' || typeof item === 'function' || typeof item === 'symbol') {
                    result[index] = "null";
                } else {
                    result[index] = jsonStringify(item);
                }
            });
            result = "[" + result + "]";
            return result.replace(/'/g, '"');
            
        } else {
            //普通对象
            /**
             * 循环引用抛错(暂未检测，循环引用时，堆栈溢出)
             * symbol key 忽略
             * undefined、函数、symbol 为属性值，被忽略
             */
            let result = [];
            Object.keys(data).forEach((item, index) => {
                if (typeof item !== 'symbol') {
                    //key 如果是symbol对象，忽略
                    if (data[item] !== undefined && typeof data[item] !== 'function'
                        && typeof data[item] !== 'symbol') {
                        //键值如果是 undefined、函数、symbol 为属性值，忽略
                        result.push('"' + item + '"' + ":" + jsonStringify(data[item]));
                    }
                }
            });
            return ("{" + result + "}").replace(/'/g, '"');
        }
    }
}

```

#### JSON.parse

**eval**

```
var rx_one = /^[\],:{}\s]*$/;
var rx_two = /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g;
var rx_three = /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g;
var rx_four = /(?:^|:|,)(?:\s*\[)+/g;

if (
    rx_one.test(
        json.replace(rx_two, "@")
            .replace(rx_three, "]")
            .replace(rx_four, "")
    )
) {
    var obj = eval("(" +json + ")");
}
```

**new Function**

```
var json = '{"name":"小姐姐", "age":20}';
var obj = (new Function('return ' + json))();
```



#### Promise

**promise超时处理**

> 需求：在微服务这种发送一个请求，如果三秒钟还没有收到结果，我们就认为失败

```js
const p = Promise.race([
	getUrl('/xx'),
	new Promise((resolve, reject) => {
		setTimeout(() => reject(new Error('timeout')), 3000)
	})
])

p.then(console.log)
.catch(console.error)
```

```js
async function async1() {
	console.log('async1 start')
	await async2()
	console.log('async1 end')
  
  // 这里可以理解为
  /*
  new Promise(function() {
    async2()
  }).then(() => {
    console.log('async1 end')
  })
  */
}
async function async2() {
	console.log('async2')
}
console.log('script start')
setTimeout(function(){
	console.log('setTimeout')
},0)
async1()
new Promise(function(resolve){
	console.log('promise1')
	resolve()
}).then(function() {
	console.log('promise2')
})
console.log('script end')

/*
	script start
	async1 start
	async2
	promise1
	script end
	async1 end
	promise2
	setTimeout
*/
```

```js
console.log("start");
setTimeout(() => {
  console.log("children2");
  Promise.resolve().then(() => {
    console.log("children3");
  });
}, 0);

new Promise(function (resolve) {
  console.log("children4");
  setTimeout(function () {
    console.log("children5");
    resolve("children6");
  }, 0);
}).then((res) => {
  console.log("children7");
  setTimeout(() => {
    console.log(res);
  }, 0);
});

/*
    start
    children4
    children2
    children3
    children5
    children7
    children6
*/

```

```js
const p = function () {
  return new Promise((resolve) => {
    const p1 = new Promise((resolve) => {
      setTimeout(() => {
        resolve(1);
      }, 0);
      resolve(2);
    });
    p1.then((res) => {
      console.log(res);
    });
    console.log(3);
    resolve(4);
  });
};

p().then((res) => {
  console.log(res);
});
console.log("end");

/*
    3
    end
    2
    4
*/

```

**Promise.all的实现**

```js
function promiseAll(arr) {
	return new Promise((resolve, reject)=>{
		let count = 0
		let result = []
		for(let i = 0; i< arr.length;i++) {
			Promise.resolve(arr[i]).then(res=> {
			  count++
				result[i] = res
				if(count == arr.length) {
				  resolve(result)
				}
			}).catch(err=>{
			  reject(err)
			})
		}
	})
}
```

**Promise.resolve**

```
Promise.resolve = function(value) {
    // 如果是 Promsie，则直接输出它
    if(value instanceof Promise){
        return value
    }
    return new Promise(resolve => resolve(value))
}
```

**Promise.reject**

```js
Promise.reject = function(reason) {
    return new Promise((resolve, reject) => reject(reason))
}
```

**Promise.race**

```js
Promise.race = function(promiseArr) {
    return new Promise((resolve, reject) => {
        promiseArr.forEach(p => {
            Promise.resolve(p).then(val => {
                resolve(val)
            }, err => {
                rejecte(err)
            })
        })
    })
}
```

**Promise.allSettled**

```
Promise.allSettled = function(promiseArr) {
    let result = []
        
    return new Promise((resolve, reject) => {
        promiseArr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                result.push({
                    status: 'fulfilled',
                    value: val
                })
                if (result.length === promiseArr.length) {
                    resolve(result) 
                }
            }, err => {
                result.push({
                    status: 'rejected',
                    reason: err
                })
                if (result.length === promiseArr.length) {
                    resolve(result) 
                }
            })
        })  
    })   
}
```

**Promise.any**

```
Promise.any = function(promiseArr) {
    let index = 0
    return new Promise((resolve, reject) => {
        if (promiseArr.length === 0) return 
        promiseArr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                resolve(val)
                
            }, err => {
                index++
                if (index === promiseArr.length) {
                  reject(new AggregateError('All promises were rejected'))
                }
            })
        })
    })
}
```

**并发控制**

> 巨量的图片资源，每次最多并发加载3个

```js
function limitLoad(urls, handler, limit){
		const sequence = [].concat(urls)
    let promises = []
    
    promises = sequence.splice(0, limit).map((url, index) => {
      return handler(url).then(() => {
        return index
      })
    })
  
  let p = Promise.race(promises)
  
  for(let i = 0;i<sequence.length;i++) {
    p = p.then(res=> {
      promises[res] = handler(sequence[i]).then(() => {
        return res
      })
      return Promise.race(promises)
    })
  }
}

const urls = [
	{
		info: 'x',
		time: 3000
	},
	{
		info: 'x2',
		time: 2000
	}
	...
]

function loadImg(url) {
	return new Promise((resolve, reject)=>{
		setTimeout(() => {
			resolve()
		}, url.time)
	})
}
limitLoad(urls, loadImg, 3)
```



