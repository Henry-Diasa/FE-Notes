#### 前言

本篇我们重点介绍以下四种模块加载规范：

1. AMD
2. CMD
3. CommonJS
4. ES6 模块

最后再延伸讲下 Babel 的编译和 webpack 的打包原理。

#### require.js

在了解 AMD 规范之前，我们先来看看 require.js 的使用方式。

项目目录为：

```
* project/
    * index.html
    * vender/
        * main.js
        * require.js
        * add.js
        * square.js
        * multiply.js
```

`index.html` 的内容如下：

```
<!DOCTYPE html>
<html>
    <head>
        <title>require.js</title>
    </head>
    <body>
        <h1>Content</h1>
        <script data-main="vender/main" src="vender/require.js"></script>
    </body>
</html>
```

`data-main="vender/main"` 表示主模块是 `vender` 下的 `main.js`。

`main.js` 的配置如下：

```
// main.js
require(['./add', './square'], function(addModule, squareModule) {
    console.log(addModule.add(1, 1))
    console.log(squareModule.square(3))
});
```

require 的第一个参数表示依赖的模块的路径，第二个参数表示此模块的内容。

由此可以看出，`主模块`依赖 `add 模块`和 `square 模块`。

我们看下 `add 模块`即 `add.js` 的内容：

```
// add.js
define(function() {
    console.log('加载了 add 模块');
    var add = function(x, y) {　
        return x + y;
    };

    return {　　　　　　
        add: add
    };
});
```

`requirejs` 为全局添加了 `define` 函数，你只要按照这种约定的方式书写这个模块即可。

那如果依赖的模块又依赖了其他模块呢？

我们来看看`主模块`依赖的 `square 模块`， `square 模块`的作用是求出一个数字的平方，比如输入 3 就返回 9，该模块依赖一个`乘法模块`，该乘法模块即 `multiply.js` 的代码如下：

```
// multiply.js
define(function() {
    console.log('加载了 multiply 模块')
    var multiply = function(x, y) {　
        return x * y;
    };

    return {　　　　　　
        multiply: multiply
    };
});
```

而 `square 模块`就要用到 `multiply 模块`，其实写法跟 main.js 添加依赖模块一样：

```
// square.js
define(['./multiply'], function(multiplyModule) {
    console.log('加载了 square 模块')
    return {　　　　　　
        square: function(num) {
            return multiplyModule.multiply(num, num)
        }
    };
});
```

require.js 会自动分析依赖关系，将需要加载的模块正确加载。

而如果我们在浏览器中打开 `index.html`，打印的顺序为：

```
加载了 add 模块
加载了 multiply 模块
加载了 square 模块
2
9
```

#### AMD

在上节，我们说了这样一句话：

> `requirejs` 为全局添加了 `define` 函数，你只要按照这种约定的方式书写这个模块即可。

那这个约定的书写方式是指什么呢？

指的便是 The Asynchronous Module Definition (AMD) 规范。

所以其实 **AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。**

你去看 [AMD 规范](https://github.com/amdjs/amdjs-api/wiki/AMD-(中文版)) 的内容，其主要内容就是定义了 define 函数该如何书写，只要你按照这个规范书写模块和依赖，require.js 就能正确的进行解析。

#### Sea.js

在国内，经常与 AMD 被一起提起的还有 CMD，CMD 又是什么呢？我们从 `sea.js` 的使用开始说起。

文件目录与 requirejs 项目目录相同:

```
* project/
    * index.html
    * vender/
        * main.js
        * require.js
        * add.js
        * square.js
        * multiply.js
```

`index.html` 的内容如下：

```
<!DOCTYPE html>
<html>
<head>
    <title>sea.js</title>
</head>
<body>
    <h1>Content</h1>
    <script src="vender/sea.js"></script>
    <script>
    // 在页面中加载主模块
    seajs.use("./vender/main");
    </script>
</body>

</html>
```

main.js 的内容如下：

```
// main.js
define(function(require, exports, module) {
    var addModule = require('./add');
    console.log(addModule.add(1, 1))

    var squareModule = require('./square');
    console.log(squareModule.square(3))
});
```

add.js 的内容如下：

```
// add.js
define(function(require, exports, module) {
    console.log('加载了 add 模块')
    var add = function(x, y) {　
        return x + y;
    };
    module.exports = {　　　　　　
        add: add
    };
});
```

square.js 的内容如下：

```
define(function(require, exports, module) {
    console.log('加载了 square 模块')
    var multiplyModule = require('./multiply');
    module.exports = {　　　　　　
        square: function(num) {
            return multiplyModule.multiply(num, num)
        }
    };

});
```

multiply.js 的内容如下：

```
define(function(require, exports, module) {
    console.log('加载了 multiply 模块')
    var multiply = function(x, y) {　
        return x * y;
    };
    module.exports = {　　　　　　
        multiply: multiply
    };
});
```

跟第一个例子是同样的依赖结构，即 main 依赖 add 和 square，square 又依赖 multiply。

而如果我们在浏览器中打开 `index.html`，打印的顺序为：

```
加载了 add 模块
2
加载了 square 模块
加载了 multiply 模块
9
```

#### CMD

与 AMD 一样，CMD 其实就是 SeaJS 在推广过程中对模块定义的规范化产出。

你去看 [CMD 规范](https://github.com/seajs/seajs/issues/242)的内容，主要内容就是描述该如何定义模块，如何引入模块，如何导出模块，只要你按照这个规范书写代码，sea.js 就能正确的进行解析。

#### AMD与CMD的区别

从 sea.js 和 require.js 的例子可以看出：

1.CMD 推崇**依赖就近**，AMD 推崇**依赖前置**。看两个项目中的 main.js：

```
// require.js 例子中的 main.js
// 依赖必须一开始就写好
require(['./add', './square'], function(addModule, squareModule) {
    console.log(addModule.add(1, 1))
    console.log(squareModule.square(3))
});
// sea.js 例子中的 main.js
define(function(require, exports, module) {
    var addModule = require('./add');
    console.log(addModule.add(1, 1))

    // 依赖可以就近书写
    var squareModule = require('./square');
    console.log(squareModule.square(3))
});
```

2.对于依赖的模块，AMD 是**提前执行**，CMD 是**延迟执行**。看两个项目中的打印顺序：

```
// require.js
加载了 add 模块
加载了 multiply 模块
加载了 square 模块
2
9
// sea.js
加载了 add 模块
2
加载了 square 模块
加载了 multiply 模块
9
```

AMD 是将需要使用的模块先加载完再执行代码，而 CMD 是在 require 的时候才去加载模块文件，加载完再接着执行。

#### CommonJS

AMD 和 CMD 都是用于浏览器端的模块规范，而在服务器端比如 node，采用的则是 CommonJS 规范。

导出模块的方式：

```
var add = function(x, y) {　
    return x + y;
};

module.exports.add = add;
```

引入模块的方式：

```
var add = require('./add.js');
console.log(add.add(1, 1));
```

我们将之前的例子改成 CommonJS 规范：

```
// main.js
var add = require('./add.js');
console.log(add.add(1, 1))

var square = require('./square.js');
console.log(square.square(3));
// add.js
console.log('加载了 add 模块')

var add = function(x, y) {　
    return x + y;
};

module.exports.add = add;
// multiply.js
console.log('加载了 multiply 模块')

var multiply = function(x, y) {　
    return x * y;
};

module.exports.multiply = multiply;
// square.js
console.log('加载了 square 模块')

var multiply = require('./multiply.js');

var square = function(num) {　
    return multiply.multiply(num, num);
};

module.exports.square = square;
```

如果我们执行 `node main.js`，打印的顺序为：

```
加载了 add 模块
2
加载了 square 模块
加载了 multiply 模块
9
```

跟 sea.js 的执行结果一致，也是在 require 的时候才去加载模块文件，加载完再接着执行。

#### CommonJS与AMD

引用阮一峰老师的[《JavaScript 标准参考教程（alpha）》](http://javascript.ruanyifeng.com/nodejs/module.html):

> CommonJS 规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。

> AMD规范则是非同步加载模块，允许指定回调函数。
>
> 由于 Node.js 主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以 CommonJS 规范比较适用。

> 但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用 AMD 规范。

#### ES6

ECMAScript2015 规定了新的模块加载方案。

导出模块的方式：

```
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

引入模块的方式：

```
import {firstName, lastName, year} from './profile';
```

我们再将上面的例子改成 ES6 规范：

目录结构与 requirejs 和 seajs 目录结构一致。

```
<!DOCTYPE html>
<html>
    <head>
        <title>ES6</title>
    </head>
    <body>
        <h1>Content</h1>
        <script src="vender/main.js" type="module"></script>
    </body>
</html>
```

注意！浏览器加载 ES6 模块，也使用 `<script>` 标签，但是要加入 `type="module"` 属性。

```
// main.js
import {add} from './add.js';
console.log(add(1, 1))

import {square} from './square.js';
console.log(square(3));
// add.js
console.log('加载了 add 模块')

var add = function(x, y) {
    return x + y;
};

export {add}
// multiply.js
console.log('加载了 multiply 模块')

var multiply = function(x, y) {　
    return x * y;
};

export {multiply}
// square.js
console.log('加载了 square 模块')

import {multiply} from './multiply.js';

var square = function(num) {　
    return multiply(num, num);
};

export {square}
```

值得注意的，在 Chrome 中，如果直接打开，会报跨域错误，必须开启服务器，保证文件同源才可以有效果。

为了验证这个效果你可以：

```
cnpm install http-server -g
```

然后进入该目录，执行

```
http-server
```

在浏览器打开 `http://localhost:8080/` 即可查看效果。

打印的顺序为：

```
加载了 add 模块
加载了 multiply 模块
加载了 square 模块
2
9
```

跟 require.js 的执行结果是一致的，也就是将需要使用的模块先加载完再执行代码。

#### ES6和CommonJS

引用阮一峰老师的 [《ECMAScript 6 入门》](http://es6.ruanyifeng.com/#docs/module-loader)：

> 它们有两个重大差异。
>
> 1. CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
> 2. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

第二个差异可以从两个项目的打印结果看出，导致这种差别的原因是：

> 因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

重点解释第一个差异。

> CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。

举个例子：

```
// 输出模块 counter.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
    counter: counter,
    incCounter: incCounter,
};
// 引入模块 main.js
var mod = require('./counter');

console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```

counter.js 模块加载以后，它的内部变化就影响不到输出的 mod.counter 了。这是因为 mod.counter 是一个原始类型的值，会被缓存。

但是如果修改 counter 为一个引用类型的话：

```
// 输出模块 counter.js
var counter = {
    value: 3
};

function incCounter() {
    counter.value++;
}
module.exports = {
    counter: counter,
    incCounter: incCounter,
};
// 引入模块 main.js
var mod = require('./counter.js');

console.log(mod.counter.value); // 3
mod.incCounter();
console.log(mod.counter.value); // 4
```

value 是会发生改变的。不过也可以说这是 "值的拷贝"，只是对于引用类型而言，值指的其实是引用。

而如果我们将这个例子改成 ES6:

```
// counter.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './counter';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

这是因为

> ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令 import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6 的 import 有点像 Unix 系统的“符号连接”，原始值变了，import 加载的值也会跟着变。因此，ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

#### Babel

鉴于浏览器支持度的问题，如果要使用 ES6 的语法，一般都会借助 Babel，可对于 import 和 export 而言，只借助 Babel 就可以吗？

让我们看看 Babel 是怎么编译 import 和 export 语法的。

```
// ES6
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
// Babel 编译后
'use strict';

Object.defineProperty(exports, "__esModule", {
  value: true
});
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

exports.firstName = firstName;
exports.lastName = lastName;
exports.year = year;
```

是不是感觉有那么一点奇怪？编译后的语法更像是 CommonJS 规范，再看 import 的编译结果：

```
// ES6
import {firstName, lastName, year} from './profile';
// Babel 编译后
'use strict';

var _profile = require('./profile');
```

你会发现 Babel 只是把 ES6 模块语法转为 CommonJS 模块语法，然而浏览器是不支持这种模块语法的，所以直接跑在浏览器会报错的，如果想要在浏览器中运行，还是需要使用打包工具将代码打包。

#### webpack

Babel 将 ES6 模块转为 CommonJS 后， webpack 又是怎么做的打包的呢？它该如何将这些文件打包在一起，从而能保证正确的处理依赖，以及能在浏览器中运行呢？

首先为什么浏览器中不支持 CommonJS 语法呢？

这是因为浏览器环境中并没有 module、 exports、 require 等环境变量。

换句话说，webpack 打包后的文件之所以在浏览器中能运行，就是靠模拟了这些变量的行为。

那怎么模拟呢？

我们以 CommonJS 项目中的 square.js 为例，它依赖了 multiply 模块：

```
console.log('加载了 square 模块')

var multiply = require('./multiply.js');


var square = function(num) {　
    return multiply.multiply(num, num);
};

module.exports.square = square;
```

webpack 会将其包裹一层，注入这些变量：

```
function(module, exports, require) {
    console.log('加载了 square 模块');

    var multiply = require("./multiply");
    module.exports = {
        square: function(num) {
            return multiply.multiply(num, num);
        }
    };
}
```

那 webpack 又会将 CommonJS 项目的代码打包成什么样呢？我写了一个精简的例子，你可以直接复制到浏览器中查看效果：

```
// 自执行函数
(function(modules) {

    // 用于储存已经加载过的模块
    var installedModules = {};

    function require(moduleName) {

        if (installedModules[moduleName]) {
            return installedModules[moduleName].exports;
        }

        var module = installedModules[moduleName] = {
            exports: {}
        };

        modules[moduleName](module, module.exports, require);

        return module.exports;
    }

    // 加载主模块
    return require("main");

})({
    "main": function(module, exports, require) {

        var addModule = require("./add");
        console.log(addModule.add(1, 1))

        var squareModule = require("./square");
        console.log(squareModule.square(3));

    },
    "./add": function(module, exports, require) {
        console.log('加载了 add 模块');

        module.exports = {
            add: function(x, y) {
                return x + y;
            }
        };
    },
    "./square": function(module, exports, require) {
        console.log('加载了 square 模块');

        var multiply = require("./multiply");
        module.exports = {
            square: function(num) {
                return multiply.multiply(num, num);
            }
        };
    },

    "./multiply": function(module, exports, require) {
        console.log('加载了 multiply 模块');

        module.exports = {
            multiply: function(x, y) {
                return x * y;
            }
        };
    }
})
```

最终的执行结果为：

```
加载了 add 模块
2
加载了 square 模块
加载了 multiply 模块
9
```

#### 参考

- [《JavaScript 标准参考教程（alpha）》](http://javascript.ruanyifeng.com/nodejs/module.html)

- [《ECMAScript6 入门》](http://es6.ruanyifeng.com/#docs/module-loader)

- [手写一个CommonJS打包工具（一）](https://zhuanlan.zhihu.com/p/20731484)

  