## JS基础

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

![构造函数与实例原型关系](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype1.png)

#### `__proto__`

> 这是每一个`JavaScript`对象(除了 null )都具有的一个属性，叫`__proto__`，这个属性会指向该对象的`原型`。

```javascript
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```

![更新关系图](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype2.png)

既然实例对象和构造函数都可以指向原型，那么原型是否有属性指向构造函数或者实例呢？

#### constructor

> 每个原型都有一个 constructor 属性指向关联的构造函数。

```javascript
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```

![](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype3.png)

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

![](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype4.png)

#### 原型链

那 `Object.prototype `的原型呢？

`null`，我们可以打印：

> null 表示“没有对象”，即该处不应该有值。

```javascript
console.log(Object.prototype.__proto__ === null) // true
```

所以 `Object.prototype.__proto__ `的值为 null 跟 Object.prototype 没有原型，其实表达了一个意思

![](https://github.com/mqyqingfeng/Blog/raw/master/Images/prototype5.png)

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

