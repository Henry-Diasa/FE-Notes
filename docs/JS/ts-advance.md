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

### 类

```javascript
    class Greeter {
        greeting: string;
        constructor(message: string) {
            this.greeting = message;
        }
        greet() {
            return "Hello, " + this.greeting;
        }
    }

    let greeter = new Greeter("world");
```

`继承`

```javascript
    class Animal {
        name: string;
        constructor(theName: string) { this.name = theName; }
        move(distanceInMeters: number = 0) {
            console.log(`${this.name} moved ${distanceInMeters}m.`);
        }
    }

    class Snake extends Animal {
        // 使用super调用父类的构造函数
        constructor(name: string) { super(name); }
        move(distanceInMeters = 5) {
            // 重写父类的方法
            console.log("Slithering...");
            super.move(distanceInMeters);
        }
    }

    class Horse extends Animal {
        constructor(name: string) { super(name); }
        move(distanceInMeters = 45) {
            console.log("Galloping...");
            super.move(distanceInMeters);
        }
    }

    let sam = new Snake("Sammy the Python");
    let tom: Animal = new Horse("Tommy the Palomino");

    sam.move();
    tom.move(34);
``` 

`公共，私有与受保护的修饰符`

默认为`public`

> 声明为public的变量在`父类`、`子类`、`子类的实例`上都可以使用

```javascript
    class Animal {
        public name: string;
        public constructor(theName: string) { this.name = theName; }
        public move(distanceInMeters: number) {
            console.log(`${this.name} moved ${distanceInMeters}m.`);
        }
    }
```
`protected`
> 可以在`父类`、`子类`中使用。

```javascript
    class Person {
        protected name: string;
        constructor(name: string) { this.name = name; }
    }

    class Employee extends Person {
        private department: string;

        constructor(name: string, department: string) {
            super(name)
            this.department = department;
        }

        public getElevatorPitch() {
            return `Hello, my name is ${this.name} and I work in ${this.department}.`;
        }
    }

    let howard = new Employee("Howard", "Sales");
    console.log(howard.getElevatorPitch());
    console.log(howard.name); // 错误
```

`private`
> 当成员被标记成 `private`时，它就`不能`在声明它的类的`外部访问`.也就是只允许`当前类`使用

```javascript
    class Animal {
        private name: string;
        constructor(theName: string) { this.name = theName; }
    }

    new Animal("Cat").name; // 错误: 'name' 是私有的.
```

`readonly修饰符`
> `readonly`关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。

```javascript
    class Octopus {
        readonly name: string;
        readonly numberOfLegs: number = 8;
        constructor (theName: string) {
            this.name = theName;
        }
    }
    let dad = new Octopus("Man with the 8 strong legs");
    dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.
```

`参数属性`
> 参数属性可以方便地让我们在一个地方`定义`并`初始化`一个成员

```javascript
    class Octopus {
        readonly numberOfLegs: number = 8;
        constructor(readonly name: string) { // 和上面的写法一致 也可以使用public、private等
            // this.name = name;
        }
    }
```

`存取器`
> get、set 可以在设置值或者获取值的时候进行一些拦截操作
```javascript
    let passcode = "secret passcode";

    class Employee {
        private _fullName: string;

        get fullName(): string {
            return this._fullName;
        }

        set fullName(newName: string) {
            if (passcode && passcode == "secret passcode") {
                this._fullName = newName;
            }
            else {
                console.log("Error: Unauthorized update of employee!");
            }
        }
    }

    let employee = new Employee();
    employee.fullName = "Bob Smith";
    if (employee.fullName) {
        alert(employee.fullName);
    }
```

`静态属性`
> static 定义的只能通过`类本身`来进行调用

```javascript
    class Grid {
        static origin = {x: 0, y: 0};
        calculateDistanceFromOrigin(point: {x: number; y: number;}) {
            let xDist = (point.x - Grid.origin.x);
            let yDist = (point.y - Grid.origin.y);
            return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
        }
        constructor (public scale: number) { }
    }

    let grid1 = new Grid(1.0);  // 1x scale
    let grid2 = new Grid(5.0);  // 5x scale

    console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
    console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

`抽象类`
> abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法。

抽象类的方法需要在子类中进行实现

```javascript
    abstract class Department {

        constructor(public name: string) {
        }

        printName(): void {
            console.log('Department name: ' + this.name);
        }

        abstract printMeeting(): void; // 必须在派生类中实现
    }

    class AccountingDepartment extends Department {

        constructor() {
            super('Accounting and Auditing'); // 在派生类的构造函数中必须调用 super()
        }

        printMeeting(): void {
            console.log('The Accounting Department meets each Monday at 10am.');
        }

        generateReports(): void {
            console.log('Generating accounting reports...');
        }
    }

    let department: Department; // 允许创建一个对抽象类型的引用
    department = new Department(); // 错误: 不能创建一个抽象类的实例
    department = new AccountingDepartment(); // 允许对一个抽象子类进行实例化和赋值
    department.printName();
    department.printMeeting();
    department.generateReports(); // 错误: 方法在声明的抽象类中不存在
```

`把类当做接口使用`
> 类定义会创建两个东西：类的实例类型和一个构造函数 (typeof)

```javascript
   class Point {
        x: number;
        y: number;
    }

    interface Point3d extends Point {
        z: number;
    }

    let point3d: Point3d = {x: 1, y: 2, z: 3}; 
```

### 函数

`函数类型`

为函数定义类型
> 定义`参数类型`和`返回值`类型
```javascript
    function add(x: number, y: number): number {
        return x + y;
    }

    let myAdd = function(x: number, y: number): number { return x + y; };

    let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

`可选参数和默认参数`

```javascript
    // 可选
    function buildName(firstName: string, lastName?: string) {
        // ...
    }
    // 默认 默认参数只有传递undefined的时候才会生效
    function buildName(firstName: string, lastName = "Smith") {
        // ...
    }
```

`剩余参数`

```javascript
    function buildName(firstName: string, ...restOfName: string[]) {
        return firstName + " " + restOfName.join(" ");
    }

    let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

`this参数`
> 可能不是很常用，需要的时候去官网查看就可以
```javascript
    interface Card {
        suit: string;
        card: number;
    }
    interface Deck {
        suits: string[];
        cards: number[];
        createCardPicker(this: Deck): () => Card;
    }
    let deck: Deck = {
        suits: ["hearts", "spades", "clubs", "diamonds"],
        cards: Array(52),
        // NOTE: The function now explicitly specifies that its callee must be of type Deck
        createCardPicker: function(this: Deck) {
            return () => {
                let pickedCard = Math.floor(Math.random() * 52);
                let pickedSuit = Math.floor(pickedCard / 13);

                return {suit: this.suits[pickedSuit], card: pickedCard % 13};
            }
        }
    }

    let cardPicker = deck.createCardPicker();
    let pickedCard = cardPicker();

    alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

`重载`

> 为了让编译器能够选择正确的检查类型，它与JavaScript里的处理流程相似。 它查找重载列表，尝试使用第一个重载定义。 如果匹配的话就使用这个。 因此，在定义重载的时候，一定要把最精确的定义放在最前面

```javascript
    let suits = ["hearts", "spades", "clubs", "diamonds"];

    function pickCard(x: {suit: string; card: number; }[]): number;
    function pickCard(x: number): {suit: string; card: number; };
    function pickCard(x): any {
        // Check to see if we're working with an object/array
        // if so, they gave us the deck and we'll pick the card
        if (typeof x == "object") {
            let pickedCard = Math.floor(Math.random() * x.length);
            return pickedCard;
        }
        // Otherwise just let them pick the card
        else if (typeof x == "number") {
            let pickedSuit = Math.floor(x / 13);
            return { suit: suits[pickedSuit], card: x % 13 };
        }
    }

    let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
    let pickedCard1 = myDeck[pickCard(myDeck)];
    alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

    let pickedCard2 = pickCard(15);
    alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

### 泛型

> 使用`泛型`来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件

```javascript
    // 定义函数传递的参数和返回值类型一致
    function identity<T>(arg: T): T {
        return arg;
    }

    // 泛型的使用

    // 第一种

    let output = identity<string>("myString");  // type of output will be 'string'

    // 第二种 利用类型推论

    let output = identity("myString");  // type of output will be 'string'

```

`使用泛型变量`

```javascript
    function loggingIdentity<T>(arg: Array<T>): Array<T> {
        // 如果arg不声明为数组是不可以直接调用.length的，因为T的类型是任意的
        console.log(arg.length);  // Array has a .length, so no more error
        return arg;
    }
```

`泛型类型`

```javascript
   function identity<T>(arg: T): T {
        return arg;
    }

    let myIdentity: <T>(arg: T) => T = identity; 

    // 或者
    interface GenericIdentityFn {
        <T>(arg: T): T;
    }

    function identity<T>(arg: T): T {
        return arg;
    }

    let myIdentity: GenericIdentityFn = identity;

    // 接口定义泛型

    interface GenericIdentityFn<T> {
        (arg: T): T;
    }

    function identity<T>(arg: T): T {
        return arg;
    }

    let myIdentity: GenericIdentityFn<number> = identity;

```

`泛型类`

```javascript
    class GenericNumber<T> {
        zeroValue: T;
        add: (x: T, y: T) => T;
    }

    let myGenericNumber = new GenericNumber<number>();
    myGenericNumber.zeroValue = 0;
    myGenericNumber.add = function(x, y) { return x + y; };
```

`泛型约束`

```javascript

    interface Lengthwise {
        length: number;
    }

    function loggingIdentity<T extends Lengthwise>(arg: T): T {
        console.log(arg.length);  // Now we know it has a .length property, so no more error
        return arg;
    }
```

`在泛型里使用类类型`

```javascript
    class BeeKeeper {
        hasMask: boolean;
    }

    class ZooKeeper {
        nametag: string;
    }

    class Animal {
        numLegs: number;
    }

    class Bee extends Animal {
        keeper: BeeKeeper;
    }

    class Lion extends Animal {
        keeper: ZooKeeper;
    }

    function createInstance<A extends Animal>(c: new () => A): A {
        // 传递的参数c实例化的时候返回A类型
        return new c();
    }

    createInstance(Lion).keeper.nametag;  // typechecks!
    createInstance(Bee).keeper.hasMask;   // typechecks!
```

### 枚举

`数字枚举`

> 如果Up不指定值的时候，将默认为0，下面也是依次递增
```javascript
    enum Direction {
        Up = 1,
        Down,  // 2
        Left,  // 3
        Right  // 4  自动增长
    }

    // 使用
    enum Response {
        No = 0,
        Yes = 1,
    }

    function respond(recipient: string, message: Response): void {
        // ...
    }

    respond("Princess Caroline", Response.Yes)
```

`字符串枚举`

```javascript
    enum Direction {
        Up = "UP",
        Down = "DOWN",
        Left = "LEFT",
        Right = "RIGHT",
    }
```

`异构枚举（Heterogeneous enums）`

```javascript
    enum BooleanLikeHeterogeneousEnum {
        No = 0,
        Yes = "YES",
    }
```

`联合枚举与枚举成员的类型`

```javascript
    enum ShapeKind {
        Circle,
        Square,
    }

    interface Circle {
        kind: ShapeKind.Circle;
        radius: number;
    }

    interface Square {
        kind: ShapeKind.Square;
        sideLength: number;
    }

    let c: Circle = {
        kind: ShapeKind.Square, // 必须是ShapeKind.Circle
        //    ~~~~~~~~~~~~~~~~ Error!
        radius: 100,
    }

```

`运行时的枚举`

```javascript
    enum E {
        X, Y, Z
    }

    function f(obj: { X: number }) {
        return obj.X;
    }

    // Works, since 'E' has a property named 'X' which is a number.
    f(E);
```

`反向映射`

```javascript
    enum Enum {
        A
    }
    let a = Enum.A;
    let nameOfA = Enum[a]; // "A"
```

上面的代码编译为JS
> 注：不会为字符串枚举成员生成反向映射。

```javascript
    var Enum;
    (function (Enum) {
        Enum[Enum["A"] = 0] = "A";
    })(Enum || (Enum = {}));
    var a = Enum.A;
    var nameOfA = Enum[a]; // "A"
```

`const枚举`

```javascript
    const enum Directions {
        Up,
        Down,
        Left,
        Right
    }

    let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```








