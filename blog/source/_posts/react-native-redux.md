---
title: react_native_redux
date: 2019-03-21 17:03:07
categories:
- react-native
tags:
---



#### sotre

`store` 是应用状态 `state` 的管理者，包含下列四个函数：

- `getState() # 获取整个 state`
- `dispatch(action) # ※ 触发 state 改变的【唯一途径】※`
- `subscribe(listener) # 您可以理解成是 DOM 中的 addEventListener`
- `replaceReducer(nextReducer) # 一般在 Webpack Code-Splitting 按需加载的时候用`

#### reducer

用户每次 `dispatch(action)` 后，都会触发 `reducer` 的执行
`reducer` 的实质是一个**函数**，根据 `action.type` 来**更新** `state` 并返回 `nextState`
最后会用 `reducer` 的返回值 `nextState` **完全替换掉**原来的 `state`

#### action 

action代表界面的操作。我们定义一个界面有几个界面操作（或者叫触发事件），使用string来描述。然后定义一个方法即action creator，传入执行这个操作需要的参数，然后返回这个操作的类型，即上面定义的string类型中的一个，和完成后的新的界面数据。这个action creator只是描述输入与产出，不负责具体的执行。总体讲action负责定义操作类型，和提供一个获取该类型的方法

```
const ADD_TODO = 'ADD_TODO'

{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

- Action Creator 
  Action Creator 是 `action` 的创造者，本质上就是一个**函数**，返回值是一个 `action`（**对象**）

  ```js
  var id = 1
  function addTodo(content) {
    return {
      type: 'ADD_TODO',
      payload: {
        id: id++,
        content: content, // 待办事项内容
        completed: false  // 是否完成的标识
      }
    }
  }
  ```

  


#### state

是应用的状态,界面状态，界面显示的值

#### 数据流

**1. 调用** [`store.dispatch(action)`](https://www.redux.org.cn/docs/api/Store.html#dispatch)。

**2. Redux store 调用传入的 reducer 函数。**

[Store](https://www.redux.org.cn/docs/basics/Store.html) 会把两个参数传入 [reducer](https://www.redux.org.cn/docs/basics/Reducers.html)： 当前的 state 树和 action。例如，在这个 todo 应用中，根 reducer 可能接收这样的数据：

**3. 根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。**

````objective-c
 function todos(state = [], action) {
   // 省略处理逻辑...
   return nextState
 }

 function visibleTodoFilter(state = 'SHOW_ALL', action) {
   // 省略处理逻辑...
   return nextState
 }

 let todoApp = combineReducers({
   todos,
   visibleTodoFilter
 })
````

**4. Redux store 保存了根 reducer 返回的完整 state 树。**

这个新的树就是应用的下一个 state！所有订阅 [`store.subscribe(listener)`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 的监听器都将被调用；监听器里可以调用 [`store.getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 获得当前 state。



### 实现容器组件

容器组件就是使用 [`store.subscribe()`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 从 Redux state 树中读取部分数据，并通过 props 来把这些数据提供给要渲染的组件。

```objective-c
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    case 'SHOW_ALL':
    default:
      return todos
  }
}

const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

// 分发方法
const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}


import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

#### 异步 action 创建函数

通过使用指定的 middleware，action 创建函数除了返回 action 对象外还可以返回函数。这时，这个 action 创建函数就成为了 [thunk](https://en.wikipedia.org/wiki/Thunk)。

当 action 创建函数返回函数时，这个函数会被 Redux Thunk middleware 执行。这个函数并不需要保持纯净；它还可以带有副作用，包括执行异步 API 请求。这个函数还可以 dispatch action，就像 dispatch 前面定义的同步 action 一样。