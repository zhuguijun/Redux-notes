<h3>Redux核心概念</h3>

Redux 本身很简单,应用的状态就是普通对象，使用普通对象来描述应用的state ，这个对象就像 “Model”，区别是它并没有 setter，因此其它的代码不能随意修改它，造成难以复现的 bug。

要想更新 state 中的数据，你需要发起一个 action。Action 就是一个普通 JavaScript 对象，Action这个对象用来描述发生了什么。


强制使用 action 来描述所有变化，带来的好处是可以清晰地知道应用中到底发生了什么。如果一些东西改变了，就可以知道为什么变。action 就像是描述发生了什么的指示器。最终，为了把 action 和 state 串起来，开发一些函数，这就是 reducer：reducer 只是一个接收 state 和 action，并返回新的 state 的函数。 对于大的应用来说，不大可能仅仅只写一个这样的函数，所以我们编写很多小函数来分别管理 state 的一部分，
这样就再开发一个 reducer，可以用来调用其他的 reducer，进而来管理整个应用的 state。


这差不多就是 Redux 全部思想：主要的想法是如何根据这些 action 对象来更新 state，而且 90% 的代码都是纯 JavaScript，没用 Redux和Redux API。

描述应用的 state 可能长这样：
<pre>
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
</pre>
用来描述发生了什么的action可能长这样：
<pre>
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
</pre>


拆分成多个小函数的action可能长这样：
<pre>
function visibilityFilter(state = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter;
  } else {
    return state;
  }
}

function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([{ text: action.text, completed: false }]);
  case 'TOGGLE_TODO':
    return state.map((todo, index) =>
      action.index === index ?
        { text: todo.text, completed: !todo.completed } :
        todo
   )
  default:
    return state;
  }
  }
 </pre>
 
 
 再开发一个 reducer 调用这两个 reducer，进而来管理整个应用的 state：
<pre>
function todoApp(state = {}, action) {
  return {
    todos: todos(state.todos, action),
    visibilityFilter: visibilityFilter(state.visibilityFilter, action)
  };
}
</pre>
我们可以理解redux就是简单的JavaScript。
