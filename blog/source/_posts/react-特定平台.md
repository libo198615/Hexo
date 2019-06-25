---
title: react-特定平台
date: 2019-03-14 11:18:07
categories:
- react-native
tags:
---

### Platform 模块

```javascript
import { Platform, StyleSheet } from "react-native";

const styles = StyleSheet.create({
    height: Platform.OS === "ios" ? 200 : 100
});
```

Platform.select()

```javascript
const styles = StyleSheet.create({
    container: {
        flex: 1,
        ...Platform.select({
            ios: {
                backgroundColor: "red"
            },
            android: {
                backgroundColor: "blue"
            }
        })
    }
});

const Component = Platform.select({
    ios: () => require("ComponentIOS"),
    android: () => require("ComponentAndroid")
})();
```

