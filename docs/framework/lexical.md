### 词法分析 - 为生成AST做准备

在 [Vue的编译器初探](http://localhost:8080/vue-design/art/80vue-compiler-start.html) 这一章节中，我们对 `Vue` 如何创建编译器，以及在这个过程中经历过的几个重要的函数做了分析，比如 `compileToFunctions` 函数以及 `compile` 函数，并且我们知道真正对模板进行编译工作的实际是 `baseCompile` 函数，而接下来我们的任务就是搞清楚 `baseCompile` 函数的内容。

`baseCompile` 函数是在 `src/compiler/index.js` 中作为 `createCompilerCreator` 函数的参数使用的，代码如下：

```js
// `createCompilerCreator` allows creating compilers that use alternative
// parser/optimizer/codegen, e.g the SSR optimizing compiler.
// Here we just export a default compiler using the default parts.
export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  optimize(ast, options)
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```

可以看到 `baseCompile` 函数接收两个参数，分别是字符串模板(`template`)和选项参数(`options`)，其中选项参数 `options` 我们已经分析过了，并且我们有对应的附录专门整理编译器的选项参数，可以在 [编译器选项整理](http://localhost:8080/vue-design/appendix/compiler-options.html) 中查看。

`baseCompile` 函数很简短，由三句代码和一个 `return` 语句组成，这三句代码的作用如下：

```js
// 调用 parse 函数将字符串模板解析成抽象语法树(AST)
const ast = parse(template.trim(), options)
// 调用 optimize 函数优化 ast
optimize(ast, options)
// 调用 generate 函数将 ast 编译成渲染函数
const code = generate(ast, options)
```

最终 `baseCompile` 的返回值如下：

```js
return {
  ast,
  render: code.render,
  staticRenderFns: code.staticRenderFns
}
```

可以看到，其最终返回了抽象语法树(`ast`)，渲染函数(`render`)，静态渲染函数(`staticRenderFns`)，且 `render` 的值为 `code.render`，`staticRenderFns` 的值为 `code.staticRenderFns`，也就是说通过 `generate` 处理 `ast` 之后得到的返回值 `code` 是一个对象，该对象的属性中包含了渲染函数（**注意以上提到的渲染函数，都以字符串的形式存在，因为真正变成函数的过程是在 `compileToFunctions` 中使用 `new Function()` 来完成的**）。

而接下来我们将会花费很大的篇幅来聚焦在一句代码上，即下面这句代码：

```js
const ast = parse(template.trim(), options)
```

也就是 `Vue` 的 `parser`，它是如何将字符串模板解析为抽象语法树(`AST`)的。

#### 对parser的简单介绍

在说 `parser` 之前，我们先了解一下编译器的概念，简单的讲编译器就是将 `源代码` 转换成 `目标代码` 的工具。详细一点如下(引用自维基百科)：

> 它主要的目的是将便于人编写、阅读、维护的高级计算机语言所写作的 `源代码` 程序，翻译为计算机能解读、运行的低阶机器语言的程序。`源代码` 一般为高阶语言（High-level language），如Pascal、C、C++、C# 、Java等，而目标语言则是汇编语言或目标机器的目标代码（Object code）。

编译器所包含的概念很多，比如 词法分析(`lexical analysis`)，句法分析(`parsing`)，类型检查/推导，代码优化，代码生成...等等，且大学中已有专门的课程，而我们这里要讲的 `parser` 就是编译器中的一部分，准确的说，`parser` 是编译器对源代码处理的第一步。

`parser` 是把某种特定格式的文本转换成某种数据结构的程序，其中“特定格式的文本”可以理解为普通的字符串，而 `parser` 的作用就是将这个字符串转换成一种数据结构(通常是一个对象)，并且这个数据结构是编译器能够理解的，因为编译器的后续步骤，比如上面提到的 句法分析，类型检查/推导，代码优化，代码生成 等等都依赖于该数据结构，正因如此我们才说 `parser` 是编译器处理源代码的第一步，并且这种数据结构是抽象的，我们常称其为抽象语法树，即 `AST`。

`Vue` 的编译器也不例外，大致也分为三个阶段，即：词法分析 -> 句法分析 -> 代码生成。在词法分析阶段 `Vue` 会把字符串模板解析成一个个的令牌(`token`)，该令牌将用于句法分析阶段，在句法分析阶段会根据令牌生成一棵 `AST`，最后再根据该 `AST` 生成最终的渲染函数，这样就完成了代码的生成。按照顺序我们需要先了解的是词法分析阶段，看一看 `Vue` 是如何对字符串模板进行拆解的

#### Vue中的html-parser

本节中大量出现 `parse` 以及 `parser` 这两个单词，不要混淆这两个单词，`parse` 是动词，代表“解析”的过程，`parser` 是名词，代表“解析器”。

回到 `baseCompile` 函数中的这句代码：

```js
const ast = parse(template.trim(), options)
```

由这句代码可知 `parse` 函数就是用来解析模板字符串的，最终生成 `AST`，根据文件头部的引用关系可知 `parse` 函数位于 `src/compiler/parser/index.js` 文件，打开该文件可以发现其的确导出了一个名字为 `parse` 的函数，如下：

```js
export function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
  // 省略...

  parseHTML(template, {
    warn,
    expectHTML: options.expectHTML,
    isUnaryTag: options.isUnaryTag,
    canBeLeftOpenTag: options.canBeLeftOpenTag,
    shouldDecodeNewlines: options.shouldDecodeNewlines,
    shouldDecodeNewlinesForHref: options.shouldDecodeNewlinesForHref,
    shouldKeepComment: options.comments,
    start (tag, attrs, unary) {
      // 省略...
    },
    end () {
      // 省略...
    },
    chars (text: string) {
      // 省略...
    },
    comment (text: string) {
      // 省略...
    }
  })
  return root
}
```

同时我们注意到在 `parse` 函数内部主要通过调用 `parseHTML` 函数对模板字符串进行解析，实际上 `parseHTML` 函数的作用就是用来做词法分析的，而 `parse` 函数的作用则是在词法分析的基础上做句法分析从而生成一棵 `AST`。本节我们主要分析一下 `Vue` 是如何对模板字符串进行词法分析的，也就是 `parseHTML` 函数的实现。

根据文件头部的引用关系可知 `parseHTML` 函数来自 `src/compiler/parser/html-parser.js` 文件，实际上整个 `html-parser.js` 文件所做的事情都是在做词法分析，接下来我们就研究一下它是如何实现的。打开该文件，其开头是一段注释：

```js
/**
 * Not type-checking this file because it's mostly vendor code.
 */

/*!
 * HTML Parser By John Resig (ejohn.org)
 * Modified by Juriy "kangax" Zaytsev
 * Original code by Erik Arvidsson, Mozilla Public License
 * http://erik.eae.net/simplehtmlparser/simplehtmlparser.js
 */
```

通过这段注释我们可以了解到，`Vue` 的 `html parser` 是 `fork` 自 [John Resig 所写的一个开源项目：http://erik.eae.net/simplehtmlparser/simplehtmlparser.js](http://erik.eae.net/simplehtmlparser/simplehtmlparser.js)，`Vue` 在此基础上做了很多完善的工作，下面我们就探究一下 `Vue` 中的 `html parser` 都做了哪些事情。

#### 正则分析

代码正文的一开始，是两句 `import` 语句，以及定义的一些正则常量：

```js
import { makeMap, no } from 'shared/util'
import { isNonPhrasingTag } from 'web/compiler/util'

// Regular Expressions for parsing tags and attributes
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
// could use https://www.w3.org/TR/1999/REC-xml-names-19990114/#NT-QName
// but for Vue templates we can enforce a simple charset
const ncname = '[a-zA-Z_][\\w\\-\\.]*'
const qnameCapture = `((?:${ncname}\\:)?${ncname})`
const startTagOpen = new RegExp(`^<${qnameCapture}`)
const startTagClose = /^\s*(\/?)>/
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`)
const doctype = /^<!DOCTYPE [^>]+>/i
// #7298: escape - to avoid being pased as HTML comment when inlined in page
const comment = /^<!\--/
const conditionalComment = /^<!\[/
```

下面我们依次来看一下这些正则：

##### attribute

首先是 `attribute` 正则常量：

```js
// Regular Expressions for parsing tags and attributes
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
```

`attribute` 顾名思义，这个正则的作用是用来匹配标签的属性(`attributes`)的

我们在观察一个复杂的正则表达式时，主要就是要观察它有几个分组(准确地说应该是有几个捕获的分组)，通过上图我们能够清晰地看到，这个表达式有五个捕获组，第一个捕获组用来匹配属性名，第二个捕获组用来匹配等于号，第三、第四、第五个捕获组都是用来匹配属性值的，同时 `?` 表明第三、四、五个分组是可选的。 这是因为在 `html` 标签中有4种写属性值的方式：

- 1、使用双引号把值引起来：`class="some-class"`
- 2、使用单引号把值引起来：`class='some-class'`
- 3、不使用引号：`class=some-class`
- 4、单独的属性名：`disabled`

正因如此，需要三个正则分组并配合可选属性来分别匹配四种情况，我们可以对这个正则做一个测试，如下：

```js
const attribute = /^\s*([^\s"'<>\/=]+)(?:\s*(=)\s*(?:"([^"]*)"+|'([^']*)'+|([^\s"'=<>`]+)))?/
console.log('class="some-class"'.match(attribute))  // 测试双引号
console.log("class='some-class'".match(attribute))  // 测试单引号
console.log('class=some-class'.match(attribute))  // 测试无引号
console.log('disabled'.match(attribute))  // 测试无属性值
```

对于双引号的情况，我们将得到以下结果：

```js
[
    'class="some-class"',
    'class',
    '=',
    'some-class',
    undefined,
    undefined
]
```

数组共有从 `0` 到 `5` 六个元素，第 `0` 个元素是被整个正则所匹配的结果，从第 `1` 至第 `5` 个元素分别对应五个捕获组的匹配结果，我们可以看到，第 `1` 个元素对应第一个捕获组，匹配到了属性名(`class`)；第 `2` 个元素对应第二个捕获组，匹配到了等号(`=`)；第 `3` 个元素对应第三个捕获组，匹配到了带双引号的属性值；而第 `4` 和第 `5` 个元素分别对应第四和第五个捕获组，由于没有匹配到所以都是 `undefined`。

所以通过以上结果我们很容易想到当属性值被单引号引起来和不使用引号的情况下所得到的匹配结果是什么，变化主要就在匹配结果数组的第 `3`、`4`、`5` 个元素，匹配到哪种情况，那么对应的位置就是属性值，其他位置则是 `undefined`，如下：

```js
// 对于单引号的情况
[
    "class='some-class'",
    'class',
    '=',
    undefined,
    'some-class',
    undefined
]
// 对于没有引号
[
    'class=some-class',
    'class',
    '=',
    undefined,
    undefined,
    'some-class'
]
// 对于单独的属性名
[
    'disabled',
    'disabled',
    undefined,
    undefined,
    undefined,
    undefined
]
```

##### ncname

接下来一句代码如下：

```js
// could use https://www.w3.org/TR/1999/REC-xml-names-19990114/#NT-QName
// but for Vue templates we can enforce a simple charset
const ncname = '[a-zA-Z_][\\w\\-\\.]*'
```

首先给大家解释几个概念并说明一些问题：

- 一、合法的 XML 名称是什么样的？

首先在 XML 中，标签是用户自己定义的，比如：`<bug></bug>`。

正因为这样，所以不同的文档中如果定义了相同的元素(标签)，就会产生冲突，为此，XML 允许用户为标签指定前缀：`<k:bug></k:bug>`，前缀是字母 `k`。

除了前缀还可以使用命名空间，即使用标签的 `xmlns` 属性，为前缀赋予与指定命名空间相关联的限定名称：

```html
<k:bug xmlns:k="http://www.xxx.com/xxx"></k:bug>
```

综上所述，一个合法的XML标签名应该是由 `前缀`、`冒号(:)` 以及 `标签名称` 组成的：`<前缀:标签名称>`

- 二、什么是 `ncname`？

`ncname` 的全称是 `An XML name that does not contain a colon (:)` 即：不包含冒号(`:`)的 XML 名称。也就是说 `ncname` 就是不包含前缀的XML标签名称。大家可以在这里找到关于 [ncname](https://msdn.microsoft.com/zh-cn/library/ms256452.aspx) 的概念。

- 三、什么是 `qname`？

我们可以在 `Vue` 的源码中看到其给出了一个链接：https://www.w3.org/TR/1999/REC-xml-names-19990114/#NT-QName，其实 `qname` 就是：`<前缀:标签名称>`，也就是合法的XML标签。

了解了这些，我们再来看 `ncname` 的正则表达式，它定义了 `ncname` 的合法组成，这个正则所匹配的内容很简单：*字母或下划线开头，后面可以跟任意数量的字符、中横线和 `.`*。

##### qnameCapture

下一个正则是 `qnameCapture`，`qnameCapture` 同样是普通字符串，只不过将来会用在 `new RegExp()` 中：

```js
const qnameCapture = `((?:${ncname}\\:)?${ncname})`
```

我们知道 `qname` 实际上就是合法的标签名称，它是由可选项的 `前缀`、`冒号` 以及 `名称` 组成，观察 `qnameCapture` 可知它有一个捕获分组，捕获的内容就是整个 `qname` 名称，即整个标签的名称。

##### startTagOpen

`startTagOpen` 是一个真正使用 `new RegExp()` 创建出来的正则表达式：

```js
const startTagOpen = new RegExp(`^<${qnameCapture}`)
```

用来匹配开始标签的一部分，这部分包括：`<` 以及后面的 `标签名称`，这个表达式的创建用到了上面定义的 `qnameCapture` 字符串，所以 `qnameCapture` 这个字符串中所设置的捕获分组，在这里同样适用，也就是说 `startTagOpen` 这个正则表达式也会有一个捕获的分组，用来捕获匹配的标签名称。

##### startTagClose

```js
const startTagClose = /^\s*(\/?)>/
startTagOpen` 用来匹配开始标签的 `<` 以及标签的名字，但是并不包括开始标签的闭合部分，即：`>` 或者 `/>`，由于标签可能是一元标签，所以开始标签的闭合部分有可能是 `/>`，比如：`<br />`，如果不是一元标签，此时就应该是：`>
```

观察 `startTagClose` 可知，这个正则拥有一个捕获分组，用来捕获开始标签结束部分的斜杠：`/`。

##### endTag

```js
const endTag = new RegExp(`^<\\/${qnameCapture}[^>]*>`)
```

`endTag` 这个正则用来匹配结束标签，由于该正则同样使用了字符串 `qnameCapture`，所以这个正则也拥有了一个捕获组，用来捕获标签名称。

##### doctype

```js
const doctype = /^<!DOCTYPE [^>]+>/i
```

这个正则用来匹配文档的 `DOCTYPE` 标签，没有捕获组。

##### comment

```js
// #7298: escape - to avoid being pased as HTML comment when inlined in page
const comment = /^<!\--/
```

这个正则用来匹配注释节点，没有捕获组。大家注意这句代码上方的注释，索引是：`#7298`。有兴趣的同学可以去 `Vue` 的 `issue` 中搜索一下相关问题。在这之前实际上 `comment` 常量的值是 `<!--` 而并不是 `<!\--`，之所以改成 `<!\--` 是为了允许把 `Vue` 代码内联到 `html` 中，否则 `<!--` 会被认为是注释节点。

##### conditionalComment

```js
const conditionalComment = /^<!\[/
```

这个正则用来匹配条件注释节点，没有捕获组。

最后很重要的一点是，大家注意，这些正则表达式有一个共同的特点，即：*他们都是从一个字符串的开头位置开始匹配的，因为有 `^` 的存在*。

在这些正则常量的下面，有着这样一段代码：

```js
let IS_REGEX_CAPTURING_BROKEN = false
'x'.replace(/x(.)?/g, function (m, g) {
  IS_REGEX_CAPTURING_BROKEN = g === ''
})
```

首先定义了变量 `IS_REGEX_CAPTURING_BROKEN` 且初始值为 `false`，接着使用一个字符串 `'x'` 的 `replace` 函数用一个带有捕获组的正则进行匹配，并将捕获组捕获到的值赋值给变量 `g`。我们观察字符串 `'x'` 和正则 `/x(.)?/` 可以发现，该正则中的捕获组应该捕获不到任何内容，所以此时 `g` 的值应该是 `undefined`，但是在老版本的火狐浏览器中存在一个问题，此时的 `g` 是一个空字符串 `''`，并不是 `undefined`。所以变量 `IS_REGEX_CAPTURING_BROKEN` 的作用就是用来标识当前宿主环境是否存在该问题。这个变量我们后面会用到，其作用到时候再说

#### 变量分析

在这些正则的下面，定义了一些常量，如下：

```js
// Special Elements (can contain anything)
export const isPlainTextElement = makeMap('script,style,textarea', true)
const reCache = {}

const decodingMap = {
  '&lt;': '<',
  '&gt;': '>',
  '&quot;': '"',
  '&amp;': '&',
  '&#10;': '\n',
  '&#9;': '\t'
}
const encodedAttr = /&(?:lt|gt|quot|amp);/g
const encodedAttrWithNewLines = /&(?:lt|gt|quot|amp|#10|#9);/g
```

上面这段代码中，包含 `5` 个常量，我们逐个来看。

首先 `isPlainTextElement` 常量是一个函数，它是通过 `makeMap` 函数生成的，用来检测给定的标签名字是不是纯文本标签（包括：`script`、`style`、`textarea`）。

然后定义了 `reCache` 常量，它被初始化为一个空的 `JSON` 对象字面量。

再往下定义了 `decodingMap` 常量，它也是一个 `JSON` 对象字面量，其中 `key` 是一些特殊的 `html` 实体，值则是这些实体对应的字符。在 `decodingMap` 常量下面的是两个正则常量：`encodedAttr` 和 `encodedAttrWithNewLines`。可以发现正则 `encodedAttrWithNewLines` 会比 `encodedAttr` 多匹配两个 `html` 实体字符，分别是 ` ` 和 `	`。对于 `decodingMap` 以及下面两个正则的作用不知道大家能不能猜得到，其实我们讲解编译器的创建时有讲到 `shouldDecodeNewlines` 和 `shouldDecodeNewlinesForHref` 这两个编译器选项，当时我们就有针对这两个选项的作用做讲解，可以在附录 [platforms/web/util 目录下的工具方法全解](http://localhost:8080/vue-design/appendix/web-util.html) 中查看。

所以这里的常量 `decodingMap` 以及两个正则 `encodedAttr` 和 `encodedAttrWithNewLines` 的作用就是用来完成对 `html` 实体进行解码的。

再往下是这样一段代码：

```js
// #5992
const isIgnoreNewlineTag = makeMap('pre,textarea', true)
const shouldIgnoreFirstNewline = (tag, html) => tag && isIgnoreNewlineTag(tag) && html[0] === '\n'
```

定义了两个常量，其中 `isIgnoreNewlineTag` 是一个通过 `makeMap` 函数生成的函数，用来检测给定的标签是否是 `<pre>` 标签或者 `<textarea>` 标签。这个函数被用在了 `shouldIgnoreFirstNewline` 函数里，`shouldIgnoreFirstNewline` 函数的作用是用来检测是否应该忽略元素内容的第一个换行符。什么意思呢？大家注意这两段代码上方的注释：`// #5992`，感兴趣的同学可以去 `vue` 的 `issue` 中搜一下，大家就会发现，这两句代码的作用是用来解决一个问题，该问题是由于历史原因造成的，即一些元素会受到额外的限制，比如 `<pre>` 标签和 `<textarea>` 会忽略其内容的第一个换行符，所以下面这段代码是等价的：

```html
<pre>内容</pre>
```

等价于：

```html
<pre>
内容</pre>
```

以上是浏览器的行为，所以 `Vue` 的编译器也要实现这个行为，否则就会出现 `issue #5992` 或者其他不可预期的问题。明白了这些我们再看 `shouldIgnoreFirstNewline` 函数就很容易理解，这个函数就是用来判断是否应该忽略标签内容的第一个换行符的，如果满足：*标签是 `pre` 或者 `textarea`* 且 *标签内容的第一个字符是换行符*，则返回 `true`，否则为 `false`。

`isIgnoreNewlineTag` 函数将被用于后面的 `parse` 过程，所以我们到时候再看，接着往下看代码，接下来定义了一个函数 `decodeAttr`，其源码如下：

```js
function decodeAttr (value, shouldDecodeNewlines) {
  const re = shouldDecodeNewlines ? encodedAttrWithNewLines : encodedAttr
  return value.replace(re, match => decodingMap[match])
}
```

`decodeAttr` 函数是用来解码 `html` 实体的。它的原理是利用前面我们讲过的正则 `encodedAttrWithNewLines` 和 `encodedAttr` 以及 `html` 实体与字符一一对应的 `decodingMap` 对象来实现将 `html` 实体转为对应的字符。该函数将会在后面 `parse` 的过程中使用到

