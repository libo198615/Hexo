---
title: react-native_通知
date: 2019-04-23 13:08:29
categories:
- react-native
tags:
---

#### DeviceEventEmitter

```javascript
import { DeviceEventEmitter } from 'react-native';
...
componentDidMount() {
    //收到监听
    this.listener = DeviceEventEmitter.addListener('通知名称', (message) => {
    //收到监听后想做的事情
    console.log(message);  //监听
    })
}
componentWillUnmount() {
    //移除监听
    if (this.listener) {
      this.listener.remove();
    }
  }
...
```

```javascript
startEmit() {
    //准备值，发监听
    const message = '监听';
    DeviceEventEmitter.emit('通知名称', message);
}
```

