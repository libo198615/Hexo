---
title: react-native-navigation
date: 2019-04-09 22:32:37
categories:
- react-native
tags:
---

### react navigation

安装

```javascript
yarn add react-navigation
yarn add react-native-gesture-handler
react-native link react-native-gesture-handler
```

导航器

```
createStackNavigator
```

```javascript
import React from 'react';
import { View, Text } from 'react-native';
import { createStackNavigator, createAppContainer } from 'react-navigation';

class HomeScreen extends React.Component {
  
  static navigationOptions = {
    title: 'Home',
    headerStyle: { // 一个应用于 header 的最外层 View 的 样式对象
      backgroundColor: '#f4511e', 
    },
    headerTintColor: '#fff', // 返回按钮和标题的颜色
    headerTitleStyle: {
      fontWeight: 'bold',
    },
  };
  
  render() {
    return (
      <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
        <Text>Home Screen</Text>
      </View>
    );
  }
}

class DetailsScreen extends React.Component {
  render() {
    return (
      <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
        <Text>Details Screen</Text>
      </View>
    );
  }
}

const RootStack = createStackNavigator(
  {
    Home: HomeScreen,
    Details: DetailsScreen,
  },
  {
    initialRouteName: 'Home', // 堆栈的初始路由是Home路由
    defaultNavigationOptions: { // 共享导航栏，具体页面可直接覆盖
			headerStyle: {
				backgroundColor: '#f4511e',
			},
			headerTintColor: '#fff',
			headerTitleStyle: {
				fontWeight: 'bold',
			},
		},
  }
);

const AppContainer = createAppContainer(RootStack);
// 导出根组件
export default class App extends React.Component {
  render() {
    return <AppContainer />;
  }
}
```

```javascript
<Button
  title="Go to Details... again"
  onPress={() => this.props.navigation.push('Details')} // 直接 push 到新页面
/>
<Button
  title="Go to Home"
  onPress={() => this.props.navigation.navigate('Home')} // 路由中有此页面才会生效，如果当前就在此页面，将不起作用
/>
<Button
  title="Go back"
  onPress={() => this.props.navigation.goBack()} // 手动触发返回
/>
```

```
// 返回根页面
navigate('Home')
navigation.popToTop()
```

```javascript
// 传递参数给路由

this.props.navigation.navigate('RouteName', {itemId: 86,
              otherParam: 'anything you want here',})

// 获取
this.props.navigation.state.params 

const { navigation } = this.props;
const itemId = navigation.getParam('itemId', 'NO-ID');
```

使用自定义组件替换标题

```javascript
class LogoTitle extends React.Component {
  render() {
    return (
      <Image
        source={require('./spiro.png')}
        style={{ width: 30, height: 30 }}
      />
    );
  }
}

class HomeScreen extends React.Component {
  static navigationOptions = {
    // headerTitle instead of title
    headerTitle: <LogoTitle />,
  };

  /* render function, etc */
}
```

标题栏按钮

```javascript
class HomeScreen extends React.Component {
  static navigationOptions = ({ navigation }) => {
    return {
      headerTitle: <LogoTitle />,
      headerRight: (
        <Button
          onPress={navigation.getParam('increaseCount')}
          title="+1"
          color="#fff"
        />
      ),
    };
  };

  componentDidMount() {
    this.props.navigation.setParams({ increaseCount: this._increaseCount });
  }

  state = {
    count: 0,
  };

  _increaseCount = () => {
    this.setState({ count: this.state.count + 1 });
  };

  /* later in the render function we display the count */
}
```

### AppContainers

App 容器负责管理应用的 state, 并将顶层的 navigator 链接到整个应用环境。

```javascript
import { createAppContainer, createStackNavigator } from 'react-navigation';
// you can also import from @react-navigation/native

const AppNavigator = createStackNavigator(...);

const AppContainer = createAppContainer(AppNavigator);

// Now AppContainer is the main component for React to render

export default AppContainer;
```

React Native 中的 `createAppContainer` prop

```js
<AppContainer
  onNavigationStateChange={handleNavigationChange}
  uriPrefix="/app"
/>
```

- `onNavigationStateChange(prevState, newState, action)`

每当导航器管理的 navigation state 发生变化时，都会调用该函数。 它接收之前的 state、navigation 的新 state 以及发布状态更改的 action。 默认情况下，它将 state 的更改打印到控制台。

- `uri前缀`

应用可能会处理的 URI 前缀， 在处理一用于提取传递给 route 的一个 [深度链接](https://reactnavigation.org/docs/zh-Hans/deep-linking.html)时将会用到。

### 专业术语

- Navigation prop

  [Navigation prop API](https://reactnavigation.org/docs/zh-Hans/navigation-prop.html)

  只能用在只有通过 React Navigation 渲染为路由的页面才会生效，即第一级组件(vc)，子组件需回调到第一级组件进行调用。即只有在UIViewController.view中可以使用

- Navigation State

  ```javascript
  {
    key: 'StackRouterRoot',
    index: 1,
    routes: [
      { key: 'A', routeName: 'Home' },
      { key: 'B', routeName: 'Profile' },
    ]
  }
  ```

- Route

  每个路由都是一个 navigation state

  ```
  {
    key: 'B',
    routeName: 'Profile',
    params: { id: '123' }
  }
  ```

  

