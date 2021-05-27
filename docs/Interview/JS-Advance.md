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

