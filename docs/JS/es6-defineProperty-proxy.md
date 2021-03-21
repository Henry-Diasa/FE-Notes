#### 前言

我们或多或少都听过“数据绑定”这个词，“数据绑定”的关键在于监听数据的变化，可是对于这样一个对象：`var obj = {value: 1}`，我们该怎么知道 obj 发生了改变呢？

#### defineProperty

ES5 提供了 Object.defineProperty 方法，该方法可以在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。

**语法**

> Object.defineProperty(obj, prop, descriptor)

**参数**

```
obj: 要在其上定义属性的对象。

prop:  要定义或修改的属性的名称。

descriptor: 将被定义或修改的属性的描述符。
```

举个例子：

```javascript
var obj = {};
Object.defineProperty(obj, "num", {
    value : 1,
    writable : true,
    enumerable : true,
    configurable : true
});
//  对象 obj 拥有属性 num，值为 1
```

虽然我们可以直接添加属性和值，但是使用这种方式，我们能进行更多的配置。

函数的第三个参数 descriptor 所表示的属性描述符有两种形式：**数据描述符和存取描述符**。

**两者均具有以下两种键值**：

**两者均具有以下两种键值**：

**configurable**

```javascript
当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，也能够被删除。默认为 false。
```

**enumerable**

```javascript
当且仅当该属性的 enumerable 为 true 时，该属性才能够出现在对象的枚举属性中。默认为 false。
```

**数据描述符同时具有以下可选键值**：

**value**

```
该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。
```

**writable**

```
当且仅当该属性的 writable 为 true 时，该属性才能被赋值运算符改变。默认为 false。
```

**存取描述符同时具有以下可选键值**：

**get**

```
一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。该方法返回值被用作属性值。默认为 undefined。
```

**set**

```
一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将接受唯一参数，并将该参数的新值分配给该属性。默认为 undefined。
```

值得注意的是：

**属性描述符必须是数据描述符或者存取描述符两种形式之一，不能同时是两者** 。这就意味着你可以：

```javascript
Object.defineProperty({}, "num", {
    value: 1,
    writable: true,
    enumerable: true,
    configurable: true
});
```

也可以：

```javascript
var value = 1;
Object.defineProperty({}, "num", {
    get : function(){
      return value;
    },
    set : function(newValue){
      value = newValue;
    },
    enumerable : true,
    configurable : true
});
```

但是不可以：

```javascript
// 报错
Object.defineProperty({}, "num", {
    value: 1,
    get: function() {
        return 1;
    }
});
```

此外，所有的属性描述符都是非必须的，但是 descriptor 这个字段是必须的，如果不进行任何配置，你可以这样：

```javascript
var obj = Object.defineProperty({}, "num", {});
console.log(obj.num); // undefined
```

#### Setter和Getter

之所以讲到 defineProperty，是因为我们要使用存取描述符中的 get 和 set，这两个方法又被称为 getter 和 setter。由 getter 和 setter 定义的属性称做”存取器属性“。

当程序查询存取器属性的值时，JavaScript 调用 getter方法。这个方法的返回值就是属性存取表达式的值。当程序设置一个存取器属性的值时，JavaScript 调用 setter 方法，将赋值表达式右侧的值当做参数传入 setter。从某种意义上讲，这个方法负责“设置”属性值。可以忽略 setter 方法的返回值。

举个例子：

```javascript
var obj = {}, value = null;
Object.defineProperty(obj, "num", {
    get: function(){
        console.log('执行了 get 操作')
        return value;
    },
    set: function(newValue) {
        console.log('执行了 set 操作')
        value = newValue;
    }
})

obj.num = 1 // 执行了 set 操作

console.log(obj.num); // 执行了 get 操作 // 1
```

这不就是我们要的监控数据改变的方法吗？我们再来封装一下：

```javascript
function Archiver() {
    var value = null;
    // archive n. 档案
    var archive = [];

    Object.defineProperty(this, 'num', {
        get: function() {
            console.log('执行了 get 操作')
            return value;
        },
        set: function(value) {
            console.log('执行了 set 操作')
            value = value;
            archive.push({ val: value });
        }
    });

    this.getArchive = function() { return archive; };
}

var arc = new Archiver();
arc.num; // 执行了 get 操作
arc.num = 11; // 执行了 set 操作
arc.num = 13; // 执行了 set 操作
console.log(arc.getArchive()); // [{ val: 11 }, { val: 13 }]
```

#### Watch api

既然可以监控数据的改变，那我可以这样设想，即当数据改变的时候，自动进行渲染工作。举个例子：

HTML 中有个 span 标签和 button 标签

```
<span id="container">1</span>
<button id="button">点击加 1</button>
```

当点击按钮的时候，span 标签里的值加 1。

传统的做法是：

```
document.getElementById('button').addEventListener("click", function(){
    var container = document.getElementById("container");
    container.innerHTML = Number(container.innerHTML) + 1;
});
```

如果使用了 defineProperty：

```
var obj = {
    value: 1
}

// 储存 obj.value 的值
var value = 1;

Object.defineProperty(obj, "value", {
    get: function() {
        return value;
    },
    set: function(newValue) {
        value = newValue;
        document.getElementById('container').innerHTML = newValue;
    }
});

document.getElementById('button').addEventListener("click", function() {
    obj.value += 1;
});
```

代码看似增多了，但是当我们需要改变 span 标签里的值的时候，直接修改 obj.value 的值就可以了。

然而，现在的写法，我们还需要单独声明一个变量存储 obj.value 的值，因为如果你在 set 中直接 `obj.value = newValue` 就会陷入无限的循环中。此外，我们可能需要监控很多属性值的改变，要是一个一个写，也很累呐，所以我们简单写个 watch 函数。使用效果如下：

```
var obj = {
    value: 1
}

watch(obj, "value", function(newvalue){
    document.getElementById('container').innerHTML = newvalue;
})

document.getElementById('button').addEventListener("click", function(){
    obj.value += 1
});
```

我们来写下这个 watch 函数：

```
(function(){
    var root = this;
    function watch(obj, name, func){
        var value = obj[name];

        Object.defineProperty(obj, name, {
            get: function() {
                return value;
            },
            set: function(newValue) {
                value = newValue;
                func(value)
            }
        });

        if (value) obj[name] = value
    }

    this.watch = watch;
})()
```

现在我们已经可以监控对象属性值的改变，并且可以根据属性值的改变，添加回调函数，棒棒哒~

#### Proxy

使用 defineProperty 只能重定义属性的读取（get）和设置（set）行为，到了 ES6，提供了 Proxy，可以重定义更多的行为，比如 in、delete、函数调用等更多行为。

Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，ES6 原生提供 Proxy 构造函数，用来生成 Proxy 实例。我们来看看它的语法：

```
var proxy = new Proxy(target, handler);
```

proxy 对象的所有用法，都是上面这种形式，不同的只是handler参数的写法。其中，new Proxy()表示生成一个Proxy实例，target参数表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。

```
var proxy = new Proxy({}, {
    get: function(obj, prop) {
        console.log('设置 get 操作')
        return obj[prop];
    },
    set: function(obj, prop, value) {
        console.log('设置 set 操作')
        obj[prop] = value;
    }
});

proxy.time = 35; // 设置 set 操作

console.log(proxy.time); // 设置 get 操作 // 35
```

除了 get 和 set 之外，proxy 可以拦截多达 13 种操作，比如 has(target, propKey)，可以拦截 propKey in proxy 的操作，返回一个布尔值。

```
// 使用 has 方法隐藏某些属性，不被 in 运算符发现
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: 'foo', prop: 'foo' };
var proxy = new Proxy(target, handler);
console.log('_prop' in proxy); // false
```

又比如说 apply 方法拦截函数的调用、call 和 apply 操作。

apply 方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组，不过这里我们简单演示一下：

```
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p();
// "I am the proxy"
```

又比如说 ownKeys 方法可以拦截对象自身属性的读取操作。具体来说，拦截以下操作：

- Object.getOwnPropertyNames()
- Object.getOwnPropertySymbols()
- Object.keys()

下面的例子是拦截第一个字符为下划线的属性名，不让它被 for of 遍历到。

```
let target = {
  _bar: 'foo',
  _prop: 'bar',
  prop: 'baz'
};

let handler = {
  ownKeys (target) {
    return Reflect.ownKeys(target).filter(key => key[0] !== '_');
  }
};

let proxy = new Proxy(target, handler);
for (let key of Object.keys(proxy)) {
  console.log(target[key]);
}
// "baz"
```

更多的拦截行为可以查看阮一峰老师的 [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/proxy)

值得注意的是，proxy 的最大问题在于浏览器支持度不够，而且很多效果无法使用 poilyfill 来弥补。

#### watch API优化

我们使用 proxy 再来写一下 watch 函数。使用效果如下：

```
(function() {
    var root = this;

    function watch(target, func) {

        var proxy = new Proxy(target, {
            get: function(target, prop) {
                return target[prop];
            },
            set: function(target, prop, value) {
                target[prop] = value;
                func(prop, value);
            }
        });

        return proxy;
    }

    this.watch = watch;
})()

var obj = {
    value: 1
}

var newObj = watch(obj, function(key, newvalue) {
    if (key == 'value') document.getElementById('container').innerHTML = newvalue;
})

document.getElementById('button').addEventListener("click", function() {
    newObj.value += 1
});
```

我们也可以发现，使用 defineProperty 和 proxy 的区别，当使用 defineProperty，我们修改原来的 obj 对象就可以触发拦截，而使用 proxy，就必须修改代理对象，即 Proxy 的实例才可以触发拦截。