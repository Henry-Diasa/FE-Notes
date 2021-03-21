#### Decorator

装饰器主要用于:

1. 装饰类
2. 装饰方法或属性

#### 装饰类

```javascript
@annotation
class MyClass { }

function annotation(target) {
   target.annotated = true;
}
```

#### 装饰方法和属性

```javascript
class MyClass {
  @readonly
  method() { }
}

function readonly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}
```

#### Babel

**安装编译**

我们可以在 Babel 官网的 [Try it out](https://babeljs.io/repl/build/8875/#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=GYVwdgxgLglg9mABAZzgWwKYBEMTgJwEMoCAKAawBtDlkAaRcjATwYBMNkIBKRAbwBQiRPgxQQ-JBy4BuAQF8BAqPmb8hiCNVqIAYnDjrhwgAKpMOPERL4NwgOZiU6DAEEIETqnylefRcKK8prEEAAWiKQY-Ph-GnhgqJQYAHTR-GTp3HLyQA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=true&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=es2015%2Creact%2Cstage-2&prettier=false&targets=&version=7.0.0-rc.1%2Bpr.8505&envVersion=)，查看 Babel 编译后的代码。

不过我们也可以选择本地编译：

```shell
npm init

npm install --save-dev @babel/core @babel/cli

npm install --save-dev @babel/plugin-proposal-decorators @babel/plugin-proposal-class-properties
```

新建 .babelrc 文件

```shell
{
  "plugins": [
    ["@babel/plugin-proposal-decorators", { "legacy": true }],
    ["@babel/plugin-proposal-class-properties", {"loose": true}]
  ]
}
```

再编译指定的文件

```shell
babel decorator.js --out-file decorator-compiled.js
```

**装饰类的编译**

编译前：

```javascript
@annotation
class MyClass { }

function annotation(target) {
   target.annotated = true;
}
```

编译后：

```javascript
var _class;

let MyClass = annotation(_class = class MyClass {}) || _class;

function annotation(target) {
  target.annotated = true;
}
```

我们可以看到对于类的装饰，其原理就是：

```javascript
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```

**装饰方法的编译**

编译前：

```javascript
class MyClass {
  @unenumerable
  @readonly
  method() { }
}

function readonly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

function unenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
```

编译后：

```javascript
var _class;

function _applyDecoratedDescriptor(target, property, decorators, descriptor, context ) {
	/**
	 * 第一部分
	 * 拷贝属性
	 */
	var desc = {};
	Object["ke" + "ys"](descriptor).forEach(function(key) {
		desc[key] = descriptor[key];
	});
	desc.enumerable = !!desc.enumerable;
	desc.configurable = !!desc.configurable;

	if ("value" in desc || desc.initializer) {
		desc.writable = true;
	}

	/**
	 * 第二部分
	 * 应用多个 decorators
	 */
	desc = decorators
		.slice()
		.reverse()
		.reduce(function(desc, decorator) {
			return decorator(target, property, desc) || desc;
		}, desc);

	/**
	 * 第三部分
	 * 设置要 decorators 的属性
	 */
	if (context && desc.initializer !== void 0) {
		desc.value = desc.initializer ? desc.initializer.call(context) : void 0;
		desc.initializer = undefined;
	}

	if (desc.initializer === void 0) {
		Object["define" + "Property"](target, property, desc);
		desc = null;
	}

	return desc;
}

let MyClass = ((_class = class MyClass {
	method() {}
}),
_applyDecoratedDescriptor(
	_class.prototype,
	"method",
	[readonly],
	Object.getOwnPropertyDescriptor(_class.prototype, "method"),
	_class.prototype
),
_class);

function readonly(target, name, descriptor) {
	descriptor.writable = false;
	return descriptor;
}
```

#### 装饰方法的编译源码解析

我们可以看到 Babel 构建了一个 _applyDecoratedDescriptor 函数，用于给方法装饰。

**Object.getOwnPropertyDescriptor()**

在传入参数的时候，我们使用了一个 Object.getOwnPropertyDescriptor() 方法，我们来看下这个方法：

> Object.getOwnPropertyDescriptor() 方法返回指定对象上的一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）

顺便注意这是一个 ES5 的方法。

举个例子：

```javascript
const foo = { value: 1 };
const bar = Object.getOwnPropertyDescriptor(foo, "value");
// bar {
//   value: 1,
//   writable: true
//   enumerable: true,
//   configurable: true,
// }

const foo = { get value() { return 1; } };
const bar = Object.getOwnPropertyDescriptor(foo, "value");
// bar {
//   get: /*the getter function*/,
//   set: undefined
//   enumerable: true,
//   configurable: true,
// }
```

**第一部分源码解析**

在 _applyDecoratedDescriptor 函数内部，我们首先将 Object.getOwnPropertyDescriptor() 返回的属性描述符对象做了一份拷贝：

```javascript
// 拷贝一份 descriptor
var desc = {};
Object["ke" + "ys"](descriptor).forEach(function(key) {
	desc[key] = descriptor[key];
});
desc.enumerable = !!desc.enumerable;
desc.configurable = !!desc.configurable;

// 如果没有 value 属性或者没有 initializer 属性，表明是 getter 和 setter
if ("value" in desc || desc.initializer) {
	desc.writable = true;
}
```

那么 initializer 属性是什么呢？Object.getOwnPropertyDescriptor() 返回的对象并不具有这个属性呀，确实，这是 Babel 的 Class 为了与 decorator 配合而产生的一个属性，比如说对于下面这种代码：

```javascript
class MyClass {
  @readonly
  born = Date.now();
}

function readonly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

var foo = new MyClass();
console.log(foo.born);
```

Babel 就会编译为：

```javascript
// ...
(_descriptor = _applyDecoratedDescriptor(_class.prototype, "born", [readonly], {
	configurable: true,
	enumerable: true,
	writable: true,
	initializer: function() {
		return Date.now();
	}
}))
// ...
```

此时传入 _applyDecoratedDescriptor 函数的 descriptor 就具有 initializer 属性。

**第二部分源码解析**

接下是应用多个 decorators：

```javascript
/**
 * 第二部分
 * @type {[type]}
 */
desc = decorators
	.slice()
	.reverse()
	.reduce(function(desc, decorator) {
		return decorator(target, property, desc) || desc;
	}, desc);
```

对于一个方法应用了多个 decorator，比如：

```javascript
class MyClass {
  @unenumerable
  @readonly
  method() { }
}
```

Babel 会编译为：

```javascript
_applyDecoratedDescriptor(
	_class.prototype,
	"method",
	[unenumerable, readonly],
	Object.getOwnPropertyDescriptor(_class.prototype, "method"),
	_class.prototype
)
```

在第二部分的源码中，执行了 reverse() 和 reduce() 操作，由此我们也可以发现，如果同一个方法有多个装饰器，会由内向外执行。

**第三部分源码解析**

```
/**
 * 第三部分
 * 设置要 decorators 的属性
 */
if (context && desc.initializer !== void 0) {
	desc.value = desc.initializer ? desc.initializer.call(context) : void 0;
	desc.initializer = undefined;
}

if (desc.initializer === void 0) {
	Object["define" + "Property"](target, property, desc);
	desc = null;
}

return desc;
```

如果 desc 有 initializer 属性，意味着当装饰的是类的属性时，会将 value 的值设置为:

```
desc.initializer.call(context)
```

而 context 的值为 `_class.prototype`，之所以要 `call(context)`，这也很好理解，因为有可能

```
class MyClass {
  @readonly
  value = this.getNum() + 1;

  getNum() {
    return 1;
  }
}
```

最后无论是装饰方法还是属性，都会执行：

```
Object["define" + "Property"](target, property, desc);
```

由此可见，装饰方法本质上还是使用 `Object.defineProperty()` 来实现的。

#### 应用

**1.log**

为一个方法添加 log 函数，检查输入的参数：

```
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function(...args) {
    console.log(`Calling ${name} with`, args);
    return oldValue.apply(this, args);
  };

  return descriptor;
}

const math = new Math();

// Calling add with [2, 4]
math.add(2, 4);
```

再完善点：

```
let log = (type) => {
  return (target, name, descriptor) => {
    const method = descriptor.value;
    descriptor.value =  (...args) => {
      console.info(`(${type}) 正在执行: ${name}(${args}) = ?`);
      let ret;
      try {
        ret = method.apply(target, args);
        console.info(`(${type}) 成功 : ${name}(${args}) => ${ret}`);
      } catch (error) {
        console.error(`(${type}) 失败: ${name}(${args}) => ${error}`);
      }
      return ret;
    }
  }
};
```

**2.autobind**

```
class Person {
  @autobind
  getPerson() {
  	return this;
  }
}

let person = new Person();
let { getPerson } = person;

getPerson() === person;
// true
```

我们很容易想到的一个场景是 React 绑定事件的时候：

```
class Toggle extends React.Component {

  @autobind
  handleClick() {
	  console.log(this)
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        button
      </button>
    );
  }
}
```

我们来写这样一个 autobind 函数：

```
const { defineProperty, getPrototypeOf} = Object;

function bind(fn, context) {
  if (fn.bind) {
    return fn.bind(context);
  } else {
    return function __autobind__() {
      return fn.apply(context, arguments);
    };
  }
}

function createDefaultSetter(key) {
  return function set(newValue) {
    Object.defineProperty(this, key, {
      configurable: true,
      writable: true,
      enumerable: true,
      value: newValue
    });

    return newValue;
  };
}

function autobind(target, key, { value: fn, configurable, enumerable }) {
  if (typeof fn !== 'function') {
    throw new SyntaxError(`@autobind can only be used on functions, not: ${fn}`);
  }

  const { constructor } = target;

  return {
    configurable,
    enumerable,

    get() {

      /**
       * 使用这种方式相当于替换了这个函数，所以当比如
       * Class.prototype.hasOwnProperty(key) 的时候，为了正确返回
       * 所以这里做了 this 的判断
       */
      if (this === target) {
        return fn;
      }

      const boundFn = bind(fn, this);

      defineProperty(this, key, {
        configurable: true,
        writable: true,
        enumerable: false,
        value: boundFn
      });

      return boundFn;
    },
    set: createDefaultSetter(key)
  };
}
```

**3.debounce**

有的时候，我们需要对执行的方法进行防抖处理:

```
class Toggle extends React.Component {

  @debounce(500, true)
  handleClick() {
    console.log('toggle')
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        button
      </button>
    );
  }
}
```

我们来实现一下：

```
function _debounce(func, wait, immediate) {

    var timeout;

    return function () {
        var context = this;
        var args = arguments;

        if (timeout) clearTimeout(timeout);
        if (immediate) {
            var callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            }, wait)
            if (callNow) func.apply(context, args)
        }
        else {
            timeout = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
    }
}

function debounce(wait, immediate) {
  return function handleDescriptor(target, key, descriptor) {
    const callback = descriptor.value;

    if (typeof callback !== 'function') {
      throw new SyntaxError('Only functions can be debounced');
    }

    var fn = _debounce(callback, wait, immediate)

    return {
      ...descriptor,
      value() {
        fn()
      }
    };
  }
}
```

**4.time**

用于统计方法执行的时间:

```
function time(prefix) {
  let count = 0;
  return function handleDescriptor(target, key, descriptor) {

    const fn = descriptor.value;

    if (prefix == null) {
      prefix = `${target.constructor.name}.${key}`;
    }

    if (typeof fn !== 'function') {
      throw new SyntaxError(`@time can only be used on functions, not: ${fn}`);
    }

    return {
      ...descriptor,
      value() {
        const label = `${prefix}-${count}`;
        count++;
        console.time(label);

        try {
          return fn.apply(this, arguments);
        } finally {
          console.timeEnd(label);
        }
      }
    }
  }
}
```

**5.mixin**

用于将对象的方法混入 Class 中：

```
const SingerMixin = {
  sing(sound) {
    alert(sound);
  }
};

const FlyMixin = {
  // All types of property descriptors are supported
  get speed() {},
  fly() {},
  land() {}
};

@mixin(SingerMixin, FlyMixin)
class Bird {
  singMatingCall() {
    this.sing('tweet tweet');
  }
}

var bird = new Bird();
bird.singMatingCall();
// alerts "tweet tweet"
```

mixin 的一个简单实现如下：

```
function mixin(...mixins) {
  return target => {
    if (!mixins.length) {
      throw new SyntaxError(`@mixin() class ${target.name} requires at least one mixin as an argument`);
    }

    for (let i = 0, l = mixins.length; i < l; i++) {
      const descs = Object.getOwnPropertyDescriptors(mixins[i]);
      const keys = Object.getOwnPropertyNames(descs);

      for (let j = 0, k = keys.length; j < k; j++) {
        const key = keys[j];

        if (!target.prototype.hasOwnProperty(key)) {
          Object.defineProperty(target.prototype, key, descs[key]);
        }
      }
    }
  };
}
```

**6.redux**

实际开发中，React 与 Redux 库结合使用时，常常需要写成下面这样。

```
class MyReactComponent extends React.Component {}

export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);
```

有了装饰器，就可以改写上面的代码。

```
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {};
```

相对来说，后一种写法看上去更容易理解。

**7.注意**

以上我们都是用于修饰类方法，我们获取值的方式为：

```
const method = descriptor.value;
```

但是如果我们修饰的是类的实例属性，因为 Babel 的缘故，通过 value 属性并不能获取值，我们可以写成：

```
const value = descriptor.initializer && descriptor.initializer();
```

#### 参考

1. [ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/decorator)
2. [core-decorators](https://github.com/jayphelps/core-decorators#mixin-alias-mixins)
3. [ES7 Decorator 装饰者模式](http://taobaofed.org/blog/2015/11/16/es7-decorator/)
4. [JS 装饰器（Decorator）场景实战](https://juejin.im/post/59f1c484f265da431c6f8940#heading-2)

