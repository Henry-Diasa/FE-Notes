

### 从原型到原型链

#### 构造函数创建对象
> Person为构造函数，通过new创建了一个实例对象
```javascript
    function Person() {

    }
    var person = new Person();
    person.name = 'Kevin';
    console.log(person.name) // Kevin
```

#### Prototype
> 每个`函数`都有一个`prototype`属性

```javascript
    function Person() {

    }
    // 虽然写在注释里，但是你要注意：
    // prototype是函数才会有的属性
    Person.prototype.name = 'Kevin';
    var person1 = new Person();
    var person2 = new Person();
    console.log(person1.name) // Kevin
    console.log(person2.name) // Kevin
```

`prototype属性指向什么呢?`

> 函数的 `prototype` 属性指向了一个`对象`，这个对象正是调用该构造函数而创建的`实例的原型`，也就是这个例子中的 person1 和 person2 的原型。

`什么是原型呢？`

> 每一个`JavaScript对象`(null除外)在创建的时候就会与之关联`另一个对象`，这个对象就是我们所说的`原型`，每一个对象都会从原型`"继承"`属性。


<img src="img/JS/prototype1.png" />

#### `__proto__`

> 这是每一个`JavaScript`对象(除了 null )都具有的一个属性，叫`__proto__`，这个属性会指向该对象的`原型`。

```javascript
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```


<img src="img/JS/prototype2.png" />

既然实例对象和构造函数都可以指向原型，那么原型是否有属性指向构造函数或者实例呢？

#### constructor

> 每个原型都有一个 constructor 属性指向关联的构造函数。

```javascript
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```


<img src="img/JS/prototype3.png" />

综上得出

```javascript
function Person() {

}

var person = new Person();

console.log(person.__proto__ == Person.prototype) // true
console.log(Person.prototype.constructor == Person) // true
// 顺便学习一个ES5的方法,可以获得对象的原型
console.log(Object.getPrototypeOf(person) === Person.prototype) // true
```

#### 实例与原型

> 当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一直找到最顶层为止。

```javascript
function Person() {

}

Person.prototype.name = 'Kevin';

var person = new Person();

person.name = 'Daisy';
console.log(person.name) // Daisy 首先从实例当中寻找

delete person.name;
console.log(person.name) // Kevin 实例没有  通过__proto__去Person.prototype中寻找
```

#### 原型的原型

> 我们已经讲了原型也是一个对象，既然是对象，我们就可以用最原始的方式创建它

```javascript
var obj = new Object();
obj.name = 'Kevin'
console.log(obj.name) // Kevin
```

其实`原型对象`就是通过 `Object 构造函数`生成的，结合之前所讲，实例的 `__proto__` 指向构造函数的 prototype ，所以我们再更新下关系图


<img src="img/JS/prototype4.png" />

#### 原型链

那 `Object.prototype `的原型呢？

`null`，我们可以打印：

> null 表示“没有对象”，即该处不应该有值。

```javascript
console.log(Object.prototype.__proto__ === null) // true
```

所以 `Object.prototype.__proto__ `的值为 null 跟 Object.prototype 没有原型，其实表达了一个意思


<img src="img/JS/prototype5.png" />

图中由相互关联的原型组成的链状结构就是原型链，也就是蓝色的这条线

#### 补充

- constructor

  ```javascript
  function Person() {
  
  }
  var person = new Person();
  console.log(person.constructor === Person); // true
  ```

  当获取 person.constructor 时，其实 person 中并没有 constructor 属性,当不能读取到constructor 属性时，会从 person 的原型也就是 Person.prototype 中读取，正好原型中有该属性，所以：

  ```javascript 
  person.constructor === Person.prototype.constructor // true
  ```

- `__proto__`

  > 其次是` __proto__` ，绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于 Person.prototype 中，实际上，它是来自于 Object.prototype ，与其说是一个属性，不如说是一个 getter/setter，当使用 obj.`__proto__` 时，可以理解成返回了 Object.getPrototypeOf(obj)。

- 真的是继承吗

  > 最后是关于继承，前面我们讲到“每一个对象都会从原型‘继承’属性”，实际上，继承是一个十分具有迷惑性的说法，引用《你不知道的JavaScript》中的话，就是：
  >
  > 继承意味着复制操作，然而 JavaScript 默认并不会复制对象的属性，相反，JavaScript 只是在两个对象之间创建一个关联，这样，一个对象就可以通过委托访问另一个对象的属性和函数，所以与其叫继承，委托的说法反而更准确些。

### 词法作用域和动态作用域

#### 作用域

> 作用域是指程序源代码中`定义变量的区域`。
>
> 作用域规定了如何`查找变量`，也就是确定当前执行代码对变量的`访问权限`。
>
> JavaScript 采用`词法作用域`(lexical scoping)，也就是`静态作用域`。

#### 静态作用域与动态作用域

> 因为 JavaScript 采用的是`词法作用域`，函数的作用域在`函数定义的时候就决定了`。
>
> 而与词法作用域相对的是`动态作用域`，函数的作用域是在`函数调用的时候才决定的`。

```javascript
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();  // 1
```

假设JavaScript采用静态作用域，让我们分析下执行过程：

执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。

假设JavaScript采用动态作用域，让我们分析下执行过程：

执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

前面我们已经说了，JavaScript采用的是静态作用域，所以这个例子的结果是 1。

#### 动态作用域

> bash 就是动态作用域，不信的话，把下面的脚本存成例如 scope.bash，然后进入相应的目录，用命令行执行 `bash ./scope.bash`

```javascript
value=1
function foo () {
    echo $value;
}
function bar () {
    local value=2;
    foo;
}
bar
```

#### 思考题

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

两段代码都会打印：`local scope`。

原因也很简单，因为JavaScript采用的是词法作用域，函数的作用域基于`函数创建的位置`

> 引用《JavaScript权威指南》的回答就是：
>
> JavaScript 函数的执行用到了作用域链，这个作用域链是在函数定义的时候创建的。嵌套的函数 f() 定义在这个作用域链里，其中的变量 scope 一定是局部变量，不管何时何地执行函数 f()，这种绑定在执行 f() 时依然有效。

虽然两段代码执行的结果一样，但是两段代码究竟有哪些不同呢？



### 执行上下文栈

#### 顺序执行

> 如果要问到 JavaScript 代码执行顺序的话，想必写过 JavaScript 的开发者都会有个直观的印象，那就是顺序执行

```javascript
var foo = function () {

    console.log('foo1');

}

foo();  // foo1

var foo = function () {

    console.log('foo2');

}

foo(); // foo2
```

```javascript
function foo() {

    console.log('foo1');

}

foo();  // foo2

function foo() {

    console.log('foo2');

}

foo(); // foo2
```

JavaScript 引擎并非一行一行地分析和执行程序，而是一段一段地分析执行。当执行一段代码的时候，会进行一个“准备工作”，比如第一个例子中的`变量提升`，和第二个例子中的`函数提升`

到底JavaScript引擎遇到一段怎样的代码时才会做“准备工作”呢？

#### 可执行代码

> 这就要说到 JavaScript 的可执行代码(executable code)的类型有哪些了？
>
> 其实很简单，就三种，全局代码、函数代码、eval代码。
>
> 举个例子，当执行到一个函数的时候，就会进行准备工作，这里的“准备工作”，让我们用个更专业一点的说法，就叫做"执行上下文(execution context)"。

#### 执行上下文栈

> 我们写的函数多了去了，如何管理创建的那么多执行上下文呢？
>
> 所以 JavaScript 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文
>
> 为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：

```javascript
ECStack = [];
```

试想当 JavaScript 开始要解释执行代码的时候，最先遇到的就是全局代码，所以初始化的时候首先就会向执行上下文栈压入一个全局执行上下文，我们用 globalContext 表示它，并且只有当整个应用程序结束的时候，ECStack 才会被清空，所以程序结束之前， ECStack 最底部永远有个 globalContext：

```javascript
ECStack = [
    globalContext
];
```

现在 JavaScript 遇到下面的这段代码了

```javascript
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出

```javascript
// 伪代码

// fun1()
ECStack.push(<fun1> functionContext);

// fun1中竟然调用了fun2，还要创建fun2的执行上下文
ECStack.push(<fun2> functionContext);

// 擦，fun2还调用了fun3！
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
```

#### 上一节作用域的思考题

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

两段代码执行的结果一样，但是他们的`执行上下文栈的变化`不一样

模拟第一段代码

```javascript
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```

模拟第二段代码

```javascript
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```

### 变量对象

#### 前言 

> 当 JavaScript 代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)

对于每个执行上下文，都有三个重要属性

- 变量对象（Variable object, VO）
- 作用域链（Scope chain）
- this

#### 变量对象

变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。

因为不同执行上下文下的变量对象稍有不同，所以我们来聊聊全局上下文下的变量对象和函数上下文下的变量对象。

#### 全局上下文

> 全局对象是预定义的对象，作为 JavaScript 的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。

> 在顶层 JavaScript 代码中，可以用关键字 this 引用全局对象。因为全局对象是作用域链的头，这意味着所有非限定性的变量和函数名都会作为该对象的属性来查询。

> 例如，当JavaScript 代码引用 parseInt() 函数时，它引用的是全局对象的 parseInt 属性。全局对象是作用域链的头，还意味着在顶层 JavaScript 代码中声明的所有变量都将成为全局对象的属性。

1、可以通过` this `引用，在客户端 `JavaScript` 中，全局对象就是 `Window` 对象。

```javascript
console.log(this);
```

2、全局对象是由` Object 构造函数`实例化的一个对象。

```javascript
console.log(this instanceof Object);
```

3、预定义了一堆函数和属性

```javascript
// 都能生效
console.log(Math.random());
console.log(this.Math.random());
```

4、作为全局变量的宿主

```javascript
var a = 1;
console.log(this.a);
```

5、客户端 JavaScript 中，全局对象有 window 属性指向自身

```javascript
var a = 1;
console.log(window.a);

this.window.b = 2;
console.log(this.b);
```

全局上下文中的`变量对象`就是`全局对象`

#### 函数上下文

在函数上下文中，我们用`活动对象(activation object, AO)`来表示变量对象。

活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎实现上的，不可在 JavaScript 环境中访问，只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 activation object 呐，而只有被激活的变量对象，也就是活动对象上的各种属性才能被访问。

活动对象是在进入函数上下文时刻被创建的，它通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。

#### 执行过程

执行上下文的代码会分成两个阶段进行处理：分析和执行，我们也可以叫做：

1、进入执行上下文

2、代码执行

`进入执行上下文`

当进入执行上下文时，这时候还没有执行代码，

变量对象会包括：

1、函数的所有形参 (如果是函数上下文)

- 由名称和对应值组成的一个变量对象的属性被创建
- 没有实参，属性值设为 undefined

2、函数声明

- 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建
- 如果变量对象已经存在相同名称的属性，则完全替换这个属性

3、变量声明

- 由名称和对应值（undefined）组成一个变量对象的属性被创建；
- 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

举个🌰

```javascript
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};

  b = 3;

}

foo(1);
```

在进入执行上下文后，这时候的AO是:

```javascript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    c: reference to function c(){},
    d: undefined
}
```

`代码执行`

在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值

还是上面的例子，当代码执行完后，这时候的 AO 是：

```
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
}
```

简洁的总结

- `全局上下文`的变量对象初始化是`全局对象`
- `函数上下文`的变量对象初始化`只包括 Arguments 对象`
- 在进入执行上下文时会给变量对象添加`形参`、`函数声明`、`变量声明`等初始的属性值
- 在代码执行阶段，会再次`修改变量对象的属性值`

#### 思考题

1.第一题

```javascript
function foo() {
    console.log(a);
    a = 1;
}

foo(); // Uncaught ReferenceError: a is not defined。

function bar() {
    a = 1;
    console.log(a);
}
bar(); // 1
```

这是因为函数中的 "a" 并没有通过 var 关键字声明，所有不会被存放在 AO 中。

第一段执行 console 的时候， AO 的值是：

```javascript
AO = {
    arguments: {
        length: 0
    }
}
```

没有 a 的值，然后就会到全局去找，全局也没有，所以会报错。

当第二段执行 console 的时候，全局对象已经被赋予了 a 属性，这时候就可以从全局找到 a 的值，所以会打印 1。

2.第二题

```javascript
console.log(foo);

function foo(){
    console.log("foo");
}

var foo = 1;
```

会打印函数，而不是 undefined 。

这是因为在进入执行上下文时，首先会处理函数声明，其次会处理变量声明，如果如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。

### 作用域链

#### 前言

上节中我们知道每个执行上下文，都会有三个重要属性

- 变量对象(Variable object，VO)
- 作用域链(Scope chain)
- this

这节重点是作用域链

#### 作用域链

在变量对象中讲到，当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做`作用域链`。

下面，让我们以一个函数`创建`和`激活`两个时期来讲解作用域链是如何创建和变化的

#### 函数创建

在词法作用域和动态作用域中讲到，函数的作用域在函数定义的时候就决定了。

这是因为函数有一个内部属性` [[scope]]`，当函数`创建`的时候，就会保存所有`父变量对象`到其中，你可以理解 [[scope]] 就是所有父变量对象的层级链，但是注意：[[scope]] 并不代表完整的作用域链！

举个🌰

```javascript
function foo() {
    function bar() {
        ...
    }
}
```

函数创建时，各自的[[scope]]为

```javascript
foo.[[scope]] = [
  globalContext.VO
];

bar.[[scope]] = [
    fooContext.AO,
    globalContext.VO
];
```

#### 函数激活

当函数激活时，进入函数上下文，创建 VO/AO 后，就会将`活动对象`添加到作用链的`前端`。

这时候执行上下文的作用域链，我们命名为 Scope：

```javascript
Scope = [AO].concat([[Scope]]);
```

至此，作用域链创建完毕

#### 捋一捋

以下面的例子为例，结合着之前讲的变量对象和执行上下文栈，我们来总结一下函数执行上下文中作用域链和变量对象的创建过程：

```javascript
var scope = "global scope";
function checkscope(){
    var scope2 = 'local scope';
    return scope2;
}
checkscope();
```

执行过程如下：

1.checkscope 函数被创建，保存作用域链到 内部属性[[scope]]

```javascript
checkscope.[[scope]] = [
    globalContext.VO
];
```

2.执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈

```javascript
ECStack = [
    checkscopeContext,
    globalContext
];
```

3.checkscope 函数并不立刻执行，开始做准备工作，第一步：复制函数[[scope]]属性创建作用域链

```javascript
checkscopeContext = {
    Scope: checkscope.[[scope]],
}
```

4.第二步：用 arguments 创建活动对象，随后初始化活动对象，加入形参、函数声明、变量声明

```javascript
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: undefined
    }，
    Scope: checkscope.[[scope]],
}
```

5.第三步：将活动对象压入 checkscope 作用域链顶端

```javascript
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: undefined
    },
    Scope: [AO, [[Scope]]]
}
```

6.准备工作做完，开始执行函数，随着函数的执行，修改 AO 的属性值

```javascript
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: 'local scope'
    },
    Scope: [AO, [[Scope]]]
}
```

7.查找到 scope2 的值，返回后函数执行完毕，函数上下文从执行上下文栈中弹出

```javascript
ECStack = [
    globalContext
];
```

### ECMAScript规范解读this

#### 前言

> 前两节讲到了变量对象和作用域链，接下来说说this

 ECMAScript 5.1 规范地址

[中文版](http://yanhaijing.com/es5/#71)

#### Types

首先是第 8 章 Types：

> Types are further subclassified into ECMAScript language types and specification types.

> An ECMAScript language type corresponds to values that are directly manipulated by an ECMAScript programmer using the ECMAScript language. The ECMAScript language types are Undefined, Null, Boolean, String, Number, and Object.

> A specification type corresponds to meta-values that are used within algorithms to describe the semantics of ECMAScript language constructs and ECMAScript language types. The specification types are Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, and Environment Record.

简单翻译一下：

ECMAScript 的类型分为`语言类型`和`规范类型`。

ECMAScript `语言类型`是开发者直接`使用 ECMAScript 可以操作`的。其实就是我们常说的Undefined, Null, Boolean, String, Number, 和 Object。

而规范类型相当于 meta-values，是用来用算法`描述 ECMAScript 语言结构`和 `ECMAScript 语言类型`的。规范类型包括：Reference, List, Completion, Property Descriptor, Property Identifier, Lexical Environment, 和 Environment Record。

我们只要知道在 ECMAScript 规范中还有一种只存在于规范中的类型，它们的作用是用来描述语言底层行为逻辑

#### Reference

让我们看 8.7 章 The Reference Specification Type：

> The Reference type is used to explain the behaviour of such operators as delete, typeof, and the assignment operators.

所以 Reference 类型就是用来解释诸如 delete、typeof 以及赋值等操作行为的

> 这里的 Reference 是一个 Specification Type，也就是 “只存在于规范里的抽象类型”。它们是为了更好地描述语言的底层行为逻辑才存在的，但并不存在于实际的 js 代码中。

具体的Reference的内容

> A Reference is a resolved name binding.

> A Reference consists of three components, the base value, the referenced name and the Boolean valued strict reference flag.

> The base value is either undefined, an Object, a Boolean, a String, a Number, or an environment record (10.2.1).

> A base value of undefined indicates that the reference could not be resolved to a binding. The referenced name is a String.

Reference 的构成，由三个组成部分，分别是：

- base value
- referenced name
- strict  reference

上面的话简单理解为

base value 就是属性所在的对象或者就是 EnvironmentRecord，它的值只可能是 undefined, an Object, a Boolean, a String, a Number, or an environment record 其中的一种。

referenced name 就是属性的名称。

举个例子

```javascript
var foo = 1;

// 对应的Reference是：
var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};
```

在举个例子

```javascript
var foo = {
    bar: function () {
        return this;
    }
};
 
foo.bar(); // foo

// bar对应的Reference是：
var BarReference = {
    base: foo,
    propertyName: 'bar',
    strict: false
};
```

而且规范中还提供了获取 Reference 组成部分的方法，比如` GetBase` 和 `IsPropertyReference`。

- GetBase

  > GetBase(V). Returns the base value component of the reference V.

  返回 reference 的 base value。

- IsPropertyReference

  > IsPropertyReference(V). Returns true if either the base value is an object or HasPrimitiveBase(V) is true; otherwise returns false.

  简单的理解：如果 base value 是一个`对象`，就返回`true`。

#### GetValue

> GetValue 返回对象属性真正的值，但是要注意：**调用 GetValue，返回的将是具体的值，而不再是一个 Reference**  (这个很重要，这个很重要，这个很重要。)

```javascript
var foo = 1;

var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};

GetValue(fooReference) // 1;
```

#### 如何确定this的值

看规范 11.2.3 Function Calls：

这里讲了当函数调用的时候，如何确定 this 的取值。

只看第一步、第六步、第七步：

> 1.Let *ref* be the result of evaluating MemberExpression.

> 6.If Type(*ref*) is Reference, then

>  ```
> a.If IsPropertyReference(ref) is true, then
>  ```

> ```
> i.Let thisValue be GetBase(ref)
> ```

> ```
> b.Else, the base of ref is an Environment Record
> ```

> ```
> i.Let thisValue be the result of calling the ImplicitThisValue concrete method of GetBase(ref).
> ```

> 7.Else, Type(*ref*) is not Reference.

> ```
> a. Let thisValue be undefined.
> ```

简单解释一下

1、计算 MemberExpression 的结果赋值给 ref

2、判断 ref 是不是一个 Reference 类型

- 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)
-  如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)
- 如果 ref 不是 Reference，那么 this 的值为 undefined

#### 具体分析

让我们一步一步看：

1. 计算 MemberExpression 的结果赋值给 ref

什么是 MemberExpression？看规范 11.2 Left-Hand-Side Expressions：

MemberExpression :

- PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
- FunctionExpression // 函数定义表达式
- MemberExpression [ Expression ] // 属性访问表达式
- MemberExpression . IdentifierName // 属性访问表达式
- new MemberExpression Arguments // 对象创建表达式

举个例子

```javascript
function foo() {
    console.log(this)
}

foo(); // MemberExpression 是 foo

function foo() {
    return function() {
        console.log(this)
    }
}

foo()(); // MemberExpression 是 foo()

var foo = {
    bar: function () {
        return this;
    }
}

foo.bar(); // MemberExpression 是 foo.bar
```

简单理解 MemberExpression 其实就是`()左边`的部分。

2.判断 ref 是不是一个 Reference 类型。

关键就在于看规范是如何处理各种 MemberExpression，返回的结果是不是一个Reference类型。

举最后一个例子：

```javascript
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

//示例1
console.log(foo.bar());
//示例2
console.log((foo.bar)());
//示例3
console.log((foo.bar = foo.bar)());
//示例4
console.log((false || foo.bar)());
//示例5
console.log((foo.bar, foo.bar)());
```

**foo.bar()**

在示例 1 中，MemberExpression 计算的结果是 foo.bar，那么 foo.bar 是不是一个 Reference 呢？

查看规范 11.2.1 Property Accessors，这里展示了一个计算的过程，什么都不管了，就看最后一步：

> Return a value of type Reference whose base value is baseValue and whose referenced name is propertyNameString, and whose strict mode flag is strict.

我们得知该表达式返回了一个 Reference 类型！

根据之前的内容，我们知道该值为：

```javascript
var Reference = {
  base: foo,
  name: 'bar',
  strict: false
};
```

接下来按照 2.1 的判断流程走：

> 2.1 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

该值是 Reference 类型，那么 IsPropertyReference(ref) 的结果是多少呢？

前面我们已经铺垫了 IsPropertyReference 方法，如果 base value 是一个对象，结果返回 true。

base value 为 foo，是一个对象，所以 IsPropertyReference(ref) 结果为 true。

这个时候我们就可以确定 this 的值了：

> this = GetBase(ref)，

GetBase 也已经铺垫了，获得 base value 值，这个例子中就是foo，所以 this 的值就是 foo ，示例1的结果就是 2！

**(foo.bar)()**

```
console.log((foo.bar)());
```

foo.bar 被 () 包住，查看规范 11.1.6 The Grouping Operator

直接看结果部分：

> Return the result of evaluating Expression. This may be of type Reference.

> NOTE This algorithm does not apply GetValue to the result of evaluating Expression.

实际上 () 并没有对 MemberExpression 进行计算，所以其实跟示例 1 的结果是一样的。

**(foo.bar = foo.bar)()** 

看示例3，有赋值操作符，查看规范 11.13.1 Simple Assignment ( = ):

计算的第三步：

> 3.Let rval be GetValue(rref).

因为使用了 GetValue，所以返回的值不是 Reference 类型，

按照之前讲的判断逻辑：

> 2.3 如果 ref 不是Reference，那么 this 的值为 undefined

this 为 undefined，非严格模式下，this 的值为 undefined 的时候，其值会被隐式转换为`全局对象`。

**(false || foo.bar)()**

看示例4，逻辑与算法，查看规范 11.11 Binary Logical Operators：

计算第二步：

> 2.Let lval be GetValue(lref).

因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined

**(foo.bar, foo.bar)()**

看示例5，逗号操作符，查看规范11.14 Comma Operator ( , )

计算第二步：

> 2.Call GetValue(lref).

因为使用了 GetValue，所以返回的不是 Reference 类型，this 为 undefined

#### 揭晓结果

```javascript
var value = 1;

var foo = {
  value: 2,
  bar: function () {
    return this.value;
  }
}

//示例1
console.log(foo.bar()); // 2
//示例2
console.log((foo.bar)()); // 2
//示例3
console.log((foo.bar = foo.bar)()); // 1
//示例4
console.log((false || foo.bar)()); // 1
//示例5
console.log((foo.bar, foo.bar)()); // 1
```

注意：以上是在非严格模式下的结果，严格模式下因为 this 返回 undefined，所以示例 3 会报错。

#### 补充

```javascript
function foo() {
    console.log(this)
}

foo(); 
```



MemberExpression 是 foo，解析标识符，查看规范 10.3.1 Identifier Resolution，会返回一个 Reference 类型的值：

```javascript
var fooReference = {
    base: EnvironmentRecord,
    name: 'foo',
    strict: false
};
```

接下来进行判断：

> 2.1 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

因为 base value 是 EnvironmentRecord，并不是一个 Object 类型，还记得前面讲过的 base value 的取值可能吗？ 只可能是 undefined, an Object, a Boolean, a String, a Number, 和 an environment record 中的一种。

IsPropertyReference(ref) 的结果为 false，进入下个判断：

> 2.2 如果 ref 是 Reference，并且 base value 值是 Environment Record, 那么this的值为 ImplicitThisValue(ref)

base value 正是 Environment Record，所以会调用 ImplicitThisValue(ref)

查看规范 10.2.1.1.6，ImplicitThisValue 方法的介绍：该函数始终返回 undefined。

所以最后 this 的值就是 undefined。



### 执行上下文

#### 思考题

在词法作用域和动态作用域中，提出这样一道思考题

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

两段代码都会打印'local scope',两者的区别在于执行上下文栈的变化不一样，然而，如果是这样笼统的回答，依然显得不够详细，本篇就会详细的解析执行上下文栈和执行上下文的具体变化过程

#### 具体的执行分析

我们分析第一段代码：

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

执行过程如下：

1.执行全局代码，创建全局执行上下文，全局上下文被压入执行上下文栈

```javascript
ECStack = [
  globalContext
];
```

2.全局上下文初始化

```javascript
    globalContext = {
        VO: [global],
        Scope: [globalContext.VO],
        this: globalContext.VO
    }
```

2.初始化的同时，checkscope 函数被创建，保存作用域链到函数的内部属性[[scope]]

```javascript
    checkscope.[[scope]] = [
      globalContext.VO
    ];
```

3.执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈

```javascript
    ECStack = [
        checkscopeContext,
        globalContext
    ];
```

4.checkscope 函数执行上下文初始化：

1. 复制函数 [[scope]] 属性创建作用域链，
2. 用 arguments 创建活动对象，
3. 初始化活动对象，即加入形参、函数声明、变量声明，
4. 将活动对象压入 checkscope 作用域链顶端。

同时 f 函数被创建，保存作用域链到 f 函数的内部属性[[scope]]

```javascript
    checkscopeContext = {
        AO: {
            arguments: {
                length: 0
            },
            scope: undefined,
            f: reference to function f(){}
        },
        Scope: [AO, globalContext.VO],
        this: undefined
    }
```

5.执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈

```javascript
    ECStack = [
        fContext,
        checkscopeContext,
        globalContext
    ];
```

6.f 函数执行上下文初始化, 以下跟第 4 步相同：

1. 复制函数 [[scope]] 属性创建作用域链
2. 用 arguments 创建活动对象
3. 初始化活动对象，即加入形参、函数声明、变量声明
4. 将活动对象压入 f 作用域链顶端

```javascript
 fContext = {
        AO: {
            arguments: {
                length: 0
            }
        },
        Scope: [AO, checkscopeContext.AO, globalContext.VO],
        this: undefined
    }
```

7.f 函数执行，沿着作用域链查找 scope 值，返回 scope 值

8.f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

```javascript
    ECStack = [
        checkscopeContext,
        globalContext
    ];
```

9.checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出

```javascript
    ECStack = [
        globalContext
    ];
```

### 闭包

#### 定义

MDN 对闭包的定义为：

> 闭包是指那些能够访问**自由变量**的函数

什么是自由变量呢？

> 自由变量是指在**函数中使用**的，但既**不是函数参数**也不是函数的**局部变量**的变量。

 由此，我们可以看出闭包有两部分组成

> 闭包 = 函数 + 函数能够访问的自由变量

举个例子

```javascript
var a = 1;

function foo() {
    console.log(a);
}

foo();
```

foo 函数可以访问变量 a，但是 a 既不是 foo 函数的局部变量，也不是 foo 函数的参数，所以 a 就是自由变量。

那么，函数 foo + foo 函数访问的自由变量 a 不就是构成了一个闭包嘛……

所以在《JavaScript权威指南》中就讲到：从技术的角度讲，所有的JavaScript函数都是闭包。

咦，这怎么跟我们平时看到的讲到的闭包不一样呢！？

别着急，这是理论上的闭包，其实还有一个实践角度上的闭包，让我们看看汤姆大叔翻译的关于闭包的文章中的定义：

ECMAScript中，闭包指的是：

1. 从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域。
2. 从实践角度：以下函数才算是闭包：
   1. 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
   2. 在代码中引用了自由变量

接下来就来讲讲实践上的闭包。

#### 分析

让我们先写个例子，例子依然是来自《JavaScript权威指南》，稍微做点改动：

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}

var foo = checkscope();
foo();
```

这里直接给出简要的执行过程：

1. 进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈
2. 全局执行上下文初始化
3. 执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 执行上下文被压入执行上下文栈
4. checkscope 执行上下文初始化，创建变量对象、作用域链、this等
5. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
6. 执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
7. f 执行上下文初始化，创建变量对象、作用域链、this等
8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

了解到这个过程，我们应该思考一个问题，那就是：

当 f 函数执行的时候，checkscope 函数上下文已经被销毁了啊(即从执行上下文栈中被弹出)，怎么还会读取到 checkscope 作用域下的 scope 值呢？

以上的代码，要是转换成 PHP，就会报错，因为在 PHP 中，f 函数只能读取到自己作用域和全局作用域里的值，所以读不到 checkscope 下的 scope 值。(这段我问的PHP同事……)

然而 JavaScript 却是可以的！

当我们了解了具体的执行过程后，我们知道 f 执行上下文维护了一个作用域链：

```javascript
fContext = {
    Scope: [AO, checkscopeContext.AO, globalContext.VO],
}
```

对的，就是因为这个作用域链，f 函数依然可以读取到 checkscopeContext.AO 的值，说明当 f 函数引用了 checkscopeContext.AO 中的值的时候，即使 checkscopeContext 被销毁了，但是 JavaScript 依然会让 checkscopeContext.AO 活在内存中，f 函数依然可以通过 f 函数的作用域链找到它，正是因为 JavaScript 做到了这一点，从而实现了闭包这个概念。

所以，让我们再看一遍实践角度上闭包的定义：

1. 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
2. 在代码中引用了自由变量

再补充一个《JavaScript权威指南》英文原版对闭包的定义:

> This combination of a function object and a scope (a set of variable bindings) in which the function’s variables are resolved is called a closure in the computer science literature.

#### 必刷题

接下来，看这道刷题必刷，面试必考的闭包题：

```javascript
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i);
  };
}

data[0]();
data[1]();
data[2]();
```


答案是都是 3，让我们分析一下原因：

当执行到 data[0] 函数之前，此时全局上下文的 VO 为：

```javascript
globalContext = {
    VO: {
        data: [...],
        i: 3
    }
}
```

当执行 data[0] 函数的时候，data[0] 函数的作用域链为：

```javascript
data[0]Context = {
    Scope: [AO, globalContext.VO]
}
```

data[0]Context 的 AO 并没有 i 值，所以会从 globalContext.VO 中查找，i 为 3，所以打印的结果就是 3。

data[1] 和 data[2] 是一样的道理。

所以让我们改成闭包看看：

```javascript
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = (function (i) {
        return function(){
            console.log(i);
        }
  })(i);
}

data[0]();
data[1]();
data[2]();
```

当执行到 data[0] 函数之前，此时全局上下文的 VO 为：

```
globalContext = {
    VO: {
        data: [...],
        i: 3
    }
}
```

跟没改之前一模一样。

当执行 data[0] 函数的时候，data[0] 函数的作用域链发生了改变：

```
data[0]Context = {
    Scope: [AO, 匿名函数Context.AO globalContext.VO]
}
```

匿名函数执行上下文的AO为：

```
匿名函数Context = {
    AO: {
        arguments: {
            0: 0,
            length: 1
        },
        i: 0
    }
}
```

data[0]Context 的 AO 并没有 i 值，所以会沿着作用域链从匿名函数 Context.AO 中查找，这时候就会找 i 为 0，找到了就不会往 globalContext.VO 中查找了，即使 globalContext.VO 也有 i 的值(值为3)，所以打印的结果就是0。

data[1] 和 data[2] 是一样的道理。

### 参数按值传递

#### 定义

在《JavaScript高级程序设计》第三版 4.1.3，讲到传递参数：

> ECMAScript中所有函数的**参数都是按值**传递的。

什么是按值传递呢？

> 也就是说，把函数外部的值**复制给函数内部的参数**，就和把值从一个变量复制到另一个变量一样。

#### 按值传递

举个例子

```javascript
var value = 1;
function foo(v) {
    v = 2;
    console.log(v); //2
}
foo(value);
console.log(value) // 1
```

很好理解，当传递 value 到函数 foo 中，相当于拷贝了一份 value，假设拷贝的这份叫 _value，函数中修改的都是 _value 的值，而不会影响原来的 value 值。

#### 引用传递？

拷贝虽然很好理解，但是当值是一个复杂的数据结构的时候，拷贝就会产生性能上的问题。

所以还有另一种传递方式叫做按引用传递。

所谓按引用传递，就是传递对象的引用，函数内部对参数的任何改变都会影响该对象的值，因为两者引用的是同一个对象。

举个例子：

```javascript
var obj = {
    value: 1
};
function foo(o) {
    o.value = 2;
    console.log(o.value); //2
}
foo(obj);
console.log(obj.value) // 2
```

哎，不对啊，连我们的红宝书都说了 ECMAScript 中所有函数的参数都是按值传递的，这怎么能按"引用传递"成功呢？

而这究竟是不是引用传递呢？

#### 第三种传递方式

```javascript
var obj = {
    value: 1
};
function foo(o) {
    o = 2;
    console.log(o); //2
}
foo(obj);
console.log(obj.value) // 1
```

如果 JavaScript 采用的是引用传递，外层的值也会被修改呐，这怎么又没被改呢？所以真的不是引用传递吗？

这就要讲到其实还有第三种传递方式，叫按**共享传递**。

而共享传递是指，在传递对象的时候，传递对象的引用的副本。

**注意： 按引用传递是传递对象的引用，而按共享传递是传递对象的引用的副本！**

所以修改 o.value，可以通过引用找到原值，但是直接修改 o，并不会修改原值。所以第二个和第三个例子其实都是按共享传递。

最后，你可以这样理解：

参数如果是基本类型是按值传递，如果是引用类型按共享传递。

但是因为拷贝副本也是一种值的拷贝，所以在高程中也直接认为是按值传递了。

### call和apply的模拟实现

#### call

> call()方法在使用一个指定的this值和若干个指定的参数值的前提下调用某个函数或方法

举个例子

```javascript
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

bar.call(foo); // 1
```

**注意两点**

1、call改变了this的指向，指向到foo

2、bar函数执行了

#### 模拟实现第一步

那么我们该怎么模拟实现这两个效果呢？

试想当调用 call 的时候，把 foo 对象改造成如下：

```javascript
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
};

foo.bar(); // 1
```

这个时候 this 就指向了 foo，是不是很简单呢？

但是这样却给 foo 对象本身添加了一个属性，这可不行呐！

不过也不用担心，我们用 delete 再删除它不就好了~

所以我们模拟的步骤可以分为：

1. 将函数设为对象的属性
2. 执行该函数
3. 删除该函数

以上个例子为例，就是：

```javascript
// 第一步
foo.fn = bar
// 第二步
foo.fn()
// 第三步
delete foo.fn
```

根据这个思路，我们可以尝试着去写第一版的 **call2** 函数：

```javascript
// 第一版
Function.prototype.call2 = function(context) {
    // 首先要获取调用call的函数，用this可以获取
    context.fn = this;
    context.fn();
    delete context.fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

bar.call2(foo); // 1
```

#### 模拟实现第二步

call 函数还能给定参数执行函数。举个例子：

```javascript
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call(foo, 'kevin', 18);
// kevin
// 18
// 1
```

传入的参数不确定，可以从Arguments对象中取值，取出第二个以后的参数，放到数组中

```javascript
// 以上个例子为例，此时的arguments为：
// arguments = {
//      0: foo,
//      1: 'kevin',
//      2: 18,
//      length: 3
// }
// 因为arguments是类数组对象，所以可以用for循环
var args = [];
for(var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']');
}

// 执行后 args为 ["arguments[1]", "arguments[2]", "arguments[3]"]
```

我们接着要把这个参数数组放到要执行的函数的参数里面去

```javascript
// 将数组里的元素作为多个参数放进函数的形参里
context.fn(args.join(','))
// (O_o)??
// 这个方法肯定是不行的啦！！！
```

可以使用**eval**方法拼成一个函数

```javascript
eval('context.fn(' + args +')')
```

这里 args 会自动调用 Array.toString() 这个方法。

```javascript
// 第二版
Function.prototype.call2 = function(context) {
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    eval('context.fn(' + args +')');
    delete context.fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}

bar.call2(foo, 'kevin', 18); 
// kevin
// 18
// 1
```

#### 模拟实现第三步

有两点要注意

**1.this 参数可以传 null，当为 null 的时候，视为指向 window**

```javascript
var value = 1;

function bar() {
    console.log(this.value);
}

bar.call(null); // 1
```

**2.函数是可以有返回值的！**

```javascript
var obj = {
    value: 1
}

function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}

console.log(bar.call(obj, 'kevin', 18));
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }
```

最终代码

```javascript
// 第三版
Function.prototype.call2 = function (context) {
    var context = context || window;
    context.fn = this;

    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }

    var result = eval('context.fn(' + args +')');

    delete context.fn
    return result;
}

// 测试一下
var value = 2;

var obj = {
    value: 1
}

function bar(name, age) {
    console.log(this.value);
    return {
        value: this.value,
        name: name,
        age: age
    }
}

bar.call2(null); // 2

console.log(bar.call2(obj, 'kevin', 18));
// 1
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }
```

#### apply的模拟实现

```javascript
Function.prototype.apply = function (context, arr) {
    var context = Object(context) || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    } else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}
```

### bind的模拟实现

#### bind

> bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )

由此我们可以首先得出 bind 函数的两个特点：

- 返回一个函数
- 可以传入参数

#### 返回函数的模拟实现

举个例子

```javascript
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

// 返回了一个函数
var bindFoo = bar.bind(foo); 

bindFoo(); // 1
```

第一版

```javascript
// 第一版
Function.prototype.bind2 = function (context) {
    var self = this;
    return function () {
        return self.apply(context);
    }

}
```

此外，之所以 `return self.apply(context)`，是考虑到绑定函数可能是有返回值的，依然是这个例子：

```javascript
var foo = {
    value: 1
};

function bar() {
	return this.value;
}

var bindFoo = bar.bind(foo);

console.log(bindFoo()); // 1
```

#### 传参的模拟实现

```javascript
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(this.value);
    console.log(name);
    console.log(age);

}

var bindFoo = bar.bind(foo, 'daisy');
bindFoo('18');
// 1
// daisy
// 18
```

函数需要传 name 和 age 两个参数，竟然还可以在 bind 的时候，只传一个 name，在执行返回的函数的时候，再传另一个参数 age!

```javascript
// 第二版
Function.prototype.bind2 = function (context) {

    var self = this;
    // 获取bind2函数从第二个参数到最后一个参数
    var args = Array.prototype.slice.call(arguments, 1);

    return function () {
        // 这个时候的arguments是指bind返回的函数传入的参数
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(context, args.concat(bindArgs));
    }

}
```

#### 构造函数效果的模拟实现

> 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

也就是说当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效。举个例子：

```javascript
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}

bar.prototype.friend = 'kevin';

var bindFoo = bar.bind(foo, 'daisy');

var obj = new bindFoo('18');
// undefined
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// kevin
```

注意：尽管在全局和 foo 中都声明了 value 值，最后依然返回了 undefind，说明绑定的 this 失效了，如果大家了解 new 的模拟实现，就会知道这个时候的 this 已经指向了 obj。

```javascript
// 第三版
Function.prototype.bind2 = function (context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
        // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
        // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
    fBound.prototype = this.prototype;
    return fBound;
}
```

#### 构造函数效果的优化实现

但是在这个写法中，我们直接将 fBound.prototype = this.prototype，我们直接修改 fBound.prototype 的时候，也会直接修改绑定函数的 prototype。这个时候，我们可以通过一个空函数来进行中转：

```javascript
// 第四版
Function.prototype.bind2 = function (context) {

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

#### 小问题

- **调用 bind 的不是函数咋办**

  ```javascript
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
  }
  ```

- 线上兼容

  ```javascript
  Function.prototype.bind = Function.prototype.bind || function () {
      ……
  };
  ```

最终代码

```javascript
Function.prototype.bind2 = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

### new的模拟实现

#### new

> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一

举个例子

```javascript
// Otaku 御宅族，简称宅
function Otaku (name, age) {
    this.name = name;
    this.age = age;

    this.habit = 'Games';
}

// 因为缺乏锻炼的缘故，身体强度让人担忧
Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); // I am Kevin
```

我们可以看到，实例 person 可以：

1、访问到Otaku构造函数里的属性

2、访问到Otaku.prototype中的属性

我们写一个函数，命名为 objectFactory，来模拟 new 的效果。用的时候是这样的

```javascript
function Otaku () {
    ……
}

// 使用 new
var person = new Otaku(……);
// 使用 objectFactory
var person = objectFactory(Otaku, ……)
```

#### 初步实现

分析：

因为 new 的结果是一个新对象，所以在模拟实现的时候，我们也要**建立一个新对象**，假设这个对象叫 obj，因为 obj 会具有 Otaku 构造函数里的属性，想想经典继承的例子，我们可以使用 Otaku.apply(obj, arguments)来给 obj 添加新的属性。

我们知道实例的 `__proto__ `属性会指向构造函数的 prototype，也正是因为建立起这样的关系，实例可以访问原型上的属性。

```javascript
// 第一版代码
function objectFactory() {

    var obj = new Object(),

    Constructor = [].shift.call(arguments);

    obj.__proto__ = Constructor.prototype;

    Constructor.apply(obj, arguments);

    return obj;

};
```

在这一版中，我们：

1. 用new Object() 的方式新建了一个对象 obj
2. 取出第一个参数，就是我们要传入的构造函数。此外因为 shift 会修改原数组，所以 arguments 会被去除第一个参数
3. 将 obj 的原型指向构造函数，这样 obj 就可以访问到构造函数原型中的属性
4. 使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性
5. 返回 obj

#### 返回值效果实现

假如构造函数有返回值

```javascript
function Otaku (name, age) {
    this.strength = 60;
    this.age = age;

    return {
        name: name,
        habit: 'Games'
    }
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // undefined
console.log(person.age) // undefined
```

在这个例子中，构造函数返回了一个对象，在实例 person 中只能访问返回的对象中的属性。

而且还要注意一点，在这里我们是返回了一个对象，假如我们只是返回一个基本类型的值呢？

再举个例子：

```javascript
function Otaku (name, age) {
    this.strength = 60;
    this.age = age;

    return 'handsome boy';
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // undefined
console.log(person.habit) // undefined
console.log(person.strength) // 60
console.log(person.age) // 18
```

结果完全颠倒过来，这次尽管有返回值，但是相当于没有返回值进行处理。

所以我们还需要判断返回的值是不是一个对象，如果是一个**对象**，我们就**返回这个对象**，如果没有，我们该返回什么就返回什么。

```javascript
// 第二版的代码
function objectFactory() {

    var obj = new Object(),

    Constructor = [].shift.call(arguments);

    obj.__proto__ = Constructor.prototype;

    var ret = Constructor.apply(obj, arguments);

    return typeof ret === 'object' ? ret : obj;

};
```

### 类数组对象与arguments

#### 类数组对象

所谓的类数组对象

> 拥有一个length属性和若干索引属性的对象

举个例子

```JavaScript
var array = ['name', 'age', 'sex'];

var arrayLike = {
    0: 'name',
    1: 'age',
    2: 'sex',
    length: 3
}
```

即便如此，为什么叫做类数组对象呢？

那让我们从读写、获取长度、遍历三个方面看看这两个对象。

#### 读写

```javascript
console.log(array[0]); // name
console.log(arrayLike[0]); // name

array[0] = 'new name';
arrayLike[0] = 'new name';
```

#### 长度

```javascript
console.log(array.length); // 3
console.log(arrayLike.length); // 3
```

#### 遍历

```javascript
for(var i = 0, len = array.length; i < len; i++) {
   ……
}
for(var i = 0, len = arrayLike.length; i < len; i++) {
    ……
}
```

是不是很像？

那类数组对象可以使用数组的方法吗？比如：

```javascript
arrayLike.push('4');
```

然而上述代码会报错: arrayLike.push is not a function

所以终归还是类数组呐……

#### 调用数组方法

如果类数组就是任性的想用数组的方法怎么办呢？

既然无法直接调用，我们可以用 Function.call 间接调用：

```javascript
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }

Array.prototype.join.call(arrayLike, '&'); // name&age&sex

Array.prototype.slice.call(arrayLike, 0); // ["name", "age", "sex"] 
// slice可以做到类数组转数组

Array.prototype.map.call(arrayLike, function(item){
    return item.toUpperCase();
}); 
// ["NAME", "AGE", "SEX"]
```

#### 类数组转数组

```javascript
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }
// 1. slice
Array.prototype.slice.call(arrayLike); // ["name", "age", "sex"] 
// 2. splice
Array.prototype.splice.call(arrayLike, 0); // ["name", "age", "sex"] 
// 3. ES6 Array.from
Array.from(arrayLike); // ["name", "age", "sex"] 
// 4. apply
Array.prototype.concat.apply([], arrayLike)
```

那么为什么会讲到类数组对象呢？以及类数组有什么应用吗？

要说到类数组对象，Arguments 对象就是一个类数组对象。在客户端 JavaScript 中，一些 DOM 方法(document.getElementsByTagName()等)也返回类数组对象。

#### Arguments对象

Arguments 对象只定义在函数体中，包括了函数的参数和其他属性。在函数体中，arguments 指代该函数的 Arguments 对象。

举个例子

```javascript
function foo(name, age, sex) {
    console.log(arguments);
}

foo('name', 'age', 'sex')
```

打印结果如下

![](https://github.com/mqyqingfeng/Blog/raw/master/Images/arguments.png)

我们可以看到除了类数组的索引属性和length属性之外，还有一个callee属性，接下来我们一个一个介绍。

**length属性**

Arguments对象的length属性，表示**实参**的长度，举个例子：

```javascript
function foo(b, c, d){
    console.log("实参的长度为：" + arguments.length)
}

console.log("形参的长度为：" + foo.length)

foo(1)

// 形参的长度为：3
// 实参的长度为：1
```

**callee属性**

Arguments 对象的 callee 属性，通过它可以调用函数自身。

讲个闭包经典面试题使用 callee 的解决方法：

```javascript
var data = [];

for (var i = 0; i < 3; i++) {
    (data[i] = function () {
       console.log(arguments.callee.i) 
    }).i = i;
}

data[0]();
data[1]();
data[2]();

// 0
// 1
// 2
```

#### arguments 和对应参数的绑定

```javascript
function foo(name, age, sex, hobbit) {

    console.log(name, arguments[0]); // name name

    // 改变形参
    name = 'new name';

    console.log(name, arguments[0]); // new name new name

    // 改变arguments
    arguments[1] = 'new age';

    console.log(age, arguments[1]); // new age new age

    // 测试未传入的是否会绑定
    console.log(sex); // undefined

    sex = 'new sex';

    console.log(sex, arguments[2]); // new sex undefined

    arguments[3] = 'new hobbit';

    console.log(hobbit, arguments[3]); // undefined new hobbit

}

foo('name', 'age')
```

**传入的参数**，实参和 arguments 的值**会共享**，当**没有传入**时，实参与 arguments 值**不会共享**

除此之外，以上是在非严格模式下，如果是在**严格模式**下，实参和 arguments 是**不会共享**的。

#### 传递参数

```javascript
// 使用 apply 将 foo 的参数传递给 bar
function foo() {
    bar.apply(this, arguments);
}
function bar(a, b, c) {
   console.log(a, b, c);
}

foo(1, 2, 3)
```

#### 强大的ES6

```javascript
function func(...arguments) {
    console.log(arguments); // [1, 2, 3]
}

func(1, 2, 3);
```

#### 应用

arguments的应用其实很多，我们会在 jQuery 的 extend 实现、函数柯里化、递归等场景看见 arguments 的身影。这篇文章就不具体展开了。

如果要总结这些场景的话，暂时能想到的包括：

1. 参数不定长
2. 函数柯里化
3. 递归调用
4. 函数重载

### 创建对象的多种方式及优缺点

#### 工厂模式

```javascript
function createPerson(name) {
    var o = new Object();
    o.name = name;
    o.getName = function () {
        console.log(this.name);
    };

    return o;
}

var person1 = createPerson('kevin');
```

缺点：对象无法识别，因为所有的实例都**指向一个原型**

#### 构造函数模式

```javascript
function Person(name) {
    this.name = name;
    this.getName = function () {
        console.log(this.name);
    };
}

var person1 = new Person('kevin');
```

优点：实例可以识别为一个特定的类型

缺点：每次创建实例时，每个方法**都要被创建一次**

**构造函数模式优化**

```javascript
function Person(name) {
    this.name = name;
    this.getName = getName;
}

function getName() {
    console.log(this.name);
}

var person1 = new Person('kevin');
```

优点：解决了每个方法都要被重新创建的问题

缺点：这叫啥封装……

#### 原型模式

```javascript
function Person(name) {

}

Person.prototype.name = 'keivn';
Person.prototype.getName = function () {
    console.log(this.name);
};

var person1 = new Person();
```

优点：方法不会重新创建

缺点：1. 所有的属性和方法都共享 2. 不能初始化参数

**原型模式优化**

```javascript
function Person(name) {

}

Person.prototype = {
    name: 'kevin',
    getName: function () {
        console.log(this.name);
    }
};

var person1 = new Person();
```

优点：封装性好了一点

缺点：重写了原型，丢失了constructor属性

```javascript
function Person(name) {

}

Person.prototype = {
    constructor: Person,
    name: 'kevin',
    getName: function () {
        console.log(this.name);
    }
};

var person1 = new Person();
```

优点：实例可以通过constructor属性找到所属构造函数

缺点：原型模式该有的缺点还是有

#### 组合模式

构造函数模式与原型模式双剑合璧。

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype = {
    constructor: Person,
    getName: function () {
        console.log(this.name);
    }
};

var person1 = new Person();
```

优点：该共享的共享，该私有的私有，使用最广泛的方式

缺点：有的人就是希望全部都写在一起，即更好的封装性

**动态原型模式**

```javascript
function Person(name) {
    this.name = name;
    if (typeof this.getName != "function") {
        Person.prototype.getName = function () {
            console.log(this.name);
        }
    }
}

var person1 = new Person();
```

注意：使用动态原型模式时，不能用对象字面量重写原型

解释下为什么：

```javascript
function Person(name) {
    this.name = name;
    if (typeof this.getName != "function") {
        Person.prototype = {
            constructor: Person,
            getName: function () {
                console.log(this.name);
            }
        }
    }
}

var person1 = new Person('kevin');
var person2 = new Person('daisy');

// 报错 并没有该方法
person1.getName();

// 注释掉上面的代码，这句是可以执行的。
person2.getName();
```

为了解释这个问题，假设开始执行`var person1 = new Person('kevin')`

注意这个时候，回顾下 apply 的实现步骤，会执行 obj.Person 方法，这个时候就会执行 if 语句里的内容，注意构造函数的 prototype 属性指向了实例的原型，使用字面量方式直接覆盖 Person.prototype，并不会更改实例的原型的值，person1 依然是指向了以前的原型，而不是 Person.prototype。而之前的原型是没有 getName 方法的，所以就报错了！

如果你就是想用字面量方式写代码，可以尝试下这种：

```javascript
function Person(name) {
    this.name = name;
    if (typeof this.getName != "function") {
        Person.prototype = {
            constructor: Person,
            getName: function () {
                console.log(this.name);
            }
        }

        return new Person(name);
    }
}

var person1 = new Person('kevin');
var person2 = new Person('daisy');

person1.getName(); // kevin
person2.getName();  // daisy
```

#### 寄生构造函数模式

```javascript
function Person(name) {

    var o = new Object();
    o.name = name;
    o.getName = function () {
        console.log(this.name);
    };

    return o;

}

var person1 = new Person('kevin');
console.log(person1 instanceof Person) // false
console.log(person1 instanceof Object)  // true
```

这样方法可以在特殊情况下使用。比如我们想创建一个具有额外方法的特殊数组，但是又不想直接修改Array构造函数，我们可以这样写：

```javascript
function SpecialArray() {
    var values = new Array();

    for (var i = 0, len = arguments.length; i < len; i++) {
        values.push(arguments[i]);
    }

    values.toPipedString = function () {
        return this.join("|");
    };
    return values;
}

var colors = new SpecialArray('red', 'blue', 'green');
var colors2 = SpecialArray('red2', 'blue2', 'green2');


console.log(colors);
console.log(colors.toPipedString()); // red|blue|green

console.log(colors2);
console.log(colors2.toPipedString()); // red2|blue2|green2
```

但是值得一提的是，上面例子中的循环：

```javascript
for (var i = 0, len = arguments.length; i < len; i++) {
    values.push(arguments[i]);
}
```

可以替换成

```javascript
values.push.apply(values, arguments);
```

**稳妥构造函数模式**

```javascript
function person(name){
    var o = new Object();
    o.sayName = function(){
        console.log(name);
    };
    return o;
}

var person1 = person('kevin');

person1.sayName(); // kevin

person1.name = "daisy";

person1.sayName(); // kevin

console.log(person1.name); // daisy
```

所谓稳妥对象，指的是没有公共属性，而且其方法也不引用 this 的对象。

与寄生构造函数模式有两点不同：

1. 新创建的实例方法不引用 this
2. 不使用 new 操作符调用构造函数

稳妥对象最适合在一些安全的环境中。

稳妥构造函数模式也跟工厂模式一样，无法识别对象所属类型。

### 继承的多种方式和优缺点

#### 原型链继承

```javascript
function Parent () {
    this.name = 'kevin';
}

Parent.prototype.getName = function () {
    console.log(this.name);
}

function Child () {

}

Child.prototype = new Parent();

var child1 = new Child();

console.log(child1.getName()) // kevin
```

问题：

1.引用类型的属性被所有实例共享，举个例子：

```javascript
function Parent () {
    this.names = ['kevin', 'daisy'];
}

function Child () {

}

Child.prototype = new Parent();

var child1 = new Child();

child1.names.push('yayu');

console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();

console.log(child2.names); // ["kevin", "daisy", "yayu"]
```

2.在创建 Child 的实例时，不能向Parent传参

#### 借用构造函数（经典继承）

```javascript
function Parent () {
    this.names = ['kevin', 'daisy'];
}

function Child () {
    Parent.call(this);
}

var child1 = new Child();

child1.names.push('yayu');

console.log(child1.names); // ["kevin", "daisy", "yayu"]

var child2 = new Child();

console.log(child2.names); // ["kevin", "daisy"]
```

优点：

1.避免了引用类型的属性被所有实例共享

2.可以在 Child 中向 Parent 传参

举个例子：

```javascript
function Parent (name) {
    this.name = name;
}

function Child (name) {
    Parent.call(this, name);
}

var child1 = new Child('kevin');

console.log(child1.name); // kevin

var child2 = new Child('daisy');

console.log(child2.name); // daisy
```

缺点：

方法都在构造函数中定义，每次创建实例都会创建一遍方法。

#### 组合继承

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {

    Parent.call(this, name);
    
    this.age = age;

}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

var child1 = new Child('kevin', '18');

child1.colors.push('black');

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

var child2 = new Child('daisy', '20');

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

优点：融合原型链继承和构造函数的优点，是 JavaScript 中最常用的继承模式。

#### 原型式继承

```javascript
function createObj(o) {
    function F(){}
    F.prototype = o;
    return new F();
}
```

就是 ES5 Object.create 的模拟实现，将传入的对象作为创建的对象的原型。

缺点：

包含引用类型的属性值始终都会共享相应的值，这点跟原型链继承一样。

```javascript
var person = {
    name: 'kevin',
    friends: ['daisy', 'kelly']
}

var person1 = createObj(person);
var person2 = createObj(person);

person1.name = 'person1';
console.log(person2.name); // kevin

person1.firends.push('taylor');
console.log(person2.friends); // ["daisy", "kelly", "taylor"]
```

注意：修改`person1.name`的值，`person2.name`的值并未发生改变，并不是因为`person1`和`person2`有独立的 name 值，而是因为`person1.name = 'person1'`，给`person1`添加了 name 值，并非修改了原型上的 name 值。

#### 寄生式继承

创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象。

```javascript
function createObj (o) {
    var clone = Object.create(o);
    clone.sayName = function () {
        console.log('hi');
    }
    return clone;
}
```

缺点：跟借用构造函数模式一样，每次创建对象都会创建一遍方法。

#### 寄生组合式继承

组合继承最大的缺点是会调用两次父构造函数。

一次是设置子类型实例的原型的时候：

```
Child.prototype = new Parent();
```

一次在创建子类型实例的时候：

```
var child1 = new Child('kevin', '18');
```

回想下 new 的模拟实现，其实在这句中，我们会执行：

```
Parent.call(this, name);
```

在这里，我们又会调用了一次 Parent 构造函数。

所以，在这个例子中，如果我们打印 child1 对象，我们会发现 Child.prototype 和 child1 都有一个属性为`colors`，属性值为`['red', 'blue', 'green']`。

那么我们该如何精益求精，避免这一次重复调用呢？

如果我们不使用 Child.prototype = new Parent() ，而是间接的让 Child.prototype 访问到 Parent.prototype 呢？

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

// 关键的三步
var F = function () {};

F.prototype = Parent.prototype;

Child.prototype = new F();


var child1 = new Child('kevin', '18');

console.log(child1);
```

最后我们封装一下这个继承方法：

```javascript
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}

function prototype(child, parent) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

// 当我们使用的时候：
prototype(Child, Parent);
```

引用《JavaScript高级程序设计》中对寄生组合式继承的夸赞就是：

这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

### 类型转换（上）

#### 前言

将值从一种类型转换为另一种类型通常称为类型转换。

ES6 前，JavaScript 共有六种数据类型：Undefined、Null、Boolean、Number、String、Object。

我们先捋一捋基本类型之间的转换。

#### 原始值转布尔

我们使用 Boolean 函数将类型转换成布尔类型，在 JavaScript 中，只有 **6 种值**可以被转换成 **false**，**其他**都会被转换成 **true**。

```javascript
console.log(Boolean()) // false

console.log(Boolean(false)) // false

console.log(Boolean(undefined)) // false
console.log(Boolean(null)) // false
console.log(Boolean(+0)) // false
console.log(Boolean(-0)) // false
console.log(Boolean(NaN)) // false
console.log(Boolean("")) // false
```

#### 原始值转数字

我们可以使用 Number 函数将类型转换成数字类型，如果参数无法被转换为数字，则返回 NaN。

在看例子之前，我们先看 [ES5 规范 15.7.1.1](http://es5.github.io/#x15.7.1.1) 中关于 Number 的介绍：

[![1](https://camo.githubusercontent.com/8c449ea45bde79874a7dfb630a0b6b1c565e693f30c019d3b2ed75ebc3e19d34/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f544231397932517a4b4c32674b306a535a506858586168765858612d313536322d3136302e6a7067)](https://camo.githubusercontent.com/8c449ea45bde79874a7dfb630a0b6b1c565e693f30c019d3b2ed75ebc3e19d34/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f544231397932517a4b4c32674b306a535a506858586168765858612d313536322d3136302e6a7067)

根据规范，如果 Number 函数不传参数，返回 +0，如果有参数，调用 `ToNumber(value)`。

注意这个 `ToNumber` 表示的是一个底层规范实现上的方法，并没有直接暴露出来。

而 `ToNumber` 则直接给了一个[对应的结果表](http://es5.github.io/#x9.3)。表如下：

| 参数类型  | 结果                                           |
| --------- | ---------------------------------------------- |
| Undefined | NaN                                            |
| Null      | +0                                             |
| Boolean   | 如果参数是 true，返回 1。参数为 false，返回 +0 |
| Number    | 返回与之相等的值                               |
| String    | 这段比较复杂，看例子                           |

让我们写几个例子验证一下：

```javascript
console.log(Number()) // +0

console.log(Number(undefined)) // NaN
console.log(Number(null)) // +0

console.log(Number(false)) // +0
console.log(Number(true)) // 1

console.log(Number("123")) // 123
console.log(Number("-123")) // -123
console.log(Number("1.2")) // 1.2
console.log(Number("000123")) // 123
console.log(Number("-000123")) // -123

console.log(Number("0x11")) // 17

console.log(Number("")) // 0
console.log(Number(" ")) // 0

console.log(Number("123 123")) // NaN
console.log(Number("foo")) // NaN
console.log(Number("100a")) // NaN
```

如果通过 Number 转换函数传入一个字符串，它会试图将其转换成一个整数或浮点数，而且会忽略所有前导的 0，如果有一个字符不是数字，结果都会返回 NaN，鉴于这种严格的判断，我们一般还会使用更加灵活的 parseInt 和 parseFloat 进行转换。

parseInt 只解析整数，parseFloat 则可以解析整数和浮点数，如果字符串前缀是 "0x" 或者"0X"，parseInt 将其解释为十六进制数，parseInt 和 parseFloat 都会跳过任意数量的前导空格，尽可能解析更多数值字符，并忽略后面的内容。如果第一个非空格字符是非法的数字直接量，将最终返回 NaN：

```javascript
console.log(parseInt("3 abc")) // 3
console.log(parseFloat("3.14 abc")) // 3.14
console.log(parseInt("-12.34")) // -12
console.log(parseInt("0xFF")) // 255
console.log(parseFloat(".1")) // 0.1
console.log(parseInt("0.1")) // 0
```

#### 原始值转字符

我们使用 `String` 函数将类型转换成字符串类型，依然先看 [规范15.5.1.1](http://es5.github.io/#x15.5.1.1)中有关 `String` 函数的介绍：

[![img](https://camo.githubusercontent.com/9eb02e20b0786aba5eab6897f43502bba70aeddccdb2bff6bea24b8ec7824a70/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f544231654e6e597a486a31674b306a535a467558586372487058612d313830342d3136322e6a7067)](https://camo.githubusercontent.com/9eb02e20b0786aba5eab6897f43502bba70aeddccdb2bff6bea24b8ec7824a70/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f544231654e6e597a486a31674b306a535a467558586372487058612d313830342d3136322e6a7067)

如果 `String` 函数不传参数，返回空字符串，如果有参数，调用 `ToString(value)`，而 `ToString` 也给了一个对应的结果表。表如下：

| 参数类型  | 结果                                                     |
| --------- | -------------------------------------------------------- |
| Undefined | "undefined"                                              |
| Null      | "null"                                                   |
| Boolean   | 如果参数是 true，返回 "true"。参数为 false，返回 "false" |
| Number    | 又是比较复杂，可以看例子                                 |
| String    | 返回与之相等的值                                         |

让我们写几个例子验证一下：

```javascript
console.log(String()) // 空字符串

console.log(String(undefined)) // undefined
console.log(String(null)) // null

console.log(String(false)) // false
console.log(String(true)) // true

console.log(String(0)) // 0
console.log(String(-0)) // 0
console.log(String(NaN)) // NaN
console.log(String(Infinity)) // Infinity
console.log(String(-Infinity)) // -Infinity
console.log(String(1)) // 1
```

注意这里的 `ToString` 和上一节的 `ToNumber` 都是底层规范实现的方法，并没有直接暴露出来。

#### 原始值转对象

原始值到对象的转换非常简单，原始值通过调用 String()、Number() 或者 Boolean() 构造函数，转换为它们各自的包装对象。

null 和 undefined 属于例外，当将它们用在期望是一个对象的地方都会造成一个类型错误 (TypeError) 异常，而不会执行正常的转换。

```javascript
var a = 1;
console.log(typeof a); // number
var b = new Number(a);
console.log(typeof b); // object
```

#### 对象转布尔值

对象到布尔值的转换非常简单：所有对象(包括数组和函数)都转换为 true。对于包装对象也是这样，举个例子：

```javascript
console.log(Boolean(new Boolean(false))) // true
```

#### 对象转字符串和数字

对象到字符串和对象到数字的转换都是通过调用待转换对象的一个方法来完成的。而 JavaScript 对象有两个不同的方法来执行转换，一个是 `toString`，一个是 `valueOf`。注意这个跟上面所说的 `ToString` 和 `ToNumber` 是不同的，这两个方法是真实暴露出来的方法。

所有的对象除了 null 和 undefined 之外的任何值都具有 `toString` 方法，通常情况下，它和使用 String 方法返回的结果一致。`toString` 方法的作用在于返回一个反映这个对象的字符串，然而这才是情况复杂的开始。

Object.prototype.toString 方法会根据这个对象的[[class]]内部属性，返回由 "[object " 和 class 和 "]" 三个部分组成的字符串。举个例子：

```javascript
Object.prototype.toString.call({a: 1}) // "[object Object]"
({a: 1}).toString() // "[object Object]"
({a: 1}).toString === Object.prototype.toString // true
```

我们可以看出当调用对象的 toString 方法时，其实调用的是 Object.prototype 上的 toString 方法。

然而 JavaScript 下的很多类根据各自的特点，定义了更多版本的 toString 方法。例如:

1. 数组的 toString 方法将每个数组元素转换成一个字符串，并在元素之间添加逗号后合并成结果字符串。
2. 函数的 toString 方法返回源代码字符串。
3. 日期的 toString 方法返回一个可读的日期和时间字符串。
4. RegExp 的 toString 方法返回一个表示正则表达式直接量的字符串。

```javascript
console.log(({}).toString()) // [object Object]

console.log([].toString()) // ""
console.log([0].toString()) // 0
console.log([1, 2, 3].toString()) // 1,2,3
console.log((function(){var a = 1;}).toString()) // function (){var a = 1;}
console.log((/\d+/g).toString()) // /\d+/g
console.log((new Date(2010, 0, 1)).toString()) // Fri Jan 01 2010 00:00:00 GMT+0800 (CST)
```

而另一个转换对象的函数是 valueOf，表示对象的原始值。默认的 valueOf 方法返回这个对象本身，数组、函数、正则简单的继承了这个默认方法，也会返回对象本身。日期是一个例外，它会返回它的一个内容表示: 1970 年 1 月 1 日以来的毫秒数。

```javascript
var date = new Date(2017, 4, 21);
console.log(date.valueOf()) // 1495296000000
```

#### 对象接着转字符串和数字

了解了 toString 方法和 valueOf 方法，我们分析下从对象到字符串是如何转换的。看规范 [ES5 9.8](http://es5.github.io/#x9.8)，其实就是 ToString 方法的对应表，只是这次我们加上 Object 的转换规则：

| 参数类型 | 结果                                                         |
| -------- | ------------------------------------------------------------ |
| Object   | 1. primValue = ToPrimitive(input, String) 2. 返回ToString(primValue). |

所谓的 ToPrimitive 方法，其实就是输入一个值，然后返回一个一定是基本类型的值。

我们总结一下，当我们用 String 方法转化一个值的时候，如果是基本类型，就参照 “原始值转字符” 这一节的对应表，如果不是基本类型，我们会将调用一个 ToPrimitive 方法，将其转为基本类型，然后再参照“原始值转字符” 这一节的对应表进行转换。

其实，从对象到数字的转换也是一样：

| 参数类型 | 结果                                                         |
| -------- | ------------------------------------------------------------ |
| Object   | 1. primValue = ToPrimitive(input, Number) 2. 返回ToNumber(primValue)。 |

虽然转换成基本值都会使用 ToPrimitive 方法，但传参有不同，最后的处理也有不同，转字符串调用的是 `ToString`，转数字调用 `ToNumber`。

#### ToPrimitive

那接下来就要看看 ToPrimitive 了，在了解了 toString 和 valueOf 方法后，这个也很简单。

让我们看规范 9.1，函数语法表示如下：

```javascript
ToPrimitive(input[, PreferredType])
```

第一个参数是 input，表示要处理的输入值。

第二个参数是 PreferredType，非必填，表示希望转换成的类型，有两个值可以选，Number 或者 String。

当不传入 PreferredType 时，如果 input 是日期类型，相当于传入 String，否则，都相当于传入 Number。

如果传入的 input 是 Undefined、Null、Boolean、Number、String 类型，直接返回该值。

如果是 ToPrimitive(obj, Number)，处理步骤如下：

1. 如果 obj 为 基本类型，直接返回
2. 否则，调用 valueOf 方法，如果返回一个原始值，则 JavaScript 将其返回。
3. 否则，调用 toString 方法，如果返回一个原始值，则 JavaScript 将其返回。
4. 否则，JavaScript 抛出一个类型错误异常。

如果是 ToPrimitive(obj, String)，处理步骤如下：

1. 如果 obj为 基本类型，直接返回
2. 否则，调用 toString 方法，如果返回一个原始值，则 JavaScript 将其返回。
3. 否则，调用 valueOf 方法，如果返回一个原始值，则 JavaScript 将其返回。
4. 否则，JavaScript 抛出一个类型错误异常。

#### 对象转字符串

所以总结下，对象转字符串(就是 Number() 函数)可以概括为：

1. 如果对象具有 toString 方法，则调用这个方法。如果他返回一个原始值，JavaScript 将这个值转换为字符串，并返回这个字符串结果。
2. 如果对象没有 toString 方法，或者这个方法并不返回一个原始值，那么 JavaScript 会调用 valueOf 方法。如果存在这个方法，则 JavaScript 调用它。如果返回值是原始值，JavaScript 将这个值转换为字符串，并返回这个字符串的结果。
3. 否则，JavaScript 无法从 toString 或者 valueOf 获得一个原始值，这时它将抛出一个类型错误异常。

#### 对象转数字

对象转数字的过程中，JavaScript 做了同样的事情，只是它会首先尝试 valueOf 方法

1. 如果对象具有 valueOf 方法，且返回一个原始值，则 JavaScript 将这个原始值转换为数字并返回这个数字
2. 否则，如果对象具有 toString 方法，且返回一个原始值，则 JavaScript 将其转换并返回。
3. 否则，JavaScript 抛出一个类型错误异常。

举个例子：

```javascript
console.log(Number({})) // NaN
console.log(Number({a : 1})) // NaN

console.log(Number([])) // 0
console.log(Number([0])) // 0
console.log(Number([1, 2, 3])) // NaN
console.log(Number(function(){var a = 1;})) // NaN
console.log(Number(/\d+/g)) // NaN
console.log(Number(new Date(2010, 0, 1))) // 1262275200000
console.log(Number(new Error('a'))) // NaN
```

注意，在这个例子中，`[]` 和 `[0]` 都返回了 0，而 `[1, 2, 3]` 却返回了一个 NaN。我们分析一下原因：

当我们 `Number([])` 的时候，先调用 `[]` 的 `valueOf` 方法，此时返回 `[]`，因为返回了一个对象而不是原始值，所以又调用了 `toString` 方法，此时返回一个空字符串，接下来调用 `ToNumber` 这个规范上的方法，参照对应表，转换为 `0`, 所以最后的结果为 `0`。

而当我们 `Number([1, 2, 3])` 的时候，先调用 `[1, 2, 3]` 的 `valueOf` 方法，此时返回 `[1, 2, 3]`，再调用 `toString` 方法，此时返回 `1,2,3`，接下来调用 `ToNumber`，参照对应表，因为无法转换为数字，所以最后的结果为 `NaN`。

#### JSON.stringify

值得一提的是：JSON.stringify() 方法可以将一个 JavaScript 值转换为一个 JSON 字符串，实现上也是调用了 toString 方法，也算是一种类型转换的方法。下面讲一讲JSON.stringify 的注意要点：

1.处理基本类型时，与使用toString基本相同，结果都是字符串，除了 undefined

```javascript
console.log(JSON.stringify(null)) // null
console.log(JSON.stringify(undefined)) // undefined，注意这个undefined不是字符串的undefined
console.log(JSON.stringify(true)) // true
console.log(JSON.stringify(42)) // 42
console.log(JSON.stringify("42")) // "42"
```

2.布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。

```javascript
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]); // "[1,"false",false]"
```

3.undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。

```javascript
JSON.stringify({x: undefined, y: Object, z: Symbol("")}); 
// "{}"

JSON.stringify([undefined, Object, Symbol("")]);          
// "[null,null,null]" 
```

4.JSON.stringify 有第二个参数 replacer，它可以是数组或者函数，用来指定对象序列化过程中哪些属性应该被处理，哪些应该被排除。

```javascript
function replacer(key, value) {
  if (typeof value === "string") {
    return undefined;
  }
  return value;
}

var foo = {foundation: "Mozilla", model: "box", week: 45, transport: "car", month: 7};
var jsonString = JSON.stringify(foo, replacer);

console.log(jsonString)
// {"week":45,"month":7}
var foo = {foundation: "Mozilla", model: "box", week: 45, transport: "car", month: 7};
console.log(JSON.stringify(foo, ['week', 'month']));
// {"week":45,"month":7}
```

5.如果一个被序列化的对象拥有 toJSON 方法，那么该 toJSON 方法就会覆盖该对象默认的序列化行为：不是那个对象被序列化，而是调用 toJSON 方法后的返回值会被序列化，例如：

```javascript
var obj = {
  foo: 'foo',
  toJSON: function () {
    return 'bar';
  }
};
JSON.stringify(obj);      // '"bar"'
JSON.stringify({x: obj}); // '{"x":"bar"}'
```

### 类型转换（下）

#### 前言

举个例子：

```javascript
console.log(1 + '1')
```

在 JavaScript 中，这是完全可以运行的，不过你有没有好奇，为什么 1 和 '1' 分属不同的数据类型，为什么就可以进行运算呢？

这其实是因为 JavaScript 自动的将数据类型进行了转换，我们通常称为隐式类型转换。但是我们都知道，`+`运算符既可以用于数字加法，也能用于字符串拼接，那在这个例子中，是将数字 `1` 转成字符串 `'1'`，进行拼接运算？还是将字符串 `'1'` 转成数字 `1`，进行加法运算呢？

先卖个关子，虽然估计你也知道答案。今天，我们就常见的隐式类型转化的场景进行介绍。

#### 一元操作符 +

```javascript
console.log(+'1');
```

当 + 运算符作为一元操作符的时候，查看 [ES5规范1.4.6](http://es5.github.io/#x11.4.6)，会调用 `ToNumber` 处理该值，相当于 `Number('1')`，最终结果返回数字 `1`。

那么下面的这些结果呢？

```javascript
console.log(+[]);
console.log(+['1']);
console.log(+['1', '2', '3']);
console.log(+{});
```

既然是调用 `ToNumber` 方法，回想上节中的内容，当输入的值是对象的时候，先调用 `ToPrimitive(input, Number)` 方法，执行的步骤是：

1. 如果 `obj` 为基本类型，直接返回
2. 否则，调用 `valueOf` 方法，如果返回一个原始值，则 `JavaScript` 将其返回。
3. 否则，调用 `toString` 方法，如果返回一个原始值，则`JavaScript` 将其返回。
4. 否则，`JavaScript` 抛出一个类型错误异常。

以 `+[]` 为例，`[]` 调用 `valueOf` 方法，返回一个空数组，因为不是原始值，调用 `toString` 方法，返回 `""`。

得到返回值后，然后再调用 `ToNumber` 方法，`""` 对应的返回值是 `0`，所以最终返回 `0`。

剩下的例子以此类推。结果是：

```javascript
console.log(+['1']); // 1
console.log(+['1', '2', '3']); // NaN
console.log(+{}); // NaN
```

#### 二元操作符 +

**规范**

现在 `+` 运算符又变成了二元操作符，毕竟它也是加减乘除中的加号

`1 + '1'` 我们知道答案是 '11'，那 `null + 1`、`[] + []`、`[] + {}`、`{} + {}` 呢？

如果要了解这些运算的结果，不可避免的我们要从规范下手。

规范地址：http://es5.github.io/#x11.6.1

不过这次就不直接大段大段的引用规范了，直接给大家讲简化后的内容。

到底当执行 `+` 运算的时候，会执行怎样的步骤呢？让我们根据规范`11.6.1` 来捋一捋：

当计算 value1 + value2时：

1. lprim = ToPrimitive(value1)
2. rprim = ToPrimitive(value2)
3. 如果 lprim 是字符串或者 rprim 是字符串，那么返回 ToString(lprim) 和 ToString(rprim)的拼接结果
4. 返回 ToNumber(lprim) 和 ToNumber(rprim)的运算结果

规范的内容就这样结束了。没有什么新的内容，`ToString`、`ToNumber`、`ToPrimitive`都是在上节中讲到过的内容，所以我们直接进分析阶段：

让我们来举几个例子：

**1.Null 与数字**

```javascript
console.log(null + 1);
```

按照规范的步骤进行分析：

1. lprim = ToPrimitive(null) 因为null是基本类型，直接返回，所以 lprim = null
2. rprim = ToPrimitive(1) 因为 1 是基本类型，直接返回，所以 rprim = 1
3. lprim 和 rprim 都不是字符串
4. 返回 ToNumber(null) 和 ToNumber(1) 的运算结果

接下来：

`ToNumber(null)` 的结果为0，(回想上篇 Number(null))，`ToNumber(1)` 的结果为 1

所以，`null + 1` 相当于 `0 + 1`，最终的结果为数字 `1`。

这个还算简单，看些稍微复杂的：

**2.数组与数组**

```javascript
console.log([] + []);
```

依然按照规范：

1. lprim = ToPrimitive([])，[]是数组，相当于ToPrimitive([], Number)，先调用valueOf方法，返回对象本身，因为不是原始值，调用toString方法，返回空字符串""
2. rprim类似。
3. lprim和rprim都是字符串，执行拼接操作

所以，`[] + []`相当于 `"" + ""`，最终的结果是空字符串`""`。

看个更复杂的：

**3.数组与对象**

```javascript
// 两者结果一致
console.log([] + {});
console.log({} + []);
```

按照规范：

1. lprim = ToPrimitive([])，lprim = ""
2. rprim = ToPrimitive({})，相当于调用 ToPrimitive({}, Number)，先调用 valueOf 方法，返回对象本身，因为不是原始值，调用 toString 方法，返回 "[object Object]"
3. lprim 和 rprim 都是字符串，执行拼接操作

所以，`[] + {}` 相当于 `"" + "[object Object]"`，最终的结果是 "[object Object]"。

下面的例子，可以按照示例类推出结果：

```javascript
console.log(1 + true);
console.log({} + {});
console.log(new Date(2017, 04, 21) + 1) // 这个知道是数字还是字符串类型就行
```

结果是：

```javascript
console.log(1 + true); // 2
console.log({} + {}); // "[object Object][object Object]"
console.log(new Date(2017, 04, 21) + 1) // "Sun May 21 2017 00:00:00 GMT+0800 (CST)1"
```

**注意**

以上的运算都是在 `console.log` 中进行，如果你直接在 `Chrome` 或者 `Firebug` 开发工具中的命令行直接输入，你也许会惊讶的看到一些结果的不同，比如：

[![type1](https://camo.githubusercontent.com/8e967a28b4ae2d67cd7bf57d86463f6c31fb39e1ec20f4e5c6c10d35da9a734c/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f5442315737704941585937674b306a535a4b7a585861696b7058612d313032322d38322e6a7067)](https://camo.githubusercontent.com/8e967a28b4ae2d67cd7bf57d86463f6c31fb39e1ec20f4e5c6c10d35da9a734c/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f5442315737704941585937674b306a535a4b7a585861696b7058612d313032322d38322e6a7067)

我们刚才才说过 `{} + []` 的结果是 `"[object Object]"` 呐，这怎么变成了 `0` 了？

不急，我们尝试着加一个括号：

[![type2](https://camo.githubusercontent.com/54abd5cf06039bff11cb67ff38c14ffece48f57e1bc9fb4e3067dd700e79ab5f/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f5442316a4d384b41686e31674b306a535a4b5058585876555858612d313132342d37362e6a7067)](https://camo.githubusercontent.com/54abd5cf06039bff11cb67ff38c14ffece48f57e1bc9fb4e3067dd700e79ab5f/68747470733a2f2f67772e616c6963646e2e636f6d2f7466732f5442316a4d384b41686e31674b306a535a4b5058585876555858612d313132342d37362e6a7067)

结果又变成了正确的值，这是为什么呢？

其实，在不加括号的时候，`{}` 被当成了一个独立的空代码块，所以 `{} + []` 变成了 `+[]`，结果就变成了 0

同样的问题还出现在 `{} + {}` 上，而且火狐和谷歌的结果还不一样：

```javascript
> {} + {}
// 火狐： NaN
// 谷歌： "[object Object][object Object]"
```

如果 `{}` 被当成一个独立的代码块，那么这句话相当于 `+{}`，相当于 `Number({})`，结果自然是 `NaN`，可是 `Chrome` 却在这里返回了正确的值。

#### ==相等

#### 规范

`"=="` 用于比较两个值是否相等，当要比较的两个值类型不一样的时候，就会发生类型的转换。

关于使用"=="进行比较的时候，具体步骤可以查看[规范11.9.5](http://es5.github.io/#x11.9.3)：

当执行x == y 时：

1. 如果x与y是同一类型：
   1. x是Undefined，返回true
   2. x是Null，返回true
   3. x是数字：
      1. x是NaN，返回false
      2. y是NaN，返回false
      3. x与y相等，返回true
      4. x是+0，y是-0，返回true
      5. x是-0，y是+0，返回true
      6. 返回false
   4. x是字符串，完全相等返回true,否则返回false
   5. x是布尔值，x和y都是true或者false，返回true，否则返回false
   6. x和y指向同一个对象，返回true，否则返回false
2. x是null并且y是undefined，返回true
3. x是undefined并且y是null，返回true
4. x是数字，y是字符串，判断x == ToNumber(y)
5. x是字符串，y是数字，判断ToNumber(x) == y
6. x是布尔值，判断ToNumber(x) == y
7. y是布尔值，判断x ==ToNumber(y)
8. x是字符串或者数字，y是对象，判断x == ToPrimitive(y)
9. x是对象，y是字符串或者数字，判断ToPrimitive(x) == y
10. 返回false

**1. null和undefined**

```javascript
console.log(null == undefined);
```

看规范第2、3步：

> 1. x是null并且y是undefined，返回true

> 1. x是undefined并且y是null，返回true

所以例子的结果自然为 `true`。

如果需要编写判断对象的类型 `type` 函数时，如果输入值是 `undefined`，就返回字符串 `undefined`，如果是 `null`，就返回字符串 `null`。

如果是你，你会怎么写呢？

下面是 jQuery 的写法：

```javascript
function type(obj) {
    if (obj == null) {
        return obj + '';
    }
    ...
}
```

**2. 字符串与数字**

```javascript
console.log('1' == 1);
```

结果肯定是true，问题在于是字符串转化成了数字和数字比较还是数字转换成了字符串和字符串比较呢？

看规范第4、5步：

> 4.x是数字，y是字符串，判断x == ToNumber(y)

> 5.x是字符串，y是数字，判断ToNumber(x) == y

结果很明显，都是转换成数字后再进行比较

**3. 布尔值和其他类型**

```javascript
console.log(true == '2')
```

当要判断的一方出现 `false` 的时候，往往最容易出错，比如上面这个例子，凭直觉应该是 `true`，毕竟 `Boolean('2')` 的结果可是true，但这道题的结果却是false。

归根到底，还是要看规范，规范第6、7步：

> 6.x是布尔值，判断ToNumber(x) == y

> 7.y是布尔值，判断x ==ToNumber(y)

当一方出现布尔值的时候，就会对这一方的值进行ToNumber处理，也就是说true会被转化成1，

`true == '2'` 就相当于 `1 == '2'` 就相当于 `1 == 2`，结果自然是 `false`。

所以当一方是布尔值的时候，会对布尔值进行转换，因为这种特性，所以尽量少使用 `xx == true` 和 `xx == false` 的写法。

比如:

```javascript
// 不建议
if (a == true) {}

// 建议
if (a) {}
// 更好
if (!!a) {}
```

**4. 对象与非对象**

```javascript
console.log( 42 == ['42'])
```

看规范第8、9步：

> 1. x是字符串或者数字，y是对象，判断x == ToPrimitive(y)

> 1. x是对象，y是字符串或者数字，判断ToPrimitive(x) == y

以这个例子为例，会使用 `ToPrimitive` 处理 `['42']`，调用`valueOf`，返回对象本身，再调用 `toString`，返回 `'42'`，所以

`42 == ['42']` 相当于 `42 == '42'` 相当于`42 == 42`，结果为 `true`。

到此为止，我们已经看完了第2、3、4、5、6、7、8、9步，其他的一概返回 false。

**其他**

再多举几个例子进行分析：

```javascript
console.log(false == undefined)
false == undefined` 相当于 `0 == undefined` 不符合上面的情形，执行最后一步 返回 `false
console.log(false == [])
false == []` 相当于 `0 == []` 相当于 `0 == ''` 相当于 `0 == 0`，结果返回 `true
console.log([] == ![])
```

首先会执行 `![]` 操作，转换成 false，相当于 `[] == false` 相当于 `[] == 0` 相当于 `'' == 0` 相当于 `0 == 0`，结果返回 `true`

最后再举一些会让人踩坑的例子：

```javascript
console.log(false == "0")
console.log(false == 0)
console.log(false == "")

console.log("" == 0)
console.log("" == [])

console.log([] == 0)

console.log("" == [null])
console.log(0 == "\n")
console.log([] == 0)
```

以上均返回 true

