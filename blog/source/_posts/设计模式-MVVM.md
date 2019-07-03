---
title: MVVM
date: 2019-01-09 17:08:37
categories:
- 设计模式
tags:
---

##### mvc

![2](http://ww2.sinaimg.cn/large/006tNc79ly1g4md25ee1cj30r20coab0.jpg)

Models： 数据层，负责数据的处理和获取的数据接口层。

Views： 展示层(GUI)，对于 iOS 来说所有以 UI 开头的类基本都属于这层。

Controller： 控制器层，它是 Model 和 View 之间的胶水或者说是中间人。一般来说，当用户对 View 有操作时它负责去修改相应 Model；当 Model 的值发生变化时它负责去更新对应 View。

每个viewcontroller都带有一个根view，这就导致C和V紧密耦合在一起构成了iOS里面的C层，目的就是为了提高开发效率，简化操作。对于简单界面来说，viewcontroller结构确实可以提高开发效率，但是一旦需要构建复杂界面，那么viewcontroller很容易就会出现代码膨胀，逻辑满天飞的问题。



Model层实现的正确姿势：

业务模型。也就是你所有业务数据和业务实现逻辑都应该定义在M层里面，而且业务逻辑的实现和定义应该和具体的界面无关，也就是和视图以及控制之间没有任何的关系，它是可以独立存在的，您甚至可以将业务模型单独编译出一个静态库来提供给第三方或者其他系统使用。

1. 定义的M层中的代码应该和V层和C层完全无关的，也就是M层的对象是不需要依赖任何C层和V层的对象而独立存在的。整个框架的设计最优结构是V层不依赖C层而独立存在，M层不依赖C层和V层独立存在，C层负责关联二者，V层只负责展示，M层持有数据和业务的具体实现，而C层则处理事件响应以及业务的调用以及通知界面更新。三者之间一定要明确的定义为单向依赖，而不应该出现双向依赖
2. M层要完成对业务逻辑实现的封装，一般业务逻辑最多的是涉及到客户端和服务器之间的业务交互。M层里面要完成对使用的网络协议(HTTP, TCP，其他)、和服务器之间交互的数据格式（XML, JSON,其他)、本地缓存和数据库存储（COREDATA, SQLITE,其他)等所有业务细节的封装，而且这些东西都不能暴露给C层。所有供C层调用的都是M层里面一个个业务类所提供的成员方法来实现。也就是说C层是不需要知道也不应该知道和客户端和服务器通信所使用的任何协议，以及数据报文格式，以及存储方面的内容。这样的好处是客户端和服务器之间的通信协议，数据格式，以及本地存储的变更都不会影响任何的应用整体框架，因为提供给C层的接口不变，只需要升级和更新M层的代码就可以了。比如说我们想将网络请求库从ASI换成AFN就只要在M层变化就可以了，整个C层和V层的代码不变。

mvc缺点：业务逻辑和业务展示强耦合

必须生成cell，然后点击cell上面的点赞按钮，才可以触发点赞的业务逻辑。
但是业务逻辑一般改变的model数据，view只是拿到model的数据进行展示。现在却把这两个原本独立的事情合在一起了。导致业务逻辑没法单独测试了。

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {

    BlogCellHelper *cellHelper = self.blogs[indexPath.row];
    BlogTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ReuseIdentifier];
    cell.title = cellHelper.blogTitleText;
    cell.summary = cellHelper.blogSummaryText;
    cell.likeState = cellHelper.isLiked;
    cell.likeCountText = cellHelper.blogLikeCountText;
    cell.shareCountText = cellHelper.blogShareCountText;

    //点赞的业务逻辑
    __weak typeof(cell) weakCell = cell;
    [cell setDidLikeHandler:^{
        if (cellHelper.blog.isLiked) {
            [self.tableView showToastWithText:@"你已经赞过它了~"];
        } else {
            [[UserAPIManager new] likeBlogWithBlogId:cellHelper.blog.blogId completionHandler:^(NSError *error, id result) {
                if (error) {
                    [self.tableView showToastWithText:error.domain];
                } else {
                    cellHelper.blog.likeCount += 1;
                    cellHelper.blog.isLiked = YES;
                    //点赞的业务展示
                    weakCell.likeState = cellHelper.blog.isLiked;
                    weakCell.likeCountText = cellHelper.blogTitleText;
                }
            }];
        }
    }];
    return cell;
}
```

#### MVP

MVC的缺点在于并没有区分业务逻辑和业务展示, 这对单元测试很不友好. MVP针对以上缺点做了优化, 它将业务逻辑和业务展示也做了一层隔离, 对应的就变成了MVCP.



> Presenter层职责

- 实现view的事件处理逻辑，暴露相应的接口给view的事件调用
- 调用model的接口获取数据，然后加工数据，封装成view可以直接用来显示的数据和状态
- 处理界面之间的跳转（这个根据实际情况来确定放在P还是C）







##### MVVM（Model View View-Mode）
MVVM：将业务逻辑从 Controller 移出放到一个新的对象里，即 View Model，促进了 UI 代码与业务逻辑的分离。
![3](http://ww2.sinaimg.cn/large/006tNc79ly1g4md9ctz33j31300fujt2.jpg)

view 和 view controller 都不能直接引用model，而是引用视图模型（viewModel）

viewModel 是一个放置用户输入验证逻辑，视图显示逻辑，发起网络请求和其他代码的地方

使用MVVM会轻微的增加代码量，但总体上减少了代码的复杂性

MVVM 配合一个绑定机制效果最好

https://objccn.io/issue-13-1/

