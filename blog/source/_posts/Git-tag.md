---
title: Git Tag
date: 2019-01-09 16:06:01
categories:
- git
tags:
---

**创建**
```
git tag 0.1  // lightweight 轻量级
// 在某次提交上创建分支
git tag 0.2 -m "v-0.2" 69324355  // annotated 含附注
```
SourceTree:
{% asset_img 1.png 图片说明 %}

**推送**
```
// 连同标签一起推送
git push origin master --tags
只推送标签
git push --tags
```
SourceTree:
{% asset_img 2.png 图片说明 %}
**更新**
```
// 获取所有的提交及标签更新
git fetch
// 仅仅获取标签更新
git fetch origin --tags
```

**查看**
```
git tag 
git ls-remote --tags  // 远程
```
**删除**
```
git tag -d 0.1
git push origin(远程名字) --delete tag 0.1
```
SourceTree:
{% asset_img 3.png 图片说明 %}

**移动**
SourceTree:
{% asset_img 4.png 图片说明 %}
