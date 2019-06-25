---
title: flutter
date: 2019-01-08 18:44:15
tags:
---

##### 镜像

打开$HOME/.bash_profile(在你的/Users/.bash_profile目录下，如果没有，可以新建，可以使用command+shift+.组合健来显示与隐藏文件)文件，加入：
```
export PUB_HOSTED_URL=https://pub.flutter-io.cn 
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
export PATH=[你克隆 Flutter 仓库的路径]/flutter/bin:$PATH
```

保存，运行 `source $HOME/.bash_profile` 刷新当前窗口。


- brew
镜像：`git remote set-url origin https://git.coding.net/homebrew/homebrew.git`

brew update  error
Error: An unexpected error occurred during the `brew link` step
The formula built, but is not symlinked into /usr/local
Permission denied @ dir_s_mkdir - /usr/local/Frameworks
Error: Permission denied @ dir_s_mkdir - /usr/local/Frameworks

```
sudo mkdir /usr/local/Frameworks
sudo chown $(whoami):admin /usr/local/Frameworks
```


- material 一种标准的移动端和web端的视觉设计语言
```
import 'package:flutter/material.dart';
```
- => Dart 中单行函数或方法的简写
```
void main() => runApp(new MyApp());
```
---
- 使用外部包(package)

在`pubspec.yaml`中
```
english_works: ^3.1.0
```
运行
```
package get
````
引入
```
import 'package:english_words/english_words.dart';
```
---
#### `Statefulwidge` 有状态的部件

需要两个类
1. `StatefulWidget` 不可变
2. `State` 在widget声明周期始终存在
```
class RandomWords extends StatefulWidget {
    @override
    createState() => new RandomWordsState();
}

class RandomWordsState extends State<RandomWords> {
    @override
    Widget build(BuildContext context) {
        final wordPair = new WordPair.randow();
        return new Text(wordPair.asPascalCase);
    }
}
```
使用
```
body: new Center (
    child: new RandomWords(),
)
```
---
#### 创建一个无限滚动的 `ListView`

```
class RandomWordsState extens State <RandomWords> {
    // _开始的变量会强制变成私有的
    final _suggestions = <WordPair>[];
    final _biggerFont = const TextStyle(fontSize: 18.0);

    // 此方法构建显示建议单词对的ListView
    Widget _buildSuggestions() {
        return new ListView.buider(
            padding: const EdgeInsets.all(16.0),

            itemBuilder:(context, i) {
                // 在每一列之前，添加一个1像素高的分割线 widget
                if (i.isOdd) return new Divider();

                final index = i ~/ 2; // 除法
                if (index >= _suggestions.length) {
                    _suggestions.allAll(generateWordPairs().take(10));
                }
                return _buildRow(_sugestions[index]);
            }
        );
    }

    // 对于每一个单词对，_buildSuggestions函数都会调用一次_buildRow
    Widget _buildRow(WordPair pair) {
        return new ListTitle(
            title: new Text(
                pair.asPascalCase,
                style: _biggerFont,
            )
        )
    }
}
```