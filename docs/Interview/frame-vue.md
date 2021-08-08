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