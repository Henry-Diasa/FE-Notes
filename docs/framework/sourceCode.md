### 了解Vue项目

#### 目录及文件

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

