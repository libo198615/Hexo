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





#### user_frameworks!

表名使用动态链接库，会生成framework；默认是static libraries方式管理，会生成.a文件(静态连接库)



```
pod ``, `~> 2.0` // 大于 2.0 的最新版
pod ``, `2.0` // 固定 2.0
```

```
// 不同的target 不同的 platform 引入不同的插件
target :'zxptUser' do
platform :ios  
pod 'Reachability',  '~> 3.0.0'  
pod 'SBJson', '~> 4.0.0'  
  
platform :ios, '7.0'  
pod 'AFNetworking', '~> 2.0'
end

target :'zxptUser_local' do
pod 'OpenUDID', '~> 1.0.0'
end
```

### Podfile.lock

当你执行`pod install`之后，除了 Podfile 外，CocoaPods 还会生成一个名为`Podfile.lock`的文件，Podfile.lock 应该加入到版本控制里面，不应该把这个文件加入到`.gitignore`中。因为`Podfile.lock`会锁定当前各依赖库的版本，之后如果多次执行`pod install` 不会更改版本，要`pod update`才会改`Podfile.lock`了。这样多人协作的时候，可以防止第三方库升级时造成大家各自的第三方库版本不一致。