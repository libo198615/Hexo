---
title: 在sourceTree中使用rebase（变基）
date: 2019-01-09 15:00:47
categories:
- 知识点
tags:
---
原始状态
{% asset_img 1.png 图片说明 %}
假如我们要在 master 分支上进行开发，在远端的 master 分支上右键，检出 一个自己的开发分支  dev-1
{% asset_img 2.png 图片说明 %}
{% asset_img 3.png 图片说明 %}
做一些开发，提交到本地，不要推送（push）到远端
切换到 master 分支，拉取远端的 master 更新，（这里另一个同事在 master 分支上提交了 dev 2 的更新）
{% asset_img 4.png 图片说明 %}
切换到自己的开发分支 dev-1
选中 master 分支，右键，选择  将当前变更变基到 master
{% asset_img 5.png 图片说明 %}
如果有冲突则合并冲突，
点击左上角的加号，选择 继续变基
{% asset_img 6.png 图片说明 %}
此时我们的本地更新是基于最新的 master 分支
{% asset_img 7.png 图片说明 %}
最后’推送’我们的开发分支 dev-1到远端
切换到master分支，点击 拉取，拉取 dev-1 的更新到 master 分支
{% asset_img 8.png 图片说明 %}
再推送 master 分支，就保证了git分支的整洁
{% asset_img 9.png 图片说明 %}
