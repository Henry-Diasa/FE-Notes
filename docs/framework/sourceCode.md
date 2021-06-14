### 了解Vue项目

#### 目录及文件

[TOC]



> 首先去vue的github上dev分支下载完整版的代码，项目目录如下

```
├── scripts ------------------------------- 构建相关的文件，一般情况下我们不需要动
│   ├── git-hooks ------------------------- 存放git钩子的目录
│   ├── alias.js -------------------------- 别名配置
│   ├── config.js ------------------------- 生成rollup配置的文件
│   ├── build.js -------------------------- 对 config.js 中所有的rollup配置进行构建
│   ├── ci.sh ----------------------------- 持续集成运行的脚本
│   ├── release.sh ------------------------ 用于自动发布新版本的脚本
├── dist ---------------------------------- 构建后文件的输出目录
├── examples ------------------------------ 存放一些使用Vue开发的应用案例
├── flow ---------------------------------- 类型声明，使用开源项目 [Flow](https://flowtype.org/)
├── packages ------------------------------ 存放独立发布的包的目录
├── test ---------------------------------- 包含所有测试文件
├── src ----------------------------------- 这个是我们最应该关注的目录，包含了源码
│   ├── compiler -------------------------- 编译器代码的存放目录，将 template 编译为 render 函数
│   ├── core ------------------------------ 存放通用的，与平台无关的代码
│   │   ├── observer ---------------------- 响应系统，包含数据观测的核心代码
│   │   ├── vdom -------------------------- 包含虚拟DOM创建(creation)和打补丁(patching)的代码
│   │   ├── instance ---------------------- 包含Vue构造函数设计相关的代码
│   │   ├── global-api -------------------- 包含给Vue构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── components -------------------- 包含抽象出来的通用组件
│   ├── server ---------------------------- 包含服务端渲染(server-side rendering)的相关代码
│   ├── platforms ------------------------- 包含平台特有的相关代码，不同平台的不同构建的入口文件也在这里
│   │   ├── web --------------------------- web平台
│   │   │   ├── entry-runtime.js ---------- 运行时构建的入口，不包含模板(template)到render函数的编译器，所以不支持 `template` 选项，我们使用vue默认导出的就是这个运行时的版本。大家使用的时候要注意
│   │   │   ├── entry-runtime-with-compiler.js -- 独立构建版本的入口，它在 entry-runtime 的基础上添加了模板(template)到render函数的编译器
│   │   │   ├── entry-compiler.js --------- vue-template-compiler 包的入口文件
│   │   │   ├── entry-server-renderer.js -- vue-server-renderer 包的入口文件
│   │   │   ├── entry-server-basic-renderer.js -- 输出 packages/vue-server-renderer/basic.js 文件
│   │   ├── weex -------------------------- 混合应用
│   ├── sfc ------------------------------- 包含单文件组件(.vue文件)的解析逻辑，用于vue-template-compiler包
│   ├── shared ---------------------------- 包含整个代码库通用的代码
├── package.json -------------------------- 不解释
├── yarn.lock ----------------------------- yarn 锁定文件
├── .editorconfig ------------------------- 针对编辑器的编码风格配置文件
├── .flowconfig --------------------------- flow 的配置文件
├── .babelrc ------------------------------ babel 配置文件
├── .eslintrc ----------------------------- eslint 配置文件
├── .eslintignore ------------------------- eslint 忽略配置
├── .gitignore ---------------------------- git 忽略配置
```

#### Vue的不同构建输出

**从构建配置了解不同的构建输出**

> vue按照输出的模块形式分类可分为三种：`UMD`、`CommonJS`和`ES Module`

```js
// scripts/config.js
const builds = [
  'web-runtime-cjs-dev': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.common.dev.js'),
    format: 'cjs',
    env: 'development',
    banner
  },
  'web-runtime-esm': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.esm.js'),
    format: 'es',
    banner
  }, 
  'web-runtime-dev': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.js'),
    format: 'umd',
    env: 'development',
    banner
  }, 
    
  ...  
]
```

可以看到不同的入口文件、不同的环境、不同的运行版本（运行时版和完整版）以及不同的format格式（cjs、es、umd）

运行时版和完整版的区别

- 运行时的入口文件名字为：`entry-runtime.js`
- 完整版的入口文件名字为：`entry-runtime-with-compiler.js`

完整版的多了一个compiler（将template编译成render函数）

**不同构建输出的区别与作用**

运行时版和完整版的区别就是：运行时版 + compiler = 完整版。compiler是用来编译为render函数的，并且不一定在代码运行时去做，构建的时候就可以完成。这样可以提升性能，同时compiler单独出来也减少了库的体积。

除了运行时版与完整版之外，为什么还要输出不同形式的模块的包？比如 `cjs`、`es` 和 `umd`？其中 `umd` 是使得你可以直接使用 `<script>` 标签引用Vue的模块形式。但我们使用 Vue 的时候更多的是结合构建工具，比如 `webpack` 之类的，而 `cjs` 形式的模块就是为 `browserify` 和 `webpack 1` 提供的，他们在加载模块的时候不能直接加载 `ES Module`。而 `webpack2+` 以及 `Rollup` 是可以直接加载 `ES Module` 的，所以就有了 `es` 形式的模块输出。

#### Package.json

```js
"main": "dist/vue.runtime.common.js",
"module": "dist/vue.runtime.esm.js",
```

`main` 和 `module` 指向的都是运行时版的Vue，不同的是：前者是 `cjs` 模块，后者是 `es` 模块。

其中 `main` 字段和 `module` 字段分别用于 `browserify 或 webpack 1` 和 `webpack2+ 或 Rollup`，后者可以直接加载 `ES Module` 且会根据 `module` 字段的配置进行加载，关于 `module` 可以参考这里：https://github.com/rollup/rollup/wiki/pkg.module。

然后我们看一下 `scripts` 字段如下，这里去掉了测试相关以及weex相关的脚本配置：

```js
"scripts": {
	  // 构建完整版 umd 模块的 Vue
    "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
    // 构建运行时 cjs 模块的 Vue
    "dev:cjs": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-cjs",
    // 构建运行时 es 模块的 Vue
    "dev:esm": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-esm",
    // 构建 web-server-renderer 包
    "dev:ssr": "rollup -w -c scripts/config.js --environment TARGET:web-server-renderer",
    // 构建 Compiler 包
    "dev:compiler": "rollup -w -c scripts/config.js --environment TARGET:web-compiler ",
    "build": "node scripts/build.js",
    "build:ssr": "npm run build -- vue.runtime.common.js,vue-server-renderer",
    "lint": "eslint src build test",
    "flow": "flow check",
    "release": "bash scripts/release.sh",
    "release:note": "node scripts/gen-release-note.js",
    "commit": "git-cz"
  },
```

### Vue构造函数

> 我们在使用vue的时候是通过new操作符进行调用，首先需要弄清楚Vue构造函数

#### Vue构造函数的原型

以`npm run dev`为切入点

```js
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
```

根据`scripts/config.js`中的配置

```js
  // Runtime+compiler development build (Browser)
  'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  }
```

`web/entry-runtime-with-compiler.js`中的web指的是哪个目录？这其实是一个别名

```js
// scripts/alias.js
const path = require('path')

const resolve = p => path.resolve(__dirname, '../', p)

module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  entries: resolve('src/entries'),
  sfc: resolve('src/sfc')
}
```

打开 `src/platforms/web/entry-runtime-with-compiler.js` 文件

```js
import Vue from './runtime/index'
```

于是我们就打开当前目录的 `runtime` 目录下的 `index.js` 

```js
import Vue from 'core/index'
```

在 `scripts/alias.js` 的配置中，`core` 指向的是 `src/core`，打开 `src/core/index.js` 你能看到这样一句：

```js
import Vue from './instance/index'
```

继续打开 `./instance/index.js` 文件

```js
// 从五个文件导入五个方法（不包括 warn）
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

// 定义 Vue 构造函数
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

// 将 Vue 作为参数传递给导入的五个方法
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

// 导出 Vue
export default Vue
```

一共调用了五个方法，分别看一下

- initMixin

  ```js
  export function initMixin (Vue: Class<Component>) {
    Vue.prototype._init = function (options?: Object) {
      // ... _init 方法的函数体，此处省略
    }
  }
  ```

  在vue的原型上添加一个`_init`方法

- stateMixin

  ```js
  const dataDef = {}
    dataDef.get = function () { return this._data }
    const propsDef = {}
    propsDef.get = function () { return this._props }
    if (process.env.NODE_ENV !== 'production') {
      dataDef.set = function (newData: Object) {
        warn(
          'Avoid replacing instance root $data. ' +
          'Use nested data properties instead.',
          this
        )
      }
      propsDef.set = function () {
        warn(`$props is readonly.`, this)
      }
    }
    Object.defineProperty(Vue.prototype, '$data', dataDef)
    Object.defineProperty(Vue.prototype, '$props', propsDef)
  ```

  在vue的原型上添加 `$data`和`$props` 两个只读的属性，并且其分别代理到`_data`和`_props`

  接下来又定义了三个方法 `$set`  `$delete` ` $watch`
  
  ```js
  Vue.prototype.$set = set
    Vue.prototype.$delete = del
  
    Vue.prototype.$watch = function (
      expOrFn: string | Function,
      cb: any,
      options?: Object
    ): Function {
    	// ...
    }
  ```
  
- lifecycleMixin

  ```js
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
  Vue.prototype.$forceUpdate = function () {}
  Vue.prototype.$destroy = function () {}
  ```

  同样是添加了一些方法

- eventsMixin

  ```js
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
  Vue.prototype.$once = function (event: string, fn: Function): Component {}
  Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
  Vue.prototype.$emit = function (event: string): Component {}
  ```

  定义一些发布订阅的一些事件方法

- renderMixin

  renderMixin` 方法在执行完 `installRenderHelpers` 函数之后，又在 `Vue.prototype` 上添加了两个方法，分别是：`$nextTick` 和 `_render

  ```js
  // installRenderHelpers 函数中
  Vue.prototype._o = markOnce
  Vue.prototype._n = toNumber
  Vue.prototype._s = toString
  Vue.prototype._l = renderList
  Vue.prototype._t = renderSlot
  Vue.prototype._q = looseEqual
  Vue.prototype._i = looseIndexOf
  Vue.prototype._m = renderStatic
  Vue.prototype._f = resolveFilter
  Vue.prototype._k = checkKeyCodes
  Vue.prototype._b = bindObjectProps
  Vue.prototype._v = createTextVNode
  Vue.prototype._e = createEmptyVNode
  Vue.prototype._u = resolveScopedSlots
  Vue.prototype._g = bindObjectListeners
  
  Vue.prototype.$nextTick = function (fn: Function) {}
  Vue.prototype._render = function (): VNode {}
  ```

#### 构造函数的静态属性和方法（全局API）

路径回溯到`core/index.js`文件

```js
// 从 Vue 的出生文件导入 Vue
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

// 将 Vue 构造函数作为参数，传递给 initGlobalAPI 方法，该方法来自 ./global-api/index.js 文件
initGlobalAPI(Vue)

// 在 Vue.prototype 上添加 $isServer 属性，该属性代理了来自 core/util/env.js 文件的 isServerRendering 方法
Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

// 在 Vue.prototype 上添加 $ssrContext 属性
Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

// Vue.version 存储了当前 Vue 的版本号
Vue.version = '__VERSION__'

// 导出 Vue
export default Vue


```



在`Vue.prototype`上添加了两个只读的属性，分别是：`$isServer` 和 `$ssrContext`。接着又在 `Vue` 构造函数上定义了 `FunctionalRenderContext` 静态属性

接着在Vue上添加version版本号（`__VERSION__`是在`scripts/config.js`中配置的）

在来看这句代码

```js
initGlobalAPI(Vue)
```

```js
 // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)
```

在Vue上添加`config`属性，同样是只读的。最终所以 `Vue.config` 代理的是从 `core/config.js` 文件导出的对象

接着是这样一段代码：

```js
// exposed util methods.
// NOTE: these are not considered part of the public API - avoid relying on
// them unless you are aware of the risk.
Vue.util = {
	warn,
	extend,
	mergeOptions,
	defineReactive
}
```

Vue上添加util属性，拥有多个属性。

然后是这样一段代码：

```js
Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick

Vue.options = Object.create(null)
```

这段代码比较简单，在 `Vue` 上添加了四个属性分别是 `set`、`delete`、`nextTick` 以及 `options`，这里要注意的是 `Vue.options`，现在它还只是一个空的对象，通过 `Object.create(null)` 创建。

接下来

```js
ASSET_TYPES.forEach(type => {
	Vue.options[type + 's'] = Object.create(null)
})

// this is used to identify the "base" constructor to extend all plain-object
// components with in Weex's multi-instance scenarios.
Vue.options._base = Vue

extend(Vue.options.components, builtInComponents)
```

经过上面的转换之后Vue.options变成这样

```js
Vue.options = {
	components: Object.create(null),
	directives: Object.create(null),
	filters: Object.create(null),
	_base: Vue
}
```

紧接着，是这句代码：

```js
extend(Vue.options.components, builtInComponents)
```

`extend` 来自于 `shared/util.js` 文件,最终经过混合`Vue.options.components` 的值如下：

```js
Vue.options.components = {
	KeepAlive
}
```

```js
Vue.options = {
	components: {
		KeepAlive
	},
	directives: Object.create(null),
	filters: Object.create(null),
	_base: Vue
}
```

接下来是以vue为参数的init*方法

```js
initUse(Vue)
initMixin(Vue)
initExtend(Vue)
initAssetRegisters(Vue)
```

- initUse

  ```js
  /* @flow */
  
  import { toArray } from '../util/index'
  
  export function initUse (Vue: GlobalAPI) {
    Vue.use = function (plugin: Function | Object) {
      // ...
    }
  }
  ```

  在Vue上添加use方法

- initMixin

  ```js
  /* @flow */
  
  import { mergeOptions } from '../util/index'
  
  export function initMixin (Vue: GlobalAPI) {
    Vue.mixin = function (mixin: Object) {
      this.options = mergeOptions(this.options, mixin)
      return this
    }
  }
  ```

  在Vue上添加mixIn方法

- initExtend

  ```js
  export function initExtend (Vue: GlobalAPI) {
    /**
     * Each instance constructor, including Vue, has a unique
     * cid. This enables us to create wrapped "child
     * constructors" for prototypal inheritance and cache them.
     */
    Vue.cid = 0
    let cid = 1
  
    /**
     * Class inheritance
     */
    Vue.extend = function (extendOptions: Object): Function {
      // ...
    }
  }
  ```

  `initExtend` 方法在 `Vue` 上添加了 `Vue.cid` 静态属性，和 `Vue.extend` 静态方法

- initAssetRegisters

  ```js
  export function initAssetRegisters (Vue: GlobalAPI) {
    /**
     * Create asset registration methods.
     */
    ASSET_TYPES.forEach(type => {
      Vue[type] = function (
        id: string,
        definition: Function | Object
      ): Function | Object | void {
        // ......
      }
    })
  }
  ```

  最终vue又多了三个静态方法

  ```js
  Vue.component
  Vue.directive
  Vue.filter
  ```

#### Vue平台化的包装

> `Vue` 是一个 `Multi-platform` 的项目（web和weex），不同平台可能会内置不同的组件、指令，或者一些平台特有的功能等等，那么这就需要对 `Vue` 根据不同的平台进行平台化地包装，这就是接下来我们要看的文件，这个文件也出现在我们寻找 `Vue` 构造函数的路线上，它就是：`platforms/web/runtime/index.js` 文件

可以看到platforms目录有两个子目录`web`和`weex`。接下来看一下`platforms/web/runtime/index.js `文件

首先是下面这些

```js
// install platform specific utils
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement
```

`Vue.config`代理的值是`core/config.js`

```js
Vue.config = {
  optionMergeStrategies: Object.create(null),
  silent: false,
  productionTip: process.env.NODE_ENV !== 'production',
  devtools: process.env.NODE_ENV !== 'production',
  performance: false,
  errorHandler: null,
  warnHandler: null,
  ignoredElements: [],
  keyCodes: Object.create(null),
  isReservedTag: no,
  isReservedAttr: no,
  isUnknownElement: no,
  getTagNamespace: noop,
  parsePlatformTagName: identity,
  mustUseProp: no,
  _lifecycleHooks: LIFECYCLE_HOOKS
}
```

上面这些是在覆盖默认导出的`config` 对象的属性。

接下来两句代码

```js
// install platform runtime directives & components
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)
```

经过`extend`之后。`Vue.options`变为

```js
Vue.options = {
	components: {
		KeepAlive,
		Transition,
		TransitionGroup
	},
	directives: {
		model,
		show
	},
	filters: Object.create(null),
	_base: Vue
}
```

我们继续往下看代码，接下来是这段：

```js
// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

在原型上添加了`__patch__`和`$mount`

现在我们就看完了 `platforms/web/runtime/index.js` 文件，该文件的作用是对 `Vue` 进行平台化地包装：

- 设置平台化的 `Vue.config`。
- 在 `Vue.options` 上混合了两个指令(`directives`)，分别是 `model` 和 `show`。
- 在 `Vue.options` 上混合了两个组件(`components`)，分别是 `Transition` 和 `TransitionGroup`。
- 在 `Vue.prototype` 上添加了两个方法：`__patch__` 和 `$mount`。

#### With compiler

接下来就是带compiler的，打开 `entry-runtime-with-compiler.js` 文件

```js
// ... 其他 import 语句

// 导入 运行时 的 Vue
import Vue from './runtime/index'

// ... 其他 import 语句

// 从 ./compiler/index.js 文件导入 compileToFunctions
import { compileToFunctions } from './compiler/index'

// 根据 id 获取元素的 innerHTML
const idToTemplate = cached(id => {
  const el = query(id)
  return el && el.innerHTML
})

// 使用 mount 变量缓存 Vue.prototype.$mount 方法
const mount = Vue.prototype.$mount
// 重写 Vue.prototype.$mount 方法
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // ... 函数体省略
}

/**
 * 获取元素的 outerHTML
 */
function getOuterHTML (el: Element): string {
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    const container = document.createElement('div')
    container.appendChild(el.cloneNode(true))
    return container.innerHTML
  }
}

// 在 Vue 上添加一个全局API `Vue.compile` 其值为上面导入进来的 compileToFunctions
Vue.compile = compileToFunctions

// 导出 Vue
export default Vue
```

重写了`Vue.prototype.$mount`方法，第二个是添加`Vue.compile`全局API

### 以一个例子为线索

我们有如下模板：

```html
<div id="app">{{test}}</div>
```

和这样一段 `js` 代码：

```js
var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    }
})
```

这段 `js` 代码很简单，只是简单地调用了 `Vue`，传递了两个选项 `el` 以及 `data`。这段代码的最终效果就是在页面中渲染为如下 `DOM`：

```html
<div id="app">1</div>
```

其中 `{{ test }}` 被替换成了 `1`，并且当我们尝试修改 `data.test` 的值的时候

```js
vm.$data.test = 2
// 或
vm.test = 2
```

那么页面的 `DOM` 也会随之变化为：

```html
<div id="app">2</div>
```

首先我们`new Vue`的时候会调用`_init`方法,这个`options`就是我们透传过来的

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

```js
options = {
    el: '#app',
    data: {
        test: 1
    }
}
```

`_init`的开始首先这这么一段

```js
// 在Vue实例中 添加uid属性
const vm: Component = this
// a uid
vm._uid = uid++
```

接下来

```js
let startTag, endTag
/* istanbul ignore if */
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    startTag = `vue-perf-start:${vm._uid}`
    endTag = `vue-perf-end:${vm._uid}`
    mark(startTag)
}

// 中间的代码省略...

/* istanbul ignore if */
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    vm._name = formatComponentName(vm, false)
    mark(endTag)
    measure(`vue ${vm._name} init`, startTag, endTag)
}
```

这段代码主要是性能追踪的。`Vue` 提供了全局配置 `Vue.config.performance`，我们通过将其设置为 `true`，即可开启性能追踪，你可以追踪四个场景的性能

- 1、组件初始化(`component init`)
- 2、编译(`compile`)，将模板(`template`)编译成渲染函数
- 3、渲染(`render`)，其实就是渲染函数的性能，或者说渲染函数执行且生成虚拟DOM(`vnode`)的性能
- 4、打补丁(`patch`)，将虚拟DOM渲染为真实DOM的性能

其中*组件初始化*的性能追踪就是我们在 `_init` 方法中看到的那样去实现的，其实现的方式就是在初始化的代码的开头和结尾分别使用 `mark` 函数打上两个标记，然后通过 `measure` 函数对这两个标记点进行性能计算。`mark` 和 `measure` 这两个函数可以在`core/utils` 

接下来看被追踪性能的代码

```js
// a flag to avoid this being observed
vm._isVue = true
// merge options
if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
} else {
    vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
    )
}
/* istanbul ignore else */
if (process.env.NODE_ENV !== 'production') {
    initProxy(vm)
} else {
    vm._renderProxy = vm
}
// expose real self
vm._self = vm
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

首先是`Vue`实例上添加`_isVue`属性，并设置为`true`，那么就代表该对象是 `Vue` 实例。这样可以避免该对象被响应系统观测（其实在其他地方也有用到，但是宗旨都是一样的，这个属性就是用来告诉你：我不是普通的对象，我是Vue实例）

接下来是下面的代码

```js
// merge options
if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
} else {
    vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
    )
}
```

由于我们例子中的options没有`_isComponent`,所以直接进入else分支，这段代码在 `Vue` 实例上添加了 `$options` 属性，在 `Vue` 的官方文档中，你能够查看到 `$options` 属性的作用，这个属性用于当前 `Vue` 的初始化，什么意思呢？大家要注意我们现在的阶段处于 `_init()` 方法中，在 `_init()` 方法的内部大家可以看到一系列 `init*` 的方法，比如

```js
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

这些初始化方法都用到了`vm.$options`,那么这个`options`的产生就要依靠`mergeOptions`了。

### Vue选项的规范化

#### mergeOptions函数的三个参数

```js
vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
)
```

第一个参数是resolveConstructorOptions的返回值，也就是解析构造者的options

接下来我们就具体看一下它是怎么做的，首先第一句：

```js
let options = Ctor.options
```

其中 `Ctor` 即传递进来的参数 `vm.constructor`，在我们的例子中他就是 `Vue` 构造函数，可能有的同学会问：难道它还有不是 `Vue` 构造函数的时候吗？当然，当你使用 `Vue.extend` 创造一个子类并使用子类创造实例时，那么 `vm.constructor` 就不是 `Vue` 构造函数，而是子类，比如：

```js
const Sub = Vue.extend()
const s = new Sub()
```

那么 `s.constructor` 自然就是 `Sub` 而非 `Vue`，大家知道这一点即可，但在我们的例子中，这里的 `Ctor` 就是 `Vue` 构造函数，而有关于 `Vue.extend` 的东西，我们后面会专门讨论的。

所以，`Ctor.options` 就是 `Vue.options`，然后我们再看 `resolveConstructorOptions` 的返回值是什么？如下：

```js
return options
```

也就是把 `Vue.options` 返回回去了，所以这个函数的确就像他的名字那样，是用来获取构造者的 `options` 的。不过同学们可能注意到了，`resolveConstructorOptions` 函数的第一句和最后一句代码中间还有一坨包裹在 `if` 语句块中的代码，那么这坨代码是干什么的呢？

我可以很明确地告诉大家，这里水稍微有那么点深，比如 `if` 语句的判断条件 `Ctor.super`，`super` 这是子类才有的属性，如下：

```js
const Sub = Vue.extend()
console.log(Sub.super)  // Vue
```

也就是说，`super` 这个属性是与 `Vue.extend` 有关系的，事实也的确如此。除此之外判断分支内的第一句代码：

```js
const superOptions = resolveConstructorOptions(Ctor.super)
```

我们发现，又递归地调用了 `resolveConstructorOptions` 函数，只不过此时的参数是构造者的父类，之后的代码中，还有一些关于父类的 `options` 属性是否被改变过的判断和操作，并且大家注意这句代码：

```js
// check if there are any late-modified/attached options (#4976)
const modifiedOptions = resolveModifiedOptions(Ctor)
```

我们要注意的是注释，有兴趣的同学可以根据注释中括号内的 `issue` 索引去搜一下相关的问题，这句代码是用来解决使用 `vue-hot-reload-api` 或者 `vue-loader` 时产生的一个 `bug` 的。

现在大家知道这里的水有多深了吗？关于这些问题，我们在讲 `Vue.extend` 时都会给大家一一解答，不过有一个因素从来没有变，那就是 `resolveConstructorOptions` 这个函数的作用永远都是用来获取当前实例构造者的 `options` 属性的，即使 `if` 判断分支内也不例外，因为 `if` 分支只不过是处理了 `options`，最终返回的永远都是 `options`。

所以根据我们的例子，`resolveConstructorOptions` 函数目前并不会走 `if` 判断分支，即此时这个函数相当于：

```js
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  return options
}
```

所以，根据我们的例子，此时的 `mergeOptions` 函数的第一个参数就是 `Vue.options`，那么大家还记得 `Vue.options` 长成什么样子吗？

通过查看我们可知 `Vue.options` 如下：

```js
Vue.options = {
	components: {
		KeepAlive
		Transition,
    	TransitionGroup
	},
	directives:{
	    model,
        show
	},
	filters: Object.create(null),
	_base: Vue
}
```

第二个参数就是我们`new Vue`时传递进来的参数

最终传递进`mergeOptions`里面的参数就变成下面的样子

```js
vm.$options = mergeOptions(
  // resolveConstructorOptions(vm.constructor)
  {
    components: {
      KeepAlive
      Transition,
      TransitionGroup
    },
    directives:{
      model,
      show
    },
    filters: Object.create(null),
    _base: Vue
  },
  // options || {}
  {
    el: '#app',
    data: {
      test: 1
    }
  },
  vm
)
```

接下来就是`mergeOptions`具体做的事情

#### 检查组件名称是否符合要求

先看`mergeOptions`方法，通过注释可以知道这个函数在实例化和继承的时候都有用到，这里要注意两点：第一，这个函数将会产生一个新的对象；第二，这个函数不仅仅在实例化对象(即`_init`方法中)的时候用到，在继承(`Vue.extend`)中也有用到，所以这个函数应该是一个用来合并两个选项对象为一个新对象的通用程序

在非生产环境下，会以 `child` 为参数调用 `checkComponents` 方法，我们看看 `checkComponents` 是做什么的，这个方法同样定义在 `core/util/options.js` 文件中，内容如下：

```js
/**
 * Validate component names
 */
function checkComponents (options: Object) {
  for (const key in options.components) {
    validateComponentName(key)
  }
}
```

```js
export function validateComponentName (name: string) {
  if (!/^[a-zA-Z][\w-]*$/.test(name)) {
    warn(
      'Invalid component name: "' + name + '". Component names ' +
      'can only contain alphanumeric characters and the hyphen, ' +
      'and must start with a letter.'
    )
  }
  if (isBuiltInTag(name) || config.isReservedTag(name)) {
    warn(
      'Do not use built-in or reserved HTML elements as component ' +
      'id: ' + name
    )
  }
}
```

这里判断名称是否合法有两点

- 第一点是通过一个正则来判断名字的合法性

- 第二点是通过`isBuiltInTag`和`isReservedTag`来判断

- `isBuiltInTag`是用来检测你所注册的组件是否是内置的标签，`slot` 和 `component` 这两个名字被 `Vue` 作为内置标签而存在的，你是不能够使用的，比如这样：

  ```js
  new Vue({
    components: {
      'slot': myComponent
    }
  })
  ```

- `config.isReservedTag` 在哪里被赋值的？前面我们讲到过在 `platforms/web/runtime/index.js` 文件中有这样一段代码：

  ```js
  // install platform specific utils
  Vue.config.mustUseProp = mustUseProp
  Vue.config.isReservedTag = isReservedTag
  Vue.config.isReservedAttr = isReservedAttr
  Vue.config.getTagNamespace = getTagNamespace
  Vue.config.isUnknownElement = isUnknownElement
  ```

  其中：

  ```js
  Vue.config.isReservedTag = isReservedTag
  ```

  就是在给 `config.isReservedTag` 赋值，其值为来自于 `platforms/web/util/element.js` 文件的 `isReservedTag` 函数，大家可以在附录 `platforms/web/util 目录下`中查看该方法的作用及实现，可知在 `Vue` 中 `html` 标签和部分 `SVG` 标签被认为是保留的。所以这段代码是在保证选项被合并前的合理合法

#### 允许合并另一个实例构造者的选项

我们继续看代码，接下来的一段代码同样是一个 `if` 语句块：

```js
if (typeof child === 'function') {
  child = child.options
}
```

这说明 `child` 参数除了是普通的选项对象外，还可以是一个函数，如果是函数的话就取该函数的 `options` 静态属性作为新的 `child`，我们想一想什么样的函数具有 `options` 静态属性呢？现在我们知道 `Vue` 构造函数本身就拥有这个属性，其实通过 `Vue.extend` 创造出来的子类也是拥有这个属性的。所以这就允许我们在进行选项合并的时候，去合并一个 `Vue` 实例构造者的选项了。

#### 规范化props(normalizeProps)

接着看代码，接下来是三个用来规范化选项的函数调用：

```js
normalizeProps(child, vm)
normalizeInject(child, vm)
normalizeDirectives(child)
```

这三个函数是用来规范选项的，什么意思呢？以 `props` 为例，我们知道在 `Vue` 中，我们在使用 `props` 的时候有两种写法，一种是使用字符串数组，如下：

```js
const ChildComponent = {
  props: ['someData']
}
```

另外一种是使用对象语法：

```js
const ChildComponent = {
  props: {
    someData: {
      type: Number,
      default: 0
    }
  }
}
```

其实不仅仅是 `props`，在 `Vue` 中拥有多种使用方法的选项有很多，这给开发者提供了非常灵活且便利的选择，但是对于 `Vue` 来讲，这并不是一件好事儿，因为 `Vue` 要对选项进行处理，这个时候好的做法就是，无论开发者使用哪一种写法，在内部都将其规范成同一种方式，这样在选项合并的时候就能够统一处理，这就是上面三个函数的作用。

现在我们就详细看看这三个规范化选项的函数都是怎么规范选项的，首先是 `normalizeProps` 函数，这看上去貌似是用来规范化 `props` 选项的，找到 `normalizeProps` 函数源码如下：

```js
/**
 * Ensure all props option syntax are normalized into the
 * Object-based format.
 */
function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  options.props = res
}
```

分三种情况最终把`options`转换为一个对象

- 数组：遍历数组，然后将中横线转换为驼峰（camelize）。最后返回的对象类似

  ```js
  props: {
    someData:{
      type: null
    }
  }
  ```

- 对象：遍历对象，如果val还是对象的话直接使用，否则把val的值当做type

  ```js
  // 转换前
  props: {
    someData1: Number,
    someData2: {
      type: String,
      default: ''
    }
  }
  // 转换后
  props: {
    someData1: {
      type: Number
    },
    someData2: {
      type: String,
      default: ''
    }
  }
  ```

#### 规范化inject(normalizeInject)

```js
/**
 * Normalize all injections into Object-based format
 */
function normalizeInject (options: Object, vm: ?Component) {
  const inject = options.inject
  if (!inject) return
  const normalized = options.inject = {}
  if (Array.isArray(inject)) {
    for (let i = 0; i < inject.length; i++) {
      normalized[inject[i]] = { from: inject[i] }
    }
  } else if (isPlainObject(inject)) {
    for (const key in inject) {
      const val = inject[key]
      normalized[key] = isPlainObject(val)
        ? extend({ from: key }, val)
        : { from: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "inject": expected an Array or an Object, ` +
      `but got ${toRawType(inject)}.`,
      vm
    )
  }
}
```

首先是赋值`inject`和相同的引用`normalized`

`inject`举例

```js
// 子组件
const ChildComponent = {
  template: '<div>child component</div>',
  created: function () {
    // 这里的 data 是父组件注入进来的
    console.log(this.data)
  },
  inject: ['data']
}

// 父组件
var vm = new Vue({
  el: '#app',
  // 向子组件提供数据
  provide: {
    data: 'test provide'
  },
  components: {
    ChildComponent
  }
})
```

```js
if (Array.isArray(inject)) {
  for (let i = 0; i < inject.length; i++) {
    normalized[inject[i]] = { from: inject[i] }
  }
} else if (isPlainObject(inject)) {
  ...
} else if (process.env.NODE_ENV !== 'production') {
  ...
}
```

首先是数组的处理，最终会变成下面这样

如果你的 `inject` 选项是这样写的：

```js
['data1', 'data2']
```

那么将被规范化为：

```js
{
  'data1': { from: 'data1' },
  'data2': { from: 'data2' }
}
```

然后是对象的处理

```js
if (Array.isArray(inject)) {
  ...
} else if (isPlainObject(inject)) {
  for (const key in inject) {
    const val = inject[key]
    normalized[key] = isPlainObject(val)
      ? extend({ from: key }, val)
      : { from: val }
  }
} else if (process.env.NODE_ENV !== 'production') {
  ...
}
```

开发者所写的对象可能是这样的：

```js
let data1 = 'data1'

// 这里为简写，这应该写在Vue的选项中
inject: {
  data1,
  d2: 'data2',
  data3: { someProperty: 'someValue' }
}
```

对于这种情况，我们将会把它规范化为：

```js
inject: {
  'data1': { from: 'data1' },
  'd2': { from: 'data2' },
  'data3': { from: 'data3', someProperty: 'someValue' }
}
```

#### 规范化directives(normalizeDirectives)

```js
/**
 * Normalize raw function directives into object format.
 */
function normalizeDirectives (options: Object) {
  const dirs = options.directives
  if (dirs) {
    for (const key in dirs) {
      const def = dirs[key]
      if (typeof def === 'function') {
        dirs[key] = { bind: def, update: def }
      }
    }
  }
}
```

这段代码是用来规范化`directives`的

```js
<div id="app" v-test1 v-test2>{{test}}</div>

var vm = new Vue({
  el: '#app',
  data: {
    test: 1
  },
  // 注册两个局部指令
  directives: {
    test1: {
      bind: function () {
        console.log('v-test1')
      }
    },
    test2: function () {
      console.log('v-test2')
    }
  }
})
```

规范化后的指令当时函数的时候直接返回`bind`和`update`属性的对象。

接下来是下面这一段代码

```js
const extendsFrom = child.extends
if (extendsFrom) {
  parent = mergeOptions(parent, extendsFrom, vm)
}
if (child.mixins) {
  for (let i = 0, l = child.mixins.length; i < l; i++) {
    parent = mergeOptions(parent, child.mixins[i], vm)
  }
}
```

很显然，这段代码是处理 `extends` 选项和 `mixins` 选项的，首先使用变量 `extendsFrom` 保存了对 `child.extends` 的引用，之后的处理都是用 `extendsFrom` 来做，然后判断 `extendsFrom` 是否为真，即 `child.extends` 是否存在，如果存在的话就递归调用 `mergeOptions` 函数将 `parent` 与 `extendsFrom` 进行合并，并将结果作为新的 `parent`。这里要注意，我们之前说过 `mergeOptions` 函数将会产生一个新的对象，所以此时的 `parent` 已经被新的对象重新赋值了。

接着检测 `child.mixins` 选项是否存在，如果存在则使用同样的方式进行操作，不同的是，由于 `mixins` 是一个数组所以要遍历一下。

### Vue选项的合并

```js
const options = {}
let key
for (key in parent) {
  mergeField(key)
}
for (key in child) {
  if (!hasOwn(parent, key)) {
    mergeField(key)
  }
}
function mergeField (key) {
  const strat = strats[key] || defaultStrat
  options[key] = strat(parent[key], child[key], vm, key)
}
return options
```

首先看第一个`for in`

```js
for (key in parent) {
  mergeField(key)
}
```

这段 `for in` 用来遍历 `parent`，并且将 `parent` 对象的键作为参数传递给 `mergeField` 函数，大家应该知道这里的 `key` 是什么，假如 `parent` 就是 `Vue.options`：

```js
Vue.options = {
  components: {
      KeepAlive,
      Transition,
      TransitionGroup
  },
  directives:{
      model,
      show
  },
  filters: Object.create(null),
  _base: Vue
}
```

在看第二个`for in`

```js
for (key in child) {
  if (!hasOwn(parent, key)) {
    mergeField(key)
  }
}
```

只有`parent` 中不存在的`key`才会进行合并

```js
function mergeField (key) {
  const strat = strats[key] || defaultStrat
  options[key] = strat(parent[key], child[key], vm, key)
}
```

这句代码就定义了 `strats` 变量，且它是一个常量，这个常量的值为 `config.optionMergeStrategies`，这个 `config` 对象是全局配置对象，来自于 `core/config.js` 文件，此时 `config.optionMergeStrategies` 还只是一个空的对象。注意一下这里的一段注释：*选项覆盖策略是处理如何将父选项值和子选项值合并到最终值的函数*。也就是说 `config.optionMergeStrategies` 是一个合并选项的策略对象，这个对象下包含很多函数，这些函数就可以认为是合并特定选项的策略。这样不同的选项使用不同的合并策略，如果你使用自定义选项，那么你也可以自定义该选项的合并策略，只需要在 `Vue.config.optionMergeStrategies` 对象上添加与自定义选项同名的函数就行。而这就是 `Vue` 文档中提过的全局配置：[optionMergeStrategies](https://vuejs.org/v2/api/#optionMergeStrategies)。

#### 选项el、propsData的合并策略

```js
/**
 * Options with restrictions
 */
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
```

首先是一段 `if` 判断分支，判断是否有传递 `vm` 参数：

```js
if (!vm) {
  warn(
    `option "${key}" can only be used during instance ` +
    'creation with the `new` keyword.'
  )
}
```

如果没有传递这个参数，那么便会给你一个警告，提示你 `el` 选项或者 `propsData` 选项只能在使用 `new` 操作符创建实例的时候可用。比如下面的代码：

```js
// 子组件
var ChildComponent = {
  el: '#app2',
  created: function () {
    console.log('child component created')
  }
}

// 父组件
new Vue({
  el: '#app',
  data: {
    test: 1
  },
  components: {
    ChildComponent
  }
})
```

`vm`参数是我们在`mergeOptions`的时候传递进来的

所以我们可以理解为：策略函数中的 `vm` 来自于 `mergeOptions` 函数的第三个参数。所以当调用 `mergeOptions` 函数且不传递第三个参数的时候，那么在策略函数中就拿不到 `vm` 参数。所以我们可以猜测到一件事，那就是 `mergeOptions` 函数除了在 `_init` 方法中被调用之外，还在其他地方被调用，且没有传递第三个参数。那么到底是在哪里被调用的呢？这里可以先明确地告诉大家，就是在 `Vue.extend` 方法中被调用的，大家可以打开 `core/global-api/extend.js` 文件找到 `Vue.extend` 方法，其中有这么一段代码：

```js
Sub.options = mergeOptions(
  Super.options,
  extendOptions
)
```

可以发现，此时调用 `mergeOptions` 函数就没有传递第三个参数，也就是说通过 `Vue.extend` 创建子类的时候 `mergeOptions` 会被调用，此时策略函数就拿不到第三个参数。

所以最终的结论就是：*如果策略函数中拿不到 `vm` 参数，那么处理的就是子组件的选项*，花了大量的口舌解释了策略函数中判断 `vm` 的意义，实际上这些解释是必要的。

我们接着看 `strats.el` 和 `strats.propsData` 策略函数的代码，在 `if` 判断分支下面，直接调用了 `defaultStrat` 函数并返回：

```js
return defaultStrat(parent, child)
```

`defaultStrat` 函数就定义在 `options.js` 文件内，源码如下：

```js
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

实际上 `defaultStrat` 函数就如同它的名字一样，它是一个默认的策略，当一个选项不需要特殊处理的时候就使用默认的合并策略，它的逻辑很简单：只要子选项不是 `undefined` 那么就是用子选项，否则使用父选项。

但是大家还要注意一点，`strats.el` 和 `strats.propsData` 这两个策略函数是只有在非生产环境才有的，在生产环境下访问这两个函数将会得到 `undefined`，那这个时候 `mergeField` 函数的第一句代码就起作用了：

```js
// 当一个选项没有对应的策略函数时，使用默认策略
const strat = strats[key] || defaultStrat
```

所以在生产环境将直接使用默认的策略函数 `defaultStrat` 来处理 `el` 和 `propsData` 这两个选项。

#### 选项data的合并策略

首先看下面的代码

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```

先看`if`分支

```js
if (childVal && typeof childVal !== 'function') {
  process.env.NODE_ENV !== 'production' && warn(
    'The "data" option should be a function ' +
    'that returns a per-instance value in component ' +
    'definitions.',
    vm
  )

  return parentVal
}
return mergeDataOrFn(parentVal, childVal)
```

首先判断是否传递了子组件的 `data` 选项(即：`childVal`)，并且检测 `childVal` 的类型是不是 `function`，如果 `childVal` 的类型不是 `function` 则会给你一个警告，也就是说 `childVal` 应该是一个函数，如果不是函数会提示你 `data` 的类型必须是一个函数，这就是我们知道的：*子组件中的 `data` 必须是一个返回对象的函数*。如果不是函数，除了给你一段警告之外，会直接返回 `parentVal`。

如果 `childVal` 是函数类型，那说明满足了子组件的 `data` 选项需要是一个函数的要求，那么就直接返回 `mergeDataOrFn` 函数的执行结果：

```js
return mergeDataOrFn(parentVal, childVal)
```

上面的情况是在 `strats.data` 策略函数拿不到 `vm` 参数时的情况，如果拿到了 `vm` 参数，那么说明处理的选项不是子组件的选项，而是正常使用 `new` 操作符创建实例时的选项，这个时候则直接返回 `mergeDataOrFn` 的函数执行结果，但是会多透传一个参数 `vm`：

```js
return mergeDataOrFn(parentVal, childVal, vm)
```

所以接下来我们要做的事儿就是看看 `mergeDataOrFn` 的代码

```js
/**
 * Data
 */
export function mergeDataOrFn (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // in a Vue.extend merge, both should be functions
    if (!childVal) {
      return parentVal
    }
    if (!parentVal) {
      return childVal
    }
    // when parentVal & childVal are both present,
    // we need to return a function that returns the
    // merged result of both functions... no need to
    // check if parentVal is a function here because
    // it has to be a function to pass previous merges.
    return function mergedDataFn () {
      return mergeData(
        typeof childVal === 'function' ? childVal.call(this, this) : childVal,
        typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
      )
    }
  } else {
    return function mergedInstanceDataFn () {
      // instance merge
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm, vm)
        : childVal
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm, vm)
        : parentVal
      if (instanceData) {
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
}
```

没有传递`vm`的分支

```js
// in a Vue.extend merge, both should be functions
```

这段注释的意思是：选项是在调用 `Vue.extend` 函数时进行合并处理的，此时父子 `data` 选项都应该是函数。

```js
if (!childVal) {
  return parentVal
}
if (!parentVal) {
  return childVal
}
```

我们看第一个 `if` 语句块，如果没有 `childVal`，也就是说子组件的选项中没有 `data` 选项，那么直接返回 `parentVal`，比如下面的代码：

```js
Vue.extend({})
```

我们使用 `Vue.extend` 函数创建子类的时候传递的子组件选项是一个空对象，即没有 `data` 选项，那么此时 `parentVal` 实际上就是 `Vue.options`，由于 `Vue.options` 上也没有 `data` 这个属性，所以压根就不会执行 `strats.data` 策略函数，也就更不会执行 `mergeDataOrFn` 函数，有的同学可能会问：既然都没有执行，那么这里的 `return parentVal` 是不是多余的？当然不多余，因为 `parentVal` 存在有值的情况。那么什么时候才会出现 `childVal` 不存在但是 `parentVal` 存在的情况呢？看下面的代码：

```js
const Parent = Vue.extend({
  data: function () {
    return {
      test: 1
    }
  }
})

const Child = Parent.extend({})
```

上面的代码中 `Parent` 类继承了 `Vue`，而 `Child` 又继承了 `Parent`，关键就在于我们使用 `Parent.extend` 创建 `Child` 子类的时候，对于 `Child` 类来讲，`childVal` 不存在，因为我们没有传递 `data` 选项，但是 `parentVal` 存在，即 `Parent.options` 下的 `data` 选项，那么 `Parent.options` 是哪里来的呢？实际就是 `Vue.extend` 函数内使用 `mergeOptions` 生成的，所以此时 `parentVal` 必定是个函数，因为 `strats.data` 策略函数在处理 `data` 选项后返回的始终是一个函数。

所以现在再看这段代码就清晰多了：

```js
if (!childVal) {
  return parentVal
}
if (!parentVal) {
  return childVal
}
```

由于 `childVal` 和 `parentVal` 必定会有其一，否则便不会执行 `strats.data` 策略函数，所以上面判断的意思就是：*如果没有子选项则使用父选项，没有父选项就直接使用子选项，且这两个选项都能保证是函数*，如果父子选项同时存在，则代码继续进行，将执行下面的代码：

```js
// when parentVal & childVal are both present,
// we need to return a function that returns the
// merged result of both functions... no need to
// check if parentVal is a function here because
// it has to be a function to pass previous merges.
return function mergedDataFn () {
  return mergeData(
    typeof childVal === 'function' ? childVal.call(this, this) : childVal,
    typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
  )
}
```

以上就是 `strats.data` 策略函数在处理子组件的 `data` 选项时所做的事，我们可以发现 `mergeDataOrFn` 函数在处理子组件选项时返回的总是一个函数，这也就间接导致 `strats.data` 策略函数在处理子组件选项时返回的也总是一个函数。

说完了处理子组件选项的情况，我们再看看处理非子组件选项的情况，也就是使用 `new` 操作符创建实例时的情况，此时程序直接执行 `strats.data` 函数的最后一句代码：

```js
return mergeDataOrFn(parentVal, childVal, vm)
```

这将会执行 `mergeDataOrFn` 的 `else` 分支：

```js
if (!vm) {
  ...
} else {
  return function mergedInstanceDataFn () {
    // instance merge
    const instanceData = typeof childVal === 'function'
      ? childVal.call(vm, vm)
      : childVal
    const defaultData = typeof parentVal === 'function'
      ? parentVal.call(vm, vm)
      : parentVal
    if (instanceData) {
      return mergeData(instanceData, defaultData)
    } else {
      return defaultData
    }
  }
}
```

如果走了 `else` 分支的话那么就直接返回 `mergedInstanceDataFn` 函数，注意此时的 `mergedInstanceDataFn` 函数同样还没有执行，它是 `mergeDataOrFn` 函数的返回值，所以这再次说明了一个问题：*`mergeDataOrFn` 函数永远返回一个函数*。

我们注意到 `mergedDataFn` 和 `mergedInstanceDataFn` 这两个函数都有类似这样的代码：

```js
typeof childVal === 'function' ? childVal.call(this, this) : childVal
typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
```

我们知道 `childVal` 要么是子组件的选项，要么是使用 `new` 操作符创建实例时的选项，无论是哪一种，总之 `childVal` 要么是函数，要么就是一个纯对象。所以如果是函数的话就通过执行该函数从而获取到一个纯对象，所以类似上面那段代码中判断 `childVal` 和 `parentVal` 的类型是否是函数的目的只有一个，获取数据对象(纯对象)。所以 `mergedDataFn` 和 `mergedInstanceDataFn` 函数内部调用 `mergeData` 方法时传递的两个参数就是两个纯对象(当然你可以简单的理解为两个JSON对象)。

所以说既然知道了 `mergeData` 函数接收的两个参数就是两个纯对象，那么再看 `mergeData` 函数的代码就轻松多了，它才是终极合并策略，其源码如下：

```js
/**
 * Helper that recursively merges two data objects together.
 */
function mergeData (to: Object, from: ?Object): Object {
  // 没有 from 直接返回 to
  if (!from) return to
  let key, toVal, fromVal
  const keys = Object.keys(from)
  // 遍历 from 的 key
  for (let i = 0; i < keys.length; i++) {
    key = keys[i]
    toVal = to[key]
    fromVal = from[key]
    // 如果 from 对象中的 key 不在 to 对象中，则使用 set 函数为 to 对象设置 key 及相应的值
    if (!hasOwn(to, key)) {
      set(to, key, fromVal)
    // 如果 from 对象中的 key 也在 to 对象中，且这两个属性的值都是纯对象则递归进行深度合并
    } else if (isPlainObject(toVal) && isPlainObject(fromVal)) {
      mergeData(toVal, fromVal)
    }
    // 其他情况什么都不做
  }
  return to
}
```

`mergeData` 函数接收两个参数 `to` 和 `from`，根据 `mergeData` 函数被调用时参数的传递顺序我们知道，`to` 对应的是 `childVal` 产生的纯对象，`from` 对应 `parentVal` 产生的纯对象，我们看 `mergeData` 第一句代码：

```js
if (!from) return to
```

如果没有 `from` 则直接返回 `to`，也就是说如果没有 `parentVal` 产生的值，就直接使用 `childVal` 产生的值。

如果有 `parentVal` 产生的值，则代码继续向下运行，我们看 `mergeData` 最后的返回值：

```js
return to
```

其返回的仍是 `to` 对象，所以你应该能猜的到 `mergeData` 函数的作用，可以简单理解为：*将 `from` 对象的属性混合到 `to` 对象中，也可以说是将 `parentVal` 对象的属性混合到 `childVal` 中*，最后返回的是处理后的 `childVal` 对象。

`mergeData` 的具体做法就是像上面 `mergeData` 函数的代码段中所注释的那样，对 `from` 对象的 `key` 进行遍历：

- 如果 `from` 对象中的 `key` 不在 `to` 对象中，则使用 `set` 函数为 `to` 对象设置 `key` 及相应的值。
- 如果 `from` 对象中的 `key` 在 `to` 对象中，且这两个属性的值都是纯对象则递归地调用 `mergeData` 函数进行深度合并。
- 其他情况不做处理。

上面提到了一个 `set` 函数，根据 `options.js` 文件头部的引用关系可知：这个函数来自于 `core/observer/index.js` 文件，实际上这个 `set` 函数就是 `Vue` 暴露给我们的全局API `Vue.set`。在这里由于我们还没有讲到 `set` 函数的具体实现，所以你就可以简单理解为 `set` 函数的功能与我们前面遇到过的 `extend` 工具函数功能相似即可。

所以我们知道了 `mergeData` 函数的执行结果才是真正的数据对象，由于 `mergedDataFn` 和 `mergedInstanceDataFn` 这两个函数的返回值就是 `mergeData` 函数的执行结果，所以 `mergedDataFn` 和 `mergedInstanceDataFn` 函数的执行将会得到数据对象，我们还知道 `data` 选项会被 `mergeOptions` 处理成函数，比如处理成 `mergedInstanceDataFn`，所以：*最终得到的 `data` 选项是一个函数，且该函数的执行结果就是最终的数据对象*。

**一、为什么最终 `strats.data` 会被处理成一个函数**

这是因为，通过函数返回数据对象，保证了每个组件实例都有一个唯一的数据副本，避免了组件间数据互相影响。后面讲到 `Vue` 的初始化的时候大家会看到，在初始化数据状态的时候，就是通过执行 `strats.data` 函数来获取数据并对其进行处理的。

**二、为什么不在合并阶段就把数据合并好，而是要等到初始化的时候再合并数据？**

这个问题是什么意思呢？我们知道在合并阶段 `strats.data` 将被处理成一个函数，但是这个函数并没有被执行，而是到了后面初始化的阶段才执行的，这个时候才会调用 `mergeData` 对数据进行合并处理，那这么做的目的是什么呢？

其实这么做是有原因的，后面讲到 `Vue` 的初始化的时候，大家就会发现 `inject` 和 `props` 这两个选项的初始化是先于 `data` 选项的，这就保证了我们能够使用 `props` 初始化 `data` 中的数据，如下：

```js
// 子组件：使用 props 初始化子组件的 childData 
const Child = {
  template: '<span></span>',
  data () {
    return {
      childData: this.parentData
    }
  },
  props: ['parentData'],
  created () {
    // 这里将输出 parent
    console.log(this.childData)
  }
}

var vm = new Vue({
    el: '#app',
    // 通过 props 向子组件传递数据
    template: '<child parent-data="parent" />',
    components: {
      Child
    }
})
```

如上例所示，子组件的数据 `childData` 的初始值就是 `parentData` 这个 `props`。而之所以能够这样做的原因有两个

- 1、由于 `props` 的初始化先于 `data` 选项的初始化
- 2、`data` 选项是在初始化的时候才求值的，你也可以理解为在初始化的时候才使用 `mergeData` 进行数据合并。

**三、你可以这么做**

在上面的例子中，子组件的 `data` 选项我们是这么写的：

```js
data () {
  return {
    childData: this.parentData
  }
}
```

但你知道吗，你也可以这么写：

```js
data (vm) {
  return {
    childData: vm.parentData
  }
}
// 或者使用更简单的解构赋值
data ({ parentData }) {
  return {
    childData: parentData
  }
}
```

我们可以通过解构赋值的方式，也就是说 `data` 函数的参数就是当前实例对象。那么这个参数是在哪里传递进来的呢？其实有两个地方，其中一个地方我们前面见过了，如下面这段代码：

```js
return function mergedDataFn () {
  return mergeData(
    typeof childVal === 'function' ? childVal.call(this, this) : childVal,
    typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
  )
}
```

注意这里的 `childVal.call(this, this)` 和 `parentVal.call(this, this)`，关键在于 `call(this, this)`，可以看到，第一个 `this` 指定了 `data` 函数的作用域，而第二个 `this` 就是传递给 `data` 函数的参数。

当然了仅仅在这里这么做是不够的，比如 `mergedDataFn` 前面的代码：

```js
if (!childVal) {
  return parentVal
}
if (!parentVal) {
  return childVal
}
```

在这段代码中，直接将 `parentVal` 或 `childVal` 返回了，我们知道这里的 `parentVal` 和 `childVal` 就是 `data` 函数，由于被直接返回，所以并没有指定其运行的作用域，且也没有传递当前实例作为参数，所以我们必然还是在其他地方做这些事情，而这个地方就是我们说的第二个地方，它在哪里呢？当然是初始化的时候，后面我们会讲到的

#### 生命周期钩子选项的合并策略

我们继续按照 `options.js` 文件的顺序看代码，接下来的一段代码如下

```js
/**
 * Hooks and props are merged as arrays.
 */
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}

LIFECYCLE_HOOKS.forEach(hook => {
  strats[hook] = mergeHook
})
```

`LIFECYCLE_HOOKS`的代码如下

```js
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured'
]
```

所以现在再回头来看那段 `forEach` 语句可知，它的作用就是在 `strats` 策略对象上添加用来合并各个生命周期钩子选项的策略函数，并且这些生命周期钩子选项的策略函数相同：*都是 `mergeHook` 函数*

```js
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}
```

整个函数体由三组*三目运算符*组成，有一点值得大家学习的就是这里写三目运算符的方式，是不是感觉非常地清晰易读？那么这段代码的分析我们同样使用与上面代码相同的格式来写：

```js
return (是否有 childVal，即判断组件的选项中是否有对应名字的生命周期钩子函数)
  ? 如果有 childVal 则判断是否有 parentVal
    ? 如果有 parentVal 则使用 concat 方法将二者合并为一个数组
    : 如果没有 parentVal 则判断 childVal 是不是一个数组
      ? 如果 childVal 是一个数组则直接返回
      : 否则将其作为数组的元素，然后返回数组
  : 如果没有 childVal 则直接返回 parentVal
```

第一个三目运算符需要注意，它判断是否有 `childVal`，即组件的选项是否写了生命周期钩子函数，如果没有则直接返回了 `parentVal`，这里有个问题：`parentVal` 一定是数组吗？答案是：*如果有 `parentVal` 那么其一定是数组，如果没有 `parentVal` 那么 `strats[hooks]` 函数根本不会执行*。我们以 `created` 生命周期钩子函数为例：

```js
new Vue({
  created: function () {
    console.log('created')
  }
})
```

如果以这段代码为例，那么对于 `strats.created` 策略函数来讲(注意这里的 `strats.created` 就是 `mergeHooks`)，`childVal` 就是我们例子中的 `created` 选项，它是一个函数。`parentVal` 应该是 `Vue.options.created`，但 `Vue.options.created` 是不存在的，所以最终经过 `strats.created` 函数的处理将返回一个数组：

```js
options.created = [
  function () {
    console.log('created')
  }  
]
```

在看下面的例子

```js
const Parent = Vue.extend({
  created: function () {
    console.log('parentVal')
  }
})

const Child = new Parent({
  created: function () {
    console.log('childVal')
  }
})
```

其中 `Child` 是使用 `new Parent` 生成的，所以对于 `Child` 来讲，`childVal` 是：

```js
created: function () {
  console.log('childVal')
}
```

而 `parentVal` 已经不是 `Vue.options.created` 了，而是 `Parent.options.created`，那么 `Parent.options.created` 是什么呢？它其实是通过 `Vue.extend` 函数内部的 `mergeOptions` 处理过的，所以它应该是这样的：

```js
Parent.options.created = [
  created: function () {
    console.log('parentVal')
  }
]
```

所以这个例子最终的结果就是既有 `childVal`，又有 `parentVal`，那么根据 `mergeHooks` 函数的逻辑：

```js
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      // 这里，合并且生成一个新数组
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}
```

关键在这句：`parentVal.concat(childVal)`，将 `parentVal` 和 `childVal` 合并成一个数组。所以最终结果如下：

```js
[
  created: function () {
    console.log('parentVal')
  },
  created: function () {
    console.log('childVal')
  }
]
```

另外我们注意第三个三目运算符：

```js
: Array.isArray(childVal)
  ? childVal
  : [childVal]
```

它判断了 `childVal` 是不是数组，这说明什么？说明了生命周期钩子是可以写成数组的，虽然 `Vue` 的文档里没有，不信你可以试试：

```js
new Vue({
  created: [
    function () {
      console.log('first')
    },
    function () {
      console.log('second')
    },
    function () {
      console.log('third')
    }
  ]
})
```

钩子函数将按顺序执行

#### 资源（assets）选项的合并策略

在 `Vue` 中 `directives`、`filters` 以及 `components` 被认为是资源，其实很好理解，指令、过滤器和组件都是可以作为第三方应用来提供的，比如你需要一个模拟滚动的组件，你当然可以选用超级强大的第三方组件 [scroll-flip-page](https://github.com/HcySunYang/scroll-flip-page)，所以这样看来 [scroll-flip-page](https://github.com/HcySunYang/scroll-flip-page) 就可以认为是资源，除了组件之外指令和过滤器也都是同样的道理。

而我们接下来要看的代码就是用来合并处理 `directives`、`filters` 以及 `components` 等资源选项的，看如下代码

```js
/**
 * Assets
 *
 * When a vm is present (instance creation), we need to do
 * a three-way merge between constructor options, instance
 * options and parent options.
 */
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal)
  } else {
    return res
  }
}

ASSET_TYPES.forEach(function (type) {
  strats[type + 's'] = mergeAssets
})
```

`ASSET_TYPES` 常量如下

```js
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
```

`mergeAssets` 代码如下：

```js
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal)
  } else {
    return res
  }
}
```

举个例子，大家知道任何组件的模板中我们都可以直接使用 `<transition/>` 组件或者 `<keep-alive/>` 等，但是我们并没有在我们自己的组件实例的 `components` 选项中显式地声明这些组件。那么这是怎么做到的呢？其实答案就在 `mergeAssets` 函数中。以下面的代码为例：

```js
var v = new Vue({
  el: '#app',
  components: {
    ChildComponent: ChildComponent
  }
})
```

上面的代码中，我们创建了一个 `Vue` 实例，并注册了一个子组件 `ChildComponent`，此时 `mergeAssets` 方法内的 `childVal` 就是例子中的 `components` 选项：

```js
components: {
  ChildComponent: ChildComponent
}
```

而 `parentVal` 就是 `Vue.options.components`，我们知道 `Vue.options` 如下：

```js
Vue.options = {
	components: {
	  KeepAlive,
	  Transition,
	  TransitionGroup
	},
	directives: Object.create(null),
	directives:{
	  model,
	  show
	},
	filters: Object.create(null),
	_base: Vue
}
```

所以 `Vue.options.components` 就应该是一个对象：

```js
{
  KeepAlive,
  Transition,
  TransitionGroup
}
```

也就是说 `parentVal` 就是如上包含三个内置组件的对象，所以经过如下这句话之后：

```js
const res = Object.create(parentVal || null)
```

然后再经过 `return extend(res, childVal)` 这句话之后，`res` 变量将被添加 `ChildComponent` 属性，最终 `res` 如下：

```js
res = {
  ChildComponent
  // 原型
  __proto__: {
    KeepAlive,
    Transition,
    TransitionGroup
  }
}
```

所以这就是为什么我们不用显式地注册组件就能够使用一些内置组件的原因，同时这也是内置组件的实现方式，通过 `Vue.extend` 创建出来的子类也是一样的道理，一层一层地通过原型进行组件的搜索。

最后说一下 `mergeAssets` 函数中的这句话：

```js
process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
```

在非生产环境下，会调用 `assertObjectType` 函数，这个函数其实是用来检测 `childVal` 是不是一个纯对象的，如果不是纯对象会给你一个警告，其源码很简单，如下：

```js
function assertObjectType (name: string, value: any, vm: ?Component) {
  if (!isPlainObject(value)) {
    warn(
      `Invalid value for option "${name}": expected an Object, ` +
      `but got ${toRawType(value)}.`,
      vm
    )
  }
}
```

就是使用 `isPlainObject` 进行判断。上面我们都在以 `components` 进行讲解，对于指令(`directives`)和过滤器(`filters`)也是一样的，因为他们都是用 `mergeAssets` 进行合并处理

#### 选项watch的合并策略

```js
/**
 * Watchers.
 *
 * Watchers hashes should not overwrite one
 * another, so we merge them as arrays.
 */
strats.watch = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  // work around Firefox's Object.prototype.watch...
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
  /* istanbul ignore if */
  if (!childVal) return Object.create(parentVal || null)
  if (process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = {}
  extend(ret, parentVal)
  for (const key in childVal) {
    let parent = ret[key]
    const child = childVal[key]
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child]
  }
  return ret
}
```

这一段代码的作用是在 `strats` 策略对象上添加 `watch` 策略函数。所以 `strats.watch` 策略函数应该是合并处理 `watch` 选项的。我们先看函数体开头的两句代码：

```js
// work around Firefox's Object.prototype.watch...
if (parentVal === nativeWatch) parentVal = undefined
if (childVal === nativeWatch) childVal = undefined
```

其中 `nativeWatch` 来自于 `core/util/env.js` 文件，在 `Firefox` 浏览器中 `Object.prototype` 拥有原生的 `watch` 函数，所以即便一个普通的对象你没有定义 `watch` 属性，但是依然可以通过原型链访问到原生的 `watch` 属性，这就会给 `Vue` 在处理选项的时候造成迷惑，因为 `Vue` 也提供了一个叫做 `watch` 的选项，即使你的组件选项中没有写 `watch` 选项，但是 `Vue` 通过原型访问到了原生的 `watch`。这不是我们想要的，所以上面两句代码的目的是一个变通方案，当发现组件选项是浏览器原生的 `watch` 时，那说明用户并没有提供 `Vue` 的 `watch` 选项，直接重置为 `undefined`。

然后是这句代码：

```js
if (!childVal) return Object.create(parentVal || null)
```

检测了是否有 `childVal`，即组件选项是否有 `watch` 选项，如果没有的话，直接以 `parentVal` 为原型创建对象并返回(如果有 `parentVal` 的话)。

如果组件选项中有 `watch` 选项，即 `childVal` 存在，则代码继续执行，接下来将执行这段代码：

```js
if (process.env.NODE_ENV !== 'production') {
  assertObjectType(key, childVal, vm)
}
if (!parentVal) return childVal
```

如果存在 `parentVal`，那么代码继续执行，此时 `parentVal` 以及 `childVal` 都将存在，那么就需要做合并处理了，也就是下面要执行的代码：

```js
// 定义 ret 常量，其值为一个对象
const ret = {}
// 将 parentVal 的属性混合到 ret 中，后面处理的都将是 ret 对象，最后返回的也是 ret 对象
extend(ret, parentVal)
// 遍历 childVal
for (const key in childVal) {
  // 由于遍历的是 childVal，所以 key 是子选项的 key，父选项中未必能获取到值，所以 parent 未必有值
  let parent = ret[key]
  // child 是肯定有值的，因为遍历的就是 childVal 本身
  const child = childVal[key]
  // 这个 if 分支的作用就是如果 parent 存在，就将其转为数组
  if (parent && !Array.isArray(parent)) {
    parent = [parent]
  }
  ret[key] = parent
    // 最后，如果 parent 存在，此时的 parent 应该已经被转为数组了，所以直接将 child concat 进去
    ? parent.concat(child)
    // 如果 parent 不存在，直接将 child 转为数组返回
    : Array.isArray(child) ? child : [child]
}
// 最后返回新的 ret 对象
return ret
```

上面的代码段中写了很详细的注释。首先定义了 `ret` 常量，最后返回的也是 `ret` 常量，所以中间的代码是在充实 `ret` 常量。之后使用 `extend` 函数将 `parentVal` 的属性混合到 `ret` 中。然后开始一个 `for in` 循环遍历 `childVal`，这个循环的目的是：*检测子选项中的值是否也在父选项中，如果在的话将父子选项合并到一个数组，否则直接把子选项变成一个数组返回*。

```js
// 创建子类
const Sub = Vue.extend({
  // 检测 test 的变化
  watch: {
    test: function () {
      console.log('extend: test change')
    }
  }
})

// 使用子类创建实例
const v = new Sub({
  el: '#app',
  data: {
    test: 1
  },
  // 检测 test 的变化
  watch: {
    test: function () {
      console.log('instance: test change')
    }
  }
})

// 修改 test 的值
v.test = 2
```

上面的代码中，当我们修改 `v.test` 的值时，两个观察 `test` 变化的函数都将被执行。

我们使用子类 `Sub` 创建了实例 `v`，对于实例 `v` 来讲，其 `childVal` 就是组件选项的 `watch`：

```js
watch: {
  test: function () {
    console.log('instance: test change')
  }
}
```

而其 `parentVal` 就是 `Sub.options`，实际上就是：

```js
watch: {
  test: function () {
    console.log('extend: test change')
  }
}
```

最终这两个 `watch` 选项将被合并为一个数组：

```js
watch: {
  test: [
    function () {
      console.log('extend: test change')
    },
    function () {
      console.log('instance: test change')
    }
  ]
}
```

我们可以通过打印实例的 `$options` 属性来确认这一点：

```js
console.log(v.$options)
```

可以发现 `watch.test` 变成了数组，但是 `watch.test` 并不一定总是数组，只有父选项(`parentVal`)也存在时它才是数组，如下：

```js
// 创建实例
const v = new Vue({
  el: '#app',
  data: {
    test: 1
  },
  // 检测 test 的变化
  watch: {
    test: function () {
      console.log('instance: test change')
    }
  }
})

// 修改 test 的值
v.test = 2
```

我们直接使用 `Vue` 创建实例，这个时候对于实例 `v` 来说，父选项是 `Vue.options`，由于 `Vue.options` 并没有 `watch` 选项，所以逻辑将直接在 `strats.watch` 函数的这句话中返回：

```js
if (!parentVal) return childVal
```

没有 `parentVal` 即父选项中没有 `watch` 选项，则直接返回 `childVal`，也就是直接返回了子选项的 `watch` 选项，如就是例子中写的对象：

```js
{
  test: function () {
    console.log('instance: test change')
  }
}
```

所以此时 `test` 字段就不再是数组了，而就是一个函数，同样可以通过打印实例的 `$options` 选项证明：

所以大家应该知道：*被合并处理后的 `watch` 选项下的每个键值，有可能是一个数组，也有可能是一个函数*。

#### 选项props、methods、inject、computed合并策略

```js
/**
 * Other object hashes.
 */
strats.props =
strats.methods =
strats.inject =
strats.computed = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  if (childVal && process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = Object.create(null)
  extend(ret, parentVal)
  if (childVal) extend(ret, childVal)
  return ret
}
```

这段代码的作用是在 `strats` 策略对象上添加 `props`、`methods`、`inject` 以及 `computed` 策略函数，顾名思义这些策略函数是分别用来合并处理同名选项的，并且所使用的策略相同。

对于 `props`、`methods`、`inject` 以及 `computed` 这四个选项有一个共同点，就是它们的结构都是纯对象，虽然我们在书写 `props` 或者 `inject` 选项的时候可能是一个数组，但是在之前我们知道，`Vue` 内部都将其规范化为了一个对象。所以我们看看 `Vue` 是如何处理这些对象散列的。

```js
// 如果存在 childVal，那么在非生产环境下要检查 childVal 的类型
if (childVal && process.env.NODE_ENV !== 'production') {
  assertObjectType(key, childVal, vm)
}
// parentVal 不存在的情况下直接返回 childVal
if (!parentVal) return childVal
// 如果 parentVal 存在，则创建 ret 对象，然后分别将 parentVal 和 childVal 的属性混合到 ret 中，注意：由于 childVal 将覆盖 parentVal 的同名属性
const ret = Object.create(null)
extend(ret, parentVal)
if (childVal) extend(ret, childVal)
// 最后返回 ret 对象。
return ret
```

首先，会检测 `childVal` 是否存在，即子选项是否有相关的属性，如果有的话在非生产环境下需要使用 `assertObjectType` 检测其类型，保证其类型是纯对象。然后会判断 `parentVal` 是否存在，不存在的话直接返回子选项。

如果 `parentVal` 存在，则使用 `extend` 方法将其属性混合到新对象 `ret` 中，如果 `childVal` 也存在的话，那么同样会再使用 `extend` 函数将其属性混合到 `ret` 中，所以如果父子选项中有相同的键，那么子选项会把父选项覆盖掉。

以上就是 `props`、`methods`、`inject` 以及 `computed` 这四个属性的通用合并策略。

#### 选项provide的合并策略

最后一个选项的合并策略，就是 `provide` 选项的合并策略，只有一句代码，如下：

```js
strats.provide = mergeDataOrFn
```

也就是说 `provide` 选项的合并策略与 `data` 选项的合并策略相同，都是使用 `mergeDataOrFn` 函数

#### 选项处理小结

现在我们了解了 `Vue` 中是如何合并处理选项的，接下来我们稍微做一个总结：

- 对于 `el`、`propsData` 选项使用默认的合并策略 `defaultStrat`。
- 对于 `data` 选项，使用 `mergeDataOrFn` 函数进行处理，最终结果是 `data` 选项将变成一个函数，且该函数的执行结果为真正的数据对象。
- 对于 `生命周期钩子` 选项，将合并成数组，使得父子选项中的钩子函数都能够被执行
- 对于 `directives`、`filters` 以及 `components` 等资源选项，父子选项将以原型链的形式被处理，正是因为这样我们才能够在任何地方都使用内置组件、指令等。
- 对于 `watch` 选项的合并处理，类似于生命周期钩子，如果父子选项都有相同的观测字段，将被合并为数组，这样观察者都将被执行。
- 对于 `props`、`methods`、`inject`、`computed` 选项，父选项始终可用，但是子选项会覆盖同名的父选项字段。
- 对于 `provide` 选项，其合并策略使用与 `data` 选项相同的 `mergeDataOrFn` 函数。
- 最后，以上没有提及到的选项都将使默认选项 `defaultStrat`。
- 最最后，默认合并策略函数 `defaultStrat` 的策略是：*只要子选项不是 `undefined` 就使用子选项，否则使用父选项*。

#### 再看mixins和extends

在之前的一节中，我们讲到了 `mergeOptions` 函数中的如下这段代码：

```js
const extendsFrom = child.extends
if (extendsFrom) {
  parent = mergeOptions(parent, extendsFrom, vm)
}
if (child.mixins) {
  for (let i = 0, l = child.mixins.length; i < l; i++) {
    parent = mergeOptions(parent, child.mixins[i], vm)
  }
}
```

当时候我们并没有深入讲解，因为当时我们还不了解 `mergeOptions` 函数的作用，但是现在我们可以回头来看一下这段代码了。

我们知道 `mixins` 在 `Vue` 中用于解决代码复用的问题，比如混入 `created` 生命周期钩子，用于打印一句话：

```js
const consoleMixin = {
  created () {
    console.log('created:mixins')
  }
}

new Vue ({
  mixins: [consoleMixin],
  created () {
    console.log('created:instance')
  }
})
```

运行以上代码，将打印两句话：

```js
// created:mixins
// created:instance
```

这是因为 `mergeOptions` 函数在处理 `mixins` 选项的时候递归调用了 `mergeOptions` 函数将 `mixins` 合并到了 `parent` 中，并将合并后生成的新对象作为新的 `parent`：

```js
if (child.mixins) {
  for (let i = 0, l = child.mixins.length; i < l; i++) {
    parent = mergeOptions(parent, child.mixins[i], vm)
  }
}
```

上例中我们只涉及到 `created` 生命周期钩子的合并，所以会使用生命周期钩子的合并策略函数进行处理，现在我们已经知道 `mergeOptions` 会把生命周期选项合并为一个数组，所以所有的生命周期钩子都会被执行。那么不仅仅是生命周期钩子，任何写在 `mixins` 中的选项，都会使用 `mergeOptions` 中相应的合并策略进行处理，这就是 `mixins` 的实现方式。

对于 `extends` 选项，与 `mixins` 相同，甚至由于 `extends` 选项只能是一个对象，而不能是数组，反而要比 `mixins` 的实现更为简单，连遍历都不需要。

### Vue的初始化之开篇

#### 用于初始化的最终选项$options

在 以一个例子为线索一节中，我们写了一个很简单的例子，这个例子如下：

```js
var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    }
})
```

我们以这个例子为线索开始了对 `Vue` 代码的讲解，我们知道了在实例化 `Vue` 实例的时候，`Vue.prototype._init` 方法被第一个执行，这个方法定义在 `src/core/instance/init.js` 文件中，在分析 `_init` 方法的时候我们遇到了下面的代码：

```js
vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
)
```

正是因为上面的代码，使得我们花了大篇章来讲解其内部实现和运作，也就是 Vue选项的规范化和 Vue选项的合并 这两节所介绍的内容。现在我们已经知道了 `mergeOptions` 函数是如何对父子选项进行合并处理的，也知道了它的作用。

我们打开 `core/util/options.js` 文件，找到 `mergeOptions` 函数，看其最后一句代码：

```js
return options
```

这说明 `mergeOptions` 函数最终将合并处理后的选项返回，并以该返回值作为 `vm.$options` 的值。`vm.$options` 在 `Vue` 的官方文档中是可以找到的，它作为实例属性暴露给开发者，那么现在你应该知道 `vm.$options` 到底是什么了。并且看文档的时候你应该更能够理解其作用，比如官方文档是这样介绍 `$options` 实例属性的：

> 用于当前 `Vue` 实例的初始化选项。需要在选项中包含自定义属性时会有用处

并且给了一个例子，如下：

```js
new Vue({
  customOption: 'foo',
  created: function () {
    console.log(this.$options.customOption) // => 'foo'
  }
})
```

上面的例子中，在创建 `Vue` 实例的时候传递了一个自定义选项：`customOption`，在之后的代码中我们可以通过 `this.$options.customOption` 进行访问。那原理其实就是使用 `mergeOptions` 函数对自定义选项进行合并处理，由于没有指定 `customOption` 选项的合并策略，所以将会使用默认的策略函数 `defaultStrat`。最终效果就是你初始化的值是什么，得到的就是什么。

另外，`Vue` 也提供了 `Vue.config.optionMergeStrategies` 全局配置，大家也可以在官方文档中找到，我们知道这个对象其实就是选项合并中的策略对象，所以我们可以通过他指定某一个选项的合并策略，常用于指定自定义选项的合并策略，比如我们给 `customOption` 选项指定一个合并策略，只需要在 `Vue.config.optionMergeStrategies` 上添加与选项同名的策略函数即可：

```js
Vue.config.optionMergeStrategies.customOption = function (parentVal, childVal) {
    return parentVal ? (parentVal + childVal) : childVal
}
```

如上代码中，我们添加了自定义选项 `customOption` 的合并策略，其策略为：如果没有 `parentVal` 则直接返回 `childVal`，否则返回两者的和。

所以如下代码：

```js
// 创建子类
const Sub = Vue.extend({
    customOption: 1
})
// 以子类创建实例
const v = new Sub({
    customOption: 2,
    created () {
        console.log(this.$options.customOption) // 3
    }
})
```

最终，在实例的 `created` 方法中将打印数字 `3`。上面的例子很简单，没有什么实际作用，但这为我们提供了自定义选项的机会，这其实是非常有用的。

现在我们需要回到正题上了，还是拿我们的例子，如下：

```js
var vm = new Vue({
    el: '#app',
    data: {
        test: 1
    }
})
```

这个时候 `mergeOptions` 函数将会把 `Vue.options` 作为 父选项，把我们传递的实例选项作为子选项进行合并，合并的结果我们可以通过打印 `$options` 属性得知。其实我们前面已经分析过了，`el` 选项将使用默认合并策略合并，最终的值就是字符串 `'#app'`，而 `data` 选项将变成一个函数，且这个函数的执行结果就是合并后的数据，即： `{test: 1}`。

#### 渲染函数的作用域代理

ok，现在我们已经足够了解 `vm.$options` 这个属性了，它才是用来做一系列初始化工作的最终选项，那么接下来我们就继续看 `_init` 方法中的代码，继续了解 `Vue` 的初始化工作。

`_init` 方法中，在经过 `mergeOptions` 合并处理选项之后，要执行的是下面这段代码：

```js
/* istanbul ignore else */
if (process.env.NODE_ENV !== 'production') {
    initProxy(vm)
} else {
    vm._renderProxy = vm
}
```

这段代码是一个判断分支，如果是非生产环境的话则执行 `initProxy(vm)` 函数，如果在生产环境则直接在实例上添加 `_renderProxy` 实例属性，该属性的值就是当前实例。

现在有一个问题需要大家思考一下，目前我们还没有看 `initProxy` 函数的具体内容，那么你能猜到 `initProxy` 函数的主要作用是什么吗？我可以直接告诉大家，这个函数的主要作用其实就是在实例对象 `vm` 上添加 `_renderProxy` 属性。为什么呢？因为生产环境和非生产环境下要保持功能一致。在上面的代码中生产环境下直接执行这句：

```js
vm._renderProxy = vm
```

那么可想而知，在非生产环境下也应该执行这句代码，但实际上却调用了 `initProxy` 函数，所以 `initProxy` 函数的作用之一必然也是在实例对象 `vm` 上添加 `_renderProxy` 属性，那么接下来我们就看看 `initProxy` 的内容，验证一下我们的判断，打开 `core/instance/proxy.js` 文件：

```js
/* not type checking this file because flow doesn't play well with Proxy */

import config from 'core/config'
import { warn, makeMap } from '../util/index'

// 声明 initProxy 变量
let initProxy

if (process.env.NODE_ENV !== 'production') {
  // ... 其他代码
  
  // 在这里初始化 initProxy
  initProxy = function initProxy (vm) {
    if (hasProxy) {
      // determine which proxy handler to use
      const options = vm.$options
      const handlers = options.render && options.render._withStripped
        ? getHandler
        : hasHandler
      vm._renderProxy = new Proxy(vm, handlers)
    } else {
      vm._renderProxy = vm
    }
  }
}

// 导出
export { initProxy }
```

上面的代码是简化后的，可以发现在文件的开头声明了 `initProxy` 变量，但并未初始化，所以目前 `initProxy` 还是 `undefined`，随后，在文件的结尾将 `initProxy` 导出，那么 `initProxy` 到底是什么呢？实际上变量 `initProxy` 的赋值是在 `if` 语句块内进行的，这个 `if` 语句块进行环境判断，如果是非生产环境的话，那么才会对 `initProxy` 变量赋值，也就是说在生产环境下我们导出的 `initProxy` 实际上就是 `undefined`。只有在非生产环境下导出的 `initProxy` 才会有值，其值就是这个`initProxy`函数：

这个函数接收一个参数，实际就是 `Vue` 实例对象，我们先从宏观角度来看一下这个函数的作用是什么，可以发现，这个函数由 `if...else` 语句块组成，但无论走 `if` 还是 `else`，其最终的效果都是在 `vm` 对象上添加了 `_renderProxy` 属性，这就验证了我们之前的猜想。如果 `hasProxy` 为真则走 `if` 分支，对于 `hasProxy` 顾名思义，这是用来判断宿主环境是否支持 `js` 原生的 `Proxy` 特性的，如果发现 `Proxy` 存在，则执行：

```js
vm._renderProxy = new Proxy(vm, handlers)
```

如果不存在，那么和生产环境一样，直接赋值就可以了：

```js
vm._renderProxy = vm
```

所以我们发现 `initProxy` 的作用实际上就是对实例对象 `vm` 的代理，通过原生的 `Proxy` 实现。

另外 `hasProxy` 变量的定义也在当前文件中，代码如下：

```js
const hasProxy =
    typeof Proxy !== 'undefined' &&
    Proxy.toString().match(/native code/)
```

上面代码的作用是判断当前宿主环境是否支持原生 `Proxy`，相信大家都能看得懂，所以就不做过多解释，接下来我们就看看它是如何做代理的，并且有什么作用。

查看 `initProxy` 函数的 `if` 语句块，内容如下：

```js
initProxy = function initProxy (vm) {
    if (hasProxy) {
        // determine which proxy handler to use
        // options 就是 vm.$options 的引用
        const options = vm.$options
        // handlers 可能是 getHandler 也可能是 hasHandler
        const handlers = options.render && options.render._withStripped
            ? getHandler
            : hasHandler
        // 代理 vm 对象
        vm._renderProxy = new Proxy(vm, handlers)
    } else {
        // ...
    }
}
```

可以发现，如果 `Proxy` 存在，那么将会使用 `Proxy` 对 `vm` 做一层代理，代理对象赋值给 `vm._renderProxy`，所以今后对 `vm._renderProxy` 的访问，如果有代理那么就会被拦截。代理对象配置参数是 `handlers`，可以发现 `handlers` 既可能是 `getHandler` 又可能是 `hasHandler`，至于到底使用哪个，是由判断条件决定的：

```js
options.render && options.render._withStripped
```

如果上面的条件为真，则使用 `getHandler`，否则使用 `hasHandler`，判断条件要求 `options.render` 和 `options.render._withStripped` 必须都为真才行，我现在明确告诉大家 `options.render._withStripped` 这个属性只在测试代码中出现过，所以一般情况下这个条件都会为假，也就是使用 `hasHandler` 作为代理配置。

`hasHandler` 常量就定义在当前文件，如下：

```js
const hasHandler = {
    has (target, key) {
        // has 常量是真实经过 in 运算符得来的结果
        const has = key in target
        // 如果 key 在 allowedGlobals 之内，或者 key 是以下划线 _ 开头的字符串，则为真
        const isAllowed = allowedGlobals(key) || (typeof key === 'string' && key.charAt(0) === '_')
        // 如果 has 和 isAllowed 都为假，使用 warnNonPresent 函数打印错误
        if (!has && !isAllowed) {
            warnNonPresent(target, key)
        }
        return has || !isAllowed
    }
}
```

我们知道 `has` 可以拦截以下操作：

- 属性查询: foo in proxy
- 继承属性查询: foo in Object.create(proxy)
- with 检查: with(proxy) { (foo); }
- Reflect.has()

`has` 函数内出现了两个函数，分别是 `allowedGlobals` 以及 `warnNonPresent`，这两个函数也是定义在当前文件中，首先我们看一下 `allowedGlobals`：

```js
const allowedGlobals = makeMap(
    'Infinity,undefined,NaN,isFinite,isNaN,' +
    'parseFloat,parseInt,decodeURI,decodeURIComponent,encodeURI,encodeURIComponent,' +
    'Math,Number,Date,Array,Object,Boolean,String,RegExp,Map,Set,JSON,Intl,' +
    'require' // for Webpack/Browserify
)
```

可以看到 `allowedGlobals` 实际上是通过 `makeMap` 生成的函数，所以 `allowedGlobals` 函数的作用是判断给定的 `key` 是否出现在上面字符串中定义的关键字中的。这些关键字都是在 `js` 中可以全局访问的。

`warnNonPresent` 函数如下：

```js
const warnNonPresent = (target, key) => {
    warn(
        `Property or method "${key}" is not defined on the instance but ` +
        'referenced during render. Make sure that this property is reactive, ' +
        'either in the data option, or for class-based components, by ' +
        'initializing the property. ' +
        'See: https://vuejs.org/v2/guide/reactivity.html#Declaring-Reactive-Properties.',
        target
    )
}
```

这个函数就是通过 `warn` 打印一段警告信息，警告信息提示你“在渲染的时候引用了 `key`，但是在实例对象上并没有定义 `key` 这个属性或方法”。其实我们很容易就可以看到这个信息，比如下面的代码：

```js
const vm = new Vue({
    el: '#app',
    template: '<div>{{a}}</div>',
    data: {
        test: 1
    }
})
```

大家注意，在模板中我们使用 `a`，但是在 `data` 属性中并没有定义这个属性，这个时候我们就能够得到以上报错信息：

大家可能比较疑惑的是为什么会这样，其实我们后面讲到渲染函数的时候你自然就知道了，不过现在大家可以先看一下，打开 `core/instance/render.js` 文件，找到 `Vue.prototype._render` 方法，里面有这样的代码：

```js
vnode = render.call(vm._renderProxy, vm.$createElement)
```

可以发现，调用 `render` 函数的时候，使用 `call` 方法指定了函数的执行环境为 `vm._renderProxy`，渲染函数长成什么样呢？还是以上面的例子为例，我们可以通过打印 `vm.$options.render` 查看，所以它长成这样：

```js
vm.$options.render = function () {
    // render 函数的 this 指向实例的 _renderProxy
    with(this){
        return _c('div', [_v(_s(a))])   // 在这里访问 a，相当于访问 vm._renderProxy.a
    }
}
```

从上面的代码可以发现，显然函数使用 `with` 语句块指定了内部代码的执行环境为 `this`，由于 `render` 函数调用的时候使用 `call` 指定了其 `this` 指向为 `vm._renderProxy`，所以 `with` 语句块内代码的执行环境就是 `vm._renderProxy`，所以在 `with` 语句块内访问 `a` 就相当于访问 `vm._renderProxy` 的 `a` 属性，前面我们提到过 `with` 语句块内访问变量将会被 `Proxy` 的 `has` 代理所拦截，所以自然就执行了 `has` 函数内的代码。最终通过 `warnNonPresent` 打印警告信息给我们，所以这个代理的作用就是为了在开发阶段给我们一个友好而准确的提示。

我们理解了 `hasHandler`，但是还有一个 `getHandler`，这个代理将会在判断条件：

```js
options.render && options.render._withStripped
```

为真的情况下被使用，那这个条件什么时候成立呢？其实 `_withStripped` 只在 `test/unit/features/instance/render-proxy.spec.js` 文件中出现过，该文件有这样一段代码：

```js
it('should warn missing property in render fns without `with`', () => {
    const render = function (h) {
        // 这里访问了 a
        return h('div', [this.a])
    }
    // 在这里将 render._withStripped 设置为 true
    render._withStripped = true
    new Vue({
        render
    }).$mount()
    // 应该得到警告
    expect(`Property or method "a" is not defined`).toHaveBeenWarned()
})
```

这个时候就会触发 `getHandler` 设置的 `get` 拦截，`getHandler` 代码如下：

```js
const getHandler = {
    get (target, key) {
        if (typeof key === 'string' && !(key in target)) {
            warnNonPresent(target, key)
        }
        return target[key]
    }
}
```

其最终实现的效果无非就是检测到访问的属性不存在就给你一个警告。但我们也提到了，只有当 `render` 函数的 `_withStripped` 为真的时候，才会给出警告，但是 `render._withStripped` 又只有写测试的时候出现过，也就是说需要我们手动设置其为 `true` 才会得到提示，否则是得不到的，比如：

```js
const render = function (h) {
    return h('div', [this.a])
}

var vm = new Vue({
    el: '#app',
    render,
    data: {
        test: 1
    }
})
```

上面的代码由于 `render` 函数是我们手动书写的，所以 `render` 函数并不会被包裹在 `with` 语句块内，当然也就触发不了 `has` 拦截，但是由于 `render._withStripped` 也未定义，所以也不会被 `get` 拦截，那这个时候我们虽然访问了不存在的 `this.a`，但是却得不到警告，想要得到警告我们需要手动设置 `render._withStripped` 为 `true`：

```js
const render = function (h) {
    return h('div', [this.a])
}
render._withStripped = true

var vm = new Vue({
    el: '#app',
    render,
    data: {
        test: 1
    }
})
```

为什么会这么设计呢？因为在使用 `webpack` 配合 `vue-loader` 的环境中， `vue-loader` 会借助 [`vuejs@component-compiler-utils`](https://github.com/vuejs/component-compiler-utils) 将 `template` 编译为不使用 `with` 语句包裹的遵循严格模式的 JavaScript，并为编译后的 `render` 方法设置 `render._withStripped = true`。在不使用 `with` 语句的 `render` 方法中，模板内的变量都是通过属性访问操作 `vm['a']` 或 `vm.a` 的形式访问的，从前文中我们了解到 `Proxy` 的 `has` 无法拦截属性访问操作，所以这里需要使用 `Proxy` 中可以拦截到属性访问的 `get`，同时也省去了 `has` 中的全局变量检查(全局变量的访问不会被 `get` 拦截)。

现在，我们基本知道了 `initProxy` 的目的，就是设置渲染函数的作用域代理，其目的是为我们提供更好的提示信息。但是我们忽略了一些细节没有讲清楚，回到下面这段代码：

```js
// has 变量是真实经过 in 运算符得来的结果
const has = key in target
// 如果 key 在 allowedGlobals 之内，或者 key 是以下划线 _ 开头的字符串，则为真
const isAllowed = allowedGlobals(key) || (typeof key === 'string' && key.charAt(0) === '_')
// 如果 has 和 isAllowed 都为假，使用 warnNonPresent 函数打印错误
if (!has && !isAllowed) {
    warnNonPresent(target, key)
}
```

上面这段代码中的 `if` 语句的判断条件是 `(!has && !isAllowed)`，其中 `!has` 我们可以理解为**你访问了一个没有定义在实例对象上(或原型链上)的属性**，所以这个时候提示错误信息是合理，但是即便 `!has` 成立也不一定要提示错误信息，因为必须要满足 `!isAllowed`，也就是说当你访问了一个**虽然不在实例对象上(或原型链上)的属性，但如果你访问的是全局对象**那么也是被允许的。这样我们就可以在模板中使用全局对象了，如：

```html
<template>
  {{Number(b) + 2}}
</template>
```

其中 `Number` 为全局对象，如果去掉 `!isAllowed` 这个判断条件，那么上面模板的写法将会得到警告信息。除了允许使用全局对象之外，还允许以 `_` 开头的属性，这么做是由于渲染函数中会包含很多以 `_` 开头的内部方法，如之前我们查看渲染函数时遇到的 `_c`、`_v` 等等。

最后对于 `proxy.js` 文件内的代码，还有一段是我们没有讲过的，就是下面这段：

```js
if (hasProxy) {
    // isBuiltInModifier 函数用来检测是否是内置的修饰符
    const isBuiltInModifier = makeMap('stop,prevent,self,ctrl,shift,alt,meta,exact')
    // 为 config.keyCodes 设置 set 代理，防止内置修饰符被覆盖
    config.keyCodes = new Proxy(config.keyCodes, {
        set (target, key, value) {
            if (isBuiltInModifier(key)) {
                warn(`Avoid overwriting built-in modifier in config.keyCodes: .${key}`)
                return false
            } else {
                target[key] = value
                return true
            }
        }
    })
}
```

上面的代码首先检测宿主环境是否支持 `Proxy`，如果支持的话才会执行里面的代码，内部的代码首先使用 `makeMap` 函数生成一个 `isBuiltInModifier` 函数，该函数用来检测给定的值是否是内置的事件修饰符，我们知道在 `Vue` 中我们可以使用事件修饰符很方便地做一些工作，比如阻止默认事件等。

然后为 `config.keyCodes` 设置了 `set` 代理，其目的是防止开发者在自定义键位别名的时候，覆盖了内置的修饰符，比如：

```js
Vue.config.keyCodes.shift = 16
```

由于 `shift` 是内置的修饰符，所以上面这句代码将会得到警告

#### 初始化之initLifecycle

`_init` 函数在执行完 `initProxy` 之后，执行的就是 `initLifecycle` 函数：

```js
vm._self = vm
initLifecycle(vm)
```

在 `initLifecycle` 函数执行之前，执行了 `vm._self = vm` 语句，这句话在 `Vue` 实例对象 `vm` 上添加了 `_self` 属性，指向真实的实例本身。注意 `vm._self` 和 `vm._renderProxy` 不同，首先在用途上来说寓意是不同的，另外 `vm._renderProxy` 有可能是一个代理对象，即 `Proxy` 实例。

接下来执行的才是 `initLifecycle` 函数，同时将当前 `Vue` 实例 `vm` 作为参数传递。打开 `core/instance/lifecycle.js` 文件找到 `initLifecycle` 函数，如下

```js
export function initLifecycle (vm: Component) {
  // 定义 options，它是 vm.$options 的引用，后面的代码使用的都是 options 常量
  const options = vm.$options

  // locate first non-abstract parent
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

上面代码是 `initLifecycle` 函数的全部内容，首先定义 `options` 常量，它是 `vm.$options` 的引用。接着将执行下面这段代码：

```js
// locate first non-abstract parent (查找第一个非抽象的父组件)
// 定义 parent，它引用当前实例的父实例
let parent = options.parent
// 如果当前实例有父组件，且当前实例不是抽象的
if (parent && !options.abstract) {
  // 使用 while 循环查找第一个非抽象的父组件
  while (parent.$options.abstract && parent.$parent) {
    parent = parent.$parent
  }
  // 经过上面的 while 循环后，parent 应该是一个非抽象的组件，将它作为当前实例的父级，所以将当前实例 vm 添加到父级的 $children 属性里
  parent.$children.push(vm)
}

// 设置当前实例的 $parent 属性，指向父级
vm.$parent = parent
// 设置 $root 属性，有父级就是用父级的 $root，否则 $root 指向自身
vm.$root = parent ? parent.$root : vm
```

上面代码的作用可以用一句话总结：*“将当前实例添加到父实例的 `$children` 属性里，并设置当前实例的 `$parent` 指向父实例”*。那么要实现这个目标首先要寻找到父级才行，那么父级的来源是哪里呢？就是这句话：

```js
// 定义 parent，它引用当前实例的父组件
let parent = options.parent
```

通过读取 `options.parent` 获取父实例，但是问题来了，我们知道 `options` 是 `vm.$options` 的引用，所以这里的 `options.parent` 相当于 `vm.$options.parent`，那么 `vm.$options.parent` 从哪里来？比如下面的例子：

```js
// 子组件本身并没有指定 parent 选项
var ChildComponent = {
  created () {
    // 但是在子组件中访问父实例，能够找到正确的父实例引用
    console.log(this.$options.parent)
  }
}

var vm = new Vue({
    el: '#app',
    components: {
      // 注册组件
      ChildComponent
    },
    data: {
        test: 1
    }
})
```

我们知道 `Vue` 给我们提供了 `parent` 选项，使得我们可以手动指定一个组件的父实例，但在上面的例子中，我们并没有手动指定 `parent` 选项，但是子组件依然能够正确地找到它的父实例，这说明 `Vue` 在寻找父实例的时候是自动检测的。那它是怎么做的呢？目前不准备给大家介绍，因为时机还不够成熟，现在讲大家很容易懵，不过可以给大家看一段代码，打开 `core/vdom/create-component.js` 文件，里面有一个函数叫做 `createComponentInstanceForVnode`，如下：

```js
export function createComponentInstanceForVnode (
  vnode: any, // we know it's MountedComponentVNode but flow doesn't
  parent: any, // activeInstance in lifecycle state
  parentElm?: ?Node,
  refElm?: ?Node
): Component {
  const vnodeComponentOptions = vnode.componentOptions
  const options: InternalComponentOptions = {
    _isComponent: true,
    parent,
    propsData: vnodeComponentOptions.propsData,
    _componentTag: vnodeComponentOptions.tag,
    _parentVnode: vnode,
    _parentListeners: vnodeComponentOptions.listeners,
    _renderChildren: vnodeComponentOptions.children,
    _parentElm: parentElm || null,
    _refElm: refElm || null
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  return new vnodeComponentOptions.Ctor(options)
}
```

这个函数是干什么的呢？我们知道当我们注册一个组件的时候，还是拿上面的例子，如下：

```js
// 子组件
var ChildComponent = {
  created () {
    console.log(this.$options.parent)
  }
}

var vm = new Vue({
    el: '#app',
    components: {
      // 注册组件
      ChildComponent
    },
    data: {
        test: 1
    }
})
```

上面的代码中，我们的子组件 `ChildComponent` 说白了就是一个 `json` 对象，或者叫做组件选项对象，在父组件的 `components` 选项中把这个子组件选项对象注册了进去，实际上在 `Vue` 内部，会首先以子组件选项对象作为参数通过 `Vue.extend` 函数创建一个子类出来，然后再通过实例化子类来创建子组件，而 `createComponentInstanceForVnode` 函数的作用，在这里大家就可以简单理解为实例化子组件，只不过这个过程是在虚拟DOM的 `patch` 算法中进行的，我们后边会详细去讲。我们看 `createComponentInstanceForVnode` 函数内部有这样一段代码：

```js
const options: InternalComponentOptions = {
  _isComponent: true,
  parent,
  propsData: vnodeComponentOptions.propsData,
  _componentTag: vnodeComponentOptions.tag,
  _parentVnode: vnode,
  _parentListeners: vnodeComponentOptions.listeners,
  _renderChildren: vnodeComponentOptions.children,
  _parentElm: parentElm || null,
  _refElm: refElm || null
}
```

这是实例化子组件时的组件选项，我们发现，第二个值就是 `parent`，那么这个 `parent` 是谁呢？它是 `createComponentInstanceForVnode` 函数的形参，所以我们需要找到 `createComponentInstanceForVnode` 函数是在哪里调用的，它的调用位置就在 `core/vdom/create-component.js` 文件内的 `componentVNodeHooks` 钩子对象的 `init` 钩子函数内，如下：

```js
// hooks to be invoked on component VNodes during patch
const componentVNodeHooks = {
  init (
    vnode: VNodeWithData,
    hydrating: boolean,
    parentElm: ?Node,
    refElm: ?Node
  ): ?boolean {
    if (!vnode.componentInstance || vnode.componentInstance._isDestroyed) {
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance,
        parentElm,
        refElm
      )
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    } else if (vnode.data.keepAlive) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    }
  },

  prepatch (oldVnode: MountedComponentVNode, vnode: MountedComponentVNode) {
    ...
  },

  insert (vnode: MountedComponentVNode) {
    ...
  },

  destroy (vnode: MountedComponentVNode) {
    ...
  }
}
```

在 `init` 函数内有这样一段代码：

```js
const child = vnode.componentInstance = createComponentInstanceForVnode(
  vnode,
  activeInstance,
  parentElm,
  refElm
)
```

第二个参数 `activeInstance` 就是我们要找的 `parent`，那么 `activeInstance` 是什么呢？根据文件顶部的 `import` 语句可知，`activeInstance` 来自于 `core/instance/lifecycle.js` 文件，也就是我们正在看的 `initLifecycle` 函数的上面，如下：

```js
export let activeInstance: any = null
```

这个变量将总是保存着当前正在渲染的实例的引用，所以它就是当前实例 `components` 下注册的子组件的父实例，所以 `Vue` 实际上就是这样做到自动侦测父级的。

所以现在我们初步知道了 `options.parent` 值的来历，且知道了它的值指向父实例，那么接下来我们继续看代码，还是这段代码：

```js
// 定义 parent，它引用当前实例的父组件
let parent = options.parent
// 如果当前实例有父组件，且当前实例不是抽象的
if (parent && !options.abstract) {
  // 使用 while 循环查找第一个非抽象的父组件
  while (parent.$options.abstract && parent.$parent) {
    parent = parent.$parent
  }
  // 经过上面的 while 循环后，parent 应该是一个非抽象的组件，将它作为当前实例的父级，所以将当前实例 vm 添加到父级的 $children 属性里
  parent.$children.push(vm)
}
```

拿到父实例 `parent` 之后，进入一个判断分支，条件是：`parent && !options.abstract`，即*父实例存在，且当前实例不是抽象的*，这里大家可能会有疑问：*什么是抽象的实例*？实际上 `Vue` 内部有一些选项是没有暴露给我们的，就比如这里的 `abstract`，通过设置这个选项为 `true`，可以指定该组件是抽象的，那么通过该组件创建的实例也都是抽象的，比如：

```js
AbsComponents = {
  abstract: true,
  created () {
    console.log('我是一个抽象的组件')
  }
}
```

抽象的组件有什么特点呢？一个最显著的特点就是它们一般不渲染真实DOM，这么说大家可能不理解，我举个例子大家就明白了，我们知道 `Vue` 内置了一些全局组件比如 `keep-alive` 或者 `transition`，我们知道这两个组件它是不会渲染DOM至页面的，但他们依然给我提供了很有用的功能。所以他们就是抽象的组件，我们可以查看一下它的源码，打开 `core/components/keep-alive.js` 文件，你能看到这样的代码：

```js
export default {
  name: 'keep-alive',
  abstract: true,
  ...
}
```

可以发现，它使用 `abstract` 选项来声明这是一个抽象组件。除了不渲染真实DOM，抽象组件还有一个特点，就是它们不会出现在父子关系的路径上。这么设计也是合理的，这是由它们的性质所决定的。

如果 `options.abstract` 为真，那说明当前实例是抽象的，所以并不会走 `if` 分支的代码，所以会跳过 `if` 语句块直接设置 `vm.$parent` 和 `vm.$root` 的值。跳过 `if` 语句块的结果将导致该抽象实例不会被添加到父实例的 `$children` 中。如果 `options.abstract` 为假，那说明当前实例不是抽象的，是一个普通的组件实例，这个时候就会走 `while` 循环，那么这个 `while` 循环是干嘛的呢？我们前面说过，抽象的组件是不能够也不应该作为父级的，所以 `while` 循环的目的就是沿着父实例链逐层向上寻找到第一个不抽象的实例作为 `parent`（父级）。并且在找到父级之后将当前实例添加到父实例的 `$children` 属性中，这样最终的目的就达成了。

在上面这段代码执行完毕之后，`initLifecycle` 函数还负责在当前实例上添加一些属性，即后面要执行的代码：

```js
vm.$children = []
vm.$refs = {}

vm._watcher = null
vm._inactive = null
vm._directInactive = false
vm._isMounted = false
vm._isDestroyed = false
vm._isBeingDestroyed = false
```

其中 `$children` 和 `$refs` 都是我们熟悉的实例属性，他们都在 `initLifecycle` 函数中被初始化，其中 `$children` 被初始化为一个数组，`$refs` 被初始化为一个空 `json` 对象，除此之外，还定义了一些内部使用的属性，大家先混个脸熟，在后面的分析中自然会知道他们的用途，但是不要忘了，既然这些属性是在 `initLifecycle` 函数中定义的，那么自然会与生命周期有关

#### 初始化之initEvents

```js
export function initEvents (vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```

首先在 `vm` 实例对象上添加两个实例属性 `_events` 和 `_hasHookEvent`，其中 `_events` 被初始化为一个空对象，`_hasHookEvent` 的初始值为 `false`。之后将执行这段代码：

```js
// init parent attached events
const listeners = vm.$options._parentListeners
if (listeners) {
  updateComponentListeners(vm, listeners)
}
```

大家肯定还是有这个疑问：`vm.$options._parentListeners` 这个 `_parentListeners` 是哪里来的？细心的同学可能已经注意到了，我们之前看过一个函数叫做 `createComponentInstanceForVnode`，他在 `core/vdom/create-component.js` 文件中，如下：

```js
export function createComponentInstanceForVnode (
  vnode: any, // we know it's MountedComponentVNode but flow doesn't
  parent: any, // activeInstance in lifecycle state
  parentElm?: ?Node,
  refElm?: ?Node
): Component {
  const vnodeComponentOptions = vnode.componentOptions
  const options: InternalComponentOptions = {
    _isComponent: true,
    parent,
    propsData: vnodeComponentOptions.propsData,
    _componentTag: vnodeComponentOptions.tag,
    _parentVnode: vnode,
    _parentListeners: vnodeComponentOptions.listeners,
    _renderChildren: vnodeComponentOptions.children,
    _parentElm: parentElm || null,
    _refElm: refElm || null
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  return new vnodeComponentOptions.Ctor(options)
}
```

我们发现 `_parentListeners` 也出现这里，也就是说在创建子组件实例的时候才会有这个参数选项，所以现在我们不做深入讨论，后面自然有机会

#### 初始化之initRender

```js
export function initRender (vm: Component) {
  vm._vnode = null // the root of the child tree
  vm._staticTrees = null // v-once cached trees
  const options = vm.$options
  const parentVnode = vm.$vnode = options._parentVnode // the placeholder node in parent tree
  const renderContext = parentVnode && parentVnode.context
  vm.$slots = resolveSlots(options._renderChildren, renderContext)
  vm.$scopedSlots = emptyObject
  // bind the createElement fn to this instance
  // so that we get proper render context inside it.
  // args order: tag, data, children, normalizationType, alwaysNormalize
  // internal version is used by render functions compiled from templates
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)

  // $attrs & $listeners are exposed for easier HOC creation.
  // they need to be reactive so that HOCs using them are always updated
  const parentData = parentVnode && parentVnode.data

  /* istanbul ignore else */
  if (process.env.NODE_ENV !== 'production') {
    defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, () => {
      !isUpdatingChildComponent && warn(`$attrs is readonly.`, vm)
    }, true)
    defineReactive(vm, '$listeners', options._parentListeners || emptyObject, () => {
      !isUpdatingChildComponent && warn(`$listeners is readonly.`, vm)
    }, true)
  } else {
    defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
    defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
  }
}
```

首先在 `Vue` 实例对象上添加两个实例属性，即 `_vnode` 和 `_staticTrees`：

```js
vm._vnode = null // the root of the child tree
vm._staticTrees = null // v-once cached trees
```

并且这两个属性都被初始化为 `null`，它们会在合适的地方被赋值并使用，到时候我们再讲其作用，现在我们暂且不介绍这两个属性的作用，你只要知道这两句话仅仅是在当前实例对象上添加了两个属性就行了。

接着是这样一段代码：

```js
const options = vm.$options
const parentVnode = vm.$vnode = options._parentVnode // the placeholder node in parent tree
const renderContext = parentVnode && parentVnode.context
vm.$slots = resolveSlots(options._renderChildren, renderContext)
vm.$scopedSlots = emptyObject
```

上面这段代码从表面上看很复杂，可以明确地告诉大家，如果你看懂了上面这段代码就意味着你已经知道了 `Vue` 是如何解析并处理 `slot` 的了。由于上面这段代码涉及内部选项比较多如：`options._parentVnode`、`options._renderChildren` 甚至 `parentVnode.context`，这些内容牵扯的东西比较多，现在大家对 `Vue` 的储备还不够，所以我们会在本节的最后阶段补讲，那个时候相信大家理解起来要容易多了。

不讲归不讲，但是有一些事儿还是要讲清楚的，比如上面这段代码无论它处理的是什么内容，其结果都是在 `Vue` 当前实例对象上添加了三个实例属性：

```js
vm.$vnode
vm.$slots
vm.$scopedSlots
```

我们把这些属性都整理到 **Vue实例的设计**中。

再往下是这段代码：

```js
// bind the createElement fn to this instance
// so that we get proper render context inside it.
// args order: tag, data, children, normalizationType, alwaysNormalize
// internal version is used by render functions compiled from templates
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
// normalization is always applied for the public version, used in
// user-written render functions.
vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
```

这段代码在 `Vue` 实例对象上添加了两个方法：`vm._c` 和 `vm.$createElement`，这两个方法实际上是对内部函数 `createElement` 的包装。其中 `vm.$createElement` 相信手写过渲染函数的同学都比较熟悉，如下代码：

```js
render: function (createElement) {
  return createElement('h2', 'Title')
}
```

我们知道，渲染函数的第一个参数是 `createElement` 函数，该函数用来创建虚拟节点，实际上你也完全可以这么做：

```js
render: function () {
  return this.$createElement('h2', 'Title')
}
```

上面两段代码是完全等价的。而对于 `vm._c` 方法，则用于编译器根据模板字符串生成的渲染函数的。`vm._c` 和 `vm.$createElement` 的不同之处就在于调用 `createElement` 函数时传递的第六个参数不同，至于这么做的原因，我们放到后面讲解。有一点需要注意，即 `$createElement` 看上去像对外暴露的接口，但其实文档上并没有体现。

再往下，就是 `initRender` 函数的最后一段代码了：

```js
// $attrs & $listeners are exposed for easier HOC creation.
// they need to be reactive so that HOCs using them are always updated
const parentData = parentVnode && parentVnode.data

/* istanbul ignore else */
if (process.env.NODE_ENV !== 'production') {
  defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, () => {
    !isUpdatingChildComponent && warn(`$attrs is readonly.`, vm)
  }, true)
  defineReactive(vm, '$listeners', options._parentListeners || emptyObject, () => {
    !isUpdatingChildComponent && warn(`$listeners is readonly.`, vm)
  }, true)
} else {
  defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
  defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
}
```

上面的代码主要作用就是在 `Vue` 实例对象上定义两个属性：`vm.$attrs` 以及 `vm.$listeners`。这两个属性在 `Vue` 的文档中是有说明的，由于这两个属性的存在使得在 `Vue` 中创建高阶组件变得更容易。

我们注意到，在为实例对象定义 `$attrs` 属性和 `$listeners` 属性时，使用了 `defineReactive` 函数，该函数的作用就是为一个对象定义响应式的属性，所以 `$attrs` 和 `$listeners` 这两个属性是响应式的，至于 `defineReactive` 函数的讲解，我们会放到 `Vue` 的响应系统中讲解。

另外，上面的代码中有一个对环境的判断，在非生产环境中调用 `defineReactive` 函数时传递的第四个参数是一个函数，实际上这个函数是一个自定义的 `setter`，这个 `setter` 会在你设置 `$attrs` 或 `$listeners` 属性时触发并执行。以 `$attrs` 属性为例，当你试图设置该属性时，会执行该函数：

```js
() => {
  !isUpdatingChildComponent && warn(`$attrs is readonly.`, vm)
}
```

可以看到，当 `!isUpdatingChildComponent` 成立时，会提示你 `$attrs` 是只读属性，你不应该手动设置它的值。同样的，对于 `$listeners` 属性也做了这样的处理。

这里使用到了 `isUpdatingChildComponent` 变量，根据引用关系，该变量来自于 `lifecycle.js` 文件，打开 `lifecycle.js` 文件，可以发现有三个地方使用了这个变量：

```js
// 定义 isUpdatingChildComponent，并初始化为 false
export let isUpdatingChildComponent: boolean = false

// 省略中间代码 ...

export function updateChildComponent (
  vm: Component,
  propsData: ?Object,
  listeners: ?Object,
  parentVnode: MountedComponentVNode,
  renderChildren: ?Array<VNode>
) {
  if (process.env.NODE_ENV !== 'production') {
    isUpdatingChildComponent = true
  }

  // 省略中间代码 ...

  // update $attrs and $listeners hash
  // these are also reactive so they may trigger child update if the child
  // used them during render
  vm.$attrs = parentVnode.data.attrs || emptyObject
  vm.$listeners = listeners || emptyObject

  // 省略中间代码 ...

  if (process.env.NODE_ENV !== 'production') {
    isUpdatingChildComponent = false
  }
}
```

上面代码是简化后的，可以发现 `isUpdatingChildComponent` 初始值为 `false`，只有当 `updateChildComponent` 函数开始执行的时候会被更新为 `true`，当 `updateChildComponent` 执行结束时又将 `isUpdatingChildComponent` 的值还原为 `false`，这是因为 `updateChildComponent` 函数需要更新实例对象的 `$attrs` 和 `$listeners` 属性，所以此时是不需要提示 `$attrs` 和 `$listeners` 是只读属性的。

最后，对于大家来讲，现在了解这些知识就足够了，至于 `$attrs` 和 `$listeners` 这两个属性的值到底是什么，等我们讲解虚拟DOM的时候再回来说明，这样大家更容易理解

#### 生命周期钩子的实现方式

在 `initRender` 函数执行完毕后，是这样一段代码：

```js
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

可以发现，`initInjections(vm)`、`initState(vm)` 以及 `initProvide(vm)` 被包裹在两个 `callHook` 函数调用的语句中。那么 `callHook` 函数的作用是什么呢？正如它的名字一样，`callHook` 函数的作用是调用生命周期钩子函数。根据引用关系可知 `callHook` 函数来自于 `lifecycle.js` 文件，打开该文件找到 `callHook` 函数如下：

```js
export function callHook (vm: Component, hook: string) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget()
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}
```

大家可能注意到了 `callHook` 函数体的代码以 `pushTarget()` 开头，并以 `popTarget()` 结尾，这里我们暂且不讲这么做的目的，这其实是为了避免在某些生命周期钩子中使用 `props` 数据导致收集冗余的依赖，我们在 `Vue` 响应系统的章节会回过头来仔细给大家讲解。下面我们开始分析 `callHook` 函数的代码的中间部分，首先获取要调用的生命周期钩子：

```js
const handlers = vm.$options[hook]
```

比如 `callHook(vm, created)`，那么上面的代码就相当于：

```js
const handlers = vm.$options.created
```

在 [Vue选项的合并](http://localhost:8080/vue-design/art/5vue-merge.html) 一节中我们讲过，对于生命周期钩子选项最终会被合并处理成一个数组，所以得到的 `handlers` 就是对应生命周期钩子的数组。接着执行的是这段代码：

```js
if (handlers) {
  for (let i = 0, j = handlers.length; i < j; i++) {
    try {
      handlers[i].call(vm)
    } catch (e) {
      handleError(e, vm, `${hook} hook`)
    }
  }
}
```

由于开发者在编写组件时未必会写生命周期钩子，所以获取到的 `handlers` 可能不存在，所以使用 `if` 语句进行判断，只有当 `handlers` 存在的时候才对 `handlers` 进行遍历，`handlers` 数组的元素就是生命周期钩子函数，所以直接执行即可：

```js
handlers[i].call(vm)
```

为了保证生命周期钩子函数内可以通过 `this` 访问实例对象，所以使用 `.call(vm)` 执行这些函数。另外由于生命周期钩子函数的函数体是开发者编写的，为了捕获可能出现的错误，使用 `try...catch` 语句块，并在 `catch` 语句块内使用 `handleError` 处理错误信息

我们回过头来再看一下这段代码：

```js
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

现在大家应该知道，`beforeCreate` 以及 `created` 这两个生命周期钩子的调用时机了。其中 `initState` 包括了：`initProps`、`initMethods`、`initData`、`initComputed` 以及 `initWatch`。所以当 `beforeCreate` 钩子被调用时，所有与 `props`、`methods`、`data`、`computed` 以及 `watch` 相关的内容都不能使用，当然了 `inject/provide` 也是不可用的。

作为对立面，`created` 生命周期钩子则恰恰是等待 `initInjections`、`initState` 以及 `initProvide` 执行完毕之后才被调用，所以在 `created` 钩子中，是完全能够使用以上提到的内容的。但由于此时还没有任何挂载的操作，所以在 `created` 中是不能访问DOM的，即不能访问 `$el`。

最后我们注意到 `callHook` 函数的最后有这样一段代码：

```js
if (vm._hasHookEvent) {
  vm.$emit('hook:' + hook)
}
```

其中 `vm._hasHookEvent` 是在 `initEvents` 函数中定义的，它的作用是判断是否存在**生命周期钩子的事件侦听器**，初始化值为 `false` 代表没有，当组件检测到存在**生命周期钩子的事件侦听器**时，会将 `vm._hasHookEvent` 设置为 `true`。那么问题来了，什么叫做**生命周期钩子的事件侦听器**呢？大家可能不知道，其实 `Vue` 是可以这么玩儿的：

```html
<child
  @hook:beforeCreate="handleChildBeforeCreate"
  @hook:created="handleChildCreated"
  @hook:mounted="handleChildMounted"
  @hook:生命周期钩子
 />
```

如上代码可以使用 `hook:` 加 `生命周期钩子名称` 的方式来监听组件相应的生命周期事件。这是 `Vue` 官方文档上没有体现的，但你确实可以这么用，不过除非你对 `Vue` 非常了解，否则不建议使用。

正是为了实现这个功能，才有了这段代码：

```js
if (vm._hasHookEvent) {
  vm.$emit('hook:' + hook)
}
```

另外大家可能会疑惑，`vm._hasHookEvent` 是在什么时候被设置为 `true` 的呢？或者换句话说，`Vue` 是如何检测是否存在生命周期事件侦听器的呢？对于这个问题等我们在讲解 `Vue` 事件系统时自然会知道

#### Vue 的初始化之 initState

实际上根据如下代码所示：

```js
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

可以看到在 `initState` 函数执行之前，先执行了 `initInjections` 函数，也就是说 `inject` 选项要更早被初始化，不过由于初始化 `inject` 选项的时候涉及到 `defineReactive` 函数，并且调用了 `toggleObserving` 函数操作了用于控制是否应该转换为响应式属性的状态标识 `observerState.shouldConvert`，所以我们决定先讲解 `initState`，之后再来讲解 `initInjections` 和 `initProvide`，这才是一个合理的顺序，并且从 `Vue` 的时间线上来看 `inject/provide` 选项确实是后来才添加的。

所以我们打开 `core/instance/state.js` 文件，找到 `initState` 函数，如下：

```js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

以上是 `initState` 函数的全部代码，我们慢慢来看，首先在 `Vue` 实例对象添加一个属性 `vm._watchers = []`，其初始值是一个数组，这个数组将用来存储所有该组件实例的 `watcher` 对象。随后定义了常量 `opts`，它是 `vm.$options` 的引用。接着执行了如下两句代码：

```js
if (opts.props) initProps(vm, opts.props)
if (opts.methods) initMethods(vm, opts.methods)
```

如果 `opts.props` 存在，即选项中有 `props`，那么就调用 `initProps` 初始化 `props` 选项。同样的，如果 `opts.methods` 存在，则调用 `initMethods` 初始化 `methods` 选项。

再往下执行的是这段代码：

```js
if (opts.data) {
  initData(vm)
} else {
  observe(vm._data = {}, true /* asRootData */)
}
```

首先判断 `data` 选项是否存在，如果存在则调用 `initData` 初始化 `data` 选项，如果不存在则直接调用 `observe` 函数观测一个空对象：`{}`。

最后执行的是如下这段代码：

```js
if (opts.computed) initComputed(vm, opts.computed)
if (opts.watch && opts.watch !== nativeWatch) {
  initWatch(vm, opts.watch)
}
```

采用同样的方式初始化 `computed` 选项，但是对于 `watch` 选项仅仅判断 `opts.watch` 是否存在是不够的，还要判断 `opts.watch` 是不是原生的 `watch` 对象。前面的章节中我们提到过，这是因为在 `Firefox` 中原生提供了 `Object.prototype.watch` 函数，所以即使没有 `opts.watch` 选项，如果在火狐浏览器中依然能够通过原型链访问到原生的 `Object.prototype.watch`。但这其实不是我们想要的结果，所以这里加了一层判断避免把原生 `watch` 函数误认为是我们预期的 `opts.watch` 选项。之后才会调用 `initWatch` 函数初始化 `opts.watch` 选项。

通过阅读 `initState` 函数，我们可以发现 `initState` 其实是很多选项初始化的汇总，包括：`props`、`methods`、`data`、`computed` 和 `watch` 等。并且我们注意到 `props` 选项的初始化要早于 `data` 选项的初始化，那么这是不是可以使用 `props` 初始化 `data` 数据的原因呢？答案是：“是的”。接下来我们就深入讲解这些初始化工作都做了什么事情。下一章节我们将重点讲解 `Vue` 初始化中的关键一步：**数据响应系统**。

### Vue 构造函数整理-原型

```js
// initMixin(Vue)    src/core/instance/init.js **************************************************
Vue.prototype._init = function (options?: Object) {}

// stateMixin(Vue)    src/core/instance/state.js **************************************************
Vue.prototype.$data
Vue.prototype.$props
Vue.prototype.$set = set
Vue.prototype.$delete = del
Vue.prototype.$watch = function (
  expOrFn: string | Function,
  cb: any,
  options?: Object
): Function {}

// eventsMixin(Vue)    src/core/instance/events.js **************************************************
Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}
Vue.prototype.$once = function (event: string, fn: Function): Component {}
Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}
Vue.prototype.$emit = function (event: string): Component {}

// lifecycleMixin(Vue)    src/core/instance/lifecycle.js **************************************************
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}

// renderMixin(Vue)    src/core/instance/render.js **************************************************
// installRenderHelpers 函数中
Vue.prototype._o = markOnce
Vue.prototype._n = toNumber
Vue.prototype._s = toString
Vue.prototype._l = renderList
Vue.prototype._t = renderSlot
Vue.prototype._q = looseEqual
Vue.prototype._i = looseIndexOf
Vue.prototype._m = renderStatic
Vue.prototype._f = resolveFilter
Vue.prototype._k = checkKeyCodes
Vue.prototype._b = bindObjectProps
Vue.prototype._v = createTextVNode
Vue.prototype._e = createEmptyVNode
Vue.prototype._u = resolveScopedSlots
Vue.prototype._g = bindObjectListeners
Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}

// core/index.js 文件中
Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// 在 runtime/index.js 文件中
Vue.prototype.__patch__ = inBrowser ? patch : noop
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}

// 在入口文件 entry-runtime-with-compiler.js 中重写了 Vue.prototype.$mount 方法
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  // ... 函数体
}
```

### Vue 构造函数整理-全局API

```js
// initGlobalAPI
Vue.config
Vue.util = {
	warn,
	extend,
	mergeOptions,
	defineReactive
}
Vue.set = set
Vue.delete = del
Vue.nextTick = nextTick
Vue.options = {
	components: {
		KeepAlive
		// Transition 和 TransitionGroup 组件在 runtime/index.js 文件中被添加
		// Transition,
    	// TransitionGroup
	},
	directives: Object.create(null),
	// 在 runtime/index.js 文件中，为 directives 添加了两个平台化的指令 model 和 show
	// directives:{
	//	model,
    //	show
	// },
	filters: Object.create(null),
	_base: Vue
}

// initUse ***************** global-api/use.js
Vue.use = function (plugin: Function | Object) {}

// initMixin ***************** global-api/mixin.js
Vue.mixin = function (mixin: Object) {}

// initExtend ***************** global-api/extend.js
Vue.cid = 0
Vue.extend = function (extendOptions: Object): Function {}

// initAssetRegisters ***************** global-api/assets.js
Vue.component =
Vue.directive =
Vue.filter = function (
  id: string,
  definition: Function | Object
): Function | Object | void {}

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'

// entry-runtime-with-compiler.js
Vue.compile = compileToFunctions
```

### Vue实例的设计

```js
// Vue.prototype._init
vm._uid = uid++     // 每个Vue实例都拥有一个唯一的 id
vm._isVue = true    // 这个表示用于避免Vue实例对象被观测(observed)
vm.$options         // 当前 Vue 实例的初始化选项，注意：这是经过 mergeOptions() 后的
vm._renderProxy = vm    // 渲染函数作用域代理
vm._self = vm       // 实例本身

// initLifecycle(vm)    src/core/instance/lifecycle.js **************************************************
vm.$parent = parent
vm.$root = parent ? parent.$root : vm

vm.$children = []
vm.$refs = {}

vm._watcher = null
vm._inactive = null
vm._directInactive = false
vm._isMounted = false
vm._isDestroyed = false
vm._isBeingDestroyed = false

// initEvents(vm)   src/core/instance/events.js **************************************************
vm._events = Object.create(null)
vm._hasHookEvent = false

// initRender(vm)   src/core/instance/render.js **************************************************
vm._vnode = null // the root of the child tree
vm._staticTrees = null // v-once cached trees

vm.$vnode
vm.$slots
vm.$scopedSlots

vm._c
vm.$createElement

vm.$attrs
vm.$listeners

// initState(vm)   src/core/instance/state.js **************************************************
vm._watchers = []
vm._data

// mountComponent()   src/core/instance/lifecycle.js
vm.$el

// initComputed()   src/core/instance/state.js
vm._computedWatchers = Object.create(null)

// initProps()    src/core/instance/state.js
vm._props = {}

// initProvide()    src/core/instance/inject.js
vm._provided
```

