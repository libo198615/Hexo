---
title: RxSwift
date: 2019-01-13 13:58:06
categories:
- Swift
tags:
---

以下是本人的笔记，如果有幸被你看到，最好看一下[官网](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/)

{% asset_img 3.jpg 图片说明 %}

### 流程
1. `Observable`发送信号`String`,`Int`
2. `map`信号变换`Bool`
3. `Observer`接收信号
### Observable 可被监听的序列
可以发送信号的序列，信号类型为`String`,`Bool`,`Int`
```
let stringObservable: Observable<String> = textField.rx.text.orEmpty.asObservable()
```
主要使用`Observable `下面的是其变种
- Single 要么只能发出一个元素，要么产生一个 error 事件
- Completable 要么只能产生一个 completed 事件，要么产生一个 error 事件。
- Driver
- ControlEvent 专门用于描述 UI 控件所产生的事件， 不会产生 error 事件
```
let numbers: Observable<Int> = Observable.create { observer -> Disposable in
            
    observer.onNext(0)
    observer.onNext(1)
    observer.onNext(2)
    observer.onCompleted()
            
    return Disposables.create()
}
```
### Observer
观察者 响应Observable发出的信号
- AnyObserver
- Binder 不会处理错误事件 确保在主线程

// create AnyObserver
```
let observer: AnyObserver<Data> = AnyObserver { (event) in
    switch event {
        case .next(let data):
            print("Data Task Success with count: \(data.count)")
        case .error(let error):
            print("Data Task Error: \(error)")
        default:
            break
        }
    }
        
    URLSession.shared.rx.data(request: URLRequest(url: URL.init(string: "a")!))
        .subscribe(observer)
        .disposed(by: disposeBag)
```
// create Binder
```
let usernameValidOutlet = UIView()
let observer: Binder<Bool> = Binder(usernameValidOutlet) { (view, isHidden) in
    view.isHidden = isHidden
}

usernameValid
    .bind(to: observer)
    .disposed(by: disposeBag)
```
让所有`view`都实现这种观察,来响应信号
```
extension Reactive where Base: UIView {
    public var isHidden: Binder<Bool> {
        return Binder(self.base) { view, hidden in
            view.isHidden = hidden
        }
    }
}

usernameValid
.bind(to: usernameValidOutlet.rx.isHidden)
.disposed(by: disposeBag)
```
```
textField.rx.text.asObservable().bind(to: label.rx.text)
```

### Observable 发送信号
// 创建 信号
1. 调用 Observable.create
2. 在构建函数里面描述元素的产生过程
3. 调用 observer.onCompleted() 表示完成

### Observer 响应信号

使用`subscribe`来响应
```
json.subscribe(onNext: { json in
    print(" json success ")
}, onError: {error in
    print(error)
}, onCompleted: {
    print(" completed ")
}).disposed(by: disposeBage)
```

---
### Event 事件
onNext onError onCompleted 被称为事件
```
public enum Event<Element> {
    case next(Element)
    case error(Swift.Error)
    case completed
}
```

---
有些事物即使信号发送者也可以是接收者


一般用户可以通过UI控制的信号是发送者，UI显示是信号接受者。像`UITextField.text`即是用户控制又是界面展示
```
// 信号发送者 作为可被监听的序列 
let observable = textField.rx.text
observable.subscribe(onNext: { text in show(text: text) })
```
```
// 信号接收者 作为观察者
let observer = textField.rx.text
let text: Observable<String?> = ...
text.bind(to: observer)
```
###  AsyncSubject
`AsyncSubject` 将在源 `Observable` 产生完成事件后，发出最后一个元素（仅仅只有最后一个元素），如果源 `Observable` 没有发出任何元素，只有一个完成事件。
### PublishSubject
`PublishSubject` 将对观察者发送订阅后产生的元素，而在订阅前发出的元素将不会发送给观察者。
### BehaviorSubject
当观察者对 `BehaviorSubject` 进行订阅时，它会将源 `Observable` 中最新的元素发送出来（如果不存在最新的元素，就发出默认元素）。然后将随后产生的元素发送出来。

对比`PublishSubject`,多了一个当前元素信号
### ControlProperty
`ControlProperty` 专门用于描述 `UI` 控件属性的，不会产生 `error` 事件,一定在` MainScheduler` 订阅（主线程订阅）


---
### Schedulers - 线程调度器
```
let rxData: Observable<Data> = ...

rxData
    .subscribeOn(ConcurrentDispatchQueueScheduler(qos: .userInitiated))
    .observeOn(MainScheduler.instance)
    .subscribe(onNext: { [weak self] data in
        self?.data = data
    })
    .disposed(by: disposeBag)
```
使用 `subscribeOn`在子线程发送信号,使用 `observeOn`在主线程监听信号

---
### combineLatest 合并信号
```
Observable.combineLatest( // 合并信号
    number1.rx.text.orEmpty, // 信号源为三个label的text
    number2.rx.text.orEmpty, 
    number3.rx.text.orEmpty) 
    { textValue1, textValue2, textValue3 -> Int in
        // 三个输入源作为参数，将其转换成Int信号
        return (Int(textValue1) ?? 0) + (Int(textValue2) ?? 0) + (Int(textValue3) ?? 0)
            }
    .map { $0.description } // 将Int信号转变为 text信号
    .bind(to: result.rx.text) // text信号和label的text绑定
    .disposed(by: disposeBag)
```
`orEmpty`会把`String?`过滤`nil`帮我们变为`String`类型。

---
### disposed 声明周期
```
// 声明 disposeBag 属性，其生命周期同此VC
var disposeBag = DisposeBag()

// 最后调用下面方法，VC释放时会释放disposeBag，从而释放绑定
.disposed(by: self.disposeBag)
```
```
// 方法里面不能直接使用self，会造成内存泄漏，添加[weak self]，相当于使用weakSelf
.subscribe { [weak self] (text) in
```
---
### just 产生特定的一个元素
一个序列只有唯一的元素 0：
```
let id = Observable.just(0)
```
---
### from 将其他类型或者数据结构转换为 Observable

将一个数组转换为 Observable：
```
let numbers = Observable.from([0, 1, 2])
```
将一个可选值转换为 Observable：
```
let optional: Int? = 1
let value = Observable.from(optional: optional)
```
---
### merge 任意一个 Observable 产生了元素，就发出这个元素

{% asset_img 1.png 图片说明 %}
```
let subject1 = PublishSubject<String>()
let subject2 = PublishSubject<String>()

Observable.of(subject1, subject2)
    .merge()
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```
---
### combineLatest 当任意一个 Observable 发出一个新的元素
{% asset_img 2.png 图片说明 %}
```
let first = PublishSubject<String>()
let second = PublishSubject<String>()

Observable.combineLatest(first, second) { $0 + $1 }
          .subscribe(onNext: { print($0) })
          .disposed(by: disposeBag)
```
---
### map 对每个元素直接转换
```
Observable.of(1, 2, 3)
    .map { $0 * 10 }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
```
---
### filter 通过判定条件过滤出一些元素
```
Observable.of(2, 30, 22, 5, 60, 1)
          .filter { $0 > 10 }
          .subscribe(onNext: { print($0) })
          .disposed(by: disposeBag)
```