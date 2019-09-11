---
title: 在sourceTree中使用rebase（变基）
date: 2019-01-09 15:00:47
categories:
- 知识点
tags:
---
原始状态
![](http://ww4.sinaimg.cn/large/006tNc79ly1g61rmhilkyj30sy0pg41h.jpg)
假如我们要在 master 分支上进行开发，在远端的 master 分支上右键，检出 一个自己的开发分支  dev-1
![](http://ww1.sinaimg.cn/large/006tNc79ly1g61rmhaf1kj30t20v2ae7.jpg)
![](http://ww3.sinaimg.cn/large/006tNc79ly1g61rmh06ekj30u40r2diy.jpg)
做一些开发，提交到本地，不要推送（push）到远端
切换到 master 分支，拉取远端的 master 更新，（这里另一个同事在 master 分支上提交了 dev 2 的更新）
![](http://ww3.sinaimg.cn/large/006tNc79ly1g61rmgsxuaj30zk0ec0xz.jpg)
切换到自己的开发分支 dev-1
选中 master 分支，右键，选择  将当前变更变基到 master
![](http://ww1.sinaimg.cn/large/006tNc79ly1g61rmgjv4aj30ta0mm0wn.jpg)
如果有冲突则合并冲突，
点击左上角的加号，选择 继续变基 (如果点击加号没反应，直接进行下一步)
![](http://ww3.sinaimg.cn/large/006tNc79ly1g61rmgcbe0j30zk0g4434.jpg)
此时我们的本地更新是基于最新的 master 分支
![](http://ww3.sinaimg.cn/large/006tNc79ly1g61rmg07pkj30uo0h20vf.jpg)
最后’推送’我们的开发分支 dev-1到远端
切换到master分支，点击 拉取，拉取 dev-1（我们自己的开发分支） 的更新到 master 分支
![](http://ww1.sinaimg.cn/large/006tNc79ly1g61rmfse17j30ym0h00vp.jpg)
再推送 master 分支，就保证了git分支的整洁
![](http://ww3.sinaimg.cn/large/006tNc79ly1g61rn5g4jzj30zk0fstbx.jpg)

最后注意：切换会自己的开发分支，提交完后目前在主分支，此时再修改代码则直接是在主分支上进行修改。所以rebase后先切换回自己的开发分支。





#### 应用场景

1，控制产品OEM。

基本上做产品，不同的客户都会提出多种不同特性需求，最简单的例子就是LOGO和标题完全不一样。但是可能产品自身的大部分功能和模块的代码一样的，这个时候如何管理多个客户定制的功能特性，并且不会干扰其他OEM版本的功能呢？

如果你一开始就用if加N多变量定义的话，早晚会累死你，如果你把代码拷贝很多份，每多一个新的OEM就多拷贝一份代码，那如果发现公用模块里面有个BUG，难道你要每个版本的源代码都要修改？万一改错地方了，或者哪个版本的忘记改了，又是一件麻烦事。

这个时候我们就可以考虑使用branch功能，在第一个OEM的基础上分支出第二个OEM，第三个OEM完全取决于和哪个版本更像，就在那个版本的基础上做新的分支，有新的OEM特性需求，就切换到那个分支上修改，放心，所有的的单独分支上的代码看起来都是独立的，不会影响其他版本。

 

2，多人协作长时间开发功能模块

如果你在一个团队中，那几乎很难做到每天都能按期完成某个模块功能，并且测试通过。那团队成员又必须每日下班前把自己的代码保存一下，万一机器故障了之类的还能有代码备份机制。如果提交了不能工作的代码，别人又获取到了，那其他人的事情就做不下去了。所以，branch另外一个适用场景就是为team单独成员开辟个人工作区域，单元测试无误之后再把成员的工作代码合并到主分支中，既能达到个人代码备份的目的，又能不影响其他人的工作。

 

其实上面所说的就是rebase和merge的不同适用场景。

在场景1的情况下，如果修改了某个公用代码的BUG，这个时候就应该是把所有的OEM版本分支rebase到这个修复BUG的分支上来，在rebase过程中，Git会要你手动解决代码上的冲突，你需要做的就是把修复BUG的代码放到目标分支代码里面去。rebase的结果是：所有的分支依然存在

在场景2的情况下，因为成员的代码开发工作已经完成了，也不需要再保留这个分支了，所以我们可以把这个成员分支merge到主分支上，当然冲突在所难免，手工解决的工作肯定逃不掉，但是利大于弊不是吗。merge以后，分支就不存在了，但是在Git的所有分支历史中还能看到身影。