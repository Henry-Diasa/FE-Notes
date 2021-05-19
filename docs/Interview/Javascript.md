#### 事件代理

- 事件代理（`Event Delegation`），又称之为事件委托。是 `JavaScript` 中常用绑定事件的常用技巧。顾名思义，“事件代理”即是把原本需要绑定的事件委托给父元素，让父元素担当事件监听的职务。事件代理的原理是DOM元素的事件冒泡。使用事件代理的好处是可以提高性能
- 可以大量节省内存占用，减少事件注册，比如在`table`上代理所有`td`的`click`事件就非常棒
- 可以实现当新增子对象时无需再次对其绑定

#### 事件模型

> `W3C`中定义事件的发生经历三个阶段：捕获阶段（`capturing`）、目标阶段（`targetin`）、冒泡阶段（`bubbling`）

- 冒泡型事件：当你使用事件冒泡时，子级元素先触发，父级元素后触发
- 捕获型事件：当你使用事件捕获时，父级元素先触发，子级元素后触发
- `DOM`事件流：同时支持两种事件模型：捕获型事件和冒泡型事件
- 阻止冒泡：在`W3c`中，使用`stopPropagation()`方法；在IE下设置`cancelBubble = true`
- 阻止捕获：阻止事件的默认行为，例如`click - <a>`后的跳转。在`W3c`中，使用`preventDefault()`方法，在`IE`下设置`window.event.returnValue = false`

#### Ajax原理

- `Ajax`的原理简单来说是在用户和服务器之间加了—个中间层(`AJAX`引擎)，通过`XmlHttpRequest`对象来向服务器发异步请求，从服务器获得数据，然后用`javascrip`t来操作`DOM`而更新页面。使用户操作与服务器响应异步化。这其中最关键的一步就是从服务器获得请求数据
- `Ajax`的过程只涉及`JavaScript`、`XMLHttpRequest`和`DOM`。`XMLHttpRequest`是`aja`x的核心机制

```javascript
/** 1. 创建连接 **/
var xhr = null;
xhr = new XMLHttpRequest()
/** 2. 连接服务器 **/
xhr.open('get', url, true)
/** 3. 发送请求 **/
xhr.send(null);
/** 4. 接受请求 **/
xhr.onreadystatechange = function(){
	if(xhr.readyState == 4){
		if(xhr.status == 200){
			success(xhr.responseText);
		} else { 
			/** false **/
			fail && fail(xhr.status);
		}
	}
}
```

**ajax 有那些优缺点?**

- 优点：
  - 通过异步模式，提升了用户体验.
  - 优化了浏览器和服务器之间的传输，减少不必要的数据往返，减少了带宽占用.
  - `Ajax`在客户端运行，承担了一部分本来由服务器承担的工作，减少了大用户量下的服务器负载。
  - `Ajax`可以实现动态不刷新（局部刷新）
- 缺点：
  - 安全问题 `AJAX`暴露了与服务器交互的细节。
  - 对搜索引擎的支持比较弱。
  - 不容易调试。

#### 如何解决跨域问题?

> 首先了解下浏览器的同源策略 同源策略`/SOP（Same origin policy）`是一种约定，由Netscape公司1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到`XSS`、`CSFR`等攻击。所谓同源是指"**协议+域名+端口**"三者相同，即便两个不同的域名指向同一个ip地址，也非同源

**那么怎样解决跨域问题的呢？**

- **通过jsonp跨域**

```javascript
var script = document.createElement('script');
script.type = 'text/javascript';

// 传参并指定回调执行函数为onBack
script.src = 'http://www.....:8080/login?user=admin&callback=onBack';
document.head.appendChild(script);

// 回调执行函数
function onBack(res) {
    alert(JSON.stringify(res));
}
```

- **document.domain + iframe跨域**

> 此方案仅限主域相同，子域不同的跨域应用场景

1.）父窗口：(http://www.domain.com/a.html)

```html
<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>
<script>
    document.domain = 'domain.com';
    var user = 'admin';
</script>
```

2.）子窗口：(http://child.domain.com/b.html)

```javascript
document.domain = 'domain.com';
// 获取父窗口中变量
alert('get js data from parent ---> ' + window.parent.user);
```

- **nginx代理跨域**
- **nodejs中间件代理跨域**
- **后端在头部信息里面设置安全域名**

#### 异步加载JS的方式有哪些？

- 设置`<script>`属性 async="async" （一旦脚本可用，则会异步执行）
- 动态创建 `script DOM`：`document.createElement('script');`
- `XmlHttpRequest` 脚本注入
- 异步加载库 `LABjs`
- 模块加载器 `Sea.js`

#### XML和JSON的区别？

- 数据体积方面
  - `JSON`相对`于XML`来讲，数据的体积小，传递的速度更快些。
- 数据交互方面
  - `JSON`与`JavaScript`的交互更加方便，更容易解析处理，更好的数据交互
- 数据描述方面
  - `JSON`对数据的描述性比`XML`较差
- 传输速度方面
  - `JSON`的速度要远远快于`XML`

#### 为什么要有同源限制？

- 同源策略指的是：协议，域名，端口相同，同源策略是一种安全协议
- 举例说明：比如一个黑客程序，他利用`Iframe`把真正的银行登录页面嵌到他的页面上，当你使用真实的用户名，密码登录时，他的页面就可以通过`Javascript`读取到你的表单中`input`中的内容，这样用户名，密码就轻松到手了。

#### offsetWidth/offsetHeight,clientWidth/clientHeight与scrollWidth/scrollHeight的区别

- `offsetWidth/offsetHeight`返回值包含**content + padding + border**，效果与e.getBoundingClientRect()相同
- `clientWidth/clientHeight`返回值只包含**content + padding**，如果有滚动条，也**不包含滚动条**
- `scrollWidth/scrollHeight`返回值包含**content + padding + 溢出内容的尺寸**

#### javascript有哪些方法定义对象

- 对象字面量： `var obj = {};`
- 构造函数： `var obj = new Object();`
- Object.create(): `var obj = Object.create(Object.prototype);`

#### JavaScript有几种类型的值

- 栈：原始数据类型（`Undefined`，`Null`，`Boolean`，`Number`、`String`）
- 堆：引用数据类型（对象、数组和函数）
- 两种类型的区别是：存储位置不同；
- 原始数据类型直接存储在栈(`stack`)中的简单数据段，占据空间小、大小固定，属于被频繁使用数据，所以放入栈中存储；
- 引用数据类型存储在堆(`heap`)中的对象,占据空间大、大小不固定,如果存储在栈中，将会影响程序运行的性能；引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体

#### null，undefined 的区别

- `undefined` 表示不存在这个值。
- `undefined` :是一个表示"无"的原始值或者说表示"缺少值"，就是此处应该有一个值，但是还没有定义。当尝试读取时会返回 `undefined`
- 例如变量被声明了，但没有赋值时，就等于`undefined`
- `null` 表示一个对象被定义了，值为“空值”
- `null` : 是一个对象(空对象, 没有任何属性和方法)
- 例如作为函数的参数，表示该函数的参数不是对象；
- 在验证`null`时，一定要使用　`===` ，因为 `==`无法分别`null` 和　`undefined`

####  ["1", "2", "3"].map(parseInt) 答案是多少

- `[1, NaN, NaN]`因为 `parseInt` 需要两个参数 `(val, radix)`，其中`radix` 表示解析时用的基数。
- `map`传了 `3`个`(element, index, array)`，对应的 `radix` 不合法导致解析失败。

#### javascript 代码中的"use strict";是什么意思

- `use strict`是一种`ECMAscript 5` 添加的（严格）运行模式,这种模式使得 Javascript 在更严格的条件下运行,使`JS`编码更加规范化的模式,消除`Javascript`语法的一些不合理、不严谨之处，减少一些怪异行为

  **严格模式的限制**

  - 变量必须声明后再使用
  - 函数的参数不能有同名属性，否则报错
  - 不能使用with语句
  - 不能对只读属性赋值，否则报错
  - 不能使用前缀0表示八进制数，否则报错
  - 不能删除不可删除的属性，否则报错
  - 不能删除变量`delete prop`，会报错，只能删除属性`delete global[prop]`
  - `eval`不会在它的外层作用域引入变量
  - `eval`和`arguments`不能被重新赋值
  - `arguments`不会自动反映函数参数的变化
  - 不能使用`arguments.callee`
  - 不能使用`arguments.caller`
  - 禁止`this`指向全局对象
  - 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
  - 增加了保留字（比如`protected`、`static`和`interface`）

#### 什么是面向对象编程及面向过程编程，它们的异同和优缺点

- 面向过程就是分析出解决问题所需要的步骤，然后用函数把这些步骤一步一步实现，使用的时候一个一个依次调用就可以了
- 面向对象是把构成问题事务分解成各个对象，建立对象的目的不是为了完成一个步骤，而是为了描叙某个事物在整个解决问题的步骤中的行为
- 面向对象是以功能来划分问题，而不是步骤
- 基本思想是使用对象，类，继承，封装等基本概念来进行程序设计
- 优点
  - 易维护
    - 采用面向对象思想设计的结构，可读性高，由于继承的存在，即使改变需求，那么维护也只是在局部模块，所以维护起来是非常方便和较低成本的
  - 易扩展
  - 开发工作的重用性、继承性高，降低重复工作量。
  - 缩短了开发周期

#### map与forEach的区别

- `forEach`方法，是最基本的方法，就是遍历与循环，默认有3个传参：分别是遍历的数组内容`item`、数组索引`index`、和当前遍历数组`Array`
- `map`方法，基本用法与`forEach`一致，但是不同的，它会返回一个新的数组，所以在callback需要有`return`值，如果没有，会返回`undefined`

#### 箭头函数与普通函数的区别？

- 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象
- 不可以当作构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误
- 不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用`Rest`参数代替
- 不可以使用`yield`命令，因此箭头函数不能用作`Generator`函数

#### Js动画与CSS动画区别及相应实现

- CSS3的动画的优点
  - 在性能上会稍微好一些，浏览器会对`CSS3`的动画做一些优化
  - 代码相对简单
- 缺点
  - 在动画控制上不够灵活
  - 兼容性不好
- `JavaScript`的动画正好弥补了这两个缺点，控制能力很强，可以单帧的控制、变换，同时写得好完全可以兼容`IE6`，并且功能强大。对于一些复杂控制的动画，使用`javascript`会比较靠谱。而在实现一些小的交互动效的时候，就多考虑考虑`CSS`吧

#### 快速的让一个数组乱序

```javascript
var arr = [1,2,3,4,5,6,7,8,9,10];
arr.sort(function(){
    return Math.random() - 0.5;
})
console.log(arr);
```

#### 如何渲染几万条数据并不卡住界面

> 这道题考察了如何在不卡住页面的情况下渲染数据，也就是说不能一次性将几万条都渲染出来，而应该一次渲染部分 `DOM`，那么就可以通过 `requestAnimationFrame` 来每 `16 ms` 刷新一次

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <ul>控件</ul>
  <script>
    setTimeout(() => {
      // 插入十万条数据
      const total = 100000
      // 一次插入 20 条，如果觉得性能不好就减少
      const once = 20
      // 渲染数据总共需要几次
      const loopCount = total / once
      let countOfRender = 0
      let ul = document.querySelector("ul");
      function add() {
        // 优化性能，插入不会造成回流
        const fragment = document.createDocumentFragment();
        for (let i = 0; i < once; i++) {
          const li = document.createElement("li");
          li.innerText = Math.floor(Math.random() * total);
          fragment.appendChild(li);
        }
        ul.appendChild(fragment);
        countOfRender += 1;
        loop();
      }
      function loop() {
        if (countOfRender < loopCount) {
          window.requestAnimationFrame(add);
        }
      }
      loop();
    }, 0);
  </script>
</body>
</html>
```

#### 希望获取到页面中所有的checkbox怎么做？

> 不使用第三方框架

```js
 var domList = document.getElementsByTagName(‘input’)
 var checkBoxList = [];
 var len = domList.length;　　//缓存到局部变量
 while (len--) {　　//使用while的效率会比for循环更高
 　　if (domList[len].type == ‘checkbox’) {
     　　checkBoxList.push(domList[len]);
 　　}
 }
```

#### 怎样添加、移除、移动、复制、创建和查找节点

**创建新节点**

```js
createDocumentFragment()    //创建一个DOM片段
createElement()   //创建一个具体的元素
createTextNode()   //创建一个文本节点
```

**添加、移除、替换、插入**

```js
appendChild()      //添加
removeChild()      //移除
replaceChild()      //替换
insertBefore()      //插入
```

**查找**

```js
getElementsByTagName()    //通过标签名称
getElementsByName()     //通过元素的Name属性的值
getElementById()        //通过元素Id，唯一性
```

####  Javascript中callee和caller的作用？

- `caller`是返回一个对函数的引用，该函数调用了当前函数；
- `callee`是返回正在被执行的`function`函数，也就是所指定的`function`对象的正文

> 那么问题来了？如果一对兔子每月生一对兔子；一对新生兔，从第二个月起就开始生兔子；假定每对兔子都是一雌一雄，试问一对兔子，第n个月能繁殖成多少对兔子？（使用`callee`完成）

```js
var result=[];
  function fn(n){  //典型的斐波那契数列
     if(n==1){
          return 1;
     }else if(n==2){
             return 1;
     }else{
          if(result[n]){
                  return result[n];
         }else{
                 //argument.callee()表示fn()
                 result[n]=arguments.callee(n-1)+arguments.callee(n-2);
                 return result[n];
         }
    }
 }
```

#### window.onload和$(document).ready

> 原生`JS`的`window.onload`与`Jquery`的`$(document).ready(function(){})`有什么不同？如何用原生JS实现Jq的`ready`方法？

- `window.onload()`方法是必须等到页面内包括图片的所有元素加载完毕后才能执行。
- `$(document).ready()`是`DOM`结构绘制完毕后就执行，不必等到加载完毕

```js
function ready(fn){
      if(document.addEventListener) {        //标准浏览器
          document.addEventListener('DOMContentLoaded', function() {
              //注销事件, 避免反复触发
              document.removeEventListener('DOMContentLoaded',arguments.callee, false);
              fn();            //执行函数
          }, false);
      }else if(document.attachEvent) {        //IE
          document.attachEvent('onreadystatechange', function() {
             if(document.readyState == 'complete') {
                 document.detachEvent('onreadystatechange', arguments.callee);
                 fn();        //函数执行
             }
         });
     }
 };
```

#### 数组去重方法总结

**方法一、利用ES6 Set去重（ES6中最常用）**

```js
function unique (arr) {
  return Array.from(new Set(arr))
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
 //[1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {}, {}]
```

**方法二、利用for嵌套for，然后splice去重（ES5中最常用）**

```js
function unique(arr){            
        for(var i=0; i<arr.length; i++){
            for(var j=i+1; j<arr.length; j++){
                if(arr[i]==arr[j]){         //第一个等同于第二个，splice方法删除第二个
                    arr.splice(j,1);
                    j--;
                }
            }
        }
	return arr;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
    //[1, "true", 15, false, undefined, NaN, NaN, "NaN", "a", {…}, {…}]     //NaN和{}没有去重，两个null直接消失了
```

- 双层循环，外层循环元素，内层循环时比较值。值相同时，则删去这个值。
- 想快速学习更多常用的`ES6`语法

**方法三、利用indexOf去重**

```js
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var array = [];
    for (var i = 0; i < arr.length; i++) {
        if (array .indexOf(arr[i]) === -1) {
            array .push(arr[i])
        }
    }
    return array;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
   // [1, "true", true, 15, false, undefined, null, NaN, NaN, "NaN", 0, "a", {…}, {…}]  //NaN、{}没有去重
```

> 新建一个空的结果数组，`for` 循环原数组，判断结果数组是否存在当前元素，如果有相同的值则跳过，不相同则`push`进数组

**方法四、利用sort()**

```js
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return;
    }
    arr = arr.sort()
    var arrry= [arr[0]];
    for (var i = 1; i < arr.length; i++) {
        if (arr[i] !== arr[i-1]) {
            arrry.push(arr[i]);
        }
    }
    return arrry;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
// [0, 1, 15, "NaN", NaN, NaN, {…}, {…}, "a", false, null, true, "true", undefined]      //NaN、{}没有去重
```

> 利用`sort()`排序方法，然后根据排序后的结果进行遍历及相邻元素比对

**方法五、利用对象的属性不能相同的特点进行去重**

```js
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var arrry= [];
     var  obj = {};
    for (var i = 0; i < arr.length; i++) {
        if (!obj[arr[i]]) {
            arrry.push(arr[i])
            obj[arr[i]] = 1
        } else {
            obj[arr[i]]++
        }
    }
    return arrry;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "true", 15, false, undefined, null, NaN, 0, "a", {…}]    //两个true直接去掉了，NaN和{}去重
```

**方法六、利用includes**

```js
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var array =[];
    for(var i = 0; i < arr.length; i++) {
            if( !array.includes( arr[i]) ) {//includes 检测数组是否有某个值
                    array.push(arr[i]);
              }
    }
    return array
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
    //[1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}, {…}]     //{}没有去重
```

**方法七、利用hasOwnProperty**

```js
function unique(arr) {
    var obj = {};
    return arr.filter(function(item, index, arr){
        return obj.hasOwnProperty(typeof item + item) ? false : (obj[typeof item + item] = true)
    })
}
    var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
        console.log(unique(arr))
//[1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}]   //所有的都去重了
```

> 利用`hasOwnProperty` 判断是否存在对象属性

**方法八、利用filter**

```js
function unique(arr) {
  return arr.filter(function(item, index, arr) {
    //当前元素，在原始数组中的第一个索引==当前索引值，否则返回当前元素
    return arr.indexOf(item, 0) === index;
  });
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "true", true, 15, false, undefined, null, "NaN", 0, "a", {…}, {…}]
```

**方法九、利用递归去重**

```js
function unique(arr) {
    var array= arr;
    var len = array.length;

	array.sort(function(a,b){   //排序后更加方便去重
		return a - b;
	})

	function loop(index){
        if(index >= 1){
            if(array[index] === array[index-1]){
            array.splice(index,1);
            }
            loop(index - 1);    //递归loop，然后数组去重
        }
	}
	loop(len-1);
	return array;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "a", "true", true, 15, false, 1, {…}, null, NaN, NaN, "NaN", 0, "a", {…}, undefined]
```

**方法十、利用Map数据结构去重**

```js
function arrayNonRepeatfy(arr) {
	let map = new Map();
		let array = new Array();  // 数组用于返回结果
		for (let i = 0; i < arr.length; i++) {
			if(map .has(arr[i])) {  // 如果有该key值
			map .set(arr[i], true);
		} else {
			map .set(arr[i], false);   // 如果没有该key值
			array .push(arr[i]);
		}
	}
	return array ;
}
 var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
    console.log(unique(arr))
//[1, "a", "true", true, 15, false, 1, {…}, null, NaN, NaN, "NaN", 0, "a", {…}, undefined]
```

> 创建一个空`Map`数据结构，遍历需要去重的数组，把数组的每一个元素作为`key`存到`Map`中。由于`Map`中不会出现相同的`key`值，所以最终得到的就是去重后的结果

**方法十一、利用reduce+includes**

```js
function unique(arr){
    return arr.reduce((prev,cur) => prev.includes(cur) ? prev : [...prev,cur],[]);
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr));
// [1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}, {…}]
```

**方法十二、[...new Set(arr)]**

```js
[...new Set(arr)]
//代码就是这么少----（其实，严格来说并不算是一种，相对于第一种方法来说只是简化了代码）
```

#### 使用js实现一个持续的动画效果

**定时器思路**

```js
var e = document.getElementById('e')
var flag = true;
var left = 0;
setInterval(() => {
    left == 0 ? flag = true : left == 100 ? flag = false : ''
    flag ? e.style.left = ` ${left++}px` : e.style.left = ` ${left--}px`
}, 1000 / 60)
```

**requestAnimationFrame**

```js
//兼容性处理
window.requestAnimFrame = (function(){
    return window.requestAnimationFrame       ||
           window.webkitRequestAnimationFrame ||
           window.mozRequestAnimationFrame    ||
           function(callback){
                window.setTimeout(callback, 1000 / 60);
           };
})();

var e = document.getElementById("e");
var flag = true;
var left = 0;

function render() {
    left == 0 ? flag = true : left == 100 ? flag = false : '';
    flag ? e.style.left = ` ${left++}px` :
        e.style.left = ` ${left--}px`;
}

(function animloop() {
    render();
    requestAnimFrame(animloop);
})();
```

**使用css实现一个持续的动画效果**

```css
animation:mymove 5s infinite;

@keyframes mymove {
    from {top:0px;}
    to {top:200px;}
}
```

- `animation-name` 规定需要绑定到选择器的 `keyframe`名称。
- `animation-duration` 规定完成动画所花费的时间，以秒或毫秒计。
- `animation-timing-function` 规定动画的速度曲线。
- `animation-delay` 规定在动画开始之前的延迟。
- `animation-iteration-count` 规定动画应该播放的次数。
- `animation-direction` 规定是否应该轮流反向播放动画

#### 封装一个函数，参数是定时器的时间，.then执行回调函数

```js
function sleep (time) {
    return new Promise((resolve) => setTimeout(resolve, time));
}
```

#### 怎么判断两个对象相等？

```js
obj={
    a:1,
    b:2
}
obj2={
    a:1,
    b:2
}
obj3={
    a:1,
    b:'2'
}
```

> 可以转换为字符串来判断

```js
JSON.stringify(obj)==JSON.stringify(obj2);//true
JSON.stringify(obj)==JSON.stringify(obj3);//false
```

#### WebSocket

> 由于 `http` 存在一个明显的弊端（消息只能有客户端推送到服务器端，而服务器端不能主动推送到客户端），导致如果服务器如果有连续的变化，这时只能使用轮询，而轮询效率过低，并不适合。于是 `WebSocket`被发明出来

> 相比与 `http` 具有以下有点

- 支持双向通信，实时性更强；
- 可以发送文本，也可以二进制文件；
- 协议标识符是 `ws`，加密后是 `wss` ；
- 较少的控制开销。连接创建后，`ws`客户端、服务端进行数据交换时，协议控制的数据包头部较小。在不包含头部的情况下，服务端到客户端的包头只有`2~10`字节（取决于数据包长度），客户端到服务端的的话，需要加上额外的4字节的掩码。而`HTTP`协议每次通信都需要携带完整的头部；
- 支持扩展。ws协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议。（比如支持自定义压缩算法等）
- 无跨域问题。

> 实现比较简单，服务端库如 `socket.io`、`ws`，可以很好的帮助我们入门。而客户端也只需要参照 `api` 实现即可

#### Hybrid

> http://blog.poetries.top/2018/10/20/fe-interview-hybrid/

#### 组件化

> http://blog.poetries.top/2018/10/20/fe-interview-component/

#### MVVM

> http://blog.poetries.top/2018/10/20/fe-interview-mvvm/

#### 我现在有一个`canvas`，上面随机布着一些黑块，请实现方法，计算canvas上有多少个黑块

> https://www.jianshu.com/p/f54d265f7aa4

#### 现在要你完成一个Dialog组件，说说你设计的思路？它应该有什么功能？

- 该组件需要提供`hook`指定渲染位置，默认渲染在body下面。
- 然后改组件可以指定外层样式，如宽度等
- 组件外层还需要一层`mask`来遮住底层内容，点击`mask`可以执行传进来的`onCancel`函数关闭`Dialog`。
- 另外组件是可控的，需要外层传入`visible`表示是否可见。
- 然后`Dialog`可能需要自定义头head和底部`footer`，默认有头部和底部，底部有一个确认按钮和取消按钮，确认按钮会执行外部传进来的`onOk`事件，然后取消按钮会执行外部传进来的`onCancel`事件。
- 当组件的`visible`为`true`时候，设置`body`的`overflow`为`hidden`，隐藏`body`的滚动条，反之显示滚动条。
- 组件高度可能大于页面高度，组件内部需要滚动条。
- 只有组件的`visible`有变化且为`ture`时候，才重渲染组件内的所有内容

#### `caller`和`callee`的区别

**caller*

> `caller`返回一个函数的引用，这个函数调用了当前的函数。

**使用这个属性要注意**

- 这个属性只有当函数在执行时才有用
- 如果在`javascript`程序中，函数是由顶层调用的，则返回`null`

> `functionName.caller: functionName`是当前正在执行的函数。

```js
function a() {
  console.log(a.caller)
}
```

**callee**

> `callee`放回正在执行的函数本身的引用，它是`arguments`的一个属性

> 使用callee时要注意:

- 这个属性只有在函数执行时才有效
- 它有一个`length`属性，可以用来获得形参的个数，因此可以用来比较形参和实参个数是否一致，即比较`arguments.length`是否等于`arguments.callee.length`
- 它可以用来递归匿名函数。

```js
function a() {
  console.log(arguments.callee)
}
```

#### ajax、axios、fetch区别

**jQuery ajax**

```js
$.ajax({
   type: 'POST',
   url: url,
   data: data,
   dataType: dataType,
   success: function () {},
   error: function () {}
});
```

优缺点：

- 本身是针对`MVC`的编程,不符合现在前端`MVVM`的浪潮
- 基于原生的`XHR`开发，`XHR`本身的架构不清晰，已经有了`fetch`的替代方案
- `JQuery`整个项目太大，单纯使用`ajax`却要引入整个`JQuery`非常的不合理（采取个性化打包的方案又不能享受CDN服务）

**axios**

```js
axios({
    method: 'post',
    url: '/user/12345',
    data: {
        firstName: 'Fred',
        lastName: 'Flintstone'
    }
})
.then(function (response) {
    console.log(response);
})
.catch(function (error) {
    console.log(error);
});
```

优缺点：

- 从浏览器中创建 `XMLHttpRequest`
- 从 `node.js` 发出 `http` 请求
- 支持 `Promise API`
- 拦截请求和响应
- 转换请求和响应数据
- 取消请求
- 自动转换`JSON`数据
- 客户端支持防止`CSRF/XSRF`

**fetch**

```js
try {
  let response = await fetch(url);
  let data = response.json();
  console.log(data);
} catch(e) {
  console.log("Oops, error", e);

}
```

优缺点：

- `fetcht`只对网络请求报错，对`400`，`500`都当做成功的请求，需要封装去处理
- `fetch`默认不会带`cookie`，需要添加配置项
- `fetch`不支持`abort`，不支持超时控制，使用`setTimeout`及`Promise.reject`的实现的超时控制并不能阻止请求过程继续在后台运行，造成了量的浪费
- `fetch`没有办法原生监测请求的进度，而XHR可以

#### 检测浏览器版本版本有哪些方式？

- 根据 `navigator.userAgent` `UA.toLowerCase().indexOf('chrome')`
- 根据 `window` 对象的成员 `'ActiveXObject' in window`

#### DOM树和渲染树的区别

- `DOM`树与`HTML`标签一一对应，包括`head`和隐藏元素
- 渲染树不包括`head`和隐藏元素，大段文本的每一个行都是独立节点，每一个节点都有对应的`css`属性

####  script 的位置是否会影响首屏显示时间

- 在解析 `HTML` 生成 `DOM` 过程中，`js` 文件的下载是并行的，不需要 `DOM` 处理到 `script` 节点。因此，`script`的位置不影响首屏显示的开始时间。
- 浏览器解析 `HTML` 是自上而下的线性过程，`script`作为 `HTML` 的一部分同样遵循这个原则
- 因此，`script` 会延迟 `DomContentLoad`，只显示其上部分首屏内容，从而影响首屏显示的完成时间

#### DOM发展

- `DOM`：文档对象模型（`Document Object Model`），定义了访问HTML和XML文档的标准，与编程语言及平台无关
- `DOM0`：提供了查询和操作Web文档的内容API。未形成标准，实现混乱。如：`document.forms['login']`
- `DOM1`：W3C提出标准化的DOM，简化了对文档中任意部分的访问和操作。如：`JavaScript中的Document`对象
- `DOM2`：原来DOM基础上扩充了鼠标事件等细分模块，增加了对CSS的支持。如：`getComputedStyle(elem, pseudo)`
- `DOM3`：增加了XPath模块和加载与保存（`Load and Save`）模块。如：`XPathEvaluator`

#### 介绍DOM0，DOM2，DOM3事件处理方式区别

- DOM0级事件处理方式：
  - `btn.onclick = func;`
  - `btn.onclick = null;`
- DOM2级事件处理方式：
  - `btn.addEventListener('click', func, false);`
  - `btn.removeEventListener('click', func, false);`
  - `btn.attachEvent("onclick", func);`
  - `btn.detachEvent("onclick", func);`
- DOM3级事件处理方式：
  - `eventUtil.addListener(input, "textInput", func);`
  - `eventUtil` 是自定义对象，`textInput` 是DOM3级事件

#### **W3C事件的 target 与 currentTarget 的区别？**

- `target` 只会出现在事件流的目标阶段
- `currentTarget` 可能出现在事件流的任何阶段
- 当事件流处在目标阶段时，二者的指向相同
- 当事件流处于捕获或冒泡阶段时：`currentTarget` 指向当前事件活动的对象(一般为父级)

#### **如何派发事件(dispatchEvent)？（如何进行事件广播？）**

- W3C: 使用 `dispatchEvent` 方法
- IE: 使用 `fireEvent` 方法

```js
var fireEvent = function(element, event){
    if (document.createEventObject){
        var mockEvent = document.createEventObject();
        return element.fireEvent('on' + event, mockEvent)
    }else{
        var mockEvent = document.createEvent('HTMLEvents');
        mockEvent.initEvent(event, true, true);
        return !element.dispatchEvent(mockEvent);
    }
}
```

#### 解释一下这段代码的意思

```js
  [].forEach.call($$("*"), function(el){
      el.style.outline = "1px solid #" + (~~(Math.random()*(1<<24))).toString(16);
  })
```

- 解释：获取页面所有的元素，遍历这些元素，为它们添加1像素随机颜色的轮廓(outline)
- 1. `$$(sel)` // $$函数被许多现代浏览器命令行支持，等价于 document.querySelectorAll(sel)
- 1. `[].forEach.call(NodeLists)` // 使用 call 函数将数组遍历函数 forEach 应到节点元素列表
- 1. `el.style.outline = "1px solid #333"` // 样式 outline 位于盒模型之外，不影响元素布局位置
- 1. `(1<<24)` // parseInt("ffffff", 16) == 16777215 == 2^24 - 1 // 1<<24 == 2^24 == 16777216
- 1. `Math.random()*(1<<24)` // 表示一个位于 0 到 16777216 之间的随机浮点数
- 1. `~~Math.random()*(1<<24)` // `~~` 作用相当于 parseInt 取整
- 1. `(~~(Math.random()*(1<<24))).toString(16)` // 转换为一个十六进制-

#### 如何删除一个cookie

- 将时间设为当前时间往前一点

```js
var date = new Date();

date.setDate(date.getDate() - 1);//真正的删除
```

> `setDate()`方法用于设置一个月的某一天

- `expires`的设置

```js
document.cookie = 'user='+ encodeURIComponent('name')  + ';expires = ' + new Date(0)
```

#### WEB应用从服务器主动推送Data到客户端有那些方式

- `AJAX` 轮询
- `html5` 服务器推送事件 `(new EventSource(SERVER_URL)).addEventListener("message", func);`
- html5 Websocket

- `(new WebSocket(SERVER_URL)).addEventListener("message", func);`

#### 有四个操作会忽略enumerable为false的属性

- `for...in`循环：只遍历对象自身的和继承的可枚举的属性。
- `Object.keys()`：返回对象自身的所有可枚举的属性的键名。
- `JSON.stringify()`：只串行化对象自身的可枚举的属性。
- `Object.assign()`： 忽略`enumerable`为`false`的属性，只拷贝对象自身的可枚举的属性。