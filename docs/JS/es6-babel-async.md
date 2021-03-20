#### 前言

本文就是简单介绍下 Async 语法编译后的代码。

#### Async

```javascript
const fetchData = (data) => new Promise((resolve) => setTimeout(resolve, 1000, data + 1))

const fetchValue = async function () {
    var value1 = await fetchData(1);
    var value2 = await fetchData(value1);
    var value3 = await fetchData(value2);
    console.log(value3)
};

fetchValue();
// 大约 3s 后输出 4
```

#### Babel

我们直接在 Babel 官网的 [Try it out](https://babeljs.io/repl) 粘贴上述代码，然后查看代码编译成什么样子：

```javascript
"use strict";

function _asyncToGenerator(fn) {
  return function() {
    var gen = fn.apply(this, arguments);
    return new Promise(function(resolve, reject) {
      function step(key, arg) {
        try {
          var info = gen[key](arg);
          var value = info.value;
        } catch (error) {
          reject(error);
          return;
        }
        if (info.done) {
          resolve(value);
        } else {
          return Promise.resolve(value).then(
            function(value) {
              step("next", value);
            },
            function(err) {
              step("throw", err);
            }
          );
        }
      }
      return step("next");
    });
  };
}

var fetchData = function fetchData(data) {
  return new Promise(function(resolve) {
    return setTimeout(resolve, 1000, data + 1);
  });
};

var fetchValue = (function() {
  var _ref = _asyncToGenerator(
    /*#__PURE__*/ regeneratorRuntime.mark(function _callee() {
      var value1, value2, value3;
      return regeneratorRuntime.wrap(
        function _callee$(_context) {
          while (1) {
            switch ((_context.prev = _context.next)) {
              case 0:
                _context.next = 2;
                return fetchData(1);

              case 2:
                value1 = _context.sent;
                _context.next = 5;
                return fetchData(value1);

              case 5:
                value2 = _context.sent;
                _context.next = 8;
                return fetchData(value2);

              case 8:
                value3 = _context.sent;

                console.log(value3);

              case 10:
              case "end":
                return _context.stop();
            }
          }
        },
        _callee,
        this
      );
    })
  );

  return function fetchValue() {
    return _ref.apply(this, arguments);
  };
})();

fetchValue();
```

#### _asyncToGenerator

regeneratorRuntime 相关的代码我们在 《ES6 系列之 Babel 将 Generator 编译成了什么样子》 中已经介绍过了，这次我们重点来看看 _asyncToGenerator 函数：

```
function _asyncToGenerator(fn) {
  return function() {
    var gen = fn.apply(this, arguments);
    return new Promise(function(resolve, reject) {
      function step(key, arg) {
        try {
          var info = gen[key](arg);
          var value = info.value;
        } catch (error) {
          reject(error);
          return;
        }
        if (info.done) {
          resolve(value);
        } else {
          return Promise.resolve(value).then(
            function(value) {
              step("next", value);
            },
            function(err) {
              step("throw", err);
            }
          );
        }
      }
      return step("next");
    });
  };
}
```

以上这段代码主要是用来实现 generator 的自动执行以及返回 Promise。

当我们执行 `fetchValue()` 的时候，执行的其实就是 `_asyncToGenerator` 返回的这个匿名函数，在匿名函数中，我们执行了

```
var gen = fn.apply(this, arguments);
```

这一步就相当于执行 Generator 函数，举个例子：

```
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
```

`var gen = fn.apply(this, arguments)` 就相当于 `var hw = helloWorldGenerator();`，返回的 gen 是一个具有 next()、throw()、return() 方法的对象。

然后我们返回了一个 Promise 对象，在 Promise 中，我们执行了 step("next")，step 函数中会执行：

```
try {
  var info = gen[key](arg);
  var value = info.value;
} catch (error) {
  reject(error);
  return;
}
```

step("next") 就相当于 `var info = gen.next()`，返回的 info 对象是一个具有 value 和 done 属性的对象：

```javascript
{value: Promise, done: false}
```

接下来又会执行：

```
if (info.done) {
  resolve(value);
} else {
  return Promise.resolve(value).then(
    function(value) {
      step("next", value);
    },
    function(err) {
      step("throw", err);
    }
  );
}
```

value 此时是一个 Promise，Promise.resolve(value) 依然会返回这个 Promise，我们给这个 Promise 添加了一个 then 函数，用于在 Promise 有结果时执行，有结果时又会执行 `step("next", value)`，从而使得 Generator 继续执行，直到 `info.done` 为 true，才会 `resolve(value)`。

#### 不完整但可用的代码

```javascript
(function() {
    var ContinueSentinel = {};

    var mark = function(genFun) {
        var generator = Object.create({
            next: function(arg) {
                return this._invoke("next", arg);
            }
        });
        genFun.prototype = generator;
        return genFun;
    };

    function wrap(innerFn, outerFn, self) {
        var generator = Object.create(outerFn.prototype);

        var context = {
            done: false,
            method: "next",
            next: 0,
            prev: 0,
            sent: undefined,
            abrupt: function(type, arg) {
                var record = {};
                record.type = type;
                record.arg = arg;

                return this.complete(record);
            },
            complete: function(record, afterLoc) {
                if (record.type === "return") {
                    this.rval = this.arg = record.arg;
                    this.method = "return";
                    this.next = "end";
                }

                return ContinueSentinel;
            },
            stop: function() {
                this.done = true;
                return this.rval;
            }
        };

        generator._invoke = makeInvokeMethod(innerFn, context);

        return generator;
    }

    function makeInvokeMethod(innerFn, context) {
        var state = "start";

        return function invoke(method, arg) {
            if (state === "completed") {
                return { value: undefined, done: true };
            }

            context.method = method;
            context.arg = arg;

            while (true) {
                state = "executing";

                if (context.method === "next") {
                    context.sent = context._sent = context.arg;
                }

                var record = {
                    type: "normal",
                    arg: innerFn.call(self, context)
                };

                if (record.type === "normal") {
                    state = context.done ? "completed" : "yield";

                    if (record.arg === ContinueSentinel) {
                        continue;
                    }

                    return {
                        value: record.arg,
                        done: context.done
                    };
                }
            }
        };
    }

    window.regeneratorRuntime = {};

    regeneratorRuntime.wrap = wrap;
    regeneratorRuntime.mark = mark;
})();

"use strict";

function _asyncToGenerator(fn) {
    return function() {
        var gen = fn.apply(this, arguments);
        return new Promise(function(resolve, reject) {
            function step(key, arg) {
                try {
                    var info = gen[key](arg);
                    var value = info.value;
                } catch (error) {
                    reject(error);
                    return;
                }
                if (info.done) {
                    resolve(value);
                } else {
                    return Promise.resolve(value).then(
                        function(value) {
                            step("next", value);
                        },
                        function(err) {
                            step("throw", err);
                        }
                    );
                }
            }
            return step("next");
        });
    };
}

var fetchData = function fetchData(data) {
    return new Promise(function(resolve) {
        return setTimeout(resolve, 1000, data + 1);
    });
};

var fetchValue = (function() {
    var _ref = _asyncToGenerator(
        /*#__PURE__*/
        regeneratorRuntime.mark(function _callee() {
            var value1, value2, value3;
            return regeneratorRuntime.wrap(
                function _callee$(_context) {
                    while (1) {
                        switch ((_context.prev = _context.next)) {
                            case 0:
                                _context.next = 2;
                                return fetchData(1);

                            case 2:
                                value1 = _context.sent;
                                _context.next = 5;
                                return fetchData(value1);

                            case 5:
                                value2 = _context.sent;
                                _context.next = 8;
                                return fetchData(value2);

                            case 8:
                                value3 = _context.sent;

                                console.log(value3);

                            case 10:
                            case "end":
                                return _context.stop();
                        }
                    }
                },
                _callee,
                this
            );
        })
    );

    return function fetchValue() {
        return _ref.apply(this, arguments);
    };
})();

fetchValue();
```

