### react基本使用

- 获取变量  插值 `{}`

- 表达式

- 子元素

- class => className

- Style   `<p style={{fontSize: '30px'}}>xx</p>`

- 原生html

  ```js
  const rawHtmlData = {
     __html: '<span>测试</span>'
  }
  
  const rawHtmlElem = <div>
  	<p dangerouslySetInnerHTML={rawHtmlData}></p>
  </div>
  ```

- 条件判断

  - if else
  - 三元表达式
  - && || 

- 列表渲染

  - map key

- 事件

  - onClick  this 在constructor中bind

  - 绑定的事件也可以静态方法 

    ```js
    clickHandler = () => {
       console.log('ok')
    }
    ```

  - 获取event

    > 这个event不是原生的event，而是一个合成事件(SyntheticEvent)，模拟出来DOM事件所有能力。获取原生事件需要使用nativeEvent。所有的事件都被挂载到document上(react 16)。react17以后事件就绑定在root组件

    ```js
    clickHandler1 = (event) => {
       event.preventDefault()
       event.stopPropagation()
       
       console.log('event', event)
       console.log('event.__proto__.constructor', event.__proto__.constructor)
       
       console.log('nativeEvent', event.nativeEvent)
       console.log('nativeEvent target', event.nativeEvent.target)
       console.log('nativeEvent current target', event.nativeEvent.currentTarget) // document
    }
    ```

  - 传递自定义参数

    ```
    <div onClick={this.clickHandler.bind(this, 'name', 'age')}></div>
    
    
    clickHandler = (name, age, event) => {
       
    }
    ```

- 表单

  - 受控组件 (受state控制)
  - input textarea select 用value
  - Checkbox radio 用checked

- 组件使用

  - props传递数据
  - props传递事件
  - 类型检查（propTypes）

- setState

  - 不可变值

    > 不要直接修改 state。要通过setState来修改一个新的值
    >
    > 修改数组的值不能通过push、pop等改变原数组的操作，要返回一个新数组
    >
    > 对象类型一样 也需要重新设置一个新的对象

  - 可能是异步更新 (也可能是同步)

    > 默认是异步的，可以通过回调函数中获取最新的值
    >
    > 在setTimeout中是同步的
    >
    > 在自定义的DOM事件中是同步的

  - 可能会被合并

    ```js
    // 传入对象，会被合并  执行结果只一次  +1
    this.setState({
    	count: this.state.count + 1
    })
    this.setState({
    	count: this.state.count + 1
    })
    this.setState({
    	count: this.state.count + 1
    })
    
    // 传入函数 不会被合并 执行结果+3
    this.setState((prevState, props) => {
      return {
        count: prevState.count + 1
      }
    })
    
    this.setState((prevState, props) => {
      return {
        count: prevState.count + 1
      }
    })
    
    this.setState((prevState, props) => {
      return {
        count: prevState.count + 1
      }
    })
    ```

- 生命周期

  - 单组件生命周期 （componentWillmount、didMount、shouldComponentUpdate、didUpdate、getDerivedStateFromProps、getSnapshotBeforeUpdate）
  - 父子组件生命周期，和vue的一样

### react高级特性

#### 函数组件

- 纯函数，输入props，输出JSX
- 没有实例，没有生命周期，没有state
- 不能扩展其他方法

#### 非受控组件

- ref
- defaultValue defaultChecked
- 手动操作DOM元素

- 使用场景
  - 必须手动操作DOM元素，setState实现不了
  - 文件上传 `<input type=file>`
  - 某些富文本编辑器，需要传入DOM元素

#### Portals

- 组件默认会按照既定层次嵌套渲染
- 如何让组件渲染到父组件以外

```
ReactDOM.createPortal(
  <div>{this.props.children}</div>, // fixed的元素
  document.body
)
```

- 场景
  - Overflow: hidden
  - 父组件z-index值太小
  - fixed需要放在body第一层级

#### context

- 公共信息（语言、主题）如何传递给每个组件
- 用props太繁琐
- 跨组件之间通信  createContext  Provider Consumer

#### 异步组件

- import()
- React.lazy
- React.Suspense

```
const ContextDemo = React.lazy(() => import('./demo'))


render() {
   return <>
   	<React.Suspense fallback={<div>loading...</div>}>
   		<ContextDemo />
   	</React.Suspense>
   </>
}
```

#### 性能优化

- shouldComponentUpdate

  > react默认： 父组件有更新，子组件则无条件也更新
  >
  > 如果子组件的props值没有改变，但是依然会有无意义的更新，因此可以依靠SCU控制不进行更新
  >
  > 必须配合不可变值一起使用

  ```
  shouldComponentUpdate(nextProps, nextState) {
    if(nextState.count!==this.state.count) {
      return true
    }
  	return false
  }
  ```

- PureComponent 和 memo

  > PureComponent,SCU中实现了浅比较
  >
  > memo,函数组件中的PureComponent
  >
  > 浅比较已适用大部分情况（尽量不要做深度比较）

- 不可变值 immutable.js

  > 彻底拥抱 不可变值
  >
  > 基于共享数据（不是深拷贝），速度好
  >
  > 有一定的学习和迁移成本，按需使用

#### 公共逻辑的抽离

- Mixin，已被React弃用

- HOC

  - Redux connect 是高阶组件

    ```
    import {connect} from 'react-redux'
    
    const VisibleTodoList = connect(
    	mapStateToProps,
    	mapDispatchToProps
    )(TodoList)
    
    export default VisibleTodoList
    ```

- render props

  ```
  class Factory extends React.Component {
  	constructor() {
  	  this.state = {
  	    
  	  }
  	}
  	render() {
  	  return <div>{this.props.render(this.state)}</div>
  	}
  }
  
  const App = () => (
  	<Factory render={
  			(props) => <p>{props.a}{props.b}</p>
  	}>
  )
  ```

#### Redux使用

- store state

- reducer

- action

- 单向数据流

  - Dispatch(action)
  - Reducer -> newState
  - subscribe触发通知

- react-redux

  - Provider
  - connect
  - mapStateToProps
  - mapDispatchToProps

- 异步action

  > 需要借助redux-thunk中间件。还有redux-promise redux-saga等

  ```
  // 同步action
  export const addTodo = text => {
    // 返回action 对象
    return {
      type: 'ADD_TODO',
      text
    }
  }
  
  // 异步action
  export const addTodoAsync = text => {
    // 返回函数，其中有dispatch 参数
    return (dispatch) => {
      // ajax异步获取数据
      fetch(url).then(res=>{
        // 执行异步action
        dispatch(addTodo(res.text))
      })
    }
  }
  ```

- redux中间件

  > 加强dispatch能力

- react-router
  - 路由模式（hash、H5 history）
  - 路由配置 （动态路由、懒加载）

### react原理

#### 函数式编程

- 一种编程范式
- 纯函数
- 不可变值

#### vdom和diff

- h函数
- vnode数据结构
- patch函数

#### JSX本质

- JSX等同于Vue模板
- Vue模板不是html
- JSX也不是JS
- 经过babel转义为React.createElement 生成vnode

#### 合成事件

- 所有的事件挂载到document上
- event不是原生的，是SyntheticEvent合成事件对象
- 和Vue不同，和DOM事件也不同

#### 为何要合成事件机制

- 更好的兼容性和跨平台
- 载到document,减少内存消耗，避免频繁解绑
- 方便事件的统一管理
- react17事件绑定到root (有利于多个react版本并存，微前端)

#### setState 和 batchUpdate

- 有时异步（普通使用），有时同步（setTimeout、DOM事件）

- 有时合并（对象形式），有时不合并（函数形式）

- 核心要点

  - setState主流程

  - batchUpdate机制

    > setState无所谓异步还是同步
    >
    > 看是否能命中batchUpdate机制 （默认是批量更新的）
    >
    > 判断isBatchingUpdates变量（正常执行一个函数时isBatchingUpdates为true，待函数执行完置为false。当遇到异步的时候，执行到异步代码的时候，isBatchingUpdates已经置为false，也就是变成同步更新了）

  - transaction（事务）机制 （before、after函数）

#### 组件渲染和更新过程

**渲染**

- Props state
- render() 生成vnode
- patch(elem, vnode)

**更新**

- setState(newState) -> dirtyComponent(可能有子组件)
- render()生成newVnode
- patch(vnode, newVnode)

**更新的两个阶段**

- 上述的patch被拆分为两个阶段
- Reconciliation 阶段 -  执行diff算法，纯js计算
- commit阶段 -  将diff结果渲染DOM

#### fiber

- 将Reconciliation 阶段进行任务拆分（commit无法拆分）
- DOM需要渲染时暂停，空闲时恢复
- window.requestIdleCallback