# 面试题系列之运行时

## 一、载入过程

1. read all classes in all new images, add them all to unconnected_hashl;
2. read all categories in all new images,
3. try to connected all classes
4. resolve selector refs and class refs
5. fix up protocol object
6. call `+load()` for classes and categories
7. all classes are ready before any categories are ready.

## 一、类别（category）

#### 什么是 category 

[参考](https://juejin.im/post/5a9d14856fb9a028e52d5568)

下面是 category 的结构体：
``` C++
typedef struct category_t *Category;

struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

 category 是 Objective-C 2.0 之后添加的语言特性，主要作用是为已经存在的类追加方法。此外 Apple 还推荐另外两个使用场景：
 
 1. 可以把类的实现分开在几个不同的文件里，这样可以减少单个文件的体积；可以对功能进行分类，组织到不同的 category 中；可由多人共同开发一个类，减少冲突；按需加载，方便配置；
 2. 声明私有方法；

#### category 的特点

- category 只能给某个已有的类追加方法，不能追加成员变量
- category 可以追加属性，但只能生成 setter 和 getter 的声明，不能生成 setter 和 getter 的实现即成员变量；
- 如果 category 中的方法和类已有方法同名，category 中的方法会覆盖掉类中的已有方法；
- 如果多个 category 中存在同名的方法啊，因为运行时加载时添加的顺序是无法保证的，所以覆盖的顺序也就不能确定，进而导致最终不知道会调用哪个。


参考里说由编译器决定，有待验证，因为这跟 runtime 里的注释不符。

#### category VS extension

- extension 运行在编译期，它是类的一部分，拓展的方法，属性和变量一起形成一个完整的类。而 category 是运行期决定的，此时对象的内存布局已经确定，无法再追加变量
- extension 一般用来隐藏类的私有信息，也就说你只能给已有源码的类添加 extension，而 category 则不存在该问题；
- category 不能添加成员变量，而 extension 则没这个限制。

#### category 原理

原理非常简单，运行时准备时，会将 category 中的方法循环添加到类的方法列表中去，包括 protocol 列表， property 列表等，也是同理。同名的方法会被覆盖。

#### 使用 runtime Associate 方法关联的对象，需要在主对象 dealloc 的时候释放吗？
无论在MRC下还是ARC下均不需要在主对象dealloc的时候释放，被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的object_dispose()方法中释放。
## 三、消息转发（objc_mgSend）

#### 为什么 Objective—C的方法不叫调用叫发消息?

```
id _Nullable objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```

#### 给一个对象发消息的过程如何，或者说如何通过 selector 找到对应的 IMP 地址：

1. 现在 cache 中查找
2. 在方法类表中查找
3. 到父类中查找
4. 查遍所有直到根类

#### 类对象也是如此吗：

是的，只是实例对象要先通过 isa 指针取得该实例的类，然后就一样了。

#### 如果消息发送失败有哪些补救措施：

- 你是不是发错人了，要不要转发给别人吗？如果要，那么转给谁？
- 这没人能处理，要不要添加一个吗？
- 全部内容都在这，你看着处理吧

### objc_msgSend函数参数

```
id _Nullable objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```

### 消息转发哪些步骤可以被利用

## 四、其它

### 如何访问并修改一个类的私有属性

1. 通过 KVC 获取
2. 通过 runtime 访问并修改私有属性

### weak实现机制？为什么对象释放后会自动置为nil？
运行时会维护一张 weak 哈希表， weak 对象的地址作为 key，该表记录了每个对象的引用计数，当计数为 0 时就会触发销毁机制，

### autorealse 如何实现的

这个解释不完善，有待进一步研究
autoreleasePool 是一个Objective-C的一种内存自动回收机制，它可以延时家如autoreleasePool中的变量释放的时机。
autoreleasePool在Runloop时间开始之前（push），释放是在一个 RunLoop 事件即将结束之前（pop）

#### 子线程中需要加autoreleasepool吗？什么时间释放？
每个线程都默认有一个autoreleasepool，在线程即将退出前释放。但是如果该线程会产生大量的内存碎片，那么建议创建runloop以及自己的释放池，以便可以及时释放。

#### autorelease 和 @autoreleasepool区别

### 如何hook一个对象的方法，而不影响其它对象。

methodswazzing，方法替换，有待进一步验证。

### 设计一个检测主线程卡顿的方案

- 基于runloop，验证一个循环是不是在1/60秒内完成；
- 我们启动一个worker线程，worker线程每隔一小段时间（delta）ping以下主线程（发送一个NSNotification），如果主线程此时有空，必然能接收到这个通知，并pong以下（发送另一个NSNotification），如果worker线程超过delta时间没有收到pong的回复，那么可以推测UI线程必然在处理其他任务了，此时我们执行第二步操作，暂停UI线程，并打印出当前UI线程的函数调用栈

### 渲染 UI 为什么要在主线程
UIKit 并不是一个 线程安全 的类，UI 操作涉及到渲染访问各种 View 对象的属性，如果异步操作下会存在读写问题，而为其加锁则会耗费大量资源并拖慢运行速度。另一方面因为整个程序的起点 UIApplication 是在主线程进行初始化，所有的用户事件都是在主线程上进行传递（如点击、拖动），所以 view 只能在主线程上才能对事件进行响应。而在渲染方面由于图像的渲染需要以60帧的刷新率在屏幕上 同时 更新，在非主线程异步化的情况下无法确定这个处理过程能够实现同步更新

[参考](https://juejin.im/post/5c406d97e51d4552475fe178)

### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

1. 不可以，因为结构的偏移已经固定了
2. 可以，这是新建，当然可以声明并定义了