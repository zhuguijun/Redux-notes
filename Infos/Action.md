<h3>Action</h3>

Action 是把数据从应用（译者注：这里之所以不叫 view 是因为这些数据有可能是服务器响应，用户输入或其它非 view 的数据 ）传到 store 的有效载荷。它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。

Action 本质上是 JavaScript 普通对象。我们约定，action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作。多数情况下，type 会被定义成字符串常量。当应用规模越来越大时，建议使用单独的模块或文件来存放 action。
<pre>
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
</pre>

添加新 todo 任务的 action 是这样的：

<pre>
const ADD_TODO = 'ADD_TODO'

{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
</pre>

<h4>Action 创建函数</h4>

<b>Action 创建函数</b> 就是生成 action 的方法。“action” 和 “action 创建函数” 这两个概念很容易混在一起，使用时最好注意区分。

在 Redux 中的 action 创建函数只是简单的返回一个 action:
<pre>
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
</pre>

Redux 中只需把 action 创建函数的结果传给 dispatch() 方法即可发起一次 dispatch 过程。
<pre>
dispatch(addTodo(text))
</pre>


或者创建一个 被绑定的 action 创建函数 来自动 dispatch：
<pre>
const boundAddTodo = text => dispatch(addTodo(text))
</pre>
然后直接调用它：
<pre>
boundAddTodo(text);
</pre>
store 里能直接通过 store.dispatch() 调用 dispatch() 方法，但是多数情况下你会使用 react-redux 提供的 connect() 帮助器来调用。bindActionCreators() 可以自动把多个 action 创建函数 绑定到 dispatch() 方法上。Action 创建函数也可以是异步非纯函数。


接下来我们要学习reducers ，用来说明在发起 action 后 state 应该如何更新。
