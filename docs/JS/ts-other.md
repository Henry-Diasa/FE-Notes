### 其他内容

#### 命名空间

> 这篇文章描述了如何在TypeScript里使用命名空间（之前叫做“内部模块”）来组织你的代码

代码示例

```javascript
    interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    let lettersRegexp = /^[A-Za-z]+$/;
    let numberRegexp = /^[0-9]+$/;

    class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }

    // Some samples to try
    let strings = ["Hello", "98052", "101"];

    // Validators to use
    let validators: { [s: string]: StringValidator; } = {};
    validators["ZIP code"] = new ZipCodeValidator();
    validators["Letters only"] = new LettersOnlyValidator();

    // Show whether each string passed each validator
    for (let s of strings) {
        for (let name in validators) {
            let isMatch = validators[name].isAcceptable(s);
            console.log(`'${ s }' ${ isMatch ? "matches" : "does not match" } '${ name }'.`);
        }
    }
```

`使用命名空间的验证器`
> 把所有与验证器相关的类型都放到一个叫做Validation的命名空间里。 因为我们想让这些接口和类在命名空间之外也是可访问的，所以需要使用 `export`


```javascript
    namespace Validation {
        export interface StringValidator {
            isAcceptable(s: string): boolean;
        }

        const lettersRegexp = /^[A-Za-z]+$/;
        const numberRegexp = /^[0-9]+$/;

        export class LettersOnlyValidator implements StringValidator {
            isAcceptable(s: string) {
                return lettersRegexp.test(s);
            }
        }

        export class ZipCodeValidator implements StringValidator {
            isAcceptable(s: string) {
                return s.length === 5 && numberRegexp.test(s);
            }
        }
    }

    // Some samples to try
    let strings = ["Hello", "98052", "101"];

    // Validators to use
    let validators: { [s: string]: Validation.StringValidator; } = {};
    validators["ZIP code"] = new Validation.ZipCodeValidator();
    validators["Letters only"] = new Validation.LettersOnlyValidator();

    // Show whether each string passed each validator
    for (let s of strings) {
        for (let name in validators) {
            console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
        }
    }
```

`分离到多文件`
> 现在，我们把Validation命名空间分割成多个文件。 尽管是不同的文件，它们仍是同一个命名空间，并且在使用的时候就如同它们在一个文件中定义的一样。 因为不同文件之间存在依赖关系，所以我们加入了引用标签来告诉编译器文件之间的关联

- Validation.ts

```javascript
    namespace Validation {
        export interface StringValidator {
            isAcceptable(s: string): boolean;
        }
    }
```

- LettersOnlyValidator.ts

```javascript
    /// <reference path="Validation.ts" />
    namespace Validation {
        const lettersRegexp = /^[A-Za-z]+$/;
        export class LettersOnlyValidator implements StringValidator {
            isAcceptable(s: string) {
                return lettersRegexp.test(s);
            }
        }
    }
```
- ZipCodeValidator.ts

```javascript
    /// <reference path="Validation.ts" />
    namespace Validation {
        const numberRegexp = /^[0-9]+$/;
        export class ZipCodeValidator implements StringValidator {
            isAcceptable(s: string) {
                return s.length === 5 && numberRegexp.test(s);
            }
        }
    }
```
- Test.ts

```javascript
    /// <reference path="Validation.ts" />
    /// <reference path="LettersOnlyValidator.ts" />
    /// <reference path="ZipCodeValidator.ts" />

    // Some samples to try
    let strings = ["Hello", "98052", "101"];

    // Validators to use
    let validators: { [s: string]: Validation.StringValidator; } = {};
    validators["ZIP code"] = new Validation.ZipCodeValidator();
    validators["Letters only"] = new Validation.LettersOnlyValidator();

    // Show whether each string passed each validator
    for (let s of strings) {
        for (let name in validators) {
            console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
        }
    }
```

当涉及到多文件时，我们必须确保所有编译后的代码都被加载了。 我们有两种方式

第一种方式，把所有的输入文件编译为一个输出文件

`tsc --outFile sample.js Test.ts`
或单独地指定每个文件。
`tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts`

第二种方式，我们可以编译每一个文件（默认方式），那么每个源文件都会对应生成一个JavaScript文件。 然后，在页面上通过 <script>标签把所有生成的JavaScript文件按正确的顺序引进来

```javascript
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

`别名`

```javascript
    namespace Shapes {
        export namespace Polygons {
            export class Triangle { }
            export class Square { }
        }
    }

    import polygons = Shapes.Polygons;
    let sq = new polygons.Square(); // Same as "new Shapes.Polygons.Square()"
```

`外部命名空间`

```javascript
   declare namespace D3 {
        export interface Selectors {
            select: {
                (selector: string): Selection;
                (element: EventTarget): Selection;
            };
        }

        export interface Event {
            x: number;
            y: number;
        }

        export interface Base extends Selectors {
            event: Event;
        }
    }

    declare var d3: D3.Base; 
```

#### 声明合并

`合并接口`

```javascript
   // 接口的非函数的成员应该是唯一的。如果它们不是唯一的，那么它们必须是相同的类型。如果两个接口中同时声明了同名的非函数成员且它们的类型不同，则编译器会报错。 
   interface Box {
        height: number;
        width: number;
    }

    interface Box {
        scale: number;
    }

    let box: Box = {height: 5, width: 6, scale: 10}; 
```

```javascript
    // 对于函数成员，每个同名函数声明都会被当成这个函数的一个重载。 同时需要注意，当接口 A与后来的接口 A合并时，后面的接口具有更高的优先级。
    interface Cloner {
        clone(animal: Animal): Animal;
    }

    interface Cloner {
        clone(animal: Sheep): Sheep;
    }

    interface Cloner {
        clone(animal: Dog): Dog;
        clone(animal: Cat): Cat;
    }

    interface Cloner {
        clone(animal: Dog): Dog;
        clone(animal: Cat): Cat;
        clone(animal: Sheep): Sheep;
        clone(animal: Animal): Animal;
    }
```

```javascript
    // 这个规则有一个例外是当出现特殊的函数签名时。 如果签名里有一个参数的类型是 单一的字符串字面量（比如，不是字符串字面量的联合类型），那么它将会被提升到重载列表的最顶端。
    interface Document {
        createElement(tagName: any): Element;
    }
    interface Document {
        createElement(tagName: "div"): HTMLDivElement;
        createElement(tagName: "span"): HTMLSpanElement;
    }
    interface Document {
        createElement(tagName: string): HTMLElement;
        createElement(tagName: "canvas"): HTMLCanvasElement;
    }

    interface Document {
        createElement(tagName: "canvas"): HTMLCanvasElement;
        createElement(tagName: "div"): HTMLDivElement;
        createElement(tagName: "span"): HTMLSpanElement;
        createElement(tagName: string): HTMLElement;
        createElement(tagName: any): Element;
    }
```

`合并命名空间`

对于命名空间的合并，模块导出的同名接口进行合并，构成单一命名空间内含合并后的接口。

对于命名空间里值的合并，如果当前已经存在给定名字的命名空间，那么后来的命名空间的导出成员会被加到已经存在的那个模块里。

```javascript
    namespace Animals {
        export class Zebra { }
    }

    namespace Animals {
        export interface Legged { numberOfLegs: number; }
        export class Dog { }
    }


    // 等同于
    namespace Animals {
        export interface Legged { numberOfLegs: number; }

        export class Zebra { }
        export class Dog { }
    }

```
除了这些合并外，你还需要了解非导出成员是如何处理的。 非导出成员仅在其原有的（合并前的）命名空间内可见。这就是说合并之后，从其它命名空间合并进来的成员无法访问非导出成员。

```javascript
    namespace Animal {
        let haveMuscles = true;

        export function animalsHaveMuscles() {
            return haveMuscles;
        }
    }

    namespace Animal {
        export function doAnimalsHaveMuscles() {
            return haveMuscles;  // Error, because haveMuscles is not accessible here
        }
    }
```

#### 装饰器

安达市多

