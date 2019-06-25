---
title: react-基础
date: 2019-04-16 11:28:35
categories:
- react-native
tags:
---

### 

```
<Text> 内容 </Text>
<Image source={} style={{}} />
```

### 组件

- 函数定义组件 使用函数或类来声明

```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
// 调用 所有的参数都会合并到 props 和ios对比，不用声明标准格式的带参数的方法
<Welcome name="Sara" />
```

- 组件的返回值

  组件的返回值只能有一个根元素，可能是基于多个组件如何排列，或如何分配参数的考虑

- 组件的`props`参数是不可更改的，需要更改则要使用`state`

### 事件

```javascript
function activateLasers() {
  
}

<button onClick={activateLasers}>
  Activate Lasers
</button>
```

```javascript
// 以下两种默认绑定 this
// 1. 函数声明时使用 箭头函数方法
handleClick = () => {
    console.log('this is:', this);
}
// 2. 如果函数声明时没有使用箭头函数方法，调用时使用箭头函数
// e 是一个合成事件
<button onClick={(e) => this.handleClick(e)}>
        Click me
</button>
```

### 状态提升

在React应用中，对应任何可变数据理应只有一个单一“数据源”。通常，状态都是首先添加在需要渲染数据的组件中。然后，如果另一个组件也需要这些数据，你可以将数据提升至离它们最近的共同祖先中。你应该依赖 [自上而下的数据流](https://react.docschina.org/docs/state-and-lifecycle.html#%E6%95%B0%E6%8D%AE%E8%87%AA%E9%A1%B6%E5%90%91%E4%B8%8B%E6%B5%81%E5%8A%A8)，而不是尝试在不同组件中同步状态。

从ios开发者的角度考虑：

`react`中的`props`是不可变的，`state`才是可变的数据。可变数据在super中以state的形式存储，通过在子类初始化时传入 this.state.parameter, function(可以理解为ios中的block)。在子类中通过props获取变量和更新函数，更新时再调用获取的更新函数来更新数据。