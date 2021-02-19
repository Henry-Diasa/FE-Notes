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







