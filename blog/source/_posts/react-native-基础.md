---
title: react_native_基础
date: 2019-03-21 17:05:54
categories:
- react-native
tags:
---



### import

```
import a from A 导入 A中的默认导出组件，并命名为a
import {a} from A 导入A中的非默认组件
```



### state

render() 方法依赖于 this.props 和 this.state ，框架会确保渲染出来的 UI 界面总是与输入（ this.props 和 this.state ）保持一致。

```javascript
// 初始化 state
state = {
    name: ''
}

// 更新 state
this.setState({
    
})
```



### this

```javascript
// 使 this 不失效
<button onClick={this.handleAdd}>"xx"</button>
<button onClick={this.handleClick.bind(this)}>"xx"</button>

//此时this指向是当前实例对象
handleAdd = ()=> {
    console.log(this)
    this.setState({
        count:5
    })
}

handleClick(){
    this.setState({
        count:6
    })
}

```

—save 安装模块的时候，同时把它写到package.json 文件中, dependencies。

 --save-dev 安装依模块的时候，把它写到package.json中，devDependencies中

"express": "4.15.2" 版本号前面什么符号都没有，它表示固定版本

"express": "~4.15.2"，版本号前面有符号~，它表示安装4.15.x 的版本

"express": "^4.15.2" , 版本号前面有符号^, 它表示可以安装4.x.x 的版本，只要中间的x  大于15就可以



### 布局

```
super flexDirection:'row' 
{
	alignSelf:'flex-start'  // 左上
	alignSelf:'center'  // 中间
	alignSelf:'flex-end'  // 左下
}

{
	alignSelf:'flex-start'  // 左上
	alignSelf:'center'  // 中间	
  alignSelf:'flex-end'  // 右上
}
```



```javascript
// css
margin // 自己距离外边框的距离
border // 边框
padding // 内边框 自己的子元素距离自己边框的距离

width: '95%', // 除了vc的第一view会默认为是屏幕尺寸，其他subview都要指定宽度
  
// flex
flexDirection: row(横向) | row-reverse | column | column-reverse
flexWrap: wrap // 换行
justifyContent: // 主轴 控制子元素
alignItems: // 交叉轴 控制子元素
alignSelf: // 允许单个项目有与其他项目不一样的对齐方式 元素本身
```

```
flexbox定位和position定位可以同时使用
positon有两个取值：relative（默认值）
absolute(绝对定位)：相对于父元素来定位 top left ..
```



![](https://ws3.sinaimg.cn/large/006tKfTcly1g1igyjyi3fj30hp0l70sn.jpg)

```
alignItems: 交叉轴上如何对齐
```

![](https://ws2.sinaimg.cn/large/006tKfTcly1g1ih0q0tt8j30h50lu3yg.jpg)

```
// flex 项目属性
flexGrow: 0 // 项目的放大比例，如果所有项目的flex-grow属性都为1，则它们将等分剩余空间
flexShrink: 1 // 项目缩小比例，
```

![](https://ws3.sinaimg.cn/large/006tKfTcly1g1ihimznq1j30kn0aua9w.jpg)

### 严格模式

```
'use strict'
```

### 持久化

```
AsyncStorage.setItem("userToken", "token");
    
const userToken = await AsyncStorage.getItem('userToken');
```

