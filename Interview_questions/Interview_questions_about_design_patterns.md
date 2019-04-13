# 面试题系列之设计模式

## 1. 什么是设计模式？

《设计模式》一书中关于设计模式的定义：对定制来解决特定场景下一半设计问题的类和互相通信的对象的描述。简而言之，设计模式时为特定场景下的问题而定制的解决方案。

常见的设计模式：

- 原型模式
- 工厂方法模式
- 抽象工厂模式
- 生成器模式
- 单例模式
- 适配器模式
- 桥接模式
- 外观模式
- 中介者模式
- 观察者模式
- 组合模式
- 迭代器模式
- 访问者模式
- 装饰模式
- 责任链模式
- 模板方法模式
- 策略模式
- 命令模式
- 共享池模式
- 代理模式
- 备忘录模式

### 1.1 简单工厂模式、工厂模式以及抽象工厂模式？

[参考一](https://www.jianshu.com/p/847af218b1f0)
[参考二](https://blog.csdn.net/shihuboke/article/details/73921535)

### 1.2 抽象工程模式在 Cocoa SDK 中的那些类中有体现？

### 1.3 单例模式

```objc
+ (instancetype)sharedInstance {
    static InstanceType *instance;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[InstanceType alloc] init];
    });
    return instance;
}
```

Swift 已经不需如此处理，它已经从语义上保证只被初始化一次。

### 1.10.1 Delegate、Notification、KVO 的区别，优缺点

- Delegate 是单向的委托，你只能委托给一个代理对象；
- Notification 可以一对多的发送消息，但是需要你主动发送，接收方需要注册；
- KVO 也可以实现一对多，且不需要主动触发，

### 1.10.2 什么时候应该使用 Notification，什么时候应该使用 KVO，它们的实现上有什么区别吗？如果用 protocol 和delegate（或者 delegate 的 Array）来实现类似的功能可能吗？如果可能，会有什么潜在的问题？如果不能，为什么？

1. 观察者和被观察者都必须是 NSObject 的子类，因为 OC 中 KVO 的实现基于 KVC 和 runtime 机制，只有是 NSObject 的子类才能利用这些特性；
2. 如果是 Swift 观察的属性需要使用 dynamic 关键字修饰，表示该属性的存取都由 runtime 在运行时来决定，由于 Swift 基于效率的考量默认禁止了动态派发机制，因此要加上该修饰符来开启动态派发。

## 2. 架构

- MVC: Model-View-Controller
- MVP: Model-View-Presenter
- MVVM：Model-View-View Model

### 2.1 MVC 架构
MVC 的优点：

MVC 的缺点：

- 增加了系统结构和实现的复杂性。对于简单的界面，严格遵循MVC，使模型、视图与控制器分离，会增加结构的复杂性，并可能产生过多的更新操作，降低运行效率。
- 视图与控制器间的过于紧密的连接。视图与控制器是相互分离，但确实联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用。
- 视图对模型数据的低效率访问。依据模型操作接口的不同，视图可能需要多次调用才能获得足够的显示数据。对未变化数据的不必要的频繁访问，也将损害操作性能。
- 目前，一般高级的界面工具或构造器不支持MVC模式。改造这些工具以适应MVC需要和建立分离的部件的代价是很高的，从而造成使用MVC的困难。

### 2.2 MVP 架构

### 2.3 MVVM 架构

#### 2.3.1 MVVM如何实现绑定













