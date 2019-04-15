# 面试题系列之设计模式
[toc]
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

### 1.2 抽象工程模式在 Cocoa SDK 中的哪些类中有体现？

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

## 2. 架构
架构（Architecture）

- MVC: Model-View-Controller
- MVP: Model-View-Presenter
- MVVM：Model-View-View Model

### 2.1 MVC 架构

![MVC架构图](http://joeyio.com/assets/mvc.png)

MVC 的优点：

- 各司其职，互不干涉：三层中某一个发生了需求变化，只需修改响应模块的逻辑可，不会影响到其它；
- 有利于开发分工：正式因为其具备互不干涉的特点，得以将开发工作分开；
- 有利于组件的重用：功能专一可以抽象成一些通用的组件，利于重用。

MVC 的缺点：

- 增加了系统结构和实现的复杂性。对于简单的界面，严格遵循MVC，使模型、视图与控制器分离，会增加结构的复杂性，并可能产生过多的更新操作，降低运行效率。
- 视图与控制器间的过于紧密的连接。视图与控制器是相互分离，但确实联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用。
- 视图对模型数据的低效率访问。依据模型操作接口的不同，视图可能需要多次调用才能获得足够的显示数据。对未变化数据的不必要的频繁访问，也将损害操作性能。
- 目前，一般高级的界面工具或构造器不支持MVC模式。改造这些工具以适应MVC需要和建立分离的部件的代价是很高的，从而造成使用MVC的困难。



### 2.2 MVP 架构
MVP（Model View Presenter）架构师从著名的 MVC 架构演变而来的。虽然 MVC 很好的将 Mode 和 View 进行了分离，但是也带来一个问题，Model 的变化以及 View 的反馈，这些消息都必须经过 Controller 来进行，随着应用的开发，Controller 也就变得越来越臃肿不堪。所以在 Android 的开发中，Google 推出了 MVP 架构，以解决该问题。

#### 2.2.1 MVP 架构介绍
从名字便可知，MVP 同样是三层架构：

- Model：数据层，它区别于 MVC 架构中的 Model，在这里不仅仅只是数据模型，还兼具对数据的存取操作，例如：对数据库的读写，网络的数据的请求等等；
- View：视图层，仅负责对数据的展示，提供与用户进行交互的窗口；
- Presenter：它是连接 View 与 Model 的桥梁，并对业务逻辑进行处理。在 MVP 架构中 Model 与 View 无法直接进行交互。所以在 Presenter 中，它会从 Model 层中获得所需的数据，进行适当的处理后交给 View 进行展示。这样通过 Presenter 将 View 与 Model 进行隔离，使得 View 和 Model 之间不存在耦合，同时也将业务逻辑从 View 中抽离。

![MVP架构关系图](https://i.stack.imgur.com/vxJf4.png)

### 2.3 MVVM 架构

#### 2.3.1 MVVM如何实现绑定











