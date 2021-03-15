#### 起源

一段标准的 for 循环代码：

```javascript
var colors = ["red", "green", "blue"];

for (var i = 0, len = colors.length; i < len; i++) {
    console.log(colors[i]);
}
```

看着很简单，但是再回顾这段代码，实际上我们仅仅是需要数组中元素的值，但是却需要提前获取数组长度，声明索引变量等，尤其当多个循环嵌套的时候，更需要使用多个索引变量，代码的复杂度就会大大增加，比如我们使用双重循环进行去重：

```javascript
function unique(array) {
    var res = [];
    for (var i = 0, arrayLen = array.length; i < arrayLen; i++) {
        for (var j = 0, resLen = res.length; j < resLen; j++) {
            if (array[i] === res[j]) {
                break;
            }
        }
        if (j === resLen) {
            res.push(array[i]);
        }
    }
    return res;
}
```

为了消除这种复杂度以及减少循环中的错误(比如错误使用其他循环中的变量)，ES6 提供了迭代器和 for of 循环**共同解决**这个问题。

#### 迭代器

所谓迭代器，其实就是一个具有 next() 方法的对象，每次调用 next() 都会返回一个结果对象，该结果对象有两个属性，value 表示当前的值，done 表示遍历是否结束。

我们直接用 ES5 的语法创建一个迭代器：

```javascript
function createIterator(items) {
    var i = 0;
    return {
        next: function() {
            var done = i >= item.length;
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };
        }
    };
}

// iterator 就是一个迭代器对象
var iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // { done: false, value: 1 }
console.log(iterator.next()); // { done: false, value: 2 }
console.log(iterator.next()); // { done: false, value: 3 }
console.log(iterator.next()); // { done: true, value: undefined }
```

#### For of

除了迭代器之外，我们还需要一个可以遍历迭代器对象的方式，ES6 提供了 for of 语句，我们直接用 for of 遍历一下我们上节生成的遍历器对象试试：

```javascript
var iterator = createIterator([1, 2, 3]);

for (let value of iterator) {
    console.log(value);
}
```

结果报错 `TypeError: iterator is not iterable`，表明我们生成的 iterator 对象并不是 iterable(可遍历的)。

那什么才是可遍历的呢？

其实一种数据结构只要部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”（iterable）。

ES6 规定，默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性，或者说，一个数据结构只要具有 Symbol.iterator 属性，就可以认为是"可遍历的"（iterable）。

举个例子：

```javascript
const obj = {
    value: 1
};

for (value of obj) {
    console.log(value);
}

// TypeError: iterator is not iterable
```

我们直接 for of 遍历一个对象，会报错，然而如果我们给该对象添加 Symbol.iterator 属性：

```javascript
const obj = {
    value: 1
};

obj[Symbol.iterator] = function() {
    return createIterator([1, 2, 3]);
};

for (value of obj) {
    console.log(value);
}

// 1
// 2
// 3
```

由此，我们也可以发现 for of 遍历的其实是对象的 Symbol.iterator 属性。

#### 默认可遍历对象

然而如果我们直接遍历一个数组对象：

```javascript
const colors = ["red", "green", "blue"];

for (let color of colors) {
    console.log(color);
}

// red
// green
// blue
```

尽管我们没有手动添加 Symbol.iterator 属性，还是可以遍历成功，这是因为 ES6 默认部署了 Symbol.iterator 属性，当然我们也可以手动修改这个属性：

```javascript
var colors = ["red", "green", "blue"];

colors[Symbol.iterator] = function() {
    return createIterator([1, 2, 3]);
};

for (let color of colors) {
    console.log(color);
}

// 1
// 2
// 3
```

除了数组之外，还有一些数据结构默认部署了 Symbol.iterator 属性。

所以 for...of 循环可以使用的范围包括：

1. 数组
2. Set
3. Map
4. 类数组对象，如 arguments 对象、DOM NodeList 对象
5. Generator 对象
6. 字符串

#### 模拟实现for of

其实模拟实现 for of 也比较简单，基本就是通过 Symbol.iterator 属性获取迭代器对象，然后使用 while 遍历一下：

```javascript
function forOf(obj, cb) {
    let iterable, result;

    if (typeof obj[Symbol.iterator] !== "function")
        throw new TypeError(result + " is not iterable");
    if (typeof cb !== "function") throw new TypeError("cb must be callable");

    iterable = obj[Symbol.iterator]();

    result = iterable.next();
    while (!result.done) {
        cb(result.value);
        result = iterable.next();
    }
}
```

#### 内建迭代器

为了更好的访问对象中的内容，比如有的时候我们仅需要数组中的值，但有的时候不仅需要使用值还需要使用索引，ES6 为数组、Map、Set 集合内建了以下三种迭代器：

1. entries() 返回一个遍历器对象，用来遍历[键名, 键值]组成的数组。对于数组，键名就是索引值。
2. keys() 返回一个遍历器对象，用来遍历所有的键名。
3. values() 返回一个遍历器对象，用来遍历所有的键值。

以数组为例：

```javascript
var colors = ["red", "green", "blue"];

for (let index of colors.keys()) {
    console.log(index);
}

// 0
// 1
// 2

for (let color of colors.values()) {
    console.log(color);
}

// red
// green
// blue

for (let item of colors.entries()) {
    console.log(item);
}

// [ 0, "red" ]
// [ 1, "green" ]
// [ 2, "blue" ]
```

Map 类型与数组类似，但是对于 Set 类型需要注意以下：

```javascript
var colors = new Set(["red", "green", "blue"]);

for (let index of colors.keys()) {
    console.log(index);
}

// red
// green
// blue

for (let color of colors.values()) {
    console.log(color);
}

// red
// green
// blue

for (let item of colors.entries()) {
    console.log(item);
}

// [ "red", "red" ]
// [ "green", "green" ]
// [ "blue", "blue" ]
```

Set 类型的 keys() 和 values() 返回的是相同的迭代器，这也意味着在 Set 这种数据结构中键名与键值相同。

而且每个集合类型都有一个默认的迭代器，在 for-of 循环中，如果没有显式指定则使用默认的迭代器。数组和 Set 集合的默认迭代器是 values() 方法，Map 集合的默认迭代器是 entries() 方法。

这也就是为什么直接 for of 遍历 Set 和 Map 数据结构，会有不同的数据结构返回：

```javascript
const values = new Set([1, 2, 3]);

for (let value of values) {
    console.log(value);
}

// 1
// 2
// 3
const values = new Map([["key1", "value1"], ["key2", "value2"]]);
for (let value of values) {
    console.log(value);
}

// ["key1", "value1"]
// ["key2", "value2"]
```

遍历 Map 数据结构的时候可以顺便结合解构赋值：

```javascript
const valuess = new Map([["key1", "value1"], ["key2", "value2"]]);

for (let [key, value] of valuess) {
    console.log(key + ":" + value);
}

// key1:value1
// key2:value2
```

#### Babel是如何编译 for of的

我们可以在 Babel 的 [Try it out](http://babeljs.io/repl) 中查看编译的结果：

```
const colors = new Set(["red", "green", "blue"]);

for (let color of colors) {
    console.log(color);
}
```

对于这样一段代码，编译的结果如下：

```
"use strict";

var colors = new Set(["red", "green", "blue"]);

var _iteratorNormalCompletion = true;
var _didIteratorError = false;
var _iteratorError = undefined;

try {
    for (
        var _iterator = colors[Symbol.iterator](), _step;
        !(_iteratorNormalCompletion = (_step = _iterator.next()).done);
        _iteratorNormalCompletion = true
    ) {
        var color = _step.value;

        console.log(color);
    }
} catch (err) {
    _didIteratorError = true;
    _iteratorError = err;
} finally {
    try {
        if (!_iteratorNormalCompletion && _iterator.return) {
            _iterator.return();
        }
    } finally {
        if (_didIteratorError) {
            throw _iteratorError;
        }
    }
}
```

至少由编译的结果可以看出，使用 `for of` 循环的背后，还是会使用 Symbol.iterator 接口。

而这段编译的代码稍微复杂的地方有两段，一段是 for 循环这里：

```
for (
    var _iterator = colors[Symbol.iterator](), _step;
    !(_iteratorNormalCompletion = (_step = _iterator.next()).done);
    _iteratorNormalCompletion = true
) {
    var color = _step.value;
    console.log(color);
}
```

跟标准的 for 循环写法有些差别，我们看下 for 语句的语法：

```
for (initialize; test; increment) statement;
```

initialize、test 和 increment 三个表达式之间用分号分割，它们分别负责`初始化操作`、`循环条件判断`和`计数器变量的更新`。

for 语句其实就相当于：

```
initialize;
while (test) {
    statement;
    increment;
}
```

代码的逻辑为：先进行初始化，然后每次循环执行之前会执行 test 表达式，并判断表达式的结果来决定是否执行循环体，如果 test 计算结果为真值，则执行循环体中的 statement。最后，执行 increment 表达式。

而且值得注意的是，其实 for 循环中的三个表达式中任意一个都可以被忽略，不过分号还是要写的。

比如 `for(;;)`，不过这就是一个死循环……

比如：

```
var i = 0,
    len = colors.length;
for (; i < len; i++) {
    console.log(colors[i]);
}
```

又比如：

```
var i = 0,
    len = colors.length;
for (; i < len; ) {
    i++;
}
```

然后我们再来看 Babel 编译的这个 for 循环表达式：

```
for (
    var _iterator = colors[Symbol.iterator](), _step;
    !(_iteratorNormalCompletion = (_step = _iterator.next()).done);
    _iteratorNormalCompletion = true
) {
    var color = _step.value;
    console.log(color);
}
```

用 while 的写法相当于：

```
var _iterator = colors[Symbol.iterator](),
    _step;
while (!(_iteratorNormalCompletion = (_step = _iterator.next()).done)) {
    var color = _step.value;
    console.log(color);
    _iteratorNormalCompletion = true;
}
```

是不是就好懂了很多呢，然后你就会发现，其实 `_iteratorNormalCompletion = true` 这句是完全没有必要的……

另外一段稍微复杂的代码是:

```
try {
  ...
} catch (err) {
  ...
} finally {
  try {
    if (!_iteratorNormalCompletion && _iterator.return) {
      _iterator.return();
    }
  } finally {
    ...
  }
}
```

因为 `_iteratorNormalCompletion = (_step = _iterator.next()).done`，所以 _iteratorNormalCompletion 表示的就是是否完成了一次完整的迭代过程，如果没有正常的迭代完成，并且迭代器有 return 方法时，就会执行该方法。

而之所以这么做，就要提到迭代器的 return 方法。

引用阮一峰老师的 [ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/iterator#遍历器对象的-return，throw):

> 遍历器对象除了具有 next 方法，还可以具有 return 方法和 throw 方法。如果你自己写遍历器对象生成函数，那么 next 方法是必须部署的，return 方法和 throw 方法是否部署是可选的。

> return 方法的使用场合是，如果 for...of 循环提前退出（通常是因为出错，或者有 break 语句或 continue 语句），就会调用 return 方法。如果一个对象在完成遍历前，需要清理或释放资源，就可以部署 return 方法。

我们可以举个例子：

```
function createIterator(items) {
    var i = 0;
    return {
        next: function() {
            var done = i >= items.length;
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };
        },
        return: function() {
            console.log("执行了 return 方法");
            return {
                value: 23333,
                done: true
            };
        }
    };
}

var colors = ["red", "green", "blue"];

var iterator = createIterator([1, 2, 3]);

colors[Symbol.iterator] = function() {
    return iterator;
};

for (let color of colors) {
    if (color == 1) break;
    console.log(color);
}
// 执行了 return 方法
```

不过正如你在编译后的代码中看到，仅仅是在有 return 函数的时候执行了 return 函数而已，return 函数中返回的值其实并不生效……

但是你不返回值或者返回一个基本类型的值的话，结果又会报错……

```
TypeError: Iterator result undefined is not an object
```

这是因为 return 方法必须返回一个对象，而这又是 Generator 规范决定的……

总之如果是在浏览器中使用的话，return 函数的返回值其实并不生效 T^T

