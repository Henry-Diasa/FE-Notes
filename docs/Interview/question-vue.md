####如何快速定位哪个组件出现性能问题

> 用 `timeline` 工具。 大意是通过 `timeline` 来查看每个函数的调用时常，定位出哪个函数的问题，从而能判断哪个组件出了问题



#### scoped样式穿透

**使用/deep/**

```
//Parent
<template>
<div class="wrap">
    <Child />
</div>
</template>

<style lang="scss" scoped>
.wrap /deep/ .box{
    background: red;
}
</style>

//Child
<template>
    <div class="box"></div>
</template>
```

**两个style标签**

```
//Parent
<template>
<div class="wrap">
    <Child />
</div>
</template>

<style lang="scss" scoped>
//其他样式
</style>
<style lang="scss">
.wrap .box{
    background: red;
}
</style>

//Child
<template>
    <div class="box"></div>
</template>
```

#### 设计一个比较友好的Header组件

> 当时的思路是头部(Header)一般分为左、中、右三个部分，分为三个区域来设计，中间为主标题，每个页面的标题肯定不同，所以可以通过vue props的方式做成可配置对外进行暴露，左侧大部分页面可能都是回退按钮，但是样式和内容不尽相同，右侧一般都是具有功能性的操作按钮，所以左右两侧可以通过vue slot插槽的方式对外暴露以实现多样化，同时也可以提供default slot默认插槽来统一页面风格

#### 既然Vue通过数据劫持可以精准探测数据变化,为什么还需要虚拟DOM进行diff检测差异

> 现代前端框架有两种方式侦测变化,一种是pull一种是push

- pull: 其代表为React,我们可以回忆一下React是如何侦测到变化的,我们通常会用setStateAPI显式更新,然后React会进行一层层的Virtual Dom Diff操作找出差异,然后Patch到DOM上,React从一开始就不知道到底是哪发生了变化,只是知道「有变化了」,然后再进行比较暴力的Diff操作查找「哪发生变化了」，另外一个代表就是Angular的脏检查操作。
- push: Vue的响应式系统则是push的代表,当Vue程序初始化的时候就会对数据data进行依赖的收集,一但数据发生变化,响应式系统就会立刻得知,因此Vue是一开始就知道是「在哪发生变化了」,但是这又会产生一个问题,如果你熟悉Vue的响应式系统就知道,通常一个绑定一个数据就需要一个Watcher,一但我们的绑定细粒度过高就会产生大量的Watcher,这会带来内存以及依赖追踪的开销,而细粒度过低会无法精准侦测变化,因此Vue的设计是选择中等细粒度的方案,在组件级别进行push侦测的方式,也就是那套响应式系统,通常我们会第一时间侦测到发生变化的组件,然后在组件内部进行Virtual Dom Diff获取更加具体的差异,而Virtual Dom Diff则是pull操作,Vue是push+pull结合的方式进行变化侦测的

#### Vue为什么没有类似于React中shouldComponentUpdate的生命周期

> 根本原因是Vue与React的变化侦测方式有所不同

- React是pull的方式侦测变化,当React知道发生变化后,会使用Virtual Dom Diff进行差异检测,但是很多组件实际上是肯定不会发生变化的,这个时候需要用shouldComponentUpdate进行手动操作来减少diff,从而提高程序整体的性能.
- Vue是pull+push的方式侦测变化的,在一开始就知道那个组件发生了变化,因此在push的阶段并不需要手动控制diff,而组件内部采用的diff方式实际上是可以引入类似于shouldComponentUpdate相关生命周期的,但是通常合理大小的组件不会有过量的diff,手动优化的价值有限,因此目前Vue并没有考虑引入shouldComponentUpdate这种手动优化的生命周期.

#### vue项目性能优化

**代码层面：**

- V-for 事件代理

- 合理使用 `v-if` 和 `v-show`
- 区分 `computed` 和 `watch` 的使用
- `v-for` 遍历为 `item` 添加 `key`
- `v-for` 遍历避免同时使用 `v-if`
- 通过 `addEventListener`添加的事件在组件销毁时要用 `removeEventListener` 手动移除这些事件的监听
- 图片懒加载
- 路由懒加载
- 第三方插件按需引入
- `SSR`服务端渲染，首屏加载速度快，`SEO`效果好
- 使用runtime版本
- Keep-alive

**Webpack 层面优化：**

- 对图片进行压缩
- 使用 `CommonsChunkPlugin` 插件提取公共代码
- 提取组件的 CSS
- 优化 `SourceMap`
- 构建结果输出分析，利用 `webpack-bundle-analyzer` 可视化分析工具

#### delete和Vue.delete删除数组的区别

- `delete`只是被删除的元素变成了 `empty/undefined` 其他的元素的键值还是不变。
- `Vue.delete`直接删除了数组 改变了数组的键值。

```
var a=[1,2,3,4]
var b=[1,2,3,4]
delete a[0]
console.log(a)  //[empty,2,3,4]
this.$delete(b,0)
console.log(b)  //[2,3,4]
```

#### 插槽与作用域插槽的区别

**插槽**

- 创建组件虚拟节点时，会将组件儿子的虚拟节点保存起来。当初始化组件时，通过插槽属性将儿子进行分类`{a:[vnode],b[vnode]}`
- 渲染组件时会拿对应的`slot` 属性的节点进行替换操作。（插槽的作用域为父组件）

**作用域插槽**

- 作用域插槽在解析的时候不会作为组件的孩子节点。会解析成函数，当子组件渲染时，会调用此函数进行渲染。
- 普通插槽渲染的作用域是父组件，作用域插槽的渲染作用域是当前子组件。





