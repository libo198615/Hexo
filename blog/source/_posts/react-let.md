---
title: react-let
date: 2019-03-14 09:00:11
categories:
- react-native
tags:

---



### export 导出

```javascript
// 导出变量 text.js
var name = '';
var age = '';
export {name, age};
```

```javascript
// 导出函数 myModule.js
export function myModule(someArg) {
    return someArg;
}
```

### import 导入

```javascript
import {myModule} form 'myModule';
import {name, age} from 'text'
```

### => function 的缩写



### string

```javascript
// 变量字符串拼接
var name = 'your name is ${name}'
```

### 延展操作符

```javascript
...

var arr = [1, 2, 3];
var arr2 = [...arr]; // arr2和arr内容一样
```

### props 组件的属性

```javascript
this.props.xxx
```

