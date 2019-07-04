---
title: 制作自己的pod
date: 2019-01-09 14:56:24
categories:
- 知识点
tags:
---
## 1. 在github上创建一个空白工程 ``APPHelp``，克隆到本地

## 2. 在终端 cd 到 ``APPHelp`` 目录，执行
```
pod spec create APPHelp
```
会创建 ``APPHelp.podspec``
{% asset_img 1.png 图片说明 %}
使用xcode打开，可以看里面的注释，我这里放一个自己使用的精简版的，可以对照着改
里面的注释被我删掉了，可能是我用的注释格式不正确“//   #”,注意点都写在了下面
```


Pod::Spec.new do |s|

s.name         = "APPHelp"

s.version      = "0.0.1"

s.summary      = "app开发常用知识点收集"

s.homepage     = "https://github.com/libo198615/APPHelp"

s.license = { :type => 'MIT', :text => <<-LICENSE
Copyright PPAbner 2016-2017
LICENSE
}

s.author             = { "boli" => "libo198615@163.com" }

s.platform     = :ios

s.source       = { :git => "https://github.com/libo198615/APPHelp.git", :tag => "#{s.version}" }

s.source_files  = "Pod/**/*"

s.public_header_files = "Pod/**/*.h"

s.requires_arc = true

s.ios.deployment_target = "8.0"

end


```

## 3. 将 ``APPHelp.podspec``和你自己的文件提交到github，创建``APPHelp.podspec``中相应的 ``0.0.1``的分支或者tag
里面的内容需要改成自己的信息，其中s.source_files和s.public_header_files中如果有其它文件夹，要用两个*，不然也会报错，应该是正则匹配的问题，这里不深究了。

## 4. 执行下面命令，验证``APPHelp.podspec``是否正确，如果有错误请细读一下``APPHelp.podspec``看是否有设置不对的地方
```
pod spec lint APPHelp.podspec
```

## 5. 提交 ``APPHelp.podspec`` 到 ``CocoaPods``
```
如果是第一次提交，需要注册一下
$ pod trunk register 邮箱 '昵称' --description=' 这里写描述'
```
```
pod trunk push APPHelp.podspec
```

