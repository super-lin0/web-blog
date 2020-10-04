---
title: Redux源码系列四--bindActionCreators
date: 2019-08-17 10:32:16
tags: [Redux, Source code analysis]
---

<center>
  本文根据Redux 4.0.1版本来分析 bindActionCreators 源代码
<center>
</br>
</center>
  标签：#Redux, #Source code analysis
</center>

<!-- more -->

### 使用方法

- TodoActionCreators.js

```
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```

- test.js

```

const TodoActions = { addTodo, removeTodo }

const dispatch = action => console.log(action)

console.log('========action1===========')
let action1 = TodoActions.addTodo()
dispatch(action1) // { type: 'ADD_TODO', text: undefined }
action1 = TodoActions.addTodo('Hello action1')
dispatch(action1) // { type: 'ADD_TODO', text: 'Hello action1' }

console.log('===========action2==============')

let action2 = TodoActions.removeTodo()
dispatch(action2) // { type: 'REMOVE_TODO', id: undefined }
dispatch(TodoActions.removeTodo('Hello action2')) // { type: 'REMOVE_TODO', id: 'Hello action2' }

console.log('===========bindActionCreators===============')
let boundActionCreators = bindActionCreators(TodoActions, dispatch)

console.log(boundActionCreators) // { addTodo: [Function], removeTodo: [Function] }
console.log(boundActionCreators.addTodo()) // { type: 'ADD_TODO', text: undefined }
console.log(boundActionCreators.addTodo('Hello bound1')) // { type: 'ADD_TODO', text: 'Hello bound1' }

```

- SomeComponent.js

```
import { Component } from 'react'
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'

import * as TodoActionCreators from './TodoActionCreators'
console.log(TodoActionCreators)
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  constructor(props) {
    super(props)

    const { dispatch } = props

    // 这是一个很好的 bindActionCreators 的使用示例：
    // 你想让你的子组件完全不感知 Redux 的存在。
    // 我们在这里对 action creator 绑定 dispatch 方法，
    // 以便稍后将其传给子组件。

    this.boundActionCreators = bindActionCreators(TodoActionCreators, dispatch)
    console.log(this.boundActionCreators)
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }
  }

  componentDidMount() {
    // 由 react-redux 注入的 dispatch：
    let { dispatch } = this.props

    // 注意：这样是行不通的：
    // TodoActionCreators.addTodo('Use Redux')

    // 你只是调用了创建 action 的方法。
    // 你必须要同时 dispatch action。

    // 这样做是可行的：
    let action = TodoActionCreators.addTodo('Use Redux')
    dispatch(action)
  }

  render() {
    // 由 react-redux 注入的 todos：
    let { todos } = this.props

    return <TodoList todos={todos} {...this.boundActionCreators} />

    // 另一替代 bindActionCreators 的做法是
    // 直接把 dispatch 函数当作 prop 传递给子组件，但这时你的子组件需要
    // 引入 action creator 并且感知它们

    // return <TodoList todos={todos} dispatch={dispatch} />;
  }
}

export default connect(state => ({ todos: state.todos }))(TodoListContainer)
```

### 源码解读

```
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}

/**
 * Turns an object whose values are action creators, into an object with the
 * same keys, but with every function wrapped into a `dispatch` call so they
 * may be invoked directly. This is just a convenience method, as you can call
 * `store.dispatch(MyActionCreators.doSomething())` yourself just fine.
 * 把一个 value 为不同 action creator 的对象，转成拥有同名 key 的对象。同时使
 * 用 dispatch 对每个 action creator 进行包装，以便可以直接调用它们。你也可以直接
 * 在 Store 实例上调用 dispatch，随便你
 *
 * For convenience, you can also pass an action creator as the first argument,
 * and get a dispatch wrapped function in return.
 * 为方便起见，你也可以传入一个函数作为第一个参数，它会返回一个函数。
 *
 * @param {Function|Object} actionCreators An object whose values are action
 * creator functions. One handy way to obtain it is to use ES6 `import * as`
 * syntax. You may also pass a single function.
 * 一个值是 action creator函数的对象。一个简便的方式是使用es6的 `import * as`。传一个单独的
 * 函数也可以
 *
 * @param {Function} dispatch The `dispatch` function available on your Redux
 * store.
 * 一个由 Store 实例提供的 dispatch 函数。
 *
 * @returns {Function|Object} The object mimicking the original object, but with
 * every action creator wrapped into the `dispatch` call. If you passed a
 * function as `actionCreators`, the return value will also be a single
 * function.
 * 一个与原对象类似的对象，只不过这个对象的 value 都是会直接 dispatch 原 action creator 返
 * 回的结果的函数。如果传入一个单独的函数作为 actionCreators，那么返回的结果也是一个单独的函数。
 */
export default function bindActionCreators(actionCreators, dispatch) {
  // 如果只是传入一个action，则通过bindActionCreator返回被绑定到dispatch的函数
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  // 第一个参数必须是对象
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  // 遍历并通过bindActionCreator分发绑定至dispatch
  const boundActionCreators = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

```

### 参考文献：

1、<a href="http://cn.redux.js.org/docs/api/bindActionCreators.html">Redux 官方文档</a>

2、<a href="https://github.com/ecmadao/Coding-Guide/blob/master/Notes/React/Redux/Redux%E5%85%A5%E5%9D%91%E8%BF%9B%E9%98%B6-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md#bindactioncreator">Redux 入坑进阶-源码解析</a>

3、<a href="https://juejin.im/post/5b4ac9ce6fb9a04fb745c8f9">Redux 源码分析--bindActionCreators 篇</a>

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
