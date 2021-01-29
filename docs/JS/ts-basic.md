## 什么是TS

[官网介绍](https://www.tslang.cn/index.html)

## 基础类型

### 静态类型和动态类型

Javascript是`动态类型`的语言,也就是我们定义一个变量以后，可以赋值为`任何类型`的值.
而TS是一个基于`静态类型`的语言，每次定义变量的时候`最好指定为类型`，当然如果没有指定，TS会为我们推断出一个类型（`类型断言`）

### 布尔值

```javascript
    let isDone:boolean = true;
```

### 数字

> number类型除了支持10进制以外，还支持2进制、8进制等

```javascript
    let decLiteral: number = 6;
    let hexLiteral: number = 0xf00d;
    let binaryLiteral: number = 0b1010;
    let octalLiteral: number = 0o744;
```

### 字符串

> 除了字符串还支持`模板字符串`,并且是支持多行的

```javascript
    let name: string = 'foo';
    let age: number = 12;
    let sentence: string = `name is ${foo}
        age is ${age}
    `;
```

### 数组

```javascript
    // 第一种 类型+[]
    let arr: string[] = ['foo', 'join'];
    // 第二种 Array[T]
    let arr: Array<string> = ['foo', 'join];
```

### 元组（Tuple）
> 元组类型允许表示一个已知元素`数量`和`类型`的数组，各元素的类型不必相同

```javascript
    let tuple: [string, number] = ['hello', 12];
```
当访问`已知索引`的元素，会得到正确的类型

```javascript
    console.log(tuple[0].substr(1)); // OK
```
当访问`越界`的元素, 会使用`联合类型`替代

```javascript
    tuple[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型

    console.log(tuple[5].toString()); // OK, 'string' 和 'number' 都有 toString

    tuple[6] = true; // Error, 布尔不是(string | number)类型
```

```javascript
const teacherList: [string, string, number][] = [['foo', 'male', 19], ['sun', 'female', 26], ['jeny', 'female', 38]];

```

### 枚举

> 使用枚举类型可以为`一组数值`赋予友好的名字

```javascript
    enum Color {Red, Green, Blue}
    let c: Color = Color.Green;
```
默认从`0`开始，也可以手动指定。或者采用全部手动赋值。枚举值的`数字`和`字符串值`是互相指定的

```javascript
    enum Color {Red = 1, Green = 2, Blue = 4}
    let c: Color = Color.Green; // 2
    let name: string = Color[2] // Green
```

### Any

> 当我们不确定数据类型的时候，可以使用any，但是这样会丧失TS的代码提示和检查。慎用。

### Void

> 某种程度上来说，`void`类型像是与`any`类型相反，它表示没有任何类型.

```javascript
    function fn(): void {
        console.log('hello');
    }
```

声明一个`void`类型的变量没有什么大用，因为你只能为它赋予`undefined`和`null`

```javascript
    let unusable: void = undefined;
```

### Null 和 Undefined
> `undefined`和`null`两者各自有自己的类型分别叫做`undefined`和`null`

```javascript
    let u: undefined = undefined;
    let n: null = null;
```

默认情况下`null`和`undefined`是所有类型的`子类型`。 就是说你可以把 `null`和`undefined`赋值给`number`类型的变量.

当你指定了`--strictNullChecks`标记，`null`和`undefined`只能赋值给`void`和它们`各自`.

### Never

never类型表示的是那些永不存在的值的类型。 例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never类型，当它们被永不为真的类型保护所约束时。

never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 即使 any也不可以赋值给never。

```javascript
    // 返回never的函数必须存在无法达到的终点
    function error(message: string): never {
        throw new Error(message);
    }

    // 推断的返回值类型为never
    function fail() {
        return error("Something failed");
    }

    // 返回never的函数必须存在无法达到的终点
    function infiniteLoop(): never {
        while (true) {
        }
    }
```

### Object

> 非原始类型

```javascript
    declare function create(o: object | null): void;
    create({ prop: 0 }); // OK
    create(null); // OK
    create(42); // Error
```

### 类型断言

> 告诉TS你知道具体是什么类型。

```javascript
    let someValue: any = "this is a string";
    // 第一种
    let strLength: number = (someValue as string).length;
    // 第二种 这种方式在 JSX 里面是不被允许的
    let strLength: number = (<string>someValue).length;
```

