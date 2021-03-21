#### 前言

在上一篇 《 ES6 系列 Babel 是如何编译 Class 的(上)》，我们知道了 Babel 是如何编译 Class 的，这篇我们学习 Babel 是如何用 ES5 实现 Class 的继承。

#### ES5寄生组合式继承

```javascript
function Parent (name) {
    this.name = name;
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

Child.prototype = Object.create(Parent.prototype);

var child1 = new Child('kevin', '18');

console.log(child1);
```

原型链示意图为：

[![寄生组合式继承原型链示意图](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/class/es5-prototype.png)](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/class/es5-prototype.png)

关于寄生组合式继承我们在 《JavaScript深入之继承的多种方式和优缺点》中介绍过。

引用《JavaScript高级程序设计》中对寄生组合式继承的夸赞就是：

> 这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

#### ES6 extend

Class 通过 extends 关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

以上 ES5 的代码对应到 ES6 就是：

```javascript
class Parent {
    constructor(name) {
        this.name = name;
    }
}

class Child extends Parent {
    constructor(name, age) {
        super(name); // 调用父类的 constructor(name)
        this.age = age;
    }
}

var child1 = new Child('kevin', '18');

console.log(child1);
```

值得注意的是：

super 关键字表示父类的构造函数，相当于 ES5 的 Parent.call(this)。

子类必须在 constructor 方法中调用 super 方法，否则新建实例时会报错。这是因为子类没有自己的 this 对象，而是继承父类的 this 对象，然后对其进行加工。如果不调用 super 方法，子类就得不到 this 对象。

也正是因为这个原因，在子类的构造函数中，只有调用 super 之后，才可以使用 this 关键字，否则会报错。

#### 子类的`__proto__`

在 ES6 中，父类的静态方法，可以被子类继承。举个例子：

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod(); // 'hello'
```

这是因为 Class 作为构造函数的语法糖，同时有 prototype 属性和 __proto__ 属性，因此同时存在两条继承链。

（1）子类的 __proto__ 属性，表示构造函数的继承，总是指向父类。

（2）子类 prototype 属性的 __proto__ 属性，表示方法的继承，总是指向父类的 prototype 属性。

```javascript
class Parent {
}

class Child extends Parent {
}

console.log(Child.__proto__ === Parent); // true
console.log(Child.prototype.__proto__ === Parent.prototype); // true
```

ES6 的原型链示意图为：

[![ES6 class 原型链示意图](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/class/class-prototype.png)](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/class/class-prototype.png)

我们会发现，相比寄生组合式继承，ES6 的 class 多了一个 `Object.setPrototypeOf(Child, Parent)` 的步骤。

#### 继承目标

extends 关键字后面可以跟多种类型的值。

```javascript
class B extends A {
}
```

上面代码的 A，只要是一个有 prototype 属性的函数，就能被 B 继承。由于函数都有 prototype 属性（除了 Function.prototype 函数），因此 A 可以是任意函数。

除了函数之外，A 的值还可以是 null，当 `extend null` 的时候：

```javascript
class A extends null {
}

console.log(A.__proto__ === Function.prototype); // true
console.log(A.prototype.__proto__ === undefined); // true
```

#### Babel编译

那 ES6 的这段代码:

```javascript
class Parent {
    constructor(name) {
        this.name = name;
    }
}

class Child extends Parent {
    constructor(name, age) {
        super(name); // 调用父类的 constructor(name)
        this.age = age;
    }
}

var child1 = new Child('kevin', '18');

console.log(child1);
```

Babel 又是如何编译的呢？我们可以在 Babel 官网的[ Try it out ](https://babeljs.io/repl)中尝试：

```javascript
'use strict';

function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

function _inherits(subClass, superClass) {
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, { constructor: { value: subClass, enumerable: false, writable: true, configurable: true } });
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Parent = function Parent(name) {
    _classCallCheck(this, Parent);

    this.name = name;
};

var Child = function(_Parent) {
    _inherits(Child, _Parent);

    function Child(name, age) {
        _classCallCheck(this, Child);

        // 调用父类的 constructor(name)
        var _this = _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name));

        _this.age = age;
        return _this;
    }

    return Child;
}(Parent);

var child1 = new Child('kevin', '18');

console.log(child1);
```

我们可以看到 Babel 创建了 _inherits 函数帮助实现继承，又创建了 _possibleConstructorReturn 函数帮助确定调用父类构造函数的返回值，我们来细致的看一看代码。

#### _inherits

```javascript
function _inherits(subClass, superClass) {
    // extend 的继承目标必须是函数或者是 null
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }

    // 类似于 ES5 的寄生组合式继承，使用 Object.create，设置子类 prototype 属性的 __proto__ 属性指向父类的 prototype 属性
    subClass.prototype = Object.create(superClass && superClass.prototype, { constructor: { value: subClass, enumerable: false, writable: true, configurable: true } });

    // 设置子类的 __proto__ 属性指向父类
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}
```

关于 Object.create()，一般我们用的时候会传入一个参数，其实是支持传入两个参数的，第二个参数表示要添加到新创建对象的属性，注意这里是给新创建的对象即返回值添加属性，而不是在新创建对象的原型对象上添加。

举个例子：

```javascript
// 创建一个以另一个空对象为原型,且拥有一个属性 p 的对象
const o = Object.create({}, { p: { value: 42 } });
console.log(o); // {p: 42}
console.log(o.p); // 42
```

再完整一点：

```javascript
const o = Object.create({}, {
    p: {
        value: 42,
        enumerable: false,
        // 该属性不可写
        writable: false,
        configurable: true
    }
});
o.p = 24;
console.log(o.p); // 42
```

那么对于这段代码：

```javascript
subClass.prototype = Object.create(superClass && superClass.prototype, { constructor: { value: subClass, enumerable: false, writable: true, configurable: true } });
```

作用就是给 subClass.prototype 添加一个可配置可写不可枚举的 constructor 属性，该属性值为 subClass。

#### _possibleConstructorReturn

函数里是这样调用的：

```javascript
var _this = _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name));
```

我们简化为：

```javascript
var _this = _possibleConstructorReturn(this, Parent.call(this, name));
```

`_possibleConstructorReturn` 的源码为：

```javascript
function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}
```

在这里我们判断 `Parent.call(this, name)` 的返回值的类型，咦？这个值还能有很多类型？

对于这样一个 class：

```javascript
class Parent {
    constructor() {
        this.xxx = xxx;
    }
}
```

Parent.call(this, name) 的值肯定是 undefined。可是如果我们在 constructor 函数中 return 了呢？比如：

```javascript
class Parent {
    constructor() {
        return {
            name: 'kevin'
        }
    }
}
```

我们可以返回各种类型的值，甚至是 null：

```javascript
class Parent {
    constructor() {
        return null
    }
}
```

我们接着看这个判断：

```javascript
call && (typeof call === "object" || typeof call === "function") ? call : self;
```

注意，这句话的意思并不是判断 call 是否存在，如果存在，就执行 `(typeof call === "object" || typeof call === "function") ? call : self`

因为 `&&` 的运算符优先级高于 `? :`，所以这句话的意思应该是：

```javascript
(call && (typeof call === "object" || typeof call === "function")) ? call : self;
```

对于 Parent.call(this) 的值，如果是 object 类型或者是 function 类型，就返回 Parent.call(this)，如果是 null 或者基本类型的值或者是 undefined，都会返回 self 也就是子类的 this。

这也是为什么这个函数被命名为 `_possibleConstructorReturn`。

#### 总结

```javascript
var Child = function(_Parent) {
    _inherits(Child, _Parent);

    function Child(name, age) {
        _classCallCheck(this, Child);

        // 调用父类的 constructor(name)
        var _this = _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name));

        _this.age = age;
        return _this;
    }

    return Child;
}(Parent);
```

最后我们总体看下如何实现继承：

首先执行 `_inherits(Child, Parent)`，建立 Child 和 Parent 的原型链关系，即 `Object.setPrototypeOf(Child.prototype, Parent.prototype)` 和 `Object.setPrototypeOf(Child, Parent)`。

然后调用 `Parent.call(this, name)`，根据 Parent 构造函数的返回值类型确定子类构造函数 this 的初始值 _this。

最终，根据子类构造函数，修改 _this 的值，然后返回该值。

