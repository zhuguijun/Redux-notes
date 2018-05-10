<h3>Middleware</h3>

middleware 是指可以被嵌入在框架接收请求到产生响应过程之中的代码。例如，Express 或者 Koa 的 middleware 可以完成添加 CORS headers、记录日志、内容压缩等工作。middleware 最优秀的特性就是可以被链式组合。你可以在一个项目中使用多个独立的第三方 middleware,middleware 可以完成包括异步 API 调用在内的各种事情。


Middleware 接收了一个 next() 的 dispatch 函数，并返回一个 dispatch 函数，返回的函数会被作为下一个 middleware 的 next()，以此类推。由于 store 中类似 getState() 的方法依旧非常有用，我们将 store 作为顶层的参数，使得它可以在所有 middleware 中被使用。
我们可以写一个 applyMiddleware() 方法替换掉原来的 applyMiddlewareByMonkeypatching()。在新的 applyMiddleware() 中，我们取得最终完整的被包装过的 dispatch() 函数，并返回一个 store 的副本：

<pre>
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
</pre>
然后是将它们引用到 Redux store 中：
<pre>
import { createStore, combineReducers, applyMiddleware } from 'redux'

const todoApp = combineReducers(reducers)
const store = createStore(
  todoApp,
  // applyMiddleware() 告诉 createStore() 如何处理中间件
  applyMiddleware(logger, crashReporter)
)
</pre>
就是这样！现在任何被发送到 store 的 action 都会经过 logger 和 crashReporter：

    // 将经过 logger 和 crashReporter 两个 middleware！

    store.dispatch(addTodo('Use Redux'))
