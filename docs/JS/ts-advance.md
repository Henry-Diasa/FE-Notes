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
注意：
```javascript
    "strictPropertyInitialization": true / 启用类属性初始化的严格检查/
    name!:string  // 类里面的属性需要加! 来避免初始化未赋值的报错
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
- 常数枚举与普通枚举的区别是，它会在编译阶段被删除，并且不能包含计算成员。
- 假如包含了计算成员，则会在编译阶段报错

```javascript
    const enum Directions {
        Up,
        Down,
        Left,
        Right,
        xx = "xx".length,  // 报错
    }

    let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

### 高级类型

#### 交叉类型（Intersection Types）
> 交叉类型是将`多个类型`合并为`一个类型`。 这让我们可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的`所有类型的特性`

```javascript
    function extend<T, U>(first: T, second: U): T & U {
         // 断言
        let result = <T & U>{};
        for (let id in first) {
            (<any>result)[id] = (<any>first)[id];
        }
        for (let id in second) {
            if (!result.hasOwnProperty(id)) {
                (<any>result)[id] = (<any>second)[id];
            }
        }
        return result;
    }

    class Person {
        constructor(public name: string) { }
    }
    interface Loggable {
        log(): void;
    }
    class ConsoleLogger implements Loggable {
        log() {
            // ...
        }
    }
    var jim = extend(new Person("Jim"), new ConsoleLogger());
    var n = jim.name;
    jim.log();
```

#### 联合类型（Union Types）

> 联合类型与交叉类型很有关联，但是使用上却完全不同。 偶尔你会遇到这种情况，一个代码库希望传入 `number`或 `string`类型的参数

```javascript

// 这样是存在问题的，因为padding传递一个不是string和number类型的时候，编译阶段并不会报错。执行的时候会抛出异常
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

// 联合类型转换后

function padLeft(value: string, padding: string | number) {
    // ...
}
padLeft("Hello world", 4); // returns "    Hello world"
```

如果一个值是`联合类型`，我们只能访问此联合类型的所有类型里`共有的`成员。

```javascript
    interface Bird {
        fly();
        layEggs();
    }

    interface Fish {
        swim();
        layEggs();
    }

    function getSmallPet(): Fish | Bird {
        // ...
    }

    let pet = getSmallPet();
    pet.layEggs(); // okay
    pet.swim();    // errors
```

#### 类型保护与区分类型（Type Guards and Differentiating Types）

```javascript
    let pet = getSmallPet();

    // 每一个成员访问都会报错
    if (pet.swim) {
        pet.swim();
    }
    else if (pet.fly) {
        pet.fly();
    }
```

类型断言

```javascript
    let pet = getSmallPet();

    if ((<Fish>pet).swim) {
        (<Fish>pet).swim();
    }
    else {
        (<Bird>pet).fly();
    }
```

`用户自定义的类型保护`

- 类型谓词

```javascript
    // pet is Fish 就是类型谓词
    function isFish(pet: Fish | Bird): pet is Fish {
        return (<Fish>pet).swim !== undefined;
    }

    // 使用 'swim' 和 'fly' 调用都没有问题了

    if (isFish(pet)) {
        pet.swim();
    }
    else {
        pet.fly();
    }

```

- `typeof` 类型保护

```javascript
    function padLeft(value: string, padding: string | number) {
        if (typeof padding === "number") {
            return Array(padding + 1).join(" ") + value;
        }
        if (typeof padding === "string") {
            return padding + value;
        }
        throw new Error(`Expected string or number, got '${padding}'.`);
    }
```

- `instanceof` 类型保护

```javascript
    interface Padder {
        getPaddingString(): string
    }

    class SpaceRepeatingPadder implements Padder {
        constructor(private numSpaces: number) { }
        getPaddingString() {
            return Array(this.numSpaces + 1).join(" ");
        }
    }

    class StringPadder implements Padder {
        constructor(private value: string) { }
        getPaddingString() {
            return this.value;
        }
    }

    function getRandomPadder() {
        return Math.random() < 0.5 ?
            new SpaceRepeatingPadder(4) :
            new StringPadder("  ");
    }

    // 类型为SpaceRepeatingPadder | StringPadder
    let padder: Padder = getRandomPadder();

    if (padder instanceof SpaceRepeatingPadder) {
        padder; // 类型细化为'SpaceRepeatingPadder'
    }
    if (padder instanceof StringPadder) {
        padder; // 类型细化为'StringPadder'
    }
```

- in
> in 操作符可以安全的检查一个对象上是否存在一个属性，它通常也被作为类型保护使用：
```javascript
    interface A {
        x: number;
    }

    interface B {
        y: string;
    }

    function doStuff(q: A | B) {
        if ('x' in q) {
            // q: A
        } else {
            // q: B
        }
    }
```
- null保护
> 如果开启了strictNullChecks选项，那么对于可能为null的变量不能调用它上面的方法和属性

```javascript
    function getFirstLetter(s: string | null) {
        //第一种方式是加上null判断
        if (s == null) {
            return '';
        }
        //第二种处理是增加一个或的处理
        s = s || '';
        return s.charAt(0);
    }
    //它并不能处理一些复杂的判断，需要加非空断言操作符
    function getFirstLetter2(s: string | null) {
        function log() {
            console.log(s!.trim());
        }
        s = s || '';
        log();
        return s.charAt(0);
    }
```

- 链判断运算符
> 链判断运算符是一种先检查属性是否存在，再尝试访问该属性的运算符，其符号为 `?. `。如果运算符左侧的操作数 ?. 计算为 undefined 或 null，则表达式求值为 undefined 。否则，正常触发目标属性访问，方法或函数调用。

> 链判断运算符 还处于 stage1 阶段,TS 也暂时不支持

```javascript
    a?.b; //如果a是null/undefined,那么返回undefined，否则返回a.b的值.
    a == null ? undefined : a.b;

    a?.[x]; //如果a是null/undefined,那么返回undefined，否则返回a[x]的值
    a == null ? undefined : a[x];

    a?.b(); // 如果a是null/undefined,那么返回undefined
    a == null ? undefined : a.b(); //如果a.b不函数的话抛类型错误异常,否则计算a.b()的结果

    a?.(); //如果a是null/undefined,那么返回undefined
    a == null ? undefined : a(); //如果A不是函数会抛出类型错误
    //否则 调用a这个函数
```

- 可辨识的联合类型
> 就是利用联合类型中的共有字段进行类型保护的一种技巧, 相同字段的不同取值就是可辨识

```javascript
    interface User {
        username: string
    }
    type Action = {
        type:'add',
        payload:User
    } | {
        type: 'delete'
        payload: number
    }
    const UserReducer = (action: Action) => {
    switch (action.type) {
        case "add":
        let user: User = action.payload;
        break;
        case "delete":
        let id: number = action.payload;
        break;
        default:
        break;
    }
    };
```

- unknown
    - TypeScript 3.0 引入了新的unknown 类型，它是 any 类型对应的安全类型
    - unknown 和 any 的主要区别是 unknown 类型会更加严格：在对 unknown 类型的值执行大多数操作之前，我们必须进行某种形式的检查。而在对 any 类型的值执行操作之前，我们不必进行任何检查
    - 就像所有类型都可以被归为 any，所有类型也都可以被归为 unknown。这使得 unknown 成为 TypeScript 类型系统的另一种顶级类型（另一种是 any）
    - 任何类型都可以赋值给unknown类型

    ```javascript
        let value: unknown;

        value = true;             // OK
        value = 42;               // OK
        value = "Hello World";    // OK
        value = [];               // OK
        value = {};               // OK
        value = Math.random;      // OK
        value = null;             // OK
        value = undefined;        // OK
        value = new TypeError();  // OK
    ```
    - unknown类型只能被赋值给any类型和unknown类型本身

    ```javascript
        let value: unknown;
        let value1: unknown = value;   // OK
        let value2: any = value;       // OK
        let value3: boolean = value;   // Error
        let value4: number = value;    // Error
        let value5: string = value;    // Error
        let value6: object = value;    // Error
        let value7: any[] = value;     // Error
        let value8: Function = value;  // Error
    ```
    - 缩小 unknown 类型范围 (如果没有类型断言或类型细化时，不能在unknown上面进行任何操作, 可以对 unknown 类型使用类型断言)

    ```javascript
        const value: unknown = "Hello World";
        const someString: string = value as string;
    ```
    - 联合类型中的 unknown 类型 
    ```javascript
        // 在联合类型中，unknown 类型会吸收任何类型。这就意味着如果任一组成类型是 unknown，联合类型也会相当于 unknown：
        type UnionType1 = unknown | null;       // unknown
        type UnionType2 = unknown | undefined;  // unknown
        type UnionType3 = unknown | string;     // unknown
        type UnionType4 = unknown | number[];   // unknown
    ```
    - 交叉类型中的 unknown 类型

    ```javascript
        // 在交叉类型中，任何类型都可以吸收 unknown 类型。这意味着将任何类型与 unknown 相交不会改变结果类型
        type IntersectionType1 = unknown & null;       // null
        type IntersectionType2 = unknown & undefined;  // undefined
        type IntersectionType3 = unknown & string;     // string
        type IntersectionType4 = unknown & number[];   // number[]
        type IntersectionType5 = unknown & any;        // any
    ```
    - never是unknown的子类型
    ```javascript
        type isNever = never extends unknown ? true : false;
    ```
    - keyof unknown 等于never

    ```javascript
        type key = keyof unknown;
    ```
    - 只能对unknown进行等或不等操作，不能进行其它操作

    ```javascript
        un1===un2;
        un1!==un2;
        un1 += un2;

        un.name  // 不能访问属性
        un();  // 不能作为函数调用
        new un();  // 不能当作类的构造函数不能创建实例
    ```

    - 映射属性

    ```javascript
        // 如果映射类型遍历的时候是unknown,不会映射属性
        type getType<T> = {
        [P in keyof T]:number
        }
        type t = getType<unknown>;
    ```


#### 可以为null的类型

> 当使用`--strictNullChecks`标记的时候，声明一个变量，它不会自动的包含`null`或者`undefined`。需要使用联合类型明确的包含他们

```javascript
    let s = "foo";
    s = null; // 错误, 'null'不能赋值给'string'
    let sn: string | null = "bar";
    sn = null; // 可以

    sn = undefined; // error, 'undefined'不能赋值给'string | null'
```

`可选参数和可选属性`
> 使用了 `--strictNullChecks`，可选参数和可选属性会被自动地加上 `| undefined`:

```javascript
    function f(x: number, y?: number) {
        return x + (y || 0);
    }
    f(1, 2);
    f(1);
    f(1, undefined);
    f(1, null); // error, 'null' is not assignable to 'number | undefined'
```

```javascript
    class C {
        a: number;
        b?: number;
    }
    let c = new C();
    c.a = 12;
    c.a = undefined; // error, 'undefined' is not assignable to 'number'
    c.b = 13;
    c.b = undefined; // ok
    c.b = null; // error, 'null' is not assignable to 'number | undefined'
```

`类型保护和类型断言`
> 由于可以为null的类型是通过联合类型实现，那么你需要使用类型保护来去除 null

```javascript
    function f(sn: string | null): string {
        if (sn == null) {
            return "default";
        }
        else {
            return sn;
        }
        // 或者通过短路运算符
        return sn || "default";
    }
```

`! 断言`

```javascript
   function broken(name: string | null): string {
    function postfix(epithet: string) {
        return name.charAt(0) + '.  the ' + epithet; // error, 'name' is possibly null
    }
    name = name || "Bob";
    return postfix("great");
    }

    function fixed(name: string | null): string {
    function postfix(epithet: string) {
        // 这里使用 ！来断言name是一定存在的
        return name!.charAt(0) + '.  the ' + epithet; // ok
    }
    name = name || "Bob";
    return postfix("great");
    } 
```

#### 类型别名

> 类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

```javascript
    type Name = string;
    type NameResolver = () => string;
    type NameOrResolver = Name | NameResolver;
    function getName(n: NameOrResolver): Name {
        if (typeof n === 'string') {
            return n;
        }
        else {
            return n();
        }
    }
```
类型别名也可以是泛型

```javascript
    type Tree<T> = {
        value: T;
        left: Tree<T>;
        right: Tree<T>;
    }
```

类型别名不能出现在声明右侧的任何地方

```javascript
    type Yikes = Array<Yikes>; // error
```

`接口 vs 类型别名`

- 接口创建了一个`新的名字`，可以在其它任何地方使用。 类型别名并`不创建新名字`—比如，错误信息就不会使用别名

```javascript
    // 在编译器中将鼠标悬停在 interfaced上，显示它返回的是 Interface，但悬停在 aliased上时，显示的却是对象字面量类型。
    type Alias = { num: number }
    interface Interface {
        num: number;
    }
    declare function aliased(arg: Alias): Alias;
    declare function interfaced(arg: Interface): Interface;
```
- 另一个重要区别是`类型别名`不能被 `extends`和 `implements`（自己也不能 `extends`和 `implements`其它类型）

- 如果你无法通过接口来描述一个类型并且需要使用`联合类型`或`元组类型`，这时通常会使用`类型别名`。

#### 字符串字面量类型

```javascript
   type Easing = "ease-in" | "ease-out" | "ease-in-out";
    class UIElement {
        animate(dx: number, dy: number, easing: Easing) {
            if (easing === "ease-in") {
                // ...
            }
            else if (easing === "ease-out") {
            }
            else if (easing === "ease-in-out") {
            }
            else {
                // error! should not pass null or undefined.
            }
        }
    }

    let button = new UIElement();
    button.animate(0, 0, "ease-in");
    button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here 
```
字符串字面量类型还可以用于区分函数重载：

```javascript
    function createElement(tagName: "img"): HTMLImageElement;
    function createElement(tagName: "input"): HTMLInputElement;
    // ... more overloads ...
    function createElement(tagName: string): Element {
        // ... code goes here ...
    }
```

#### 数字字面量类型

```javascript
    function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {
        // ...
    }
```

#### 可辨识联合（Discriminated Unions）

```javascript
    interface Square {
        kind: "square";
        size: number;
    }
    interface Rectangle {
        kind: "rectangle";
        width: number;
        height: number;
    }
    interface Circle {
        kind: "circle";
        radius: number;
    }
    type Shape = Square | Rectangle | Circle;
    function area(s: Shape) {
        switch (s.kind) {
            case "square": return s.size * s.size;
            case "rectangle": return s.height * s.width;
            case "circle": return Math.PI * s.radius ** 2;
        }
    }
```

#### 多态的 `this`类型

```javascript
    class BasicCalculator {
        public constructor(protected value: number = 0) { }
        public currentValue(): number {
            return this.value;
        }
        public add(operand: number): this {
            this.value += operand;
            return this;
        }
        public multiply(operand: number): this {
            this.value *= operand;
            return this;
        }
        // ... other operations go here ...
    }

    let v = new BasicCalculator(2)
                .multiply(5)
                .add(1)
                .currentValue();
```
#### 索引类型（Index types）
> 下面是如何在TypeScript里使用获取对象`key`相对应的`value`值，通过 `索引类型查询`和 `索引访问`操作符：

```javascript
    // keyof 获取T类型的 已知的 公共属性名的联合
    function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
        return names.map(n => o[n]);
    }

    interface Person {
        name: string;
        age: number;
    }
    let person: Person = {
        name: 'Jarid',
        age: 35
    };
    let strings: string[] = pluck(person, ['name']); // ok, string[]

    let personProps: keyof Person; // 'name' | 'age'
```

`T[K]`索引访问操作符。

#### 映射类型

> TypeScript提供了从旧类型中创建新类型的一种方式
```javascript
    type Readonly<T> = {
        readonly [P in keyof T]: T[P];
    }
    type Partial<T> = {
        [P in keyof T]?: T[P];
    }
    // 使用
    type PersonPartial = Partial<Person>;
    type ReadonlyPerson = Readonly<Person>;
```

#### Record<Keys,Type>
> Constructs an object type whose property keys are `Keys `and whose property values are `Type`. This utility can be used to map the properties of a type to another type.

```javascript
    interface PageInfo {
        title: string;
    }

    type Page = "home" | "about" | "contact";

    const nav: Record<Page, PageInfo> = {
    about: { title: "about" },
    contact: { title: "contact" },
    home: { title: "home" },
    };
```

#### 其他
- typeof
> 可以获取一个变量的类型
```javascript
    //先定义变量，再定义类型
    let p1 = {
        name:'zhufeng',
        age:10,
        gender:'male'
    }
    type People = typeof p1;
    function getName(p:People):string{
        return p.name;
    }
    getName(p1);
```
- 索引访问操作符
> 可以通过[]获取一个类型的子类型
```javascript
    interface Person{
        name:string;
        age:number;
        job:{
            name:string
        };
        interests:{name:string,level:number}[]
    }
    let FrontEndJob:Person['job'] = {
        name:'前端工程师'
    }
    let interestLevel:Person['interests'][0]['level'] = 2;
```
- keyof
> 索引类型查询操作符
```javascript
    interface Person{
    name:string;
    age:number;
    gender:'male'|'female';
    }
    //type PersonKey = 'name'|'age'|'gender';
    type PersonKey = keyof Person;

    function getValueByKey(p:Person,key:PersonKey){
    return p[key];
    }
    let val = getValueByKey({name:'zhufeng',age:10,gender:'male'},'name');
    console.log(val);
```
- 映射类型
    - 在定义的时候用in操作符去批量定义类型中的属性
    ```javascript
        interface Person{
            name:string;
            age:number;
            gender:'male'|'female';
        }
        //批量把一个接口中的属性都变成可选的
        type PartPerson = {
        [Key in keyof Person]?:Person[Key]
        }

        let p1:PartPerson={};
        //也可以使用泛型
        type Part<T> = {
        [key in keyof T]?:T[key]
        }
        let p2:Part<Person>={};
    ```
    - 通过key的数组获取值的数组
    ```javascript
        function pick<T, K extends keyof T>(o: T, names: K[]): T[K][] {
            return names.map((n) => o[n]);
        }
        let user = { id: 1, name: 'zhufeng' };
        type User = typeof user;
        const res = pick<User, keyof User>(user, ["id", "name"]);
        console.log(res);
    ```

- 内置条件类型

    - Exclude<T, U> -- 从T中剔除可以赋值给U的类型。
    ```javascript
        type Exclude<T, U> = T extends U ? never : T;

        type  E = Exclude<string|number,string>;
        let e:E = 10;
    ```
    - Extract<T, U> -- 提取T中可以赋值给U的类型。
    ```javascript
        type Extract<T, U> = T extends U ? T : never;

        type  E = Extract<string|number,string>;
        let e:E = '1';
    ```
    - NonNullable<T> -- 从T中剔除null和undefined。
    ```javascript
        type NonNullable<T> = T extends null | undefined ? never : T;

        type  E = NonNullable<string|number|null|undefined>;
        let e:E = null;
    ```
    - ReturnType<T> -- 获取函数返回值类型。
    - Parameters -- Constructs a tuple type of the types of the parameters of a function type T
    ```javascript
        type T0 = Parameters<() => string>;  // []
        type T1 = Parameters<(s: string) => void>;  // [string]
        type T2 = Parameters<(<T>(arg: T) => T)>;  // [unknown]
    ```
    - InstanceType<T> -- 获取构造函数类型的实例类型。

    ```javascript
        class Person {
            name: string;
            constructor(name: string) {
                this.name = name;
            }
            getName() { console.log(this.name) }
        }
        //构造函数参数
        type constructorParameters = ConstructorParameters<typeof Person>;
        let params: constructorParameters = ['zhufeng']
        //实例类型
        type Instance = InstanceType<typeof Person>;
        let instance: Instance = { name: 'zhufeng', getName() { } };
    ```

```javascript
    type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
    type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"

    type T02 = Exclude<string | number | (() => void), Function>;  // string | number
    type T03 = Extract<string | number | (() => void), Function>;  // () => void

    type T04 = NonNullable<string | number | undefined>;  // string | number
    type T05 = NonNullable<(() => string) | string[] | null | undefined>;  // (() => string) | string[]

    function f1(s: string) {
        return { a: 1, b: s };
    }

    class C {
        x = 0;
        y = 0;
    }

    type T10 = ReturnType<() => string>;  // string
    type T11 = ReturnType<(s: string) => void>;  // void
    type T12 = ReturnType<(<T>() => T)>;  // {}
    type T13 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
    type T14 = ReturnType<typeof f1>;  // { a: number, b: string }
    type T15 = ReturnType<any>;  // any
    type T16 = ReturnType<never>;  // any
    type T17 = ReturnType<string>;  // Error
    type T18 = ReturnType<Function>;  // Error

    type T20 = InstanceType<typeof C>;  // C
    type T21 = InstanceType<any>;  // any
    type T22 = InstanceType<never>;  // any
    type T23 = InstanceType<string>;  // Error
    type T24 = InstanceType<Function>;  // Error
```

#### 内置工具类型
> TypeScript中增加了对映射类型修饰符的控制。具体而言，一个 readonly 或 ? 修饰符在一个映射类型里可以用前缀 +（可选） 或-（必选）来表示这个修饰符应该被添加或移除
- Partial
> Partial 可以将传入的属性由非可选变为可选，具体使用如下

```javascript
    type Partial<T> = { [P in keyof T]?: T[P] };

    interface A {
    a1: string;
    a2: number;
    a3: boolean;
    }

    type aPartial = Partial<A>;

    const a: aPartial = {}; // 不会报错
```

- 类型递归

```javascript
    interface Company {
        id: number
        name: string
    }

    interface Person {
        id: number
        name: string
        company: Company
    }
    type DeepPartial<T> = {
        [U in keyof T]?: T[U] extends object
        ? DeepPartial<T[U]>
        : T[U]
    };

    type R2 = DeepPartial<Person>
```

- Required
> Required 可以将传入的属性中的可选项变为必选项，这里用了 -? 修饰符来实现。

```javascript
    interface Person{
        name:string;
        age:number;
        gender?:'male'|'female';
    }
    /**
    * type Require<T> = { [P in keyof T]-?: T[P] };
    */
    let p:Required<Person> = {
    name:'zhufeng',
    age:10,
    //gender:'male'
    }
```
- Readonly

```javascript
    interface Person{
    name:string;
    age:number;
    gender?:'male'|'female';
    }
    //type Readonly<T> = { readonly [P in keyof T]: T[P] };
    let p:Readonly<Person> = {
    name:'zhufeng',
    age:10,
    gender:'male'
    }
    p.age = 11;
```

- Pick
> Pick 能够帮助我们从传入的属性中摘取某一项返回

```javascript
    interface Animal {
    name: string;
    age: number;
    gender:number
    }
    /**
    * From T pick a set of properties K
    * type Pick<T, K extends keyof T> = { [P in K]: T[P] };
    */
    // 摘取 Animal 中的 name 属性
    interface Person {
        name: string;
        age: number;
        married: boolean
    }
    function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
        const result: any = {};
        keys.map(key => {
            result[key] = obj[key];
        });
        return result
    }
    let person: Person = { name: 'zhufeng', age: 10, married: true };
    let result: Pick<Person, 'name' | 'age'> = pick<Person, 'name' | 'age'>(person, ['name', 'age']);
    console.log(result);
```

- Record

> Record 是 TypeScript 的一个高级类型。他会将一个类型的所有属性值都映射到另一个类型上并创造一个新的类型

```javascript
    /**
    * Construct a type with a set of properties K of type T
    */
    type Record<K extends keyof any, T> = {
        [P in K]: T;
    };

    function mapObject<K extends string | number, T, U>(obj: Record<K, T>, map: (x: T) => U): Record<K, U> {
        let result: any = {};
        for (const key in obj) {
            result[key] = map(obj[key]);
        }
        return result;
    }
    let names = { 0: 'hello', 1: 'world' };
    let lengths = mapObject<string | number, string, number>(names, (s: string) => s.length);
    console.log(lengths);//{ '0': 5, '1': 5 }

    type Point = 'x' | 'y';
    type PointList = Record<Point, { value: number }>
    const cars: PointList = {
        x: { value: 10 },
        y: { value: 20 },
    }
```



















