#### 单个异步任务

```javascript
var fetch = require('node-fetch');

function* gen(){
    var url = 'https://api.github.com/users/github';
    var result = yield fetch(url);
    console.log(result.bio);
}
```

为了获得最终的执行结果，你需要这样做：

```javascript
var g = gen();
var result = g.next();

result.value.then(function(data){
    return data.json();
}).then(function(data){
    g.next(data);
});
```

首先执行 Generator 函数，获取遍历器对象。

然后使用 next 方法，执行异步任务的第一阶段，即 fetch(url)。

注意，由于 fetch(url) 会返回一个 Promise 对象，所以 result 的值为：

```javascript
{ value: Promise { <pending> }, done: false }
```

最后我们为这个 Promise 对象添加一个 then 方法，先将其返回的数据格式化(`data.json()`)，再调用 g.next，将获得的数据传进去，由此可以执行异步任务的第二阶段，代码执行完毕。

#### 多个异步任务

上节我们只调用了一个接口，那如果我们调用了多个接口，使用了多个 yield，我们岂不是要在 then 函数中不断的嵌套下去……

所以我们来看看执行多个异步任务的情况：

```javascript
var fetch = require('node-fetch');

function* gen() {
    var r1 = yield fetch('https://api.github.com/users/github');
    var r2 = yield fetch('https://api.github.com/users/github/followers');
    var r3 = yield fetch('https://api.github.com/users/github/repos');

    console.log([r1.bio, r2[0].login, r3[0].full_name].join('\n'));
}
```

为了获得最终的执行结果，你可能要写成：

```javascript
var g = gen();
var result1 = g.next();

result1.value.then(function(data){
    return data.json();
})
.then(function(data){
    return g.next(data).value;
})
.then(function(data){
    return data.json();
})
.then(function(data){
    return g.next(data).value
})
.then(function(data){
    return data.json();
})
.then(function(data){
    g.next(data)
});
```

但我知道你肯定不想写成这样……

其实，利用递归，我们可以这样写：

```javascript
function run(gen) {
    var g = gen();

    function next(data) {
        var result = g.next(data);

        if (result.done) return;

        result.value.then(function(data) {
            return data.json();
        }).then(function(data) {
            next(data);
        });

    }

    next();
}

run(gen);
```

其中的关键就是 yield 的时候返回一个 Promise 对象，给这个 Promise 对象添加 then 方法，当异步操作成功时执行 then 中的 onFullfilled 函数，onFullfilled 函数中又去执行 g.next，从而让 Generator 继续执行，然后再返回一个 Promise，再在成功时执行 g.next，然后再返回……

#### 启动器函数

在 run 这个启动器函数中，我们在 then 函数中将数据格式化 `data.json()`，但在更广泛的情况下，比如 yield 直接跟一个 Promise，而非一个 fetch 函数返回的 Promise，因为没有 json 方法，代码就会报错。所以为了更具备通用性，连同这个例子和启动器，我们修改为：

```javascript
var fetch = require('node-fetch');

function* gen() {
    var r1 = yield fetch('https://api.github.com/users/github');
    var json1 = yield r1.json();
    var r2 = yield fetch('https://api.github.com/users/github/followers');
    var json2 = yield r2.json();
    var r3 = yield fetch('https://api.github.com/users/github/repos');
    var json3 = yield r3.json();

    console.log([json1.bio, json2[0].login, json3[0].full_name].join('\n'));
}

function run(gen) {
    var g = gen();

    function next(data) {
        var result = g.next(data);

        if (result.done) return;

        result.value.then(function(data) {
            next(data);
        });

    }

    next();
}

run(gen);
```

只要 yield 后跟着一个 Promise 对象，我们就可以利用这个 run 函数将 Generator 函数自动执行。

#### 回调函数

yield 后一定要跟着一个 Promise 对象才能保证 Generator 的自动执行吗？如果只是一个回调函数呢？我们来看个例子：

首先我们来模拟一个普通的异步请求：

```javascript
function fetchData(url, cb) {
    setTimeout(function(){
        cb({status: 200, data: url})
    }, 1000)
}
```

我们将这种函数改造成：

```javascript
function fetchData(url) {
    return function(cb){
        setTimeout(function(){
            cb({status: 200, data: url})
        }, 1000)
    }
}
```

对于这样的 Generator 函数：

```javascript
function* gen() {
    var r1 = yield fetchData('https://api.github.com/users/github');
    var r2 = yield fetchData('https://api.github.com/users/github/followers');

    console.log([r1.data, r2.data].join('\n'));
}
```

如果要获得最终的结果：

```javascript
var g = gen();

var r1 = g.next();

r1.value(function(data) {
    var r2 = g.next(data);
    r2.value(function(data) {
        g.next(data);
    });
});
```

如果写成这样的话，我们会面临跟第一节同样的问题，那就是当使用多个 yield 时，代码会循环嵌套起来……

同样利用递归，所以我们可以将其改造为：

```javascript
function run(gen) {
    var g = gen();

    function next(data) {
        var result = g.next(data);

        if (result.done) return;

        result.value(next);
    }

    next();
}

run(gen);
```

#### run

由此可以看到 Generator 函数的自动执行需要一种机制，即当异步操作有了结果，能够自动交回执行权。

而两种方法可以做到这一点。

（1）回调函数。将异步操作进行包装，暴露出回调函数，在回调函数里面交回执行权。

（2）Promise 对象。将异步操作包装成 Promise 对象，用 then 方法交回执行权。

在两种方法中，我们各写了一个 run 启动器函数，那我们能不能将这两种方式结合在一些，写一个通用的 run 函数呢？我们尝试一下：

```javascript
// 第一版
function run(gen) {
    var gen = gen();

    function next(data) {
        var result = gen.next(data);
        if (result.done) return;

        if (isPromise(result.value)) {
            result.value.then(function(data) {
                next(data);
            });
        } else {
            result.value(next)
        }
    }

    next()
}

function isPromise(obj) {
    return 'function' == typeof obj.then;
}

module.exports = run;
```

其实实现的很简单，判断 result.value 是否是 Promise，是就添加 then 函数，不是就直接执行。

#### Return Promise

我们已经写了一个不错的启动器函数，支持 yield 后跟回调函数或者 Promise 对象。

现在有一个问题需要思考，就是我们如何获得 Generator 函数的返回值呢？又如果 Generator 函数中出现了错误，就比如 fetch 了一个不存在的接口，这个错误该如何捕获呢？

这很容易让人想到 Promise，如果这个启动器函数返回一个 Promise，我们就可以给这个 Promise 对象添加 then 函数，当所有的异步操作执行成功后，我们执行 onFullfilled 函数，如果有任何失败，就执行 onRejected 函数。

我们写一版：

```javascript
// 第二版
function run(gen) {
    var gen = gen();

    return new Promise(function(resolve, reject) {

        function next(data) {
            try {
                var result = gen.next(data);
            } catch (e) {
                return reject(e);
            }

            if (result.done) {
                return resolve(result.value)
            };

            var value = toPromise(result.value);

            value.then(function(data) {
                next(data);
            }, function(e) {
                reject(e)
            });
        }

        next()
    })

}

function isPromise(obj) {
    return 'function' == typeof obj.then;
}

function toPromise(obj) {
    if (isPromise(obj)) return obj;
    if ('function' == typeof obj) return thunkToPromise(obj);
    return obj;
}

function thunkToPromise(fn) {
    return new Promise(function(resolve, reject) {
        fn(function(err, res) {
            if (err) return reject(err);
            resolve(res);
        });
    });
}

module.exports = run;
```

与第一版有很大的不同：

首先，我们返回了一个 Promise，当 `result.done` 为 true 的时候，我们将该值 `resolve(result.value)`，如果执行的过程中出现错误，被 catch 住，我们会将原因 `reject(e)`。

其次，我们会使用 `thunkToPromise` 将回调函数包装成一个 Promise，然后统一的添加 then 函数。在这里值得注意的是，在 `thunkToPromise` 函数中，我们遵循了 error first 的原则，这意味着当我们处理回调函数的情况时：

```javascript
// 模拟数据请求
function fetchData(url) {
    return function(cb) {
        setTimeout(function() {
            cb(null, { status: 200, data: url })
        }, 1000)
    }
}
```

在成功时，第一个参数应该返回 null，表示没有错误原因。

#### 优化

我们在第二版的基础上将代码写的更加简洁优雅一点，最终的代码如下：

```javascript
// 第三版
function run(gen) {

    return new Promise(function(resolve, reject) {
        if (typeof gen == 'function') gen = gen();

        // 如果 gen 不是一个迭代器
        if (!gen || typeof gen.next !== 'function') return resolve(gen)

        onFulfilled();

        function onFulfilled(res) {
            var ret;
            try {
                ret = gen.next(res);
            } catch (e) {
                return reject(e);
            }
            next(ret);
        }

        function onRejected(err) {
            var ret;
            try {
                ret = gen.throw(err);
            } catch (e) {
                return reject(e);
            }
            next(ret);
        }

        function next(ret) {
            if (ret.done) return resolve(ret.value);
            var value = toPromise(ret.value);
            if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
            return onRejected(new TypeError('You may only yield a function, promise ' +
                'but the following object was passed: "' + String(ret.value) + '"'));
        }
    })
}

function isPromise(obj) {
    return 'function' == typeof obj.then;
}

function toPromise(obj) {
    if (isPromise(obj)) return obj;
    if ('function' == typeof obj) return thunkToPromise(obj);
    return obj;
}

function thunkToPromise(fn) {
    return new Promise(function(resolve, reject) {
        fn(function(err, res) {
            if (err) return reject(err);
            resolve(res);
        });
    });
}

module.exports = run;
```

#### co

如果我们再将这个启动器函数写的完善一些，我们就相当于写了一个 co，实际上，上面的代码确实是来自于 co……

而 co 是什么？ co 是大神 TJ Holowaychuk 于 2013 年 6 月发布的一个小模块，用于 Generator 函数的自动执行。

如果直接使用 co 模块，这两种不同的例子可以简写为：

```javascript
// yield 后是一个 Promise
var fetch = require('node-fetch');
var co = require('co');

function* gen() {
    var r1 = yield fetch('https://api.github.com/users/github');
    var json1 = yield r1.json();
    var r2 = yield fetch('https://api.github.com/users/github/followers');
    var json2 = yield r2.json();
    var r3 = yield fetch('https://api.github.com/users/github/repos');
    var json3 = yield r3.json();

    console.log([json1.bio, json2[0].login, json3[0].full_name].join('\n'));
}

co(gen);
// yield 后是一个回调函数
var co = require('co');

function fetchData(url) {
    return function(cb) {
        setTimeout(function() {
            cb(null, { status: 200, data: url })
        }, 1000)
    }
}

function* gen() {
    var r1 = yield fetchData('https://api.github.com/users/github');
    var r2 = yield fetchData('https://api.github.com/users/github/followers');

    console.log([r1.data, r2.data].join('\n'));
}

co(gen);
```

是不是特别的好用？