---
title: Redux源码系列二--combineReducers
date: 2019-08-16 19:12:16
tags: [Redux, Source code analysis]
---

<center>
  本文根据Redux 4.0.1版本来分析 combineReducers 源代码
<center>
</br>
</center>
  标签：#Redux, #Source code analysis
</center>

<!-- more -->

### 使用方法

```
import { createStore, combineReducers, applyMiddleware, compose } from "redux";

import thunkMiddleware from "redux-thunk";

import { reducer as weatherReducer } from "./weather/";

const reducer = combineReducers({
  weather: weatherReducer
});

const middlewares = [thunkMiddleware];

const storeEnhancers = compose(applyMiddleware(...middlewares));

export default createStore(reducer, {}, storeEnhancers);

```

### 源码解读

```
import ActionTypes from './utils/actionTypes'
import warning from './utils/warning'
import isPlainObject from './utils/isPlainObject'

function getUndefinedStateErrorMessage(key, action) {
  const actionType = action && action.type
  const actionDescription =
    (actionType && `action "${String(actionType)}"`) || 'an action'

  return (
    `Given ${actionDescription}, reducer "${key}" returned undefined. ` +
    `To ignore an action, you must explicitly return the previous state. ` +
    `If you want this reducer to hold no value, you can return null instead of undefined.`
  )
}

function getUnexpectedStateShapeWarningMessage(
  inputState,
  reducers,
  action,
  unexpectedKeyCache
) {
  const reducerKeys = Object.keys(reducers)
  const argumentName =
    action && action.type === ActionTypes.INIT
      ? 'preloadedState argument passed to createStore'
      : 'previous state received by the reducer'

  if (reducerKeys.length === 0) {
    return (
      'Store does not have a valid reducer. Make sure the argument passed ' +
      'to combineReducers is an object whose values are reducers.'
    )
  }

  if (!isPlainObject(inputState)) {
    return (
      `The ${argumentName} has unexpected type of "` +
      {}.toString.call(inputState).match(/\s([a-z|A-Z]+)/)[1] +
      `". Expected argument to be an object with the following ` +
      `keys: "${reducerKeys.join('", "')}"`
    )
  }

  const unexpectedKeys = Object.keys(inputState).filter(
    key => !reducers.hasOwnProperty(key) && !unexpectedKeyCache[key]
  )

  unexpectedKeys.forEach(key => {
    unexpectedKeyCache[key] = true
  })

  if (action && action.type === ActionTypes.REPLACE) return

  if (unexpectedKeys.length > 0) {
    return (
      `Unexpected ${unexpectedKeys.length > 1 ? 'keys' : 'key'} ` +
      `"${unexpectedKeys.join('", "')}" found in ${argumentName}. ` +
      `Expected to find one of the known reducer keys instead: ` +
      `"${reducerKeys.join('", "')}". Unexpected keys will be ignored.`
    )
  }
}

function assertReducerShape(reducers) {
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key]
    const initialState = reducer(undefined, { type: ActionTypes.INIT })

    if (typeof initialState === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined during initialization. ` +
          `If the state passed to the reducer is undefined, you must ` +
          `explicitly return the initial state. The initial state may ` +
          `not be undefined. If you don't want to set a value for this reducer, ` +
          `you can use null instead of undefined.`
      )
    }

    if (
      typeof reducer(undefined, {
        type: ActionTypes.PROBE_UNKNOWN_ACTION()
      }) === 'undefined'
    ) {
      throw new Error(
        `Reducer "${key}" returned undefined when probed with a random type. ` +
          `Don't try to handle ${
            ActionTypes.INIT
          } or other actions in "redux/*" ` +
          `namespace. They are considered private. Instead, you must return the ` +
          `current state for any unknown actions, unless it is undefined, ` +
          `in which case you must return the initial state, regardless of the ` +
          `action type. The initial state may not be undefined, but can be null.`
      )
    }
  })
}

/**
 * 随着应用变得越来越复杂，可以考虑将reducer函数拆分成多个单独的函数，拆分后的每个函数负责独
 * 立管理state的一部分
 * combineReducer: 接收多个 reducer 函数，并进行整合，归一化成一个 rootReducer 的方法。
 * 即把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数。
 *
 * Turns an object whose values are different reducer functions, into a single
 * reducer function. It will call every child reducer, and gather their results
 * into a single state object, whose keys correspond to the keys of the passed
 * reducer functions.
 *
 * @param {Object} reducers An object whose values correspond to different
 * reducer functions that need to be combined into one. One handy way to obtain
 * it is to use ES6 `import * as reducers` syntax. The reducers may never return
 * undefined for any action. Instead, they should return their initial state
 * if the state passed to them was undefined, and the current state for any
 * unrecognized action.
 *
 * reducers: 一个对象，它的值（value）对应不同的 reducer 函数，这些 reducer 函数后面会被合并成一个。
 * 阐述了不同reducer函数和页面状态数据树不同部分的映射匹配关系。
 * 注意：
 * 1、所有未匹配到的 action，必须把它接收到的第一个参数也就是那个 state 原封不动返回。
 * 2、永远不能返回 undefined。
 * 3、如果传入的 state 就是 undefined，一定要返回对应 reducer 的初始 state。
 *
 * @returns {Function} A reducer function that invokes every reducer inside the
 * passed object, and builds a state object with the same shape.
 *
 * 一个调用 reducers 对象里所有 reducer 的 reducer，并且构造一个与 reducers 对象结构相同
 * 的 state 对象。
 *
 * const myTodosReducer = (todoState, action) => todoState
 * const myCounterReducer = (counterState, action) => counterState
 * combineReducers({ todos: myTodosReducer, counter: myCounterReducer }) ===> { todos, counter }
 */
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers) // reducerKeys [ 'todos', 'counter' ]
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    // 筛选调 reducers 中不是 function 的键值对
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)
  // finalReducers: { todos: myTodosReducer, counter: myCounterReducer }
  // finalReducerKeys: [ 'todos', 'counter' ]

  // This is used to make sure we don't warn about the same
  // keys multiple times.
  let unexpectedKeyCache
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  // 确保单个reducer不能返回 undefined。在开发过程中，注意
  // reducer函数默认返回初始state（switch default: return state）
  let shapeAssertionError
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  // 返回一个标准的reducer
  return function combination(state = {}, action) {
    // 如果之前判断的有 reducer 返回 undefined，直接抛出错误
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    // 如果不是production环境则抛出warning
    // 规则： 1、state必须是原生对象； 2、finalReducers 数组长度不能为0
    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState = {}

    // const myTodosReducer = (todoState, action) => todoState
    // const myCounterReducer = (counterState, action) => counterState
    // finalReducers: { todos: myTodosReducer, counter: myCounterReducer }
    // finalReducerKeys: [ 'todos', 'counter' ]
    // 遍历所有的key和reducer，分别将reducer对应的key所代表的state，代入到reducer中进行函数调用
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i] // "todos" "counter"
      const reducer = finalReducers[key] // myTodosReducer  myCounterReducer
      const previousStateForKey = state[key] // undefined undefined
      const nextStateForKey = reducer(previousStateForKey, action) // myTodosReducer(undefined, action) myCounterReducer(undefined, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey // {todos: myTodosReducer} {todo: myTodosReducer, counter: myCounterReducer(undefined, action)}
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey // true
    }
    return hasChanged ? nextState : state // {todos: myTodosReducer, counter: myCounterReducer(undefined, action)}
  }
}

```

### 参考文献：

1、<a href="http://cn.redux.js.org/docs/api/combineReducers.html">Redux 官方文档</a>

2、<a href="https://github.com/ecmadao/Coding-Guide/blob/master/Notes/React/Redux/Redux%E5%85%A5%E5%9D%91%E8%BF%9B%E9%98%B6-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md#combinereducers">Redux 入坑进阶-源码解析</a>

3、《React 状态管理与同构实战》

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
