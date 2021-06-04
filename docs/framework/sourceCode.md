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

