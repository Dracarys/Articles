# 面试题系列之设计模式

## 什么是设计模式？

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

### 简单工厂模式、工厂模式以及抽象工厂模式？

[参考一](https://www.jianshu.com/p/847af218b1f0)
[参考二](https://blog.csdn.net/shihuboke/article/details/73921535)

### 抽象工程模式在 Cocoa SDK 中的那些类中有体现？

### 单例模式

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

## 架构

- MVC: Model-View-Controller
- MVP: Model-View-Presenter
- MVVM：Model-View-View Model

### MVC 架构
MVC 的优点：

MVC 的缺点：

- 增加了系统结构和实现的复杂性。对于简单的界面，严格遵循MVC，使模型、视图与控制器分离，会增加结构的复杂性，并可能产生过多的更新操作，降低运行效率。
- 视图与控制器间的过于紧密的连接。视图与控制器是相互分离，但确实联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用。
- 视图对模型数据的低效率访问。依据模型操作接口的不同，视图可能需要多次调用才能获得足够的显示数据。对未变化数据的不必要的频繁访问，也将损害操作性能。
- 目前，一般高级的界面工具或构造器不支持MVC模式。改造这些工具以适应MVC需要和建立分离的部件的代价是很高的，从而造成使用MVC的困难。

### MVP 架构

### MVVM 架构

#### MVVM如何实现绑定











