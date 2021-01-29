## 进阶类型

### 接口
> 接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

```javascript
    // 必须包含一个name属性，且类型为string
    interface name {
        name: string
    }
    function logName(obj: name) {
        console.log(obj.name)
    }
    logName({name: 'xx', age: 12})
```

`可选属性`
> 接口里的属性不全都是必需的。 有些是只在某些条件下存在，或者根本不存在，带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个`?`符号。

```javascript
    // age属性是可选的
    interface name {
        name: string
        age?: number
    }
    function logName(obj: name) {
        console.log(obj.name)
    }
    logName({name: 'xx', age: 12})
    logName({name: 'xx'})
```

`只读属性`
> 一些对象属性只能在对象刚刚创建的时候修改其值。 你可以在属性名前用 `readonly`来指定只读属性

```javascript
    interface Point {
        readonly x: number;
        readonly y: number;
    }

    let p1: Point = { x: 10, y: 20 };
    p1.x = 5; // error!
```

TypeScript具有`ReadonlyArray<T>`类型

```javascript
    let a: number[] = [1, 2, 3, 4];
    let ro: ReadonlyArray<number> = a;
    ro[0] = 12; // error!
    ro.push(5); // error!
    ro.length = 100; // error!
    a = ro; // error!
```

判断该用`readonly`还是`const`的方法是看要把它做为变量使用还是做为一个属性。 做为`变量`使用的话用 `const`，若做为`属性`则使用`readonly`。

`额外的属性检查`

表示的是`SquareConfig`可以有任意数量的属性，并且只要它们不是`color`和`width`，那么就无所谓它们的类型是什么

```javascript
    interface SquareConfig {
        color?: string;
        width?: number; 
        [propName: string]: any; // 额外的属性检查
    }
```

`函数类型`

```javascript
    interface SearchFunc {
        (source: string, subString: string): boolean;  // 定义参数和返回值
    }
```

```javascript
    let mySearch: SearchFunc;
    mySearch = function(source: string, subString: string) {
        let result = source.search(subString);
        return result > -1;
    }
```
函数的`参数名字`是`任意的`

```
    mySearch = function(src: string, sub: string): boolean {
        let result = src.search(sub);
        return result > -1;
    }
```

`可索引的类型`

```javascript
    interface StringArray {
        [index: number]: string; // 索引是数字类型，索引值是字符串类型
    }

    let myArray: StringArray;
    myArray = ["Bob", "Fred"];

    let myStr: string = myArray[0];
```

```javascript
    interface NumberDictionary {
        [index: string]: number;
        length: number;    // 可以，length是number类型
        name: string       // 错误，`name`的类型与索引类型返回值的类型不匹配
    }
```

`类类型`

```javascript
    interface ClockInterface {
        currentTime: Date; // 类中的属性
        setTime(d: Date); // 类中实现的方法
    }

    class Clock implements ClockInterface {
        currentTime: Date;
        setTime(d: Date) {
            this.currentTime = d;
        }
        constructor(h: number, m: number) { }
    }
```

`继承接口`

```javascript
    interface Shape {
        color: string;
    }
    interface PenStroke {
        penWidth: number;
    }
    interface Square extends Shape, PenStroke{ // 也可以继承多个
        sideLength: number;
    }

    let square = <Square>{}; // 类型断言

    square.color = "blue";
    square.sideLength = 10;
```

`混合类型`

> 一个对象可以同时做为函数和对象使用，并带有额外的属性

```javascript
    interface Counter { // 既是函数也是对象
        (start: number): string;
        interval: number;
        reset(): void;
    }

    function getCounter(): Counter {
        // 断言为Counter
        let counter = <Counter>function (start: number) { }; 
        counter.interval = 123;
        counter.reset = function () { };
        return counter;
    }

    let c = getCounter();
    c(10);
    c.reset();
    c.interval = 5.0;
```


