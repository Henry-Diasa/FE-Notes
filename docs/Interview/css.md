#### css sprite是什么,有什么优缺点

- 概念：将多个小图片拼接到一个图片中。通过`background-position`和元素尺寸调节需要显示的背景图案。
- 优点：
  - 减少`HTTP`请求数，极大地提高页面加载速度
  - 增加图片信息重复度，提高压缩比，减少图片大小
  - 更换风格方便，只需在一张或几张图片上修改颜色或样式即可实现
- 缺点：
  - 图片合并麻烦
  - 维护麻烦，修改一个图片可能需要从新布局整个图片，样式

#### `display: none;`与`visibility: hidden;`的区别

- 联系：它们都能让元素不可见
- 区别：
  - `display:none`;会让元素完全从渲染树中消失，渲染的时候不占据任何空间；`visibility: hidden`;不会让元素从渲染树消失，渲染师元素继续占据空间，只是内容不可见
  - `display: none`;是非继承属性，子孙节点消失由于元素从渲染树消失造成，通过修改子孙节点属性无法显示`；visibility: hidden;`是继承属性，子孙节点消失由于继承了`hidden`，通过设置`visibility: visible;`可以让子孙节点显式
  - 修改常规流中元素的`display`通常会造成文档重排。修改`visibility`属性只会造成本元素的重绘。
  - 读屏器不会读取`display: none`;元素内容；会读取`visibility: hidden;`元素内容

#### `link`与`@import`的区别

1. `link`是`HTML`方式， `@import`是CSS方式
2. `link`最大限度支持并行下载，`@import`过多嵌套导致串行下载，出现`FOUC`(文档样式短暂失效)
3. `link`可以通过`rel="alternate stylesheet"`指定候选样式
4. 浏览器对`link`支持早于`@import`，可以使用`@import`对老浏览器隐藏样式
5. `@import`必须在样式规则之前，可以在css文件中引用其他文件
6. 总体来说：`link`优于`@import`

#### 什么是FOUC?如何避免

- `Flash Of Unstyled Content`：用户定义样式表加载之前浏览器使用默认样式显示文档，用户样式加载渲染之后再从新显示文档，造成页面闪烁。
- **解决方法**：把样式表放到文档的`<head>`

#### css3有哪些新特性

- 新增选择器 `p:nth-child(n){color: rgba(255, 0, 0, 0.75)}`
- 弹性盒模型 `display: flex;`
- 多列布局 `column-count: 5;`
- 媒体查询 `@media (max-width: 480px) {.box: {column-count: 1;}}`
- 个性化字体 `@font-face{font-family: BorderWeb; src:url(BORDERW0.eot);}`
- 颜色透明度 `color: rgba(255, 0, 0, 0.75);`
- 圆角 `border-radius: 5px;`
- 渐变 `background:linear-gradient(red, green, blue);`
- 阴影 `box-shadow:3px 3px 3px rgba(0, 64, 128, 0.3);`
- 倒影 `box-reflect: below 2px;`
- 文字装饰 `text-stroke-color: red;`
- 文字溢出 `text-overflow:ellipsis;`
- 背景效果 `background-size: 100px 100px;`
- 边框效果 `border-image:url(bt_blue.png) 0 10;`
- 转换
  - 旋转 `transform: rotate(20deg);`
  - 倾斜 `transform: skew(150deg, -10deg);`
  - 位移 `transform: translate(20px, 20px);`
  - 缩放 `transform: scale(.5);`
- 平滑过渡 `transition: all .3s ease-in .1s;`
- 动画 `@keyframes anim-1 {50% {border-radius: 50%;}} animation: anim-1 1s;`

**CSS3新增伪类有那些？**

- `p:first-of-type` 选择属于其父元素的首个`<p>`元素的每个`<p>` 元素。
- `p:last-of-type` 选择属于其父元素的最后 `<p>` 元素的每个`<p>` 元素。
- `p:only-of-type` 选择属于其父元素唯一的 `<p>`元素的每个 `<p>` 元素。
- `p:only-child` 选择属于其父元素的唯一子元素的每个 `<p>` 元素。
- `p:nth-child(2)` 选择属于其父元素的第二个子元素的每个 `<p>` 元素。
- `:after` 在元素之前添加内容,也可以用来做清除浮动。
- `:before` 在元素之后添加内容。
- `:enabled` 已启用的表单元素。
- `:disabled` 已禁用的表单元素。
- `:checked` 单选框或复选框被选中。

#### 介绍一下标准的CSS的盒子模型？低版本IE的盒子模型有什么不同的？

> - 有两种， `IE`盒子模型、`W3C`盒子模型；
> - 盒模型： 内容(content)、填充(`padding`)、边界(`margin`)、 边框(`border`)；
> - 区 别： `IE`的c`ontent`部分把 `border` 和 `padding`计算了进去;

- 盒子模型构成：内容(`content`)、内填充(`padding`)、 边框(`border`)、外边距(`margin`)
- `IE8`及其以下版本浏览器，未声明 `DOCTYPE`，内容宽高会包含内填充和边框，称为怪异盒模型(`IE`盒模型)
- 标准(`W3C`)盒模型：元素宽度 = `width + padding + border + margin`
- 怪异(`IE`)盒模型：元素宽度 = `width + margin`
- 标准浏览器通过设置 css3 的 `box-sizing: border-box` 属性，触发“怪异模式”解析计算宽高

**box-sizing 常用的属性有哪些？分别有什么作用**

- `box-sizing: content-box;` 默认的标准(W3C)盒模型元素效果
- `box-sizing: border-box;` 触发怪异(IE)盒模型元素的效果
- `box-sizing: inherit;` 继承父元素 `box-sizing` 属性的值

#### CSS优先级算法如何计算？

- 优先级就近原则，同权重情况下样式定义最近者为准
- 载入样式以最后载入的定位为准
- 优先级为: `!important > id > class > tag`; `!important` 比 内联优先级高

#### display:inline-block 什么时候不会显示间隙？(携程)

- 移除空格
- 使用`margin`负值
- 使用`font-size:0`
- `letter-spacing`
- `word-spacing`

#### PNG\GIF\JPG的区别及如何选

- `GIF`
  - `8`位像素，`256`色
  - 无损压缩
  - 支持简单动画
  - 支持`boolean`透明
  - 适合简单动画
- `JPEG`
  - 颜色限于`256`
  - 有损压缩
  - 可控制压缩质量
  - 不支持透明
  - 适合照片
- `PNG`
  - 有`PNG8`和`truecolor PNG`
  - `PNG8`类似`GIF`颜色上限为`256`，文件小，支持`alpha`透明度，无动画
  - 适合图标、背景、按钮

#### 行内元素float:left后是否变为块级元素？

> 行内元素设置成浮动之后变得更加像是`inline-block`（行内块级元素，设置成这个属性的元素会同时拥有行内和块级的特性，最明显的不同是它的默认宽度不是`100%`），这时候给行内元素设置`padding-top`和`padding-bottom`或者`width`、`height`都是有效果的

#### ::before 和 :after中双冒号和单冒号 有什么区别？解释一下这2个伪元素的作用

- 单冒号(`:`)用于`CSS3`伪类，双冒号(`::`)用于`CSS3`伪元素
- 用于区分伪类和伪元素

#### 如果需要手动写动画，你认为最小时间间隔是多久，为什么？（阿里）

- 多数显示器默认频率是`60Hz`，即`1`秒刷新`60`次，所以理论上最小间隔为`1/60*1000ms ＝ 16.7ms`

#### CSS不同选择器的权重(CSS层叠的规则)

- `！important`规则最重要，大于其它规则
- 行内样式规则，加`1000`
- 对于选择器中给定的各个`ID`属性值，加`100`
- 对于选择器中给定的各个类属性、属性选择器或者伪类选择器，加`10`
- 对于选择其中给定的各个元素标签选择器，加1
- 如果权值一样，则按照样式规则的先后顺序来应用，顺序靠后的覆盖靠前的规则

> 以下是权重的规则：标签的权重为1，class的权重为10，id的权重为100，以下/// 例子是演示各种定义的权重值：

```css
/*权重为1*/
div{
}
/*权重为10*/
.class1{
}
/*权重为100*/
#id1{
}
/*权重为100+1=101*/
#id1 div{
}
/*权重为10+1=11*/
.class1 div{
}
/*权重为10+10+1=21*/
.class1 .class2 div{
}
```

> 如果权重相同，则最后定义的样式会起作用，但是应该避免这种情况出现

#### CSS3动画（简单动画的实现，如旋转等）

- 依靠`CSS3`中提出的三个属性：`transition`、`transform`、`animation`
- `transition`：定义了元素在变化过程中是怎么样的，包含`transition-property`、`transition-duration`、`transition-timing-function`、`transition-delay`。
- `transform`：定义元素的变化结果，包含`rotate`、`scale`、`skew`、`translate`。
- `animation`：动画定义了动作的每一帧（`@keyframes`）有什么效果，包括`animation-name`，`animation-duration`、`animation-timing-function`、`animation-delay`、`animation-iteration-count`、`animation-direction`

####  base64的原理及优缺点

- 优点可以加密，减少了`HTTTP`请求
- 缺点是需要消耗`CPU`进行编解码

#### postcss的作用

- 可以直观的理解为：它就是一个平台。为什么说它是一个平台呢？因为我们直接用它，感觉不能干什么事情，但是如果让一些插件在它上面跑，那么将会很强大
- `PostCSS` 提供了一个解析器，它能够将 `CSS` 解析成抽象语法树
- 通过在 `PostCSS` 这个平台上，我们能够开发一些插件，来处理我们的`CSS`，比如热门的：`autoprefixer`
- `postcss`可以对sass处理过后的`css`再处理 最常见的就是`autoprefixer`

#### 什么是外边距重叠？重叠的结果是什么？

> 外边距重叠就是margin-collapse

- 在CSS当中，相邻的两个盒子（可能是兄弟关系也可能是祖先关系）的外边距可以结合成一个单独的外边距。这种合并外边距的方式被称为折叠，并且因而所结合成的外边距称为折叠外边距。

**折叠结果遵循下列计算规则**：

- 两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值。
- 两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值。
- 两个外边距一正一负时，折叠结果是两者的相加的和。

#### rgba()和opacity的透明效果有什么不同？

- `rgba()`和`opacity`都能实现透明效果，但最大的不同是`opacity（继承属性）`作用于元素，以及元素内的所有内容的透明度，
- 而`rgba()`只作用于元素的颜色或其背景色。（设置`rgba`透明的元素的子元素不会继承透明效果！）

#### px和em的区别

- `px`和`em`都是长度单位，区别是，`px`的值是固定的，指定是多少就是多少，计算比较容易。`em`得值不是固定的，并且`em`会继承父级元素的字体大小。
- 浏览器的默认字体高都是`16px`。所以未经调整的浏览器都符合: `1em=16px`。那么`12px=0.75em`, `10px=0.625em`。

> - px 相对于显示器屏幕分辨率，无法用浏览器字体放大功能
> - em 值并不是固定的，会继承父级的字体大小： em = 像素值 / 父级font-size

#### sass、less是什么-大家为什么要使用他们 Sass、LESS是什么？大家为什么要使用他们？

- 他们是`CSS`预处理器。他是`CSS`上的一种抽象层。他们是一种特殊的语法/语言编译成`CSS`。
- 例如Less是一种动态样式语言. 将CSS赋予了动态语言的特性，如变量，继承，运算， 函数. `LESS` 既可以在客户端上运行 (支持`IE 6+`, `Webkit`, `Firefox`)，也可一在服务端运行 (借助 `Node.js`)

**为什么要使用它们？**

- 结构清晰，便于扩展。
- 可以方便地屏蔽浏览器私有语法差异。这个不用多说，封装对- 浏览器语法差异的重复处理，减少无意义的机械劳动。
- 可以轻松实现多重继承。
- 完全兼容 CSS 代码，可以方便地应用到老项目中。LESS 只- 是在 CSS 语法上做了扩展，所以老的 CSS 代码也可以与 LESS 代码一同编译

#### 知道css有个content属性吗？有什么作用？有什么应用？

> css的`content`属性专门应用在 `before/after`伪元素上，用于来插入生成内容。最常见的应用是利用伪类清除浮动。

```css
/**一种常见利用伪类清除浮动的代码**/
.clearfix:after {
    content:".";       //这里利用到了content属性
    display:block;
    height:0;
    visibility:hidden;
    clear:both; 
 }
.clearfix {
    *zoom:1;
}
```

#### 如何使用CSS实现硬件加速？

> 硬件加速是指通过创建独立的复合图层，让GPU来渲染这个图层，从而提高性能，

- 一般触发硬件加速的`CSS`属性有`transform`、`opacity`、`filter`，为了避免2D动画在 开始和结束的时候的`repaint`操作，一般使用`transform:translateZ(0)`

#### 说一说css3的animation

- css3的`animation`是css3新增的动画属性，这个css3动画的每一帧是通过`@keyframes`来声明的，`keyframes`声明了动画的名称，通过`from`、`to`或者是百分比来定义
- 每一帧动画元素的状态，通过`animation-name`来引用这个动画，同时css3动画也可以定义动画运行的时长、动画开始时间、动画播放方向、动画循环次数、动画播放的方式，
- 这些相关的动画子属性有：`animation-name`定义动画名、`animation-duration`定义动画播放的时长、`animation-delay`定义动画延迟播放的时间、`animation-direction`定义 动画的播放方向、`animation-iteration-count`定义播放次数、`animation-fill-mode`定义动画播放之后的状态、`animation-play-state`定义播放状态，如暂停运行等、`animation-timing-function`
- 定义播放的方式，如恒速播放、艰涩播放等。

#### 左边宽度固定，右边自适应

> 左侧固定宽度，右侧自适应宽度的两列布局实现

html结构

```html
<div class="outer">
    <div class="left">固定宽度</div>
    <div class="right">自适应宽度</div>
</div>
```

> 在外层`div`（类名为`outer`）的`div`中，有两个子`div`，类名分别为`left`和`right`，其中`left`为固定宽度，而`right`为自适应宽度

**方法1：左侧div设置成浮动：float: left，右侧div宽度会自拉升适应**

```css
.outer {
    width: 100%;
    height: 500px;
    background-color: yellow;
}
.left {
    width: 200px;
    height: 200px;
    background-color: red;
    float: left;
}
.right {
    height: 200px;
    background-color: blue;
}
```

**方法2：对右侧:div进行绝对定位，然后再设置right=0，即可以实现宽度自适应**

> 绝对定位元素的第一个高级特性就是其具有自动伸缩的功能，当我们将 `width`设置为 `auto` 的时候（或者不设置，默认为 `auto` ），绝对定位元素会根据其 `left` 和 `right` 自动伸缩其大小

```css
.outer {
    width: 100%;
    height: 500px;
    background-color: yellow;
    position: relative;
}
.left {
    width: 200px;
    height: 200px;
    background-color: red;
}
.right {
    height: 200px;
    background-color: blue;
    position: absolute;
    left: 200px;
    top:0;          
    right: 0;
}
```

**方法3：将左侧`div`进行绝对定位，然后右侧`div`设置`margin-left: 200px`**

```css
.outer {
    width: 100%;
    height: 500px;
    background-color: yellow;
    position: relative;
}
.left {
    width: 200px;
    height: 200px;
    background-color: red;
    position: absolute;
}
.right {
    height: 200px;
    background-color: blue;
    margin-left: 200px;
}
```

**方法4：使用flex布局**

```css
.outer {
    width: 100%;
    height: 500px;
    background-color: yellow;
    display: flex;
    flex-direction: row;
}
.left {
    width: 200px;
    height: 200px;
    background-color: red;
}
.right {
    height: 200px;
    background-color: blue;
    flex: 1;
}
```

#### 两种以上方式实现已知或者未知宽度的垂直水平居中

```css
/** 1 **/
.wraper {
  position: relative;
  .box {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 100px;
    height: 100px;
    margin: -50px 0 0 -50px;
  }
}

/** 2 **/
.wraper {
  position: relative;
  .box {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
}

/** 3 **/
.wraper {
  .box {
    display: flex;
    justify-content:center;
    align-items: center;
    height: 100px;
  }
}

/** 4 **/
.wraper {
  display: table;
  .box {
    display: table-cell;
    vertical-align: middle;
  }
}
```

#### 如何实现小于12px的字体效果

> `transform:scale()`这个属性只可以缩放可以定义宽高的元素，而行内元素是没有宽高的，我们可以加上一个`display:inline-block`;

```css
transform: scale(0.7);
```

#### CSS选择符有哪些？哪些属性可以继承

- id选择器（ `# myid`）
- 类选择器（`.myclassname`）
- 标签选择器（`div`, `h1`, `p`）
- 相邻选择器（`h1 + p`） 后面第一个相邻
- 子选择器（`ul > li`）
- 后代选择器（`li a`）
- 通配符选择器（ `*` ）
- 属性选择器（`a[rel = "external"]`）
- 伪类选择器（`a:hover, li:nth-child`）

**CSS哪些属性可以继承？哪些属性不可以继承**

- 可继承的样式： `font-size font-family color, UL LI DL DD DT`
- 不可继承的样式：`border padding margin width height`

#### 用纯CSS创建一个三角形的原理是什么

```css
/* 把上、左、右三条边隐藏掉（颜色设为 transparent） */
#demo {
  width: 0;
  height: 0;
  border-width: 20px;
  border-style: solid;
  border-color: transparent transparent red transparent;
}
```

#### 请列举几种隐藏元素的方法

- `visibility: hidden;` 这个属性只是简单的隐藏某个元素，但是元素占用的空间任然存在
- `opacity: 0;` `CSS3`属性，设置`0`可以使一个元素完全透明
- `position: absolute;` 设置一个很大的 `left` 负值定位，使元素定位在可见区域之外
- `display: none;` 元素会变得不可见，并且不会再占用文档的空间。
- `transform: scale(0);` 将一个元素设置为缩放无限小，元素将不可见，元素原来所在的位置将被保留
- `<div hidden="hidden">` HTML5属性,效果和`display:none;`相同，但这个属性用于记录一个元素的状态
- `height: 0;` 将元素高度设为 `0` ，并消除边框
- `filter: blur(0);` CSS3属性，将一个元素的模糊度设置为`0`，从而使这个元素“消失”在页面中

#### 经常遇到的浏览器的JS兼容性有哪些？解决方法是什么

- 当前样式：`getComputedStyle(el, null) VS el.currentStyle`
- 事件对象：`e VS window.event`
- 鼠标坐标：`e.pageX, e.pageY VS window.event.x, window.event.y`
- 按键码：`e.which VS event.keyCode`
- 文本节点：`el.textContent VS el.innerText`

#### 浏览器是怎样解析CSS选择器的

- 浏览器解析 CSS 选择器的方式是从右到左

#### 抽离样式模块怎么写，说出思路

- CSS可以拆分成2部分：公共CSS 和 业务CSS：
  - 网站的配色，字体，交互提取出为公共CSS。这部分CSS命名不应涉及具体的业务
  - 对于业务CSS，需要有统一的命名，使用公用的前缀。可以参考面向对象的CSS

#### 元素竖向的百分比设定是相对于容器的高度吗

> 元素竖向的百分比设定是相对于**容器的宽度**，而不是高度

#### 什么是响应式设计？响应式设计的基本原理是什么？如何兼容低版本的IE

- 响应式设计就是网站能够兼容多个终端，而不是为每个终端做一个特定的版本
- 基本原理是利用CSS3媒体查询，为不同尺寸的设备适配不同样式
- 对于低版本的IE，可采用JS获取屏幕宽度，然后通过resize方法来实现兼容：

```js
$(window).resize(function () {
  screenRespond();
});
screenRespond();
function screenRespond(){
var screenWidth = $(window).width();
if(screenWidth <= 1800){
  $("body").attr("class", "w1800");
}
if(screenWidth <= 1400){
  $("body").attr("class", "w1400");
}
if(screenWidth > 1800){
  $("body").attr("class", "");
}
}
```

#### a标签上四个伪类的执行顺序是怎么样的

> ```
> link > visited > hover > active
> ```

- `L-V-H-A` `love hate` 用喜欢和讨厌两个词来方便记忆

#### 伪元素和伪类的区别和作用

- 伪元素 -- 在内容元素的前后插入额外的元素或样式，但是这些元素实际上并不在文档中生成。
- 它们只在外部显示可见，但不会在文档的源代码中找到它们，因此，称为“伪”元素。例如：

```css
p::before {content:"第一章：";}
p::after {content:"Hot!";}
p::first-line {background:red;}
p::first-letter {font-size:30px;}
```

- 伪类 -- 将特殊的效果添加到特定选择器上。它是已有元素上添加类别的，不会产生新的元素。例如：

```css
a:hover {color: #FF00FF}
p:first-child {color: red}
```