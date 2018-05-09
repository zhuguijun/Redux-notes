<h3>搭配 React</h3>

Redux 和 React 之间没有关系。Redux 支持 React、Angular、Ember、jQuery 甚至纯 JavaScript。

尽管如此，Redux 还是和 React 和 Deku 这类库搭配起来用最好，因为这类库允许你以 state 函数的形式来描述界面，Redux 通过 action 的形式来发起 state 变化,Redux 的 React 绑定库是基于 容器组件和展示组件相分离 的开发思想。
大部分的组件都应该是展示型的，但一般需要少数的几个容器组件把它们和 Redux store 连接起来。这并不意味着容器组件必须位于组件树的最顶层,如果一个容器组件变得太复杂，那么可以在组件树中引入另一个容器。


<h4>设计组件层次结构</h4>

| |展示组件|容器组件
|-|-|-
作用|描述如何展现（骨架、样式）|描述如何运行（数据获取、状态更新）
直接使用 Redux|否|是
数据来源|props|监听 Redux state
数据修改|从 props 调用回调函数|向 Redux 派发 actions
调用方式|手动|通常由 React Redux 生成

<b>展示组件</b>:这些组件只定义外观并不关心数据来源和如何改变。传入什么就渲染什么。如果你把代码从 Redux 迁移到别的架构，这些组件可以不做任何改动直接使用。它们并不依赖于 Redux。

<b>容器组件</b>:需要一些容器组件来把展示组件连接到 Redux。


<b>实现展示组件</b>:它们只是普通的 React 组件，可以使用函数式无状态组件，除非需要本地 state 或生命周期函数的场景。这并不是说展示组件必须是函数 -- 只是因为这样做容易些。如果你需要使用本地 state，生命周期方法，或者性能优化，可以将它们转成 class。比如：
<pre>
  import React from 'react'
  import PropTypes from 'prop-types'

  const Todo = ({ onClick, completed, text }) => (
    &lt;li
      onClick={onClick}
      style={ {
        textDecoration: completed ? 'line-through' : 'none'
      }}
    >
      {text}
    &lt;/li>
  )

  Todo.propTypes = {
    onClick: PropTypes.func.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }

  export default Todo
</pre>


<b>实现容器组件</b>:创建一些容器组件把这些展示组件和 Redux 关联起来。技术上讲，容器组件就是使用 store.subscribe() 从 Redux state 树中读取部分数据，并通过 props 来把这些数据提供给要渲染的组件。你可以手工来开发容器组件，但建议使用 React Redux 库的 connect() 方法来生成，这个方法做了性能优化来避免很多不必要的重复渲染。（这样你就不必为了性能而手动实现 React 性能优化建议 中的 shouldComponentUpdate 方法。）
<pre>
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
</pre>

<b>将容器放到一个组件</b>:
<pre>components/App.js</pre>


  <pre>
import React from 'react'
import Footer from './Footer'
import AddTodo from '../containers/AddTodo'
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  &lt;div>
    &lt;AddTodo />
    &lt;VisibleTodoList />
    &lt;Footer />
  &lt;/div>
)

export default App
</pre>

<b>传入 Store</b>:所有容器组件都可以访问 Redux store，所以可以手动监听它。一种方式是把它以 props 的形式传入到所有容器组件中。但这太麻烦了，因为必须要用 store 把展示组件包裹一层，仅仅是因为恰好在组件树中渲染了一个容器组件。

建议的方式是使用指定的 React Redux 组件 <Provider> 来 魔法般的 让所有容器组件都可以访问 store，而不必显示地传递它。只需要在渲染根组件时使用即可。

<pre>index.js</pre>


  <pre>
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp)

render(
  &lt;Provider store={store}>
   &lt;App />
  &lt;/Provider>,
  document.getElementById('root')
)
</pre>
