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

