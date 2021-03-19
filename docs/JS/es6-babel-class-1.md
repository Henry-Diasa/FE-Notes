#### 前言

在了解 Babel 是如何编译 class 前，我们先看看 ES6 的 class 和 ES5 的构造函数是如何对应的。毕竟，ES6 的 class 可以看作一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

#### constructor

ES6 中：

```javascript
class Person {
    constructor(name) {
        this.name = name;
    }

    sayHello() {
        return 'hello, I am ' + this.name;
    }
}

var kevin = new Person('Kevin');
kevin.sayHello(); // hello, I am Kevin
```

对应到 ES5 中就是:

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.sayHello = function () {
    return 'hello, I am ' + this.name;
};

var kevin = new Person('Kevin');
kevin.sayHello(); // hello, I am Kevin
```

我们可以看到 ES5 的构造函数 Person，对应 ES6 的 Person 类的 constructor 方法。

值得注意的是：**类的内部所有定义的方法，都是不可枚举的（non-enumerable）**

以上面的例子为例，在 ES6 中：

```javascript
Object.keys(Person.prototype); // []
Object.getOwnPropertyNames(Person.prototype); // ["constructor", "sayHello"]
```

然而在 ES5 中：

```javascript
Object.keys(Person.prototype); // ['sayHello']
Object.getOwnPropertyNames(Person.prototype); // ["constructor", "sayHello"]
```

#### 实例属性

以前，我们定义实例属性，只能写在类的 constructor 方法里面。比如：

```javascript
class Person {
    constructor() {
        this.state = {
            count: 0
        };
    }
}
```

然而现在有一个提案，对实例属性和静态属性都规定了新的写法，而且 Babel 已经支持。现在我们可以写成：

```javascript
class Person {
    state = {
        count: 0
    };
}
```

对应到 ES5 都是：

```javascript
function Person() {
    this.state = {
        count: 0
    };
}
```

#### 静态方法

所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

ES6 中：

```javascript
class Person {
    static sayHello() {
        return 'hello';
    }
}

Person.sayHello() // 'hello'

var kevin = new Person();
kevin.sayHello(); // TypeError: kevin.sayHello is not a function
```

对应 ES5：

```javascript
function Person() {}

Person.sayHello = function() {
    return 'hello';
};

Person.sayHello(); // 'hello'

var kevin = new Person();
kevin.sayHello(); // TypeError: kevin.sayHello is not a function
```

#### 静态属性

静态属性指的是 Class 本身的属性，即 Class.propName，而不是定义在实例对象（this）上的属性。以前，我们添加静态属性只可以这样：

```javascript
class Person {}

Person.name = 'kevin';
```

因为上面提到的提案，现在可以写成：

```javascript
class Person {
  static name = 'kevin';
}
```

对应到 ES5 都是：

```javascript
function Person() {};

Person.name = 'kevin';
```

#### new调用

值得注意的是：类必须使用 new 调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用 new 也可以执行。

```javascript
class Person {}

Person(); // TypeError: Class constructor Foo cannot be invoked without 'new'
```

#### getter和setter

与 ES5 一样，在“类”的内部可以使用 get 和 set 关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class Person {
    get name() {
        return 'kevin';
    }
    set name(newName) {
        console.log('new name 为：' + newName)
    }
}

let person = new Person();

person.name = 'daisy';
// new name 为：daisy

console.log(person.name);
// kevin
```

对应到 ES5 中：

```javascript
function Person(name) {}

Person.prototype = {
    get name() {
        return 'kevin';
    },
    set name(newName) {
        console.log('new name 为：' + newName)
    }
}

let person = new Person();

person.name = 'daisy';
// new name 为：daisy

console.log(person.name);
// kevin
```

#### babel编译

至此，我们已经知道了有关“类”的方法中，ES6 与 ES5 是如何对应的，实际上 Babel 在编译时并不会直接就转成这种形式，Babel 会自己生成一些辅助函数，帮助实现 ES6 的特性。

我们可以在 Babel 官网的[ Try it out ](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-2&targets=&browsers=&builtIns=false&debug=false&code_lz=Q)页面查看 ES6 的代码编译成什么样子。

**编译（一）**

ES6 代码为：

```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
}
```

Babel 编译为：

```javascript
"use strict";

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Person = function Person(name) {
    _classCallCheck(this, Person);

    this.name = name;
};
```

_classCallCheck 的作用是检查 Person 是否是通过 new 的方式调用，在上面，我们也说过，类必须使用 new 调用，否则会报错。

当我们使用 `var person = Person()` 的形式调用的时候，this 指向 window，所以 `instance instanceof Constructor` 就会为 false，与 ES6 的要求一致。

**编译（二）**

ES6 代码为：

```javascript
class Person {
    // 实例属性
    foo = 'foo';
    // 静态属性
    static bar = 'bar';

    constructor(name) {
        this.name = name;
    }
}
```

Babel 编译为：

```javascript
'use strict';

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Person = function Person(name) {
    _classCallCheck(this, Person);

    this.foo = 'foo';

    this.name = name;
};

Person.bar = 'bar';
```

**编译（三）**

ES6 代码为：

```javascript
class Person {
    constructor(name) {
        this.name = name;
    }

    sayHello() {
        return 'hello, I am ' + this.name;
    }

    static onlySayHello() {
        return 'hello'
    }

    get name() {
        return 'kevin';
    }

    set name(newName) {
        console.log('new name 为：' + newName)
    }
}
```

对应到 ES5 的代码应该是：

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype =  {
    sayHello: function () {
        return 'hello, I am ' + this.name;
    },
    get name() {
        return 'kevin';
    },
    set name(newName) {
        console.log('new name 为：' + newName)
    }
}

Person.onlySayHello = function () {
    return 'hello'
};
```

Babel 编译后为：

```javascript
'use strict';

var _createClass = function() {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function(Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
}();

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Person = function() {
    function Person(name) {
        _classCallCheck(this, Person);

        this.name = name;
    }

    _createClass(Person, [{
        key: 'sayHello',
        value: function sayHello() {
            return 'hello, I am ' + this.name;
        }
    }, {
        key: 'name',
        get: function get() {
            return 'kevin';
        },
        set: function set(newName) {
            console.log('new name 为：' + newName);
        }
    }], [{
        key: 'onlySayHello',
        value: function onlySayHello() {
            return 'hello';
        }
    }]);

    return Person;
}();
```

我们可以看到 Babel 生成了一个 _createClass 辅助函数，该函数传入三个参数，第一个是构造函数，在这个例子中也就是 Person，第二个是要添加到原型上的函数数组，第三个是要添加到构造函数本身的函数数组，也就是所有添加 static 关键字的函数。该函数的作用就是将函数数组中的方法添加到构造函数或者构造函数的原型中，最后返回这个构造函数。

在其中，又生成了一个 defineProperties 辅助函数，使用 Object.defineProperty 方法添加属性。

默认 enumerable 为 false，configurable 为 true，这个在上面也有强调过，是为了防止 Object.keys() 之类的方法遍历到。然后通过判断 value 是否存在，来判断是否是 getter 和 setter。如果存在 value，就为 descriptor 添加 value 和 writable 属性，如果不存在，就直接使用 get 和 set 属性。