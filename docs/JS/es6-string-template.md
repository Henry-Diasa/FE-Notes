#### 基础用法

```javascript
let message = `Hello World`;
console.log(message);
```

如果你碰巧要在字符串中使用反撇号，你可以使用反斜杠转义：

```javascript
let message = `Hello \` World`;
console.log(message);
```

值得一提的是，在模板字符串中，空格、缩进、换行都会被保留：

```javascript
let message = `
	<ul>
		<li>1</li>
		<li>2</li>
	</ul>
`;
console.log(message);
```

注意，打印的结果中第一行是一个换行，你可以使用 trim 函数消除换行：

```javascript
let message = `
	<ul>
		<li>1</li>
		<li>2</li>
	</ul>
`.trim();
console.log(message);
```

#### 嵌入变量

模板字符串支持嵌入变量，只需要将变量名写在 ${} 之中，其实不止变量，任意的 JavaScript 表达式都是可以的：

```javascript
let x = 1, y = 2;
let message = `<ul><li>${x}</li><li>${x + y}</li></ul>`;
console.log(message); // <ul><li>1</li><li>3</li></ul>
```

值得一提的是，模板字符串支持嵌套:

```javascript
let arr = [{value: 1}, {value: 2}];
let message = `
	<ul>
		${arr.map((item) => {
			return `
				<li>${item.value}</li>
			`
		})}
	</ul>
`;
console.log(message);
```

打印结果如下：

[![string](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/string/string3.png)](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/string/string3.png)

注意，在 li 标签中间多了一个逗号，这是因为当大括号中的值不是字符串时，会将其转为字符串，比如一个数组 [1, 2, 3] 就会被转为 1,2,3，逗号就是这样产生的。

如果你要消除这个逗号，你可以先 join 一下：

```javascript
let arr = [{value: 1}, {value: 2}];
let message = `
	<ul>
		${arr.map((item) => {
			return `
				<li>${item.value}</li>
			`
		}).join('')}
	</ul>
`;
console.log(message);
```

打印结果如下：

[![string](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/string/string4.png)](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/string/string4.png)

#### 标签模板

模板标签是一个非常重要的能力，模板字符串可以紧跟在一个函数名后面，该函数将被调用来处理这个模板字符串，举个例子：

```javascript
let x = 'Hi', y = 'Kevin';
var res = message`${x}, I am ${y}`;
console.log(res);
```

我们可以自定义 message 函数来处理返回的字符串:

```javascript
// literals 文字
// 注意在这个例子中 literals 的第一个元素和最后一个元素都是空字符串
function message(literals, value1, value2) {
	console.log(literals); // [ "", ", I am ", "" ]
	console.log(value1); // Hi
	console.log(value2); // Kevin
}
```

我们利用这些参数将其拼合回去：

```javascript
function message(literals, ...values) {
	let result = '';

	for (let i = 0; i < values.length; i++) {
		result += literals[i];
		result += values[i];
	}

	result += literals[literals.length - 1];

	return result;
}
```

你也可以这样写：

```javascript
function message(literals, ...values) {
	let result = literals.reduce((prev, next, i) => {
	    let value = values[i - 1];
	    return prev + value + next;
	});

	return result;
}
```

学着拼合回去是一件非常重要的事情，因为我们经过各种处理，最终都还是要拼回去的……

#### oneLine

讲完了基础，我们可以来看一些实际的需求：

```
let message = `
	Hi,
	Daisy!
	I am
	Kevin.
`;
```

出于可读性或者其他原因，我希望书写的时候是换行的，但是最终输出的字符是在一行，这就需要借助模板标签来实现了，我们尝试写一个这样的函数：

```
// oneLine 第一版
function oneLine(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {
        let expression = expressions[i - 1];
        return prev + expression + next;
    });

    result = result.replace(/(\s+)/g, " ");
    result = result.trim();

    return result;
}
```

实现原理很简单，拼合回去然后将多个空白符如换行符、空格等替换成一个空格。

使用如下：

```
let message = oneLine `
    Hi,
    Daisy!
    I am
    Kevin.
`;
console.log(message); // Hi, Daisy! I am Kevin.
```

不过你再用下去就会发现一个问题，如果字符间就包括多个空格呢？举个例子：

```
let message = oneLine`
  Preserve eg sentences.  Double
  spaces within input lines.
`;
```

如果使用这种匹配方式，`sentences.` 与 `Double` 之间的两个空格也会被替换成一个空格。

我们可以再优化一下，我们想要的效果是将每行前面的多个空格替换成一个空格，其实应该匹配的是换行符以及换行符后面的多个空格，然后将其替换成一个空格，我们可以将正则改成：

```
result = result.replace(/(\n\s*)/g, " ");
```

就可以正确的匹配代码。最终的代码如下：

```
// oneLine 第二版
function oneLine(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {
        let expression = expressions[i - 1];
        return prev + expression + next;
    });

    result = result.replace(/(\n\s*)/g, " ");
    result = result.trim();

    return result;
}
```

#### stripIndents

假设有这样一段 HTML：

```
let html = `
	<span>1<span>
	<span>2<span>
		<span>3<span>
`;
```

为了保持可读性，我希望最终输入的样式为：

```
<span>1<span>
<span>2<span>
<span>3<span>
```

其实就是匹配每行前面的空格，然后将其替换为空字符串。

```
// stripIndents 第一版
function stripIndents(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {
        let expression = expressions[i - 1];
        return prev + expression + next;
    });


    result = result.replace(/\n[^\S\n]*/g, '\n');
    result = result.trim();

    return result;
}
```

最难的或许就是这个正则表达式了：

```
result = result.replace(/\n[^\S\n]*/g, '\n');
```

`\S` 表示匹配一个非空白字符

`[^\S\n]` 表示匹配`非空白字符`和`换行符`之外的字符，其实也就是空白字符去除换行符

`\n[^\S\n]*` 表示匹配换行符以及换行符后的多个不包含换行符的空白字符

`replace(/\n[^\S\n]*/g, '\n')` 表示将一个换行符以及换行符后的多个不包含换行符的空白字符替换成一个换行符，其实也就是将换行符后面的空白字符消掉的意思

其实吧，不用写的这么麻烦，我们还可以这样写：

```
result = result.replace(/^[^\S\n]+/gm, '');
```

看似简单了一点，之所以能这样写，是因为匹配模式的缘故，你会发现，这次除了匹配全局之外，这次我们还匹配了多行，m 标志用于指定多行输入字符串时应该被视为多个行，而且如果使用 m 标志，^ 和 $ 匹配的开始或结束是输入字符串中的每一行，而不是整个字符串的开始或结束。

[^\S\n] 表示匹配空白字符去除换行符

^[^\S\n]+ 表示匹配以`去除换行符的空白字符`为开头的一个或者多个字符

result.replace(/^[^\S\n]+/gm, '') 表示将每行开头一个或多个`去除换行符的空白字符`替换成空字符串，也同样达到了目的。

最终的代码如下：

```
// stripIndents 第二版
function stripIndents(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {
        let expression = expressions[i - 1];
        return prev + expression + next;
    });


    result = result.replace(/^[^\S\n]+/gm, '');
    result = result.trim();

    return result;
}
```

#### stripIndent

注意，这次的 stripIndent 相比上面一节的标题少了一个字母 s，而我们想要实现的功能是：

```
let html = `
	<ul>
		<li>1</li>
		<li>2</li>
		<li>3</li>
	<ul>
`;
```

[![string](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/string/string5.png)](https://raw.githubusercontent.com/mqyqingfeng/Blog/master/Images/ES6/string/string5.png)

其实也就是去除第一行的换行以及每一行的部分缩进。

这个实现就稍微麻烦了一点，因为我们要计算出每一行到底要去除多少个空白字符。

实现的思路如下：

1. 使用 match 函数，匹配每一行的空白字符，得到一个包含每一行空白字符的数组
2. 数组遍历比较，得到最小的空白字符长度
3. 构建一个正则表达式，然后每一行都替换掉最小长度的空白字符

实现的代码如下：

```
let html = `
	<ul>
		<li>1</li>
		<li>2</li>
		<li>3</li>
	<ul>
`;


function stripIndent(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {
        let expression = expressions[i - 1];
        return prev + expression + next;
    });

    const match = result.match(/^[^\S\n]*(?=\S)/gm);
    console.log(match); // Array [ "    ", "        ", "        ", "        ", "    " ]

    const indent = match && Math.min(...match.map(el => el.length));
    console.log(indent); // 4

    if (indent) {
        const regexp = new RegExp(`^.{${indent}}`, 'gm');
        console.log(regexp); // /^.{4}/gm

        result =  result.replace(regexp, '');
    }

    result = result.trim();

    return result;
}
```

值得一提的是，我们一般会以为正则中 `.` 表示匹配任意字符，其实是匹配除换行符之外的任何单个字符。

最终精简的代码如下：

```
function stripIndent(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {
        let expression = expressions[i - 1];
        return prev + expression + next;
    });

    const match = result.match(/^[^\S\n]*(?=\S)/gm);
    const indent = match && Math.min(...match.map(el => el.length));

    if (indent) {
        const regexp = new RegExp(`^.{${indent}}`, 'gm');
        result =  result.replace(regexp, '');
    }

    result = result.trim();

    return result;
}
```

#### includeArrays

前面我们讲到为了避免 ${} 表达式中返回一个数组，自动转换会导致多个逗号的问题，需要每次都将数组最后再 join('') 一下，再看一遍例子：

```
let arr = [{value: 1}, {value: 2}];
let message = `
	<ul>
		${arr.map((item) => {
			return `
				<li>${item.value}</li>
			`
		}).join('')}
	</ul>
`;
console.log(message);
```

利用标签模板，我们可以轻松的解决这个问题：

```
function includeArrays(template, ...expressions) {
    let result = template.reduce((prev, next, i) => {

        let expression = expressions[i - 1];

        if (Array.isArray(expression)) {
            expression = expression.join('');
        }

        return prev + expression + next;
    });

    result = result.trim();

    return result;
}
```

#### 最后

你会发现以上这些函数拼合的部分都是重复的，我们完全可以将其封装在一起，根据不同的配置实现不能的功能。如果你想在项目中使用这些函数，可以自己封装一个或者直接使用 [common-tags](https://github.com/declandewet/common-tags)。

