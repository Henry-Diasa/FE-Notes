#### 前言

本篇文章是私有变量的实现

#### 1.约定

**实现**

```javascript
class Example {
	constructor() {
		this._private = 'private';
	}
	getName() {
		return this._private
	}
}

var ex = new Example();

console.log(ex.getName()); // private
```

**优点**

1. 写法简单
2. 调试方便
3. 兼容性好

**缺点**

1. 外部可以访问和修改
2. 语言没有配合的机制，如 for in 语句会将所有属性枚举出来
3. 命名冲突

#### 2.闭包

**实现一**

```
/**
 * 实现一
 */
class Example {
  constructor() {
    var _private = '';
    _private = 'private';
    this.getName = function() {return _private}
  }
}

var ex = new Example();

console.log(ex.getName()); // private
console.log(ex._private); // undefined
```

**优点**

1. 无命名冲突
2. 外部无法访问和修改

**缺点**

1. constructor 的逻辑变得复杂。构造函数应该只做对象初始化的事情，现在为了实现私有变量，必须包含部分方法的实现，代码组织上略不清晰。
2. 方法存在于实例，而非原型上，子类也无法使用 super 调用
3. 构建增加一点点开销

**实现二**

```
/**
 * 实现二
 */
const Example = (function() {
  var _private = '';

  class Example {
    constructor() {
      _private = 'private';
    }
    getName() {
      return _private;
    }
  }

  return Example;

})();

var ex = new Example();

console.log(ex.getName()); // private
console.log(ex._private); // undefined
```

**优点**

1. 无命名冲突
2. 外部无法访问和修改

**缺点**

1. 写法有一点复杂
2. 构建增加一点点开销

#### 3.Symbol

**实现**

```
const Example = (function() {
    var _private = Symbol('private');

    class Example {
        constructor() {
          this[_private] = 'private';
        }
        getName() {
          return this[_private];
        }
    }

    return Example;
})();

var ex = new Example();

console.log(ex.getName()); // private
console.log(ex.name); // undefined
```

**优点**

1. 无命名冲突
2. 外部无法访问和修改
3. 无性能损失

**缺点**

1. 写法稍微复杂
2. 兼容性也还好

#### 4.WeakMap

**实现**

```
/**
 * 实现一
 */
const _private = new WeakMap();

class Example {
  constructor() {
    _private.set(this, 'private');
  }
  getName() {
  	return _private.get(this);
  }
}

var ex = new Example();

console.log(ex.getName()); // private
console.log(ex.name); // undefined
```

如果这样写，你可能觉得封装性不够，你也可以这样写：

```
/**
 * 实现二
 */
const Example = (function() {
  var _private = new WeakMap(); // 私有成员存储容器

  class Example {
    constructor() {
      _private.set(this, 'private');
    }
    getName() {
    	return _private.get(this);
    }
  }

  return Example;
})();

var ex = new Example();

console.log(ex.getName()); // private
console.log(ex.name); // undefined
```

**优点**

1. 无命名冲突
2. 外部无法访问和修改

**缺点**

1. 写法比较麻烦
2. 兼容性有点问题
3. 有一定性能代价

#### 5.最新提案

```
class Point {
  #x;
  #y;

  constructor(x, y) {
    this.#x = x;
    this.#y = y;
  }

  equals(point) {
    return this.#x === point.#x && this.#y === point.#y;
  }
}
```

那么为什么不直接使用 private 字段呢？比如说这样：

```
class Foo {
  private value;

  equals(foo) {
    return this.value === foo.value;
  }
}
```

简单点来说，就是嫌麻烦，当然也有性能上的考虑……

举个例子，如果我们不使用 #，而是使用 private 关键字：

```
class Foo {
  private value = '1';

  equals(foo) {
    return this.value === foo.value;
  }
}

var foo1 = new Foo();
var foo2 = new Foo();

console.log(foo1.equals(foo2));
```

在这里我们新建了两个实例，然后将 foo2 作为参数传入了 foo1 的实例方法中。

那么我们可以获取 foo2.value 的值吗？如果我们直接 `foo2.value` 肯定是获取不到值的，毕竟是私有变量，可是 equals 是 Foo 的一个类方法，那么可以获取到的吗？

答案是可以的。

其实这点在其他语言，比如说 Java 和 C++ 中也是一样的，**类的成员函数中可以访问同类型实例的私有变量**，这是因为私有是为了实现“对外”的信息隐藏，在类自己内部，没有必要禁止私有变量的访问，你也可以理解为私有变量的限制是以类为单位，而不是以对象为单位，此外这样做也可以为使用者带来便利。

既然获取值是可以的，那么打印的结果应该为 true，但是如果我们传入的值不是 Foo 的实例，而是一个其他对象呢？

```
var foo1 = new Foo();

console.log(foo1.equals({
  value: 2
}));
```

当然这里代码也是可以正常运行的，但是对于编译器来说，就有一点麻烦了，因为编译器不知道 value 到底是 foo 的正常属性还是私有属性，所以编译器需要做判断，先判断 foo 是不是 Foo 的实例，然后再接着获取值。

这也意味着每次属性访问都需要做这样一个判断，而引擎已经围绕属性访问做了高度优化，懒得改，而且还降低速度。

不过除了这个工作之外，还会有一些其他的内容需要考虑，比如说：

1. 你必须将私有的 key 编码进每个词法环境
2. for in 可以遍历这些属性吗？
3. 私有属性和正常属性同名的时候，谁会屏蔽谁？
4. 怎么防止私有属性的名称不被探测出来。

关于使用 # 而不使用 private 更多的讨论可以参考这个 [Issue](https://github.com/tc39/proposal-private-fields/issues/14)。

当然这些问题都可以被解决啦，就是麻烦了点。

而如果你选择 #，实现的方式将跟 JavaScript 对象属性完全没有关系，将会使用 `private slots` 的方式以及使用一个新的 slot 查找语法，总之就是会比 private 的实现方式简单很多。



#### 参考

1. [《编程语言如何演化——以JS的private为例》贺师俊](https://v.qq.com/x/page/c0529qeku63.html)
2. [Exploring ES6](http://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes)
3. [译 JS 新语法：私有属性](https://zhuanlan.zhihu.com/p/27360252)

