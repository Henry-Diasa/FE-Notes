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

