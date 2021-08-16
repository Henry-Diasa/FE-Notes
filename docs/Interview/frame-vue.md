### Vue基础使用

- 插值 （{{}}）
- 表达式 （只能是表达式，不能是js语句）
- v-html会覆盖子元素，有xss风险
- computed有缓存，data不变则不会重新计算
- watch如何实现深度监听（deep）
- watch监听引用类型，拿不到oldVal
- Class 和 style (可以使用数组形式和对象形式)
- 条件渲染 （v-if、v-show）及场景
  + v-if不会产生dom结构、v-show是通过样式display来进行控制的
  + 切换频繁的情况用v-show、反之v-if
- 列表渲染 
  + v-for可以遍历对象和数组
  + key的重要性
  + v-for和v-if不能一起使用（v-for的优先级更高一些）
- 事件
  + event参数，自定义参数（自定义参数在函数的前面，事件参数event放到最后，模板中通过$event传递）
  + 事件修饰符，按键修饰符
  + 事件被绑定到哪里 （当前dom元素）

- 表单
  + v-model 
- Vue组件使用
  + props 和 $emit
  + 组件间通信 - 自定义事件（EventBus Vue的实例，自定义事件注意解绑）
  + 组件生命周期 
    + 子组件先挂载/更新，然后才是父组件挂载/更新

### Vue高级特性

- 自定义v-model

  ```html
  <template>
    <div>
    	<CustomModel v-model="name">
    </div>
  </template>
  
  <script>
   export default {
      data() {
        return {
           name: ''
        }
      }
   }
  </script>
  
  // customModel
  <template>
    <div>
    	<input :value="text" @input="$emit('change', $event.target.value)">
    </div>
  </template>
  
  <script>
  	export default {
  	  model: {
  	    prop: 'text'
  	    event: 'change'
  	  },
  	  props: {
  	    text: String,
  	    default: ''
  	  }
  	}
  </script>
  ```

- $nextTick

  + Vue是异步渲染，$nextTick待DOM渲染完后再回调
  + data改变之后，DOM不会立刻渲染，页面渲染会将data 的修改做整合，多次data的修改只会渲染一次
  + $nextTick会在DOM渲染之后被触发，以获取最新的DOM节点

- Slot

  + 插槽、具名插槽、作用域插槽

- 动态、异步组件

  + 动态组件（`<component :is="component-name" />`）
  + 异步组件
    + import函数
    + 按需加载，异步加载大组件

- keep-alive

  + 缓存组件
  + 频繁切换，不需要重复渲染

- mixin

  + 多个组件有相同的逻辑，抽离出来
  + 变量来源不明确，不利于阅读
  + 多mixin可能会造成命名冲突

### Vuex使用

- action里面做异步操作

### Vue-router使用

- 路由模式（hash、H5 history）
  - history模式需要后端支持
- 路由配置（动态路由、懒加载）
  - path、component
  - component使用import实现懒加载

### Vue原理

- MVVM 
  + Model 
  + View
  + ViewModel
  + Model->View  通过数据绑定
  + View->Model  通过事件监听
- Vue响应式
  - 核心API - Object.defineProperty及缺点
    - 监听对象，监听数组
    - 复杂对象，深度监听
    - 递归监听，set值的时候也需要对设置的值进行observe, 深度监听，需要递归到底，一次性计算量大
    - 无法监听新增属性/删除属性(Vue.set Vue.delete)
    - 无法原生监听数组，需要特殊处理
- vDOM
  - js来模拟dom结构
  - 新旧dom对比，得出最小的更新范围，最后更新DOM
  - 数据驱动视图的模式下，有效控制DOM操作

- diff算法
  - 只比较同一层，不跨级比较
  - tag不相同，则直接删掉重建，不在深度比较
  - tag和key，两者都相同，则认为是相同节点，不再深度比较
- patch过程 [连接](https://juejin.cn/post/6844903897262194696)
  - 首先是不同的节点的条件分支判断来进行增删节点 （设置text节点、删除children节点等）
  - 新旧节点都具有children节点，将进行updateChildren
    - 定义四个索引 oldStartIdx、oldEndIdx、newStartIdx、newEndIdx
    - 头对比、尾对比、头尾对比、尾头对比。如果有相同的则将节点插入到指定的位置（oldStartIdx的前方）
    - 上面四种都不满足，则用新节点在老节点当中寻找，存在的话插入固定位置，不存在创建新节点同样插入到指定的位置
    - 最后`while(oldStartIdx<=oldEndIdx && newStartIdx<=newEndIdx)`不满足，退出循环
      - `oldStartIdx > oldEndIdx`说明老节点遍历完成，新节点比较多，多出来的节点创建出来并添加
      - `newStartIdx > newEndIdx`说明新节点遍历完成，老节点比较多，直接删除[oldStartIdx, oldEndIdx]区间的节点

- 模板编译

  - with语法
    - 改变`{}`内自由变量的查找规则，当做obj属性来查找
    - 如果找不到匹配的obj属性，就会报错
    - with慎用，它打破了作用域规则，易读性变差
  - 编译模板
    - 模板编译为render函数，执行render函数返回vnode
    - 基于vnode在执行patch和diff
    - 使用webpack vue-loader，会在开发环境下编译模板
  - 初次渲染过程
    - 解析模板为render函数（或在开发环境已完成，vue-loader）
    - 触发响应式，监听data属性getter setter
    - 执行render函数，生成vnode，patch（elem, vnode）
  - 更新过程
    - 修改data，触发setter
    - 重新执行render函数，生成newVnode
    - patch(vnode, newVnode)
  - 异步渲染
    - $nextTick

### 路由原理

- 路由模式
  - hash
  - history
- hash的特点
  - hash变化会触发网页跳转，即浏览器的前进，后退
  - hash变化不会刷新页面，SPA必须的特点
  - hash永远不会提交到server端
  - onHashChange 监听
- H5 history
  - 用url规范的路由，但跳转时不刷新页面
  - History.pushState
  - Window.onpopstate
  - 需要后端配合

### 面试题目

- -show 和 v-if 的区别

  > v-show通过CSS display 控制显示和隐藏
  >
  > v-if组件真正的渲染和销毁，而不是显示和隐藏
  >
  > 频繁切换显示状态用v-show,否则用v-if

- 为何在v-for中使用key

  > 必须用key，且不能是index和random
  >
  > diff算法中通过tag和key来判断，是否是sameNode
  >
  > 减少渲染次数，提升渲染性能

- 描述vue组件生命周期（父子组件）

  > 单组件生命周期图
  >
  > 父子组件生命周期关系

- vue组件如何通信

  > 父子组件props和this.$emit
  >
  > 自定义事件 `event.$no`  ` event.$​​off` `event.$emit`
  >
  > vuex

- 组件渲染和更新的过程

- v-model的实现原理

  > Input 元素的value = this.name
  >
  > 绑定input事件 this.name = $event.target.value
  >
  > data更新触发re-render

- 对MVVM的理解

- Computed 有何特点

  > 缓存，data不变不会重新计算
  >
  > 提高性能

- 为何组件data必须是一个函数

  > 如果不是一个函数而是一个对象，那么组件之前共用的时候会造成数据之间互相影响

- ajax请求应该放在哪个生命周期

  > mounted
  >
  > JS是单线程的，ajax异步获取数据

- 如何将组件所有props传递给子组件

  > $props
  >
  > <User v-bind="$props">
  >
  > 细节知识点，优先级不高

- 多个组件有相同的逻辑，如何抽离

  > mixin
  >
  > 以及mixin的一些缺点

- 何时要使用异步组件

  > 加载大组件
  >
  > 路由异步加载

- 何时使用keep-alive

  > 缓存组件，不需要重复渲染
  >
  > 如多个静态tab页的切换
  >
  > 优化性能

- 何时需要使用beforeDestory

  > 解绑自定义事件 `event.$off`
  >
  > 清楚定时器
  >
  > 解绑自定义的DOM事件，如window.scroll等

- 什么是作用域插槽

- vuex中action和mutation有何区别

  > Action 中处理异步，mutation不可以
  >
  > mutation做原子操作
  >
  > action可以整合多个mutation

- Vue-router常用的路由模式

  > hash
  >
  > H5 history

- 配置Vue-router异步加载

  ```
  path: '/',
  component: () => import('../component/index')
  ```

- vnode描述dom结构

  > tag
  >
  > props
  >
  > Children

- 监听data变化的核心API

  > Object.defineProperty 
  >
  > 深度监听、监听数组

- Vue如何监听数组变化

  > Object.defineProperty 不能监听数组变化
  >
  > 重新定义原型，重写push pop等方法，实现监听
  >
  > Proxy可以原型支持监听数组变化

- 描述响应式原理

  > 监听data变化
  >
  > 组件渲染和更新的流程

- diff算法的时间复杂度

  > O(n)
  >
  > 在O(n^3)基础上做了一些调整

- diff算法过程

  > Patch(elem, vnode)  和 patch(vnode, newVnode)
  >
  > patchVnode 和 addVnodes 和 removeVnodes
  >
  > updateChildren

- Vue为何是异步渲染， $nextTick何用

  > 异步渲染（以及合并data修改），以提高渲染性能
  >
  > 在nextTick后可以获取最新的DOM

- 