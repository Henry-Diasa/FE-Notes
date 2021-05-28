#### 类型和检测方式

**JS内置类型**

- 基本数据类型：`Undefined`、`Null`、`Boolean`、`Number`、`String`、`Symbol`（`es6`新增，表示独一无二的值）和`BigInt`（`es10`新增）；
- 引用数据类型: 对象`Object`（包含普通对象-`Object`，数组对象-`Array`，正则对象-`RegExp`，日期对象-`Date`，数学函数-`Math`，函数对象-`Function`）

**数据存储**

- 基础类型存储在栈内存，被引用或拷贝时，会创建一个完全相等的变量；占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储。
- 引用类型存储在堆内存，存储的是地址，多个引用指向同一个地址，这里会涉及一个“共享”的概念；占据空间大、大小不固定。引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。
- 在编译过程中，如果 JavaScript 引擎判断到一个闭包，也会在堆空间创建换一个`“closure(fn)”`的对象（这是一个内部对象，JavaScript 是无法访问的），用来保存闭包中的变量。所以**闭包中的变量是存储在“堆空间”中的**。
- JavaScript 引擎需要用栈来维护程序执行期间上下文的状态，如果栈空间大了话，所有的数据都存放在栈空间里面，那么会影响到上下文切换的效率，进而又影响到整个程序的执行效率。通常情况下，栈空间都不会设置太大，主要用来存放一些原始类型的小数据。而引用类型的数据占用的空间都比较大，所以这一类数据会被存放到堆中，堆空间很大，能存放很多大的数据，不过缺点是分配内存和回收内存都会占用一定的时间。因此需要“栈”和“堆”两种空间。

**数据类型检测**

- typeof

  > typeof 对于原始类型来说，除了 null 都可以显示正确的类型, typeof` 对于对象来说，除了函数都会显示 `object`，所以说 `typeof` 并不能准确判断变量到底是什么类型,所以想判断一个对象的正确类型，这时候可以考虑使用 `instanceof

  

  ```js
  console.log(typeof 2);               // number
  console.log(typeof true);            // boolean
  console.log(typeof 'str');           // string
  console.log(typeof []);              // object     []数组的数据类型在 typeof 中被解释为 object
  console.log(typeof function(){});    // function
  console.log(typeof {});              // object
  console.log(typeof undefined);       // undefined
  console.log(typeof null);            // object     null 的数据类型被 typeof 解释为 object
  ```

- instanceof

  > instanceof` 可以正确的判断对象的类型，因为内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype
  >
  > `instanceof` 可以准确地判断复杂引用数据类型，但是不能正确判断基础数据类型
  >
  > 而 `typeof` 也存在弊端，它虽然可以判断基础数据类型（`null` 除外），但是引用数据类型中，除了 `function` 类型以外，其他的也无法判断

  

  ```js
  console.log(2 instanceof Number);                    // false
  console.log(true instanceof Boolean);                // false 
  console.log('str' instanceof String);                // false  
  console.log([] instanceof Array);                    // true
  console.log(function(){} instanceof Function);       // true
  console.log({} instanceof Object);                   // true    
  // console.log(undefined instanceof Undefined);
  // console.log(null instanceof Null);
  ```

  ```js
  // 我们也可以试着实现一下 instanceof
  function instanceof(left, right) {
      // 获得类型的原型
      let prototype = right.prototype
      // 获得对象的原型
      left = left.__proto__
      // 判断对象的类型是否等于类型的原型
      while (true) {
      	if (left === null)
      		return false
      	if (prototype === left)
      		return true
      	left = left.__proto__
      }
  }
  ```

- constructor

  ```js
  console.log((2).constructor === Number); // true
  console.log((true).constructor === Boolean); // true
  console.log(('str').constructor === String); // true
  console.log(([]).constructor === Array); // true
  console.log((function() {}).constructor === Function); // true
  console.log(({}).constructor === Object); // true
  ```

  > 如果我创建一个对象，更改它的原型，`constructor`就会变得不可靠了,所以记得重新指向

  

  ```js
  function Fn(){};
   
  Fn.prototype=new Array();
   
  var f=new Fn();
   
  console.log(f.constructor===Fn);    // false
  console.log(f.constructor===Array); // true 
  ```

- Object.prototype.toString.call()

  ```js
  Object.prototype.toString({})       // "[object Object]"
  Object.prototype.toString.call({})  // 同上结果，加上call也ok
  Object.prototype.toString.call(1)    // "[object Number]"
  Object.prototype.toString.call('1')  // "[object String]"
  Object.prototype.toString.call(true)  // "[object Boolean]"
  Object.prototype.toString.call(function(){})  // "[object Function]"
  Object.prototype.toString.call(null)   //"[object Null]"
  Object.prototype.toString.call(undefined) //"[object Undefined]"
  Object.prototype.toString.call(/123/g)    //"[object RegExp]"
  Object.prototype.toString.call(new Date()) //"[object Date]"
  Object.prototype.toString.call([])       //"[object Array]"
  Object.prototype.toString.call(document)  //"[object HTMLDocument]"
  Object.prototype.toString.call(window)   //"[object Window]"
  
  // 从上面这段代码可以看出，Object.prototype.toString.call() 可以很好地判断引用类型，甚至可以把 document 和 window 都区分开来。
  
  ```

  ```js
  function getType(obj){
    let type  = typeof obj;
    if (type !== "object") {    // 先进行typeof判断，如果是基础数据类型，直接返回
      return type;
    }
    // 对于typeof返回结果是object的，再进行如下的判断，正则返回结果
    return Object.prototype.toString.call(obj).replace(/^\[object (\S+)\]$/, '$1');  // 注意正则中间有个空格
  }
  /* 代码验证，需要注意大小写，哪些是typeof判断，哪些是toString判断？思考下 */
  getType([])     // "Array" typeof []是object，因此toString返回
  getType('123')  // "string" typeof 直接返回
  getType(window) // "Window" toString返回
  getType(null)   // "Null"首字母大写，typeof null是object，需toString来判断
  getType(undefined)   // "undefined" typeof 直接返回
  getType()            // "undefined" typeof 直接返回
  getType(function(){}) // "function" typeof能判断，因此首字母小写
  getType(/123/g)      //"RegExp" toString返回
  ```

**数据类型转换**

> ​	JS中的类型转换只有三种情况：转布尔、转数字和转字符串

- 转Boolean

  > 在条件判断时，除了 `undefined`，`null`， `false`， `NaN`， `''`， `0`， `-0`，其他所有值都转为 `true`，包括所有对象

  

  ```js
  Boolean(0)          //false
  Boolean(null)       //false
  Boolean(undefined)  //false
  Boolean(NaN)        //false
  Boolean(1)          //true
  Boolean(13)         //true
  Boolean('12')       //true
  ```

- 对象转原始类型

  > 对象在转换类型的时候，会调用内置的 `[[ToPrimitive]]` 函数，对于该函数来说，算法逻辑一般来说如下
  >
  > - 如果已经是原始类型了，那就不需要转换了
  > - 调用 `x.valueOf()`，如果转换为基础类型，就返回转换的值
  > - 调用 `x.toString()`，如果转换为基础类型，就返回转换的值
  > - 如果都没有返回原始类型，就会报错
  > - 可以重写Symbol.toPrimitive，且该方法优先级最高

  ```js
  let a = {
    valueOf() {
      return 0
    },
    toString() {
      return '1'
    },
    [Symbol.toPrimitive]() {
      return 2
    }
  }
  1 + a // => 3
  ```

- 四则运算符

  > - 运算中其中一方为字符串，那么就会把另一方也转换为字符串
  > - 如果一方不是字符串或者数字，那么会将它转换为数字或者字符串
  > - 除了加法的运算符来说，只要其中一方是数字，那么另一方就会被转为数字

  ```js
  1 + '1' // '11'
  true + true // 2
  4 + [1,2,3] // "41,2,3"
  'a' + + 'b' // -> "aNaN"  +'b' 转化数字为NaN
  4 * '3' // 12
  4 * [] // 0
  4 * [1, 2] // NaN
  ```

- 比较运算符

  > - 如果是对象，就通过 `toPrimitive` 转换对象
  > - 如果是字符串，就通过 `unicode` 字符索引来比较

  ```js
  let a = {
    valueOf() {
      return 0
    },
    toString() {
      return '1'
    }
  }
  a > -1 // true  调用valueOf
  ```

- 强制类型转换

  ```js
  Number(true);        // 1
  Number(false);       // 0
  Number('0111');      //111
  Number(null);        //0
  Number('');          //0
  Number('1a');        //NaN
  Number(-0X11);       //-17
  Number('0X11')       //17
  ```

- Object 的转换规则

  > - 如果部署了 `Symbol.toPrimitive` 方法，优先调用再返回；
  > - 调用 `valueOf()`，如果转换为基础类型，则返回；
  > - 调用 `toString()`，如果转换为基础类型，则返回；
  > - 如果都没有返回基础类型，会报错。

  ```js
  var obj = {
    value: 1,
    valueOf() {
      return 2;
    },
    toString() {
      return '3'
    },
    [Symbol.toPrimitive]() {
      return 4
    }
  }
  console.log(obj + 1); // 输出5
  // 因为有Symbol.toPrimitive，就优先执行这个；如果Symbol.toPrimitive这段代码删掉，则执行valueOf打印结果为3；如果valueOf也去掉，则调用toString返回'31'(字符串拼接)
  // 再看两个特殊的case：
  10 + {}
  // "10[object Object]"，注意：{}会默认调用valueOf是{}，不是基础类型继续转换，调用toString，返回结果"[object Object]"，于是和10进行'+'运算，按照字符串拼接规则来，参考'+'的规则C
  [1,2,undefined,4,5] + 10
  // "1,2,,4,510"，注意[1,2,undefined,4,5]会默认先调用valueOf结果还是这个数组，不是基础数据类型继续转换，也还是调用toString，返回"1,2,,4,5"，然后再和10进行运算，还是按照字符串拼接规则，参考'+'的第3条规则
  ```

- ==隐式类型转换

  > - 如果类型相同，无须进行类型转换；
  > - 如果其中一个操作值是 `null` 或者 `undefined`，那么另一个操作符必须为 `null` 或者 `undefined`，才会返回 `true`，否则都返回 `false`；
  > - 如果其中一个是 `Symbol` 类型，那么返回 `false`；
  > - 两个操作值如果为 `string` 和 number 类型，那么就会将字符串转换为 `number`；
  > - 如果一个操作值是 `boolean`，那么转换成 `number`；
  > - 如果一个操作值为 `object` 且另一方为 `string`、`number` 或者 `symbol`，就会把 `object` 转为原始类型再进行判断（调用 `object` 的 `valueOf/toString` 方法进行转换）。

  ```js
  null == undefined       // true  规则2
  null == 0               // false 规则2
  '' == null              // false 规则2
  '' == 0                 // true  规则4 字符串转隐式转换成Number之后再对比
  '123' == 123            // true  规则4 字符串转隐式转换成Number之后再对比
  0 == false              // true  e规则 布尔型隐式转换成Number之后再对比
  1 == true               // true  e规则 布尔型隐式转换成Number之后再对比
  var a = {
    value: 0,
    valueOf: function() {
      this.value++;
      return this.value;
    }
  };
  // 注意这里a又可以等于1、2、3
  console.log(a == 1 && a == 2 && a ==3);  //true f规则 Object隐式转换
  // 注：但是执行过3遍之后，再重新执行a==3或之前的数字就是false，因为value已经加上去了，这里需要注意一下
  ```

- '+'的隐式类型转换

  > '+' 号操作符，不仅可以用作数字相加，还可以用作字符串拼接。仅当 '+' 号两边都是数字时，进行的是加法运算；如果两边都是字符串，则直接拼接，无须进行隐式类型转换。
  >
  > - 如果其中有一个是字符串，另外一个是 `undefined`、`null` 或布尔型，则调用 `toString()` 方法进行字符串拼接；如果是纯对象、数组、正则等，则默认调用对象的转换方法会存在优先级，然后再进行拼接。
  > - 如果其中有一个是数字，另外一个是 `undefined`、`null`、布尔型或数字，则会将其转换成数字进行加法运算，对象的情况还是参考上一条规则。
  > - 如果其中一个是字符串、一个是数字，则按照字符串规则进行拼接

  ```js
  1 + 2        // 3  常规情况
  '1' + '2'    // '12' 常规情况
  // 下面看一下特殊情况
  '1' + undefined   // "1undefined" 规则1，undefined转换字符串
  '1' + null        // "1null" 规则1，null转换字符串
  '1' + true        // "1true" 规则1，true转换字符串
  '1' + 1n          // '11' 比较特殊字符串和BigInt相加，BigInt转换为字符串
  1 + undefined     // NaN  规则2，undefined转换数字相加NaN
  1 + null          // 1    规则2，null转换为0
  1 + true          // 2    规则2，true转换为1，二者相加为2
  1 + 1n            // 错误  不能把BigInt和Number类型直接混合相加
  '1' + 3           // '13' 规则3，字符串拼接
  ```

- Null 和 undefined

  - 首先 `Undefined` 和 `Null` 都是基本数据类型，这两个基本数据类型分别都只有一个值，就是 `undefined` 和 `null`。
  - `undefined` 代表的含义是未定义， `null` 代表的含义是空对象（其实不是真的对象，请看下面的注意！）。一般变量声明了但还没有定义的时候会返回 `undefined`，`null` 主要用于赋值给一些可能会返回对象的变量，作为初始化。

  > 其实 null 不是对象，虽然 typeof null 会输出 object，但是这只是 JS 存在的一个悠久 Bug。在 JS 的最初版本中使用的是 32 位系统，为了性能考虑使用低位存储变量的类型信息，000 开头代表是对象，然而 null 表示为全零，所以将它错误的判断为 object 。虽然现在的内部类型判断代码已经改变了，但是对于这个 Bug 却是一直流传下来。

  

  - undefined 在 js 中不是一个保留字，这意味着我们可以使用 `undefined` 来作为一个变量名，这样的做法是非常危险的，它会影响我们对 undefined 值的判断。但是我们可以通过一些方法获得安全的 `undefined` 值，比如说 `void 0`。
  - 当我们对两种类型使用 typeof 进行判断的时候，Null 类型化会返回 “object”，这是一个历史遗留的问题。当我们使用双等号对两种类型的值进行比较时会返回 true，使用三个等号时会返回 false。

#### This

- 函数直接调用（foo()），this指向window(严格模式是undefined)

- 对象属性调用（obj.foo()）, this指向obj

- new的方式调用，this永远指向实例对象

- 箭头函数的this只取决包裹箭头函数的第一个普通函数的 this,箭头函数使用bind是无效的

- 最后种情况也就是 bind 这些改变上下文的 API 了，对于这些函数来说，this 取决于第一个参数，如果第一个参数为空，那么就是window

- bind绑定多次，this只取决于**第一次bind**的值

  ```js
  let a = {}
  let fn = function () { console.log(this) }
  fn.bind().bind(a)() // => ?
  
  // fn.bind().bind(a) 等于
  let fn2 = function fn1() {
    return function() {
      return fn.apply()
    }.apply(a)
  }
  fn2()
  ```

- this优先级： new > bind等 > obj.foo > foo

#### apply/call/bind 原理

> JS基础系列有相关内容

#### 执行上下文

- 对于非匿名的立即执行函数需要注意以下一点

```js
var foo = 1
(function foo() {
    foo = 10
    console.log(foo)
}()) // -> ƒ foo() { foo = 10 ; console.log(foo) }
```

> 因为当 `JS` 解释器在遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，然后将函数名称作为这个对象的属性，因此函数内部才可以访问到 `foo`，但是这个值又是只读的，所以对它的赋值并不生效，所以打印的结果还是这个函数，并且外部的值也没有发生更改。

#### 作用域

#### 闭包

- 不一定**返回函数**才产生闭包

  ```js
  // 我们只需要让父级作用域的引用存在即可
  var fun3;
  function fun1() {
    var a = 2
    fun3 = function() {
      console.log(a);
    }
  }
  fun1();
  fun3();
  ```

  ```js
  for (var i = 0; i < 6; i++) {
    setTimeout(() => {
      console.log(i)
    })
  }
  ```

  上面输出结果和如何正确的按照顺序输出

  - 闭包

    ```js
    for(var i = 1;i <= 5;i++){
      (function(j){
        setTimeout(function timer(){
          console.log(j)
        }, 0)
      })(i)
    }
    ```

  - let

    ```js
    for(let i = 1; i <= 5; i++){
      setTimeout(function() {
        console.log(i);
      },0)
    }
    ```

    

  - setTimeout第三个参数

    ```js
    for(var i=1;i<=5;i++){
      setTimeout(function(j) {
        console.log(j)
      }, 0, i)
    }
    ```

#### New的原理

```js
function create(fn, ...args) {
  if(typeof fn !== 'function') {
    throw 'fn must be a function';
  }
	// 1、用new Object() 的方式新建了一个对象obj
  // var obj = new Object()
	// 2、给该对象的__proto__赋值为fn.prototype，即设置原型链
  // obj.__proto__ = fn.prototype

  // 1、2步骤合并
  // 创建一个空对象，且这个空对象继承构造函数的 prototype 属性
  // 即实现 obj.__proto__ === constructor.prototype
  var obj = Object.create(fn.prototype);

	// 3、执行fn，并将obj作为内部this。使用 apply，改变构造函数 this 的指向到新建的对象，这样 obj 就可以访问到构造函数中的属性
  var res = fn.apply(obj, args);
	// 4、如果fn有返回值，则将其作为new操作返回内容，否则返回obj
	return res instanceof Object ? res : obj;
};
```

#### 原型和原型链

#### 继承

#### 面向对象

**编程思想**

- 基本思想是使用对象，类，继承，封装等基本概念来进行程序设计
- 优点
  - 易维护
    - 采用面向对象思想设计的结构，可读性高，由于继承的存在，即使改变需求，那么维护也只是在局部模块，所以维护起来是非常方便和较低成本的
  - 易扩展
  - 开发工作的重用性、继承性高，降低重复工作量。
  - 缩短了开发周期

> 一般面向对象包含：继承，封装，多态，抽象

**1. 对象形式的继承**

浅拷贝

```js
var Person = {
    name: 'allin',
    age: 18,
    address: {
        home: 'home',
        office: 'office',
    }
    sclools: ['x','z'],
};

var programer = {
    language: 'js',
};

function extend(p, c){
    var c = c || {};
    for( var prop in p){
        c[prop] = p[prop];
    }
}
extend(Person, programer);
programer.name;  // allin
programer.address.home;  // home
programer.address.home = 'house';  //house
Person.address.home;  // house
```

> 从上面的结果看出，浅拷贝的缺陷在于修改了子对象中引用类型的值，会影响到父对象中的值，因为在浅拷贝中对引用类型的拷贝只是拷贝了地址，指向了内存中同一个副本

深拷贝

```js
function extendDeeply(p, c){
    var c = c || {};
    for (var prop in p){
        if(typeof p[prop] === "object"){
            c[prop] = (p[prop].constructor === Array)?[]:{};
            extendDeeply(p[prop], c[prop]);
        }else{
            c[prop] = p[prop];
        }
    }
}
```

> 利用递归进行深拷贝，这样子对象的修改就不会影响到父对象

```js
extendDeeply(Person, programer);
programer.address.home = 'allin';
Person.address.home; // home
```

**利用call和apply继承**

```js
function Parent(){
    this.name = "abc";
    this.address = {home: "home"};
}
function Child(){
    Parent.call(this);
    this.language = "js"; 
}
```

**ES5中的Object.create()**

```js
var p = { name : 'allin'};
var obj = Object.create(o);
obj.name; // allin
```

> `Object.create()`作为new操作符的替代方案是ES5之后才出来的。我们也可以自己模拟该方法：

```js
//模拟Object.create()方法
function myCreate(o){
    function F(){};
    F.prototype = o;
    o = new F();
    return o;
}
var p = { name : 'allin'};
var obj = myCreate(o);
obj.name; // allin
```

目前，各大浏览器的最新版本（包括IE9）都部署了这个方法。如果遇到老式浏览器，可以用下面的代码自行部署

```js
　if (!Object.create) {
　　　　Object.create = function (o) {
　　　　　　 function F() {}
　　　　　　F.prototype = o;
　　　　　　return new F();
　　　　};
　　}
```

**2. 类的继承**

> ```
> Object.create()
> ```

```js
function Person(name, age){}
Person.prototype.headCount = 1;
Person.prototype.eat = function(){
    console.log('eating...');
}
function Programmer(name, age, title){}

Programmer.prototype = Object.create(Person.prototype); //建立继承关系
Programmer.prototype.constructor = Programmer;  // 修改constructor的指向
```

**调用父类方法**

```js
function Person(name, age){
    this.name = name;
    this.age = age;
}
Person.prototype.headCount = 1;
Person.prototype.eat = function(){
    console.log('eating...');
}

function Programmer(name, age, title){
    Person.apply(this, arguments); // 调用父类的构造器
}


Programmer.prototype = Object.create(Person.prototype);
Programmer.prototype.constructor = Programmer;

Programmer.prototype.language = "js";
Programmer.prototype.work = function(){
    console.log('i am working code in '+ this.language);
    Person.prototype.eat.apply(this, arguments); // 调用父类上的方法
}
```

**3. 封装**

- 命名空间
  - js是没有命名空间的，因此可以用对象模拟

```js
var app = {};  // 命名空间app
//模块1
app.module1 = {
    name: 'allin',
    f: function(){
        console.log('hi robot');
    }
};
app.module1.name; // "allin"
app.module1.f();  // hi robot
```

> 对象的属性外界是可读可写 如何来达到封装的额目的？答：可通过`闭包+局部变量`来完成

- 在构造函数内部声明局部变量 和普通方法
- 因为作用域的关系 只有构造函数内的方法
- 才能访问局部变量 而方法对于外界是开放的
- 因此可以通过方法来访问 原本外界访问不到的局部变量 达到函数封装的目的

```js
function Girl(name,age){
	var love = '小明';//love 是局部变量 准确说不属于对象 属于这个函数的额激活对象 函数调用时必将产生一个激活对象 love在激活对象身上   激活对象有作用域的关系 有办法访问  加一个函数提供外界访问
	this.name = name;
	this.age = age;
	this.say = function () {
		return love;
	};

	this.movelove = function (){
		love = '小轩'; //35
	}

} 

var g = new Girl('yinghong',22);

console.log(g);
console.log(g.say());//小明
console.log(g.movelove());//undefined  因为35行没有返回
console.log(g.say());//小轩



function fn(){
	function t(){
		//var age = 22;//声明age变量 在t的激活对象上
		age = 22;//赋值操作 t的激活对象上找age属性 ，找不到 找fn的激活对象....再找到 最终找到window.age = 22;
				//不加var就是操作window全局属性
	
	}
	t();
}
console.log(fn());//undefined
```

**4. 静态成员**

> 面向对象中的静态方法-静态属性：没有new对象 也能引用静态方法属性

```js
function Person(name){
    var age = 100;
    this.name = name;
}
//静态成员
Person.walk = function(){
    console.log('static');
};
Person.walk();  // static
```

**5. 私有与公有**

```js
function Person(id){
    // 私有属性与方法
    var name = 'allin';
    var work = function(){
        console.log(this.id);
    };
    //公有属性与方法
    this.id = id;
    this.say = function(){
        console.log('say hello');
        work.call(this);
    };
};
var p1 = new Person(123);
p1.name; // undefined
p1.id;  // 123
p1.say();  // say hello 123
```

**6. 模块化**

```js
var moduleA;
moduleA = function() {
    var prop = 1;

    function func() {}

    return {
        func: func,
        prop: prop
    };
}(); // 立即执行匿名函数
```

**7. 多态**

> 多态:同一个父类继承出来的子类各有各的形态

```js
function Cat(){
	this.eat = '肉';
}

function Tiger(){
	this.color = '黑黄相间';
}

function Cheetah(){
	this.color = '报文';
}

function Lion(){
	this.color = '土黄色';
}

Tiger.prototype =  Cheetah.prototype = Lion.prototype = new Cat();//共享一个祖先 Cat

var T = new Tiger();
var C = new Cheetah();
var L = new Lion();

console.log(T.color);
console.log(C.color);
console.log(L.color);


console.log(T.eat);
console.log(C.eat);
console.log(L.eat);
```

**8. 抽象类**

> 在构造器中 `throw new Error('')`; 抛异常。这样防止这个类被直接调用

```js
function DetectorBase() {
    throw new Error('Abstract class can not be invoked directly!');
}

DetectorBase.prototype.detect = function() {
    console.log('Detection starting...');
};
DetectorBase.prototype.stop = function() {
    console.log('Detection stopped.');
};
DetectorBase.prototype.init = function() {
    throw new Error('Error');
};

// var d = new DetectorBase();
// Uncaught Error: Abstract class can not be invoked directly!

function LinkDetector() {}
LinkDetector.prototype = Object.create(DetectorBase.prototype);
LinkDetector.prototype.constructor = LinkDetector;

var l = new LinkDetector();
console.log(l); //LinkDetector {}__proto__: LinkDetector
l.detect(); //Detection starting...
l.init(); //Uncaught Error: Error
```

#### 事件机制

> 涉及面试题：事件的触发过程是怎么样的？知道什么是事件代理嘛？

**1. 简介**

> 事件流是一个事件沿着特定数据结构传播的过程。冒泡和捕获是事件流在`DOM`中两种不同的传播方法

**事件流有三个阶段**

- 事件捕获阶段
- 处于目标阶段
- 事件冒泡阶段

**事件捕获**

> 事件捕获（`event capturing`）：通俗的理解就是，当鼠标点击或者触发`dom`事件时，浏览器会从根节点开始由外到内进行事件传播，即点击了子元素，如果父元素通过事件捕获方式注册了对应的事件的话，会先触发父元素绑定的事件

**事件冒泡**

> 事件冒泡（dubbed bubbling）：与事件捕获恰恰相反，事件冒泡顺序是由内到外进行事件传播，直到根节点

无论是事件捕获还是事件冒泡，它们都有一个共同的行为，就是事件传播

![img](https://poetries1.gitee.io/img-repo/2019/10/319.png)

**2. 捕获和冒泡**

```html
<div id="div1">
  <div id="div2"></div>
</div>

<script>
    let div1 = document.getElementById('div1');
    let div2 = document.getElementById('div2');
    
    div1.onClick = function(){
        alert('1')
    }
    
    div2.onClick = function(){
        alert('2');
    }

</script>
```

> 当点击 `div2`时，会弹出两个弹出框。在 `ie8/9/10`、`chrome`浏览器，会先弹出”2”再弹出“1”，这就是事件冒泡：事件从最底层的节点向上冒泡传播。事件捕获则跟事件冒泡相反

> W3C的标准是先捕获再冒泡， `addEventListener`的第三个参数决定把事件注册在捕获（`true`）还是冒泡(`false`)

**3. 事件对象**

![img](https://poetries1.gitee.io/img-repo/2019/10/320.png)

**4. 事件流阻止**

> 在一些情况下需要阻止事件流的传播，阻止默认动作的发生

- `event.preventDefault()`：取消事件对象的默认动作以及继续传播。
- `event.stopPropagation()/ event.cancelBubble = true`：阻止事件冒泡。

**事件的阻止在不同浏览器有不同处理**

- 在`IE`下使用 `event.returnValue= false`，
- 在非`IE`下则使用 `event.preventDefault()`进行阻止

**preventDefault与stopPropagation的区别**

- `preventDefault`告诉浏览器不用执行与事件相关联的默认动作（如表单提交）
- `stopPropagation`是停止事件继续冒泡，但是对IE9以下的浏览器无效

**5. 事件注册**

- 通常我们使用 `addEventListener` 注册事件，该函数的第三个参数可以是布尔值，也可以是对象。对于布尔值 `useCapture` 参数来说，该参数默认值为 `false`。`useCapture` 决定了注册的事件是捕获事件还是冒泡事件
- 一般来说，我们只希望事件只触发在目标上，这时候可以使用 `stopPropagation` 来阻止事件的进一步传播。通常我们认为 `stopPropagation` 是用来阻止事件冒泡的，其实该函数也可以阻止捕获事件。`stopImmediatePropagation` 同样也能实现阻止事件，但是还能阻止该事件目标执行别的注册事件

```js
node.addEventListener('click',(event) =>{
	event.stopImmediatePropagation()
	console.log('冒泡')
},false);
// 点击 node 只会执行上面的函数，该函数不会执行
node.addEventListener('click',(event) => {
	console.log('捕获 ')
},true)
```

**6. 事件委托**

- 在`js`中性能优化的其中一个主要思想是减少`dom`操作。
- 节省内存
- 不需要给子节点注销事件

> 假设有`100`个`li`，每个`li`有相同的点击事件。如果为每`个Li`都添加事件，则会造成`dom`访问次数过多，引起浏览器重绘与重排的次数过多，性能则会降低。 使用事件委托则可以解决这样的问题

**原理**

> 实现事件委托是利用了事件的冒泡原理实现的。当我们为最外层的节点添加点击事件，那么里面的`ul`、`li`、`a`的点击事件都会冒泡到最外层节点上，委托它代为执行事件

```html
<ul id="ul">
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ul>
window.onload = function(){
    var ulEle = document.getElementById('ul');
    ul.onclick = function(ev){
        //兼容IE
        ev = ev || window.event;
        var target = ev.target || ev.srcElement;
        
        if(target.nodeName.toLowerCase() == 'li'){
            alert( target.innerHTML);
        }
        
    }
}
```

#### 模块化

> js 中现在比较成熟的有四种模块加载方案：

- 第一种是 CommonJS 方案，它通过 require 来引入模块，通过 module.exports 定义模块的输出接口。这种模块加载方案是服务器端的解决方案，它是以同步的方式来引入模块的，因为在服务端文件都存储在本地磁盘，所以读取非常快，所以以同步的方式加载没有问题。但如果是在浏览器端，由于模块的加载是使用网络请求，因此使用异步加载的方式更加合适。
- 第二种是 AMD 方案，这种方案采用异步加载的方式来加载模块，模块的加载不影响后面语句的执行，所有依赖这个模块的语句都定义在一个回调函数里，等到加载完成后再执行回调函数。require.js 实现了 AMD 规范
- 第三种是 CMD 方案，这种方案和 AMD 方案都是为了解决异步模块加载的问题，sea.js 实现了 CMD 规范。它和require.js的区别在于模块定义时对依赖的处理不同和对依赖模块的执行时机的处理不同。
- 第四种方案是 ES6 提出的方案，使用 import 和 export 的形式来导入导出模块

> 在有 `Babel` 的情况下，我们可以直接使用 `ES6`的模块化

```js
// file a.js
export function a() {}
export function b() {}
// file b.js
export default function() {}

import {a, b} from './a.js'
import XXX from './b.js'
```

**CommonJS**

> `CommonJs` 是 `Node` 独有的规范，浏览器中使用就需要用到 `Browserify`解析了。

```js
// a.js
module.exports = {
    a: 1
}
// or
exports.a = 1

// b.js
var module = require('./a.js')
module.a // -> log 1
```

> 在上述代码中，`module.exports` 和 `exports` 很容易混淆，让我们来看看大致内部实现

```js
var module = require('./a.js')
module.a
// 这里其实就是包装了一层立即执行函数，这样就不会污染全局变量了，
// 重要的是 module 这里，module 是 Node 独有的一个变量
module.exports = {
    a: 1
}
// 基本实现
var module = {
  exports: {} // exports 就是个空对象
}
// 这个是为什么 exports 和 module.exports 用法相似的原因
var exports = module.exports
var load = function (module) {
    // 导出的东西
    var a = 1
    module.exports = a
    return module.exports
};
```

> 再来说说 `module.exports` 和`exports`，用法其实是相似的，但是不能对 `exports` 直接赋值，不会有任何效果。

> 对于 `CommonJS` 和 `ES6` 中的模块化的两者区别是：

- 前者支持动态导入，也就是 `require(${path}/xx.js)`，后者目前不支持，但是已有提案,前者是同步导入，因为用于服务端，文件都在本地，同步导入即使卡住主线程影响也不大。
- 而后者是异步导入，因为用于浏览器，需要下载文件，如果也采用同步导入会对渲染有很大影响
- 前者在导出时都是值拷贝，就算导出的值变了，导入的值也不会改变，所以如果想更新值，必须重新导入一次。
- 但是后者采用实时绑定的方式，导入导出的值都指向同一个内存地址，所以导入值会跟随导出值变化
- 后者会编译成 `require/exports` 来执行的

**AMD**

> `AMD` 是由 `RequireJS` 提出的

**AMD 和 CMD 规范的区别？**

- 第一个方面是在模块定义时对依赖的处理不同。AMD推崇依赖前置，在定义模块的时候就要声明其依赖的模块。而 CMD 推崇就近依赖，只有在用到某个模块的时候再去 require。
- 第二个方面是对依赖模块的执行时机处理不同。首先 AMD 和 CMD 对于模块的加载方式都是异步加载，不过它们的区别在于模块的执行时机，AMD 在依赖模块加载完成后就直接执行依赖模块，依赖模块的执行顺序和我们书写的顺序不一定一致。而 CMD在依赖模块加载完成后并不执行，只是下载而已，等到所有的依赖模块都加载好后，进入回调函数逻辑，遇到 require 语句的时候才执行对应的模块，这样模块的执行顺序就和我们书写的顺序保持一致了。

```js
// CMD
define(function(require, exports, module) {
  var a = require("./a");
  a.doSomething();
  // 此处略去 100 行
  var b = require("./b"); // 依赖可以就近书写
  b.doSomething();
  // ...
});

// AMD 默认推荐
define(["./a", "./b"], function(a, b) {
  // 依赖必须一开始就写好
  a.doSomething();
  // 此处略去 100 行
  b.doSomething();
  // ...
})
```

- **AMD**：`requirejs` 在推广过程中对模块定义的规范化产出，提前执行，推崇依赖前置
- **CMD**：`seajs` 在推广过程中对模块定义的规范化产出，延迟执行，推崇依赖就近
- **CommonJs**：模块输出的是一个值的 `copy`，运行时加载，加载的是一个对象（`module.exports` 属性），该对象只有在脚本运行完才会生成
- **ES6 Module**：模块输出的是一个值的引用，编译时输出接口，`ES6`模块不是对象，它对外接口只是一种静态定义，在代码静态解析阶段就会生成。

**谈谈对模块化开发的理解**

- 我对模块的理解是，一个模块是实现一个特定功能的一组方法。在最开始的时候，js 只实现一些简单的功能，所以并没有模块的概念，但随着程序越来越复杂，代码的模块化开发变得越来越重要。
- 由于函数具有独立作用域的特点，最原始的写法是使用函数来作为模块，几个函数作为一个模块，但是这种方式容易造成全局变量的污染，并且模块间没有联系。
- 后面提出了对象写法，通过将函数作为一个对象的方法来实现，这样解决了直接使用函数作为模块的一些缺点，但是这种办法会暴露所有的所有的模块成员，外部代码可以修改内部属性的值。
- 现在最常用的是立即执行函数的写法，通过利用闭包来实现模块私有作用域的建立，同时不会对全局作用域造成污染。

#### Iterator迭代器

> `Iterator`（迭代器）是一种接口，也可以说是一种规范。为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署`Iterator`接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。

Iterator语法：

```js
const obj = {
    [Symbol.iterator]:function(){}
}
```

> `[Symbol.iterator]` 属性名是固定的写法，只要拥有了该属性的对象，就能够用迭代器的方式进行遍历。

- 迭代器的遍历方法是首先获得一个迭代器的指针，初始时该指针指向第一条数据之前，接着通过调用 next 方法，改变指针的指向，让其指向下一条数据

- 每一次的

   

  ```
  next
  ```

   

  都会返回一个对象，该对象有两个属性

  - `value` 代表想要获取的数据
  - `done` 布尔值，false表示当前指针指向的数据有值，true表示遍历已经结束

**Iterator 的作用有三个：**

- 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
- 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
- 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
- 不断调用指针对象的next方法，直到它指向数据结构的结束位置。

> 每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。

```js
let arr = [{num:1},2,3]
let it = arr[Symbol.iterator]() // 获取数组中的迭代器
console.log(it.next())  // { value: Object { num: 1 }, done: false }
console.log(it.next())  // { value: 2, done: false }
console.log(it.next())  // { value: 3, done: false }
console.log(it.next())  // { value: undefined, done: true }
```

> 对象没有布局Iterator接口，无法使用`for of` 遍历。下面使得对象具备Iterator接口

- 一个数据结构只要有Symbol.iterator属性，就可以认为是“可遍历的”
- 原型部署了Iterator接口的数据结构有三种，具体包含四种，分别是数组，类似数组的对象，Set和Map结构

**为什么对象（Object）没有部署Iterator接口呢？**

- 一是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。然而遍历遍历器是一种线性处理，对于非线性的数据结构，部署遍历器接口，就等于要部署一种线性转换
- 对对象部署`Iterator`接口并不是很必要，因为`Map`弥补了它的缺陷，又正好有`Iteraotr`接口

```js
let obj = {
    id: '123',
    name: '张三',
    age: 18,
    gender: '男',
    hobbie: '睡觉'
}

obj[Symbol.iterator] = function () {
    let keyArr = Object.keys(obj)
    let index = 0
    return {
        next() {
            return index < keyArr.length ? {
                value: {
                    key: keyArr[index],
                    val: obj[keyArr[index++]]
                }
            } : {
                done: true
            }
        }
    }
}

for (let key of obj) {
  console.log(key)
}
```

![img](http://img-repo.poetries.top/images/20210407141323.png)

### Promise

> 这里你谈 `promise`的时候，除了将他解决的痛点以及常用的 `API` 之外，最好进行拓展把 `eventloop` 带进来好好讲一下，`microtask`(微任务)、`macrotask`(任务) 的执行顺序，如果看过 `promise` 源码，最好可以谈一谈 原生 `Promise` 是如何实现的。`Promise` 的关键点在于`callback` 的两个参数，一个是 `resovle`，一个是 `reject`。还有就是 `Promise` 的链式调用（`Promise.then()`，每一个 `then` 都是一个责任人）

- `Promise` 是 `ES6` 新增的语法，解决了回调地狱的问题。
- 可以把 `Promise`看成一个状态机。初始是 `pending` 状态，可以通过函数 `resolve` 和 `reject`，将状态转变为 `resolved` 或者 `rejected` 状态，状态一旦改变就不能再次变化。
- `then` 函数会返回一个 `Promise` 实例，并且该返回值是一个新的实例而不是之前的实例。因为 `Promise` 规范规定除了 `pending` 状态，其他状态是不可以改变的，如果返回的是一个相同实例的话，多个 `then` 调用就失去意义了。 对于 `then` 来说，本质上可以把它看成是 `flatMap`

**1. Promise 的基本情况**

> 简单来说它就是一个容器，里面保存着某个未来才会结束的事件（通常是异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息

一般 Promise 在执行过程中，必然会处于以下几种状态之一。

- 待定（`pending`）：初始状态，既没有被完成，也没有被拒绝。
- 已完成（`fulfilled`）：操作成功完成。
- 已拒绝（`rejected`）：操作失败。

> 待定状态的 `Promise` 对象执行的话，最后要么会通过一个值完成，要么会通过一个原因被拒绝。当其中一种情况发生时，我们用 `Promise` 的 `then` 方法排列起来的相关处理程序就会被调用。因为最后 `Promise.prototype.then` 和 `Promise.prototype.catch` 方法返回的是一个 `Promise`， 所以它们可以继续被链式调用

关于 Promise 的状态流转情况，有一点值得注意的是，内部状态改变之后不可逆，你需要在编程过程中加以注意。文字描述比较晦涩，我们直接通过一张图就能很清晰地看出 Promise 内部状态流转的情况

![img](http://img-repo.poetries.top/images/20210414175500.png)

从上图可以看出，我们最开始创建一个新的 `Promise` 返回给 `p1` ，然后开始执行，状态是 pending，当执行 `resolve`之后状态就切换为 `fulfilled`，执行 `reject` 之后就变为 `rejected` 的状态

**2. Promise 的静态方法**

- all 方法
  - 语法： `Promise.all（iterable）`
  - 参数： 一个可迭代对象，如 `Array`。
  - 描述： 此方法对于汇总多个promise的结果很有用，在 ES6 中可以将多个Promise.all异步请求并行操作，返回结果一般有下面两种情况。
    - 当所有结果成功返回时按照请求顺序返回成功结果。
    - 当其中有一个失败方法时，则进入失败方法
- 我们来看下业务的场景，对于下面这个业务场景页面的加载，将多个请求合并到一起，用 all 来实现可能效果会更好，请看代码片段

```js
// 在一个页面中需要加载获取轮播列表、获取店铺列表、获取分类列表这三个操作，页面需要同时发出请求进行页面渲染，这样用 `Promise.all` 来实现，看起来更清晰、一目了然。


//1.获取轮播数据列表
function getBannerList(){
  return new Promise((resolve,reject)=>{
      setTimeout(function(){
        resolve('轮播数据')
      },300) 
  })
}
//2.获取店铺列表
function getStoreList(){
  return new Promise((resolve,reject)=>{
    setTimeout(function(){
      resolve('店铺数据')
    },500)
  })
}
//3.获取分类列表
function getCategoryList(){
  return new Promise((resolve,reject)=>{
    setTimeout(function(){
      resolve('分类数据')
    },700)
  })
}
function initLoad(){ 
  Promise.all([getBannerList(),getStoreList(),getCategoryList()])
  .then(res=>{
    console.log(res) 
  }).catch(err=>{
    console.log(err)
  })
} 
initLoad()
```

- allSettled方法
  - `Promise.allSettled` 的语法及参数跟 `Promise.all` 类似，其参数接受一个 `Promise` 的数组，返回一个新的 `Promise`。`唯一的不同在于，执行完之后不会失败`，也就是说当 `Promise.allSettled` 全部处理完成后，我们可以拿到每个 `Promise` 的状态，而不管其是否处理成功
- 我们来看一下用 `allSettled` 实现的一段代码

```js
const resolved = Promise.resolve(2);
const rejected = Promise.reject(-1);
const allSettledPromise = Promise.allSettled([resolved, rejected]);
allSettledPromise.then(function (results) {
  console.log(results);
});
// 返回结果：
// [
//    { status: 'fulfilled', value: 2 },
//    { status: 'rejected', reason: -1 }
// ]
```

> 从上面代码中可以看到，`Promise.allSettled` 最后返回的是一个数组，记录传进来的参数中每个 Promise 的返回值，这就是和 all 方法不太一样的地方。

- any方法
  - 语法： `Promise.any（iterable）`
  - 参数： `iterable` 可迭代的对象，例如 `Array`。
  - 描述： `any` 方法返回一个 `Promise`，只要参数 `Promise` 实例有一个变成 `fulfilled`状态，最后 `any`返回的实例就会变成 `fulfilled` 状态；如果所有参数 `Promise` 实例都变成 `rejected` 状态，包装实例就会变成 `rejected` 状态。

```js
const resolved = Promise.resolve(2);
const rejected = Promise.reject(-1);
const anyPromise = Promise.any([resolved, rejected]);
anyPromise.then(function (results) {
  console.log(results);
});
// 返回结果：
// 2
```

> 从改造后的代码中可以看出，只要其中一个 `Promise` 变成 `fulfilled`状态，那么 `any` 最后就返回这个`p romise`。由于上面 `resolved` 这个 Promise 已经是 `resolve` 的了，故最后返回结果为 `2`

- race方法
  - 语法： `Promise.race（iterable）`
  - 参数： `iterable` 可迭代的对象，例如 `Array`。
  - 描述： `race`方法返回一个 `Promise`，只要参数的 `Promise` 之中有一个实例率先改变状态，则 `race` 方法的返回状态就跟着改变。那个率先改变的 `Promise` 实例的返回值，就传递给 `race` 方法的回调函数
- 我们来看一下这个业务场景，对于图片的加载，特别适合用 race 方法来解决，将图片请求和超时判断放到一起，用 race 来实现图片的超时判断。请看代码片段。

```js
//请求某个图片资源
function requestImg(){
  var p = new Promise(function(resolve, reject){
    var img = new Image();
    img.onload = function(){ resolve(img); }
    img.src = 'http://www.baidu.com/img/flexible/logo/pc/result.png';
  });
  return p;
}
//延时函数，用于给请求计时
function timeout(){
  var p = new Promise(function(resolve, reject){
    setTimeout(function(){ reject('图片请求超时'); }, 5000);
  });
  return p;
}
Promise.race([requestImg(), timeout()])
.then(function(results){
  console.log(results);
})
.catch(function(reason){
  console.log(reason);
});


// 从上面的代码中可以看出，采用 Promise 的方式来判断图片是否加载成功，也是针对 Promise.race 方法的一个比较好的业务场景
```

![img](http://img-repo.poetries.top/images/20210414210720.png)

#### Generator

> `Generator` 是 `ES6`中新增的语法，和 `Promise` 一样，都可以用来异步编程。Generator函数可以说是Iterator接口的具体实现方式。Generator 最大的特点就是可以控制函数的执行。

- `function*` 用来声明一个函数是生成器函数，它比普通的函数声明多了一个`*`,`*`的位置比较随意可以挨着 `function` 关键字，也可以挨着函数名
- `yield` 产出的意思，这个关键字只能出现在生成器函数体内，但是生成器中也可以没有`yield` 关键字，函数遇到 `yield` 的时候会暂停，并把 `yield` 后面的表达式结果抛出去
- `next`作用是将代码的控制权交还给生成器函数

```js
function *foo(x) {
  let y = 2 * (yield (x + 1))
  let z = yield (y / 3)
  return (x + y + z)
}
let it = foo(5)
console.log(it.next())   // => {value: 6, done: false}
console.log(it.next(12)) // => {value: 8, done: false}
console.log(it.next(13)) // => {value: 42, done: true}
```

上面这个示例就是一个Generator函数，我们来分析其执行过程：

- 首先 Generator 函数调用时它会返回一个迭代器
- 当执行第一次 next 时，传参会被忽略，并且函数暂停在 yield (x + 1) 处，所以返回 5 + 1 = 6
- 当执行第二次 next 时，传入的参数等于上一个 yield 的返回值，如果你不传参，yield 永远返回 undefined。此时 let y = 2 * 12，所以第二个 yield 等于 2 * 12 / 3 = 8
- 当执行第三次 next 时，传入的参数会传递给 z，所以 z = 13, x = 5, y = 24，相加等于 42

`yield`实际就是暂缓执行的标示，每执行一次`next()`，相当于指针移动到下一个`yield`位置

![img](http://img-repo.poetries.top/images/20210408100806.png)

> **总结一下**，`Generator`函数是`ES6`提供的一种异步编程解决方案。通过`yield`标识位和`next()`方法调用，实现函数的分段执行

**遍历器对象生成函数，最大的特点是可以交出函数的执行权**

- `function` 关键字与函数名之间有一个星号；
- 函数体内部使用 `yield`表达式，定义不同的内部状态；
- `next`指针移向下一个状态

> 这里你可以说说 `Generator`的异步编程，以及它的语法糖 `async` 和 `awiat`，传统的异步编程。`ES6` 之前，异步编程大致如下

- 回调函数
- 事件监听
- 发布/订阅

> 传统异步编程方案之一：协程，多个线程互相协作，完成异步任务。

```js
// 使用 * 表示这是一个 Generator 函数
// 内部可以通过 yield 暂停代码
// 通过调用 next 恢复执行
function* test() {
  let a = 1 + 2;
  yield 2;
  yield 3;
}
let b = test();
console.log(b.next()); // >  { value: 2, done: false }
console.log(b.next()); // >  { value: 3, done: false }
console.log(b.next()); // >  { value: undefined, done: true }
```

> 从以上代码可以发现，加上 `*`的函数执行后拥有了 `next` 函数，也就是说函数执行后返回了一个对象。每次调用 `next` 函数可以继续执行被暂停的代码。以下是 `Generator` 函数的简单实现

```js
// cb 也就是编译过的 test 函数
function generator(cb) {
  return (function() {
    var object = {
      next: 0,
      stop: function() {}
    };

    return {
      next: function() {
        var ret = cb(object);
        if (ret === undefined) return { value: undefined, done: true };
        return {
          value: ret,
          done: false
        };
      }
    };
  })();
}
// 如果你使用 babel 编译后可以发现 test 函数变成了这样
function test() {
  var a;
  return generator(function(_context) {
    while (1) {
      switch ((_context.prev = _context.next)) {
        // 可以发现通过 yield 将代码分割成几块
        // 每次执行 next 函数就执行一块代码
        // 并且表明下次需要执行哪块代码
        case 0:
          a = 1 + 2;
          _context.next = 4;
          return 2;
        case 4:
          _context.next = 6;
          return 3;
		// 执行完毕
        case 6:
        case "end":
          return _context.stop();
      }
    }
  });
}
```

#### async/await

> `Generator` 函数的语法糖。有更好的语义、更好的适用性、返回值是 `Promise`。

- await 和 promise 一样，更多的是考笔试题，当然偶尔也会问到和 promise 的一些区别。
- await 相比直接使用 Promise 来说，优势在于处理 then 的调用链，能够更清晰准确的写出代码。缺点在于滥用 await 可能会导致性能问题，因为 await 会阻塞代码，也许之后的异步代码并不依赖于前者，但仍然需要等待前者完成，导致代码失去了并发性，此时更应该使用 Promise.all。
- 一个函数如果加上 async ，那么该函数就会返回一个 Promise

> - `async => *`
> - `await => yield`

```js
// 基本用法

async function timeout (ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms)    
  })
}
async function asyncConsole (value, ms) {
  await timeout(ms)
  console.log(value)
}
asyncConsole('hello async and await', 1000)
```

下面来看一个使用 `await` 的代码。

```js
var a = 0
var b = async () => {
  a = a + await 10
  console.log('2', a) // -> '2' 10
  a = (await 10) + a
  console.log('3', a) // -> '3' 20
}
b()
a++
console.log('1', a) // -> '1' 1
```

- 首先函数`b` 先执行，在执行到 `await 10` 之前变量 `a` 还是 `0`，因为在 `await` 内部实现了 `generators` ，`generators` 会保留堆栈中东西，所以这时候 `a = 0` 被保存了下来
- 因为 `await` 是异步操作，遇到`await`就会立即返回一个`pending`状态的`Promise`对象，暂时返回执行代码的控制权，使得函数外的代码得以继续执行，所以会先执行 `console.log('1', a)`
- 这时候同步代码执行完毕，开始执行异步代码，将保存下来的值拿出来使用，这时候 `a = 10`
- 然后后面就是常规执行代码了

**优缺点：**

> `async/await`的优势在于处理 then 的调用链，能够更清晰准确的写出代码，并且也能优雅地解决回调地狱问题。当然也存在一些缺点，因为 await 将异步代码改造成了同步代码，如果多个异步代码没有依赖性却使用了 await 会导致性能上的降低。

**async原理**

> `async/await`语法糖就是使用`Generator`函数+自动执行器来运作的

```js
// 定义了一个promise，用来模拟异步请求，作用是传入参数++
function getNum(num){
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(num+1)
        }, 1000)
    })
}

//自动执行器，如果一个Generator函数没有执行完，则递归调用
function asyncFun(func){
  var gen = func();

  function next(data){
    var result = gen.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

// 所需要执行的Generator函数，内部的数据在执行完成一步的promise之后，再调用下一步
var func = function* (){
  var f1 = yield getNum(1);
  var f2 = yield getNum(f1);
  console.log(f2) ;
};
asyncFun(func);
```

- 在执行的过程中，判断一个函数的`promise`是否完成，如果已经完成，将结果传入下一个函数，继续重复此步骤
- 每一个 `next()` 方法返回值的 `value` 属性为一个 `Promise` 对象，所以我们为其添加 `then` 方法， 在 `then` 方法里面接着运行 `next` 方法挪移遍历器指针，直到 `Generator`函数运行完成

![img](http://img-repo.poetries.top/images/20210414210917.png)

#### 事件循环

![img](http://img-repo.poetries.top/images/20210516161332.png)

- 默认代码从上到下执行，执行环境通过`script`来执行（宏任务）
- 在代码执行过程中，调用定时器 `promise` `click`事件...不会立即执行，需要等待当前代码全部执行完毕
- 给异步方法划分队列，分别存放到微任务（立即存放）和宏任务（时间到了或事情发生了才存放）到队列中
- `script`执行完毕后，会清空所有的微任务
- 微任务执行完毕后，会渲染页面（不是每次都调用）
- 再去宏任务队列中看有没有到达时间的，拿出来其中一个执行
- 执行完毕后，按照上述步骤不停的循环

**例子**

![img](http://img-repo.poetries.top/images/20210516162034.png) ![img](http://img-repo.poetries.top/images/20210516162330.png)

> 自动执行的情况 会输出 listener1 listener2 task1 task2

![img](http://img-repo.poetries.top/images/20210516162704.png) ![img](http://img-repo.poetries.top/images/20210516163038.png)

> 如果手动点击click 会一个宏任务取出来一个个执行，先执行click的宏任务，取出微任务去执行。会输出 listener1 task1 listener2 task2

![img](http://img-repo.poetries.top/images/20210516163647.png)

```js
console.log(1)

async function asyncFunc(){
  console.log(2)
  // await xx ==> promise.resolve(()=>{console.log(3)}).then()
  // console.log(3) 放到promise.resolve或立即执行
  await console.log(3) 
  // 相当于把console.log(4)放到了then promise.resolve(()=>{console.log(3)}).then(()=>{
  //   console.log(4)
  // })
  // 微任务谁先注册谁先执行
  console.log(4)
}

setTimeout(()=>{console.log(5)})

const promise = new Promise((resolve,reject)=>{
  console.log(6)
  resolve(7)
})

promise.then(d=>{console.log(d)})

asyncFunc()

console.log(8)

// 输出 1 6 2 3 8 7 4 5
```

**1. 浏览器事件循环**

> 涉及面试题：异步代码执行顺序？解释一下什么是 `Event Loop` ？

> JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变

> js代码执行过程中会有很多任务，这些任务总的分成两类：

- 同步任务
- 异步任务

> 当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。，我们用导图来说明：

![img](https://poetries1.gitee.io/img-repo/2020/09/10.png)

**我们解释一下这张图：**

- 同步和异步任务分别进入不同的执行"场所"，同步的进入主线程，异步的进入Event Table并注册函数。
- 当指定的事情完成时，Event Table会将这个函数移入Event Queue。
- 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，进入主线程执行。
- 上述过程会不断重复，也就是常说的Event Loop(事件循环)。

> 那主线程执行栈何时为空呢？js引擎存在monitoring process进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数

以上就是js运行的整体流程

**面试中该如何回答呢？ 下面是我个人推荐的回答：**

- 首先js 是单线程运行的，在代码执行的时候，通过将不同函数的执行上下文压入执行栈中来保证代码的有序执行
- 在执行同步代码的时候，如果遇到了异步事件，js 引擎并不会一直等待其返回结果，而是会将这个事件挂起，继续执行执行栈中的其他任务
- 当同步事件执行完毕后，再将异步事件对应的回调加入到与当前执行栈中不同的另一个任务队列中等待执行
- 任务队列可以分为宏任务对列和微任务对列，当当前执行栈中的事件执行完毕后，js 引擎首先会判断微任务对列中是否有任务可以执行，如果有就将微任务队首的事件压入栈中执行
- 当微任务对列中的任务都执行完成后再去判断宏任务对列中的任务。

```js
setTimeout(function() {
  console.log(1)
}, 0);
new Promise(function(resolve, reject) {
  console.log(2);
  resolve()
}).then(function() {
  console.log(3)
});
process.nextTick(function () {
  console.log(4)
})
console.log(5)
```

- 第一轮：主线程开始执行，遇到`setTimeout`，将setTimeout的回调函数丢到宏任务队列中，在往下执行`new Promise`立即执行，输出2，then的回调函数丢到微任务队列中，再继续执行，遇到`process.nextTick`，同样将回调函数扔到为任务队列，再继续执行，输出5，当所有同步任务执行完成后看有没有可以执行的微任务，发现有then函数和`nextTick`两个微任务，先执行哪个呢？`process.nextTick`指定的异步任务总是发生在所有异步任务之前，因此先执行process.nextTick输出4然后执行then函数输出3，第一轮执行结束。
- 第二轮：从宏任务队列开始，发现setTimeout回调，输出1执行完毕，因此结果是25431

> `JS` 在执行的过程中会产生执行环境，这些执行环境会被顺序的加入到执行栈中。如果遇到异步的代码，会被挂起并加入到 `Task`（有多种 `task`） 队列中。一旦执行栈为空，`Event` `Loop` 就会从 `Task` 队列中拿出需要执行的代码并放入执行栈中执行，所以本质上来说 `JS` 中的异步还是同步行为

![img](https://poetries1.gitee.io/img-repo/2020/07/fe/4.png)

```js
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

console.log('script end');
```

> 不同的任务源会被分配到不同的 `Task` 队列中，任务源可以分为 微任务（`microtask`） 和 宏任务（`macrotask`）。在 `ES6` 规范中，`microtask` 称为 `jobs`，`macrotask` 称为 `task`

```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

new Promise((resolve) => {
    console.log('Promise')
    resolve()
}).then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
// script start => Promise => script end => promise1 => promise2 => setTimeout
```

> 以上代码虽然 `setTimeout` 写在 `Promise` 之前，但是因为 `Promise` 属于微任务而 `setTimeout` 属于宏任务

**微任务**

- `process.nextTick`

- `promise`

- `Object.observe`

- ```
  MutationObserver
  ```

  - ![img](http://img-repo.poetries.top/images/20210516165328.png)

**宏任务**

- `script`
- `setTimeout`
- `setInterval`
- `setImmediate`
- `I/O` 网络请求完成、文件读写完成事件
- `UI rendering`
- 用户交互事件（比如鼠标点击、滚动页面、放大缩小等）

> 宏任务中包括了 `script` ，浏览器会先执行一个宏任务，接下来有异步代码的话就先执行微任务

![img](http://img-repo.poetries.top/images/20210414213126.png)

**所以正确的一次 Event loop 顺序是这样的**

- 执行同步代码，这属于宏任务
- 执行栈为空，查询是否有微任务需要执行
- 执行所有微任务
- 必要的话渲染 UI
- 然后开始下一轮 `Event loop`，执行宏任务中的异步代码

> 通过上述的 `Event loop` 顺序可知，如果宏任务中的异步代码有大量的计算并且需要操作 `DOM` 的话，为了更快的响应界面响应，我们可以把操作 `DOM` 放入微任务中

- JavaScript 引擎首先从宏任务队列（macrotask queue）中取出第一个任务
- 执行完毕后，再将微任务（microtask queue）中的所有任务取出，按照顺序分别全部执行（这里包括不仅指开始执行时队列里的微任务），如果在这一步过程中产生新的微任务，也需要执行；
- 然后再从宏任务队列中取下一个，执行完毕后，再次将 microtask queue 中的全部取出，循环往复，直到两个 queue 中的任务都取完。

![img](http://img-repo.poetries.top/images/20210414211816.png)

> 总结起来就是：`一次 Eventloop 循环会处理一个宏任务和所有这次循环中产生的微任务`。

**2. Node 中的 Event loop**

> 当 Node.js 开始启动时，会初始化一个 Eventloop，处理输入的代码脚本，这些脚本会进行 API 异步调用，`process.nextTick()` 方法会开始处理事件循环。下面就是 Node.js 官网提供的 `Eventloop` 事件循环参考流程

- `Node` 中的 `Event loop` 和浏览器中的不相同。
- `Node` 的 `Event loop` 分为`6`个阶段，它们会按照顺序反复运行

![img](http://img-repo.poetries.top/images/20210414211850.png) ![img](http://img-repo.poetries.top/images/20210516214402.png) ![img](http://img-repo.poetries.top/images/20210516221825.png)

- 每次执行执行一个宏任务后会清空微任务（执行顺序和浏览器一致，在node11版本以上）
- `process.nextTick` node中的微任务，当前执行栈的底部，优先级比`promise`要高

> 整个流程分为六个阶段，当这六个阶段执行完一次之后，才可以算得上执行了一次 Eventloop 的循环过程。我们来分别看下这六个阶段都做了哪些事情。

- **Timers 阶段**：这个阶段执行 `setTimeout` 和 `setInterval`的回调函数，简单理解就是由这两个函数启动的回调函数。
- **I/O callbacks 阶段**：这个阶段主要执行系统级别的回调函数，比如 TCP 连接失败的回调。
- **idle，prepare 阶段**：仅系统内部使用，你只需要知道有这 2 个阶段就可以。
- **poll 阶段**：`poll` 阶段是一个重要且复杂的阶段，几乎所有 `I/O` 相关的回调，都在这个阶段执行（除了`setTimeout`、`setInterval`、`setImmediate` 以及一些因为 `exception` 意外关闭产生的回调）。`检索新的 I/O 事件，执行与 I/O 相关的回调`，其他情况 Node.js 将在适当的时候在此阻塞。这也是最复杂的一个阶段，所有的事件循环以及回调处理都在这个阶段执行。这个阶段的主要流程如下图所示。

![img](http://img-repo.poetries.top/images/20210414212124.png)

- **check 阶段**：`setImmediate()` 回调函数在这里执行，`setImmediate` 并不是立马执行，而是当事件循环 `poll 中没有新的事件处理时就执行该部分`，如下代码所示。

```js
const fs = require('fs');
setTimeout(() => { // 新的事件循环的起点
    console.log('1'); 
}, 0);
setImmediate( () => {
    console.log('setImmediate 1');
});
/// 将会在 poll 阶段执行
fs.readFile('./test.conf', {encoding: 'utf-8'}, (err, data) => {
    if (err) throw err;
    console.log('read file success');
});
/// 该部分将会在首次事件循环中执行
Promise.resolve().then(()=>{
    console.log('poll callback');
});
// 首次事件循环执行
console.log('2');
```

在这一代码中有一个非常奇特的地方，就是 `setImmediate` 会在 `setTimeout` 之后输出。有以下几点原因：

> - `setTimeout` 如果不设置时间或者设置时间为 `0`，则会默认为 `1ms`
> - 主流程执行完成后，超过 `1ms` 时，会将 `setTimeout` 回调函数逻辑插入到待执行回调函数 `poll` 队列中；
> - 由于当前 `poll` 队列中存在可执行回调函数，因此需要先执行完，待完全执行完成后，才会执行`check：setImmediate`。

> 因此这也验证了这句话，`先执行回调函数，再执行 setImmediate`

- **close callbacks 阶段**：执行一些关闭的回调函数，如 `socket.on('close', ...)`

> 除了把 Eventloop 的宏任务细分到不同阶段外。node 还引入了一个新的任务队列 `Process.nextTick()`

可以认为，`Process.nextTick()` 会在上述各个阶段结束时，在`进入下一个阶段之前立即执行`（优先级甚至超过 `microtask` 队列）

**事件循环的主要包含微任务和宏任务。具体是怎么进行循环的呢**

![img](http://img-repo.poetries.top/images/20210424174311.png)

- **微任务**：在 Node.js 中微任务包含 2 种——`process.nextTick` 和 `Promise`。`微任务在事件循环中优先级是最高的`，因此在同一个事件循环中有其他任务存在时，优先执行微任务队列。并且`process.nextTick 和 Promise`也存在优先级，`process.nextTick` 高于 `Promise`
- **宏任务**：在 Node.js 中宏任务包含 4 种——`setTimeout`、`setInterval`、`setImmediate` 和 `I/O`。宏任务在微任务执行之后执行，因此在同一个事件循环周期内，如果既存在微任务队列又存在宏任务队列，那么优先将微任务队列清空，再执行宏任务队列

我们可以看到有一个核心的主线程，它的执行阶段主要处理三个核心逻辑。

- 同步代码。
- 将异步任务插入到微任务队列或者宏任务队列中。
- 执行微任务或者宏任务的回调函数。在主线程处理回调函数的同时，也需要判断是否插入微任务和宏任务。根据优先级，先判断微任务队列是否存在任务，存在则先执行微任务，不存在则判断在宏任务队列是否有任务，有则执行。

```js
const fs = require('fs');
// 首次事件循环执行
console.log('start');
/// 将会在新的事件循环中的阶段执行
fs.readFile('./test.conf', {encoding: 'utf-8'}, (err, data) => {
    if (err) throw err;
    console.log('read file success');
});
setTimeout(() => { // 新的事件循环的起点
    console.log('setTimeout'); 
}, 0);
/// 该部分将会在首次事件循环中执行
Promise.resolve().then(()=>{
    console.log('Promise callback');
});
/// 执行 process.nextTick
process.nextTick(() => {
    console.log('nextTick callback');
});
// 首次事件循环执行
console.log('end');
```

分析下上面代码的执行过程

- 第一个事件循环主线程发起，因此先执行同步代码，所以先输出 start，然后输出 end
- 第一个事件循环主线程发起，因此先执行同步代码，所以先输出 start，然后输出 end；
- 再从上往下分析，遇到微任务，插入微任务队列，遇到宏任务，插入宏任务队列，分析完成后，微任务队列包含：`Promise.resolve 和 process.nextTick`，宏任务队列包含：`fs.readFile 和 setTimeout`；
- 先执行微任务队列，但是根据优先级，先执行 `process.nextTick 再执行 Promise.resolve`，所以先输出 `nextTick callback` 再输出 `Promise callback`；
- 再执行宏任务队列，根据`宏任务插入先后顺序执行 setTimeout 再执行 fs.readFile`，这里需要注意，先执行 `setTimeout` 由于其回调时间较短，因此回调也先执行，并非是 `setTimeout` 先执行所以才先执行回调函数，但是它执行需要时间肯定大于 `1ms`，所以虽然 `fs.readFile` 先于`setTimeout` 执行，但是 `setTimeout` 执行更快，所以先输出 `setTimeout` ，最后输出 `read file success`。

```js
// 输出结果
start
end
nextTick callback
Promise callback
setTimeout
read file success
```

![img](http://img-repo.poetries.top/images/20210516224232.png)

> 当微任务和宏任务又产生新的微任务和宏任务时，又应该如何处理呢？如下代码所示：

```js
const fs = require('fs');
setTimeout(() => { // 新的事件循环的起点
    console.log('1'); 
    fs.readFile('./config/test.conf', {encoding: 'utf-8'}, (err, data) => {
        if (err) throw err;
        console.log('read file sync success');
    });
}, 0);
/// 回调将会在新的事件循环之前
fs.readFile('./config/test.conf', {encoding: 'utf-8'}, (err, data) => {
    if (err) throw err;
    console.log('read file success');
});
/// 该部分将会在首次事件循环中执行
Promise.resolve().then(()=>{
    console.log('poll callback');
});
// 首次事件循环执行
console.log('2');
```

在上面代码中，有 2 个宏任务和 1 个微任务，宏任务是 `setTimeout 和 fs.readFile`，微任务是 `Promise.resolve`。

- 整个过程优先执行主线程的第一个事件循环过程，所以先执行同步逻辑，先输出 2。
- 接下来执行微任务，输出 `poll callback`。
- 再执行宏任务中的 `fs.readFile 和 setTimeout`，由于 `fs.readFile` 优先级高，先执行 `fs.readFile`。但是处理时间长于 `1ms`，因此会先执行 `setTimeout` 的回调函数，输出 `1`。这个阶段在执行过程中又会产生新的宏任务 `fs.readFile`，因此又将该 `fs.readFile 插入宏任务队列`
- 最后由于只剩下宏任务了 `fs.readFile`，因此执行该宏任务，并等待处理完成后的回调，输出 `read file sync success`。

```js
// 结果
2
poll callback
1
read file success
read file sync success
```

**Process.nextick() 和 Vue 的 nextick**

![img](http://img-repo.poetries.top/images/20210414213602.png)

> `Node.js` 和浏览器端宏任务队列的另一个很重要的不同点是，浏览器端任务队列每轮事件循环仅出队一个回调函数接着去执行微任务队列；而 `Node.js` 端只要轮到执行某个宏任务队列，则会执行完队列中所有的当前任务，但是当前轮次新添加到队尾的任务则会等到下一轮次才会执行。

```js
setTimeout(() => {
    console.log('setTimeout');
}, 0);
setImmediate(() => {
    console.log('setImmediate');
})
// 这里可能会输出 setTimeout，setImmediate
// 可能也会相反的输出，这取决于性能
// 因为可能进入 event loop 用了不到 1 毫秒，这时候会执行 setImmediate
// 否则会执行 setTimeout
```

> 上面介绍的都是 `macrotask` 的执行情况，`microtask` 会在以上每个阶段完成后立即执行

```js
setTimeout(()=>{
    console.log('timer1')

    Promise.resolve().then(function() {
        console.log('promise1')
    })
}, 0)

setTimeout(()=>{
    console.log('timer2')

    Promise.resolve().then(function() {
        console.log('promise2')
    })
}, 0)

// 以上代码在浏览器和 node 中打印情况是不同的
// 浏览器中一定打印 timer1, promise1, timer2, promise2
// node 中可能打印 timer1, timer2, promise1, promise2
// 也可能打印 timer1, promise1, timer2, promise2
```

> `Node` 中的 `process.nextTick` 会先于其他 `microtask` 执行

```js
setTimeout(() => {
 console.log("timer1");

 Promise.resolve().then(function() {
   console.log("promise1");
 });
}, 0);

process.nextTick(() => {
 console.log("nextTick");
});
// nextTick, timer1, promise1
```

> 对于 `microtask` 来说，它会在以上每个阶段完成前清空 `microtask` 队列，下图中的 `Tick` 就代表了 `microtask`

![img](https://poetries1.gitee.io/img-repo/2020/07/fe/5.png)

**谁来启动这个循环过程，循环条件是什么？**

> 当 Node.js 启动后，会初始化事件循环，处理已提供的输入脚本，它可能会先调用一些异步的 API、调度定时器，或者 `process.nextTick()`，然后再开始处理事件循环。因此可以这样理解，Node.js 进程启动后，就发起了一个新的事件循环，也就是事件循环的起点。

总结来说，Node.js 事件循环的发起点有 4 个：

- `Node.js` 启动后；
- `setTimeout` 回调函数；
- `setInterval` 回调函数；
- 也可能是一次 `I/O` 后的回调函数。

**无限循环有没有终点**

> 当所有的微任务和宏任务都清空的时候，虽然当前没有任务可执行了，但是也并不能代表循环结束了。因为可能存在当前还未回调的异步 I/O，所以这个循环是没有终点的，只要进程在，并且有新的任务存在，就会去执行

**Node.js 是单线程的还是多线程的？**

> `主线程是单线程执行的`，但是 Node.js `存在多线程执行`，多线程包括 `setTimeout 和异步 I/O 事件`。其实 Node.js 还存在其他的线程，包括`垃圾回收、内存优化`等

**EventLoop 对渲染的影响**

- 想必你之前在业务开发中也遇到过 `requestIdlecallback 和 requestAnimationFrame`，这两个函数在我们之前的内容中没有讲过，但是当你开始考虑它们在 Eventloop 的生命周期的哪一步触发，或者这两个方法的回调会在微任务队列还是宏任务队列执行的时候，才发现好像没有想象中那么简单。这两个方法其实也并不属于 JS 的原生方法，而是浏览器宿主环境提供的方法，因为它们牵扯到另一个问题：渲染。

- 我们知道浏览器作为一个复杂的应用是多线程工作的，除了运行 JS 的线程外，还有渲染线程、定时器触发线程、HTTP 请求线程，等等。JS 线程可以读取并且修改 DOM，而渲染线程也需要读取 DOM，这是一个典型的多线程竞争临界资源的问题。所以浏览器就把这两个线程设计成互斥的，即同时只能有一个线程在执行

- 渲染原本就不应该出现在 Eventloop 相关的知识体系里，但是因为 Eventloop 显然是在讨论 JS 如何运行的问题，而渲染则是浏览器另外一个线程的工作。但是 `requestAnimationFrame`的出现却把这两件事情给关联起来

- 通过调用requestAnimationFrame

   

  我们可以在下次渲染之前执行回调函数。那下次渲染具体是哪个时间点呢？渲染和 Eventloop 有什么关系呢？

  - 简单来说，就是在每一次 `Eventloop` 的末尾，`判断当前页面是否处于渲染时机，就是重新渲染`

- 有屏幕的硬件限制，比如 60Hz 刷新率，简而言之就是 1 秒刷新了 60 次，16.6ms 刷新一次。这个时候浏览器的渲染间隔时间就没必要小于 `16.6ms`，因为就算渲染了屏幕上也看不到。当然浏览器也不能保证一定会每 16.6ms 会渲染一次，因为还会受到处理器的性能、JavaScript 执行效率等其他因素影响。

- 回到 `requestAnimationFrame`，这个 API 保证在下次浏览器渲染之前一定会被调用，实际上我们完全可以把它看成是一个高级版的 `setInterval`。它们都是在一段时间后执行回调，但是前者的间隔时间是由浏览器自己不断调整的，而后者只能由用户指定。这样的特性也决定了 `requestAnimationFrame` 更适合用来做针对每一帧来修改的动画效果

- 当然 `requestAnimationFrame` 不是 `Eventloop` 里的宏任务，或者说它并不在 `Eventloop` 的生命周期里，只是浏览器又开放的一个在渲染之前发生的新的 hook。另外需要注意的是微任务的认知概念也需要更新，在执行 animation callback 时也有可能产生微任务（比如 promise 的 callback），会放到 animation queue 处理完后再执行。所以微任务并不是像之前说的那样在每一轮 Eventloop 后处理，而是在 JS 的函数调用栈清空后处理

但是 `requestIdlecallback` 却是一个更好理解的概念。当宏任务队列中没有任务可以处理时，浏览器可能存在“空闲状态”。这段空闲时间可以被 `requestIdlecallback` 利用起来执行一些优先级不高、不必立即执行的任务，如下图所示：

![img](http://img-repo.poetries.top/images/20210414212916.png)

#### 垃圾回收

> 老生代、新生代、回收算法（引用计数（老版本）、标记清除、标记整理）

#### 内存泄漏

- 意外的全局变量: 无法被回收
- 定时器: 未被正确关闭，导致所引用的外部变量无法被释放
- 事件监听: 没有正确销毁 (低版本浏览器可能出现)
- 闭包
  - 第一种情况是我们由于使用未声明的变量，而意外的创建了一个全局变量，而使这个变量一直留在内存中无法被回收。
  - 第二种情况是我们设置了setInterval定时器，而忘记取消它，如果循环函数有对外部变量的引用的话，那么这个变量会被一直留在内存中，而无法被回收。
  - 第三种情况是我们获取一个DOM元素的引用，而后面这个元素被删除，由于我们一直保留了对这个元素的引用，所以它也无法被回收。
  - 第四种情况是不合理的使用闭包，从而导致某些变量一直被留在内存当中。
- `dom` 引用: `dom` 元素被删除时，内存中的引用未被正确清空
- 控制台`console.log`打印的东西

> 可用 `chrome` 中的 `timeline` 进行内存标记，可视化查看内存的变化情况，找出异常点。

[内存泄露排查方法](https://juejin.cn/post/6947841638118998029?utm_source=gold_browser_extension)

#### 深浅拷贝

![img](http://img-repo.poetries.top/images/20210414142630.png)

**1. 浅拷贝的原理和实现**

> 自己创建一个新的对象，来接受你要重新复制或引用的对象值。如果对象属性是基本的数据类型，复制的就是基本类型的值给新对象；但如果属性是引用数据类型，复制的就是内存中的地址，如果其中一个对象改变了这个内存中的地址，肯定会影响到另一个对象

**方法一：object.assign**

> `object.assign`是 ES6 中 `object` 的一个方法，该方法可以用于 JS 对象的合并等多个用途，`其中一个用途就是可以进行浅拷贝`。该方法的第一个参数是拷贝的目标对象，后面的参数是拷贝的来源对象（也可以是多个来源）。

```text
object.assign 的语法为：Object.assign(target, ...sources)
```

object.assign 的示例代码如下：

```js
let target = {};
let source = { a: { b: 1 } };
Object.assign(target, source);
console.log(target); // { a: { b: 1 } };
```

**但是使用 object.assign 方法有几点需要注意**

- 它不会拷贝对象的继承属性；
- 它不会拷贝对象的不可枚举的属性；
- 可以拷贝 `Symbol` 类型的属性。

```js
let obj1 = { a:{ b:1 }, sym:Symbol(1)}; 
Object.defineProperty(obj1, 'innumerable' ,{
    value:'不可枚举属性',
    enumerable:false
});
let obj2 = {};
Object.assign(obj2,obj1)
obj1.a.b = 2;
console.log('obj1',obj1);
console.log('obj2',obj2);
```

![img](http://img-repo.poetries.top/images/20210414134752.png)

> 从上面的样例代码中可以看到，利用 `object.assign` 也可以拷贝 `Symbol` 类型的对象，但是如果到了对象的第二层属性 obj1.a.b 这里的时候，前者值的改变也会影响后者的第二层属性的值，说明其中`依旧存在着访问共同堆内存的问题`，也就是说`这种方法还不能进一步复制，而只是完成了浅拷贝的功能`

**方法二：扩展运算符方式**

- 我们也可以利用 JS 的扩展运算符，在构造对象的同时完成浅拷贝的功能。
- 扩展运算符的语法为：`let cloneObj = { ...obj };`

```js
/* 对象的拷贝 */
let obj = {a:1,b:{c:1}}
let obj2 = {...obj}
obj.a = 2
console.log(obj)  //{a:2,b:{c:1}} console.log(obj2); //{a:1,b:{c:1}}
obj.b.c = 2
console.log(obj)  //{a:2,b:{c:2}} console.log(obj2); //{a:1,b:{c:2}}
/* 数组的拷贝 */
let arr = [1, 2, 3];
let newArr = [...arr]; //跟arr.slice()是一样的效果
```

> 扩展运算符 和 `object.assign` 有同样的缺陷，也就是`实现的浅拷贝的功能差不多`，但是如果属性都是`基本类型的值，使用扩展运算符进行浅拷贝会更加方便`

**方法三：concat 拷贝数组**

> 数组的 `concat` 方法其实也是浅拷贝，所以连接一个含有引用类型的数组时，需要注意修改原数组中的元素的属性，因为它会影响拷贝之后连接的数组。不过 `concat` 只能用于数组的浅拷贝，使用场景比较局限。代码如下所示。

```js
let arr = [1, 2, 3];
let newArr = arr.concat();
newArr[1] = 100;
console.log(arr);  // [ 1, 2, 3 ]
console.log(newArr); // [ 1, 100, 3 ]
```

**方法四：slice 拷贝数组**

> `slice` 方法也比较有局限性，因为`它仅仅针对数组类型`。`slice方法会返回一个新的数组对象`，这一对象由该方法的前两个参数来决定原数组截取的开始和结束时间，是不会影响和改变原始数组的。

```text
slice 的语法为：arr.slice(begin, end);
let arr = [1, 2, {val: 4}];
let newArr = arr.slice();
newArr[2].val = 1000;
console.log(arr);  //[ 1, 2, { val: 1000 } ]
```

> 从上面的代码中可以看出，这就是`浅拷贝的限制所在了——它只能拷贝一层对象`。如果`存在对象的嵌套，那么浅拷贝将无能为力`。因此深拷贝就是为了解决这个问题而生的，它能解决多层对象嵌套问题，彻底实现拷贝

**手工实现一个浅拷贝**

根据以上对浅拷贝的理解，如果让你自己实现一个浅拷贝，大致的思路分为两点：

- 对基础类型做一个最基本的一个拷贝；
- 对引用类型开辟一个新的存储，并且拷贝一层对象属性。

```js
const shallowClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = target[prop];
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}
```

> 利用类型判断，针对引用类型的对象进行 for 循环遍历对象属性赋值给目标对象的属性，基本就可以手工实现一个浅拷贝的代码了

**2. 深拷贝的原理和实现**

`浅拷贝只是创建了一个新的对象，复制了原有对象的基本类型的值，而引用数据类型只拷贝了一层属性，再深层的还是无法进行拷贝`。深拷贝则不同，对于复杂引用数据类型，其在堆内存中完全开辟了一块内存地址，并将原有的对象完全复制过来存放。

这两个对象是相互独立、不受影响的，彻底实现了内存上的分离。总的来说，`深拷贝的原理可以总结如下`：

> 将一个对象从内存中完整地拷贝出来一份给目标对象，并从堆内存中开辟一个全新的空间存放新对象，且新对象的修改并不会改变原对象，二者实现真正的分离。

**方法一：乞丐版（JSON.stringify）**

> `JSON.stringify()` 是目前开发过程中最简单的深拷贝方法，其实就是把一个对象序列化成为 `JSON` 的字符串，并将对象里面的内容转换成字符串，最后再用 `JSON.parse()` 的方法将 `JSON` 字符串生成一个新的对象

```js
let a = {
    age: 1,
    jobs: {
        first: 'FE'
    }
}
let b = JSON.parse(JSON.stringify(a))
a.jobs.first = 'native'
console.log(b.jobs.first) // FE
```

**但是该方法也是有局限性的**：

- 会忽略 `undefined`
- 会忽略 `symbol`
- 不能序列化函数
- 无法拷贝不可枚举的属性
- 无法拷贝对象的原型链
- 拷贝 `RegExp` 引用类型会变成空对象
- 拷贝 `Date` 引用类型会变成字符串
- 对象中含有 `NaN`、`Infinity` 以及 `-Infinity`，`JSON` 序列化的结果会变成 `null`
- 不能解决循环引用的对象，即对象成环 (`obj[key] = obj`)。

```js
function Obj() { 
  this.func = function () { alert(1) }; 
  this.obj = {a:1};
  this.arr = [1,2,3];
  this.und = undefined; 
  this.reg = /123/; 
  this.date = new Date(0); 
  this.NaN = NaN;
  this.infinity = Infinity;
  this.sym = Symbol(1);
} 
let obj1 = new Obj();
Object.defineProperty(obj1,'innumerable',{ 
  enumerable:false,
  value:'innumerable'
});
console.log('obj1',obj1);
let str = JSON.stringify(obj1);
let obj2 = JSON.parse(str);
console.log('obj2',obj2);
```

![img](http://img-repo.poetries.top/images/20210414141731.png)

> 使用 `JSON.stringify` 方法实现深拷贝对象，虽然到目前为止还有很多无法实现的功能，但是这种方法足以满足日常的开发需求，并且是最简单和快捷的。而对于其他的也要实现深拷贝的，比较麻烦的属性对应的数据类型，`JSON.stringify` 暂时还是无法满足的，那么就需要下面的几种方法了

**方法二：基础版（手写递归实现）**

> 下面是一个实现 deepClone 函数封装的例子，通过 `for in` 遍历传入参数的属性值，如果值是引用类型则再次递归调用该函数，如果是基础数据类型就直接复制

```js
let obj1 = {
  a:{
    b:1
  }
}
function deepClone(obj) { 
  let cloneObj = {}
  for(let key in obj) {                 //遍历
    if(typeof obj[key] ==='object') { 
      cloneObj[key] = deepClone(obj[key])  //是对象就再次调用该函数递归
    } else {
      cloneObj[key] = obj[key]  //基本类型的话直接复制值
    }
  }
  return cloneObj
}
let obj2 = deepClone(obj1);
obj1.a.b = 2;
console.log(obj2);   //  {a:{b:1}}
```

虽然利用递归能实现一个深拷贝，但是同上面的 `JSON.stringify` 一样，还是有一些问题没有完全解决，例如：

- 这个深拷贝函数并不能复制不可枚举的属性以及 `Symbol` 类型；
- 这种方法`只是针对普通的引用类型的值做递归复制`，而对于 `Array、Date、RegExp、Error、Function` 这样的引用类型并不能正确地拷贝；
- 对象的属性里面成环，即`循环引用没有解决`。

这种基础版本的写法也比较简单，可以应对大部分的应用情况。但是你在面试的过程中，如果只能写出这样的一个有缺陷的深拷贝方法，有可能不会通过。

所以为了“拯救”这些缺陷，下面我带你一起看看改进的版本，以便于你可以在面试种呈现出更好的深拷贝方法，赢得面试官的青睐。

**方法三：改进版（改进后递归实现）**

> 针对上面几个待解决问题，我先通过四点相关的理论告诉你分别应该怎么做。

- 针对能够遍历对象的不可枚举属性以及 `Symbol` 类型，我们可以使用 `Reflect.ownKeys` 方法；
- 当参数为 `Date、RegExp` 类型，则直接生成一个新的实例返回；
- 利用 `Object` 的 `getOwnPropertyDescriptors` 方法可以获得对象的所有属性，以及对应的特性，顺便结合 `Object.create` 方法创建一个新对象，并继承传入原对象的原型链；
- 利用 `WeakMap` 类型作为 `Hash` 表，因为 `WeakMap` 是弱引用类型，可以有效防止内存泄漏（你可以关注一下 `Map` 和 `weakMap` 的关键区别，这里要用 `weakMap`），作为检测循环引用很有帮助，如果存在循环，则引用直接返回 `WeakMap` 存储的值

如果你在考虑到循环引用的问题之后，还能用 `WeakMap` 来很好地解决，并且向面试官解释这样做的目的，那么你所展示的代码，以及你对问题思考的全面性，在面试官眼中应该算是合格的了

**实现深拷贝**

```js
const isComplexDataType = obj => (typeof obj === 'object' || typeof obj === 'function') && (obj !== null)

const deepClone = function (obj, hash = new WeakMap()) {
  if (obj.constructor === Date) {
    return new Date(obj)       // 日期对象直接返回一个新的日期对象
  }
  
  if (obj.constructor === RegExp){
    return new RegExp(obj)     //正则对象直接返回一个新的正则对象
  }
  
  //如果循环引用了就用 weakMap 来解决
  if (hash.has(obj)) {
    return hash.get(obj)
  }
  let allDesc = Object.getOwnPropertyDescriptors(obj)

  //遍历传入参数所有键的特性
  let cloneObj = Object.create(Object.getPrototypeOf(obj), allDesc)

  //继承原型链
  hash.set(obj, cloneObj)

  for (let key of Reflect.ownKeys(obj)) { 
    cloneObj[key] = (isComplexDataType(obj[key]) && typeof obj[key] !== 'function') ? deepClone(obj[key], hash) : obj[key]
  }
  return cloneObj
}
// 下面是验证代码
let obj = {
  num: 0,
  str: '',
  boolean: true,
  unf: undefined,
  nul: null,
  obj: { name: '我是一个对象', id: 1 },
  arr: [0, 1, 2],
  func: function () { console.log('我是一个函数') },
  date: new Date(0),
  reg: new RegExp('/我是一个正则/ig'),
  [Symbol('1')]: 1,
};
Object.defineProperty(obj, 'innumerable', {
  enumerable: false, value: '不可枚举属性' }
);
obj = Object.create(obj, Object.getOwnPropertyDescriptors(obj))
obj.loop = obj    // 设置loop成循环引用的属性
let cloneObj = deepClone(obj)
cloneObj.arr.push(4)
console.log('obj', obj)
console.log('cloneObj', cloneObj)
```

我们看一下结果，`cloneObj` 在 `obj` 的基础上进行了一次深拷贝，`cloneObj` 里的 `arr` 数组进行了修改，并未影响到 `obj.arr` 的变化，如下图所示

![img](http://img-repo.poetries.top/images/20210414142525.png)

### [#](http://interview.poetries.top/docs/simply.html#_22-节流与防抖)