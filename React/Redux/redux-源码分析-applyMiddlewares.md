---
title: Redux源码系列三--applyMiddleware
date: 2019-08-16 19:43:16
tags: [Redux, Source code analysis]
---

<center>
  本文根据Redux 4.0.1版本来分析 applyMiddleware 源代码
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
import compose from './compose'

/**
 * applyMiddleware是将各个需要应用的中间件串联起来，将最原始的store.dispatch作为参数传入
 * 组合后的中间件链中。
 *
 * 组合：把一个函数的输出作为输入发送给另一个函数的方式
 * Creates a store enhancer that applies middleware to the dispatch method
 * of the Redux store. This is handy for a variety of tasks, such as expressing
 * asynchronous actions in a concise manner, or logging every action payload.
 *
 * See `redux-thunk` package as an example of the Redux middleware.
 *
 * Because middleware is potentially asynchronous, this should be the first
 * store enhancer in the composition chain.
 *
 * Note that each middleware will be given the `dispatch` and `getState` functions
 * as named arguments.
 *
 * 中间件：位于action被派发之后，到达reducer之前的扩展
 * @param {...Function} middlewares The middleware chain to be applied.
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    // 相当于
    // applyMiddleware(...middlewares)(createStore)(reducer, initState)
    // 借用原始的 createStore 方法，创建一个新的增强版的 store
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    // 第三方中间件需要使用的参数，即原始的store.getState和dispatch方法
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // 针对每个中间件传入state，dispatch，倒序执行
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // compose(f, g, h)(store.dispatch) 等价于  f(g(h(store.dispatch)))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}

```

### 参考文献：

1、<a href="https://github.com/super-lin0/functional-es6/tree/master/src/chapter-07" >函数式编程-组合</a>

2、<a href="http://cn.redux.js.org/docs/api/applyMiddleware.html">Redux 官方文档</a>

3、<a href="https://github.com/ecmadao/Coding-Guide/blob/master/Notes/React/Redux/Redux%E5%85%A5%E5%9D%91%E8%BF%9B%E9%98%B6-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md#applymiddleware">Redux 入坑进阶-源码解析</a>

4、《React 状态管理与同构实战》

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
