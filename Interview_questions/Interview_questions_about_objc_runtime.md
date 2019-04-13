# 面试题系列之Runtime

## 1. 载入过程

1. read all classes in all new images, add them all to unconnected_hashl;
2. read all categories in all new images,
3. try to connected all classes
4. resolve selector refs and class refs
5. fix up protocol object
6. call `+load()` for classes and categories
7. all classes are ready before any categories are ready.

### 1.1 load 和 initialize 方法的调用时机？

- load 当类被加载到 runtime 的时候运行，在 main 函数执行之前，也就是类加载器加载时，每个类默认只会调用一次；通常被用来进行 Method Swizzle，但是这会增加 App 启动时间。
- initialize 在类接收到第一条消息之前被用调用。每个类只会调用一次。子类未实现会向上查找。一搬用来初始化全局变量或静态变量。

## 2. 类别（category）

### 2.1 什么是 category 
下面是 category 的结构体：

```cpp
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

### 2.2 category 的特点

- category 只能给某个已有的类追加方法，不能追加成员变量
- category 可以追加属性，但只能生成 setter 和 getter 的声明，不能生成 setter 和 getter 的实现即成员变量；
- 如果 category 中的方法和类已有方法同名，category 中的方法会覆盖掉类中的已有方法；
- 如果多个 category 中存在同名的方法啊，因为运行时加载时添加的顺序是无法保证的，所以覆盖的顺序也就不能确定，进而导致最终不知道会调用哪个。

参考里说由编译器决定，有待验证，因为这跟 runtime 里的注释不符。

### 2.3 category VS extension

- extension 运行在编译期，它是类的一部分，拓展的方法，属性和变量一起形成一个完整的类。而 category 是运行期决定的，此时对象的内存布局已经确定，无法再追加变量
- extension 一般用来隐藏类的私有信息，也就说你只能给已有源码的类添加 extension，而 category 则不存在该问题；
- category 不能添加成员变量，而 extension 则没这个限制。

### 2.4 category 原理



### 2.5 使用 runtime Associate 方法关联的对象，需要在主对象 dealloc 的时候释放吗？
无论在MRC下还是ARC下均不需要在主对象 dealloc 的时候释放，被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的object_dispose()方法中释放。

### 2.6 关联属性如何显示 weak ？

利用中间对象包装一层？？？？

## 3. 消息转发（objc_mgSend）

### 3.1 为什么 Objective—C 的方法不叫调用叫发消息?
因为最终所有方法的调用都会转为下面这样的调用形式，所以被称为“发消息”。

```
id _Nullable objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```
> ⚠️注意参数

### 3.2 给一个对象发消息的过程如何，或者说如何通过 selector 找到对应的 IMP 地址：

1. 先在 cache 中查找
2. 在方法类表中查找
3. 到父类中查找
4. 查遍所有直到根类

### 3.3 类对象也是如此吗：
>类方法，也就是工厂方法是存放在元类中的，有待验证。

是的，只是实例对象要先通过 isa 指针取得该实例的类，然后就一样了。

### 3.4 如果消息发送失败有哪些补救措施：

- 你不能处理啊，那你要添加一个的吗？实例方法调用 `+(BOOL)resolveInstanceMethod:(SEL)selector`，如果是类方法，那么调用 `+(BOOL)resolveClassMethod:(SEL)selector`，如果要动态添加，就重写响应方法，在方法实现中添加，并返回 YES， 如果不，要记得调用 super；
- 添加不了啊，那要我转发给别人吗？`-(id)forwardingTargetForSelector:(SEL)selector`，如果有备用处理对象，那么在此返回，否则返回 nil。通过此方法，可以配合 composition（在对象内部封入子对象，将任务交给子对象处理） 来模拟“多重继承”的某些特性。
- 全部内容都在这里了（NSInvocation），你看着办吧。`-(void)forwardInvocation:(NSInvocation *)invocation`，此方法可以实现的很简单，只需该面调用目标，使消息在新目标上得以调用即可。这就与上一步类似了。还可以在触发消息前，先以某种方式改变消息内容，比如追加另外一个参数，或者该换选择子（selector），等等。

接受者在以上的每一步都有机会处理消息，越往后代价越大。最好能在第一步就处理完毕，这样的话，运行时可以将此方法缓存起来。

### 消息转发哪些步骤可以被利用

消息转发时可以结合组合来模拟多重继承的某些特性。

## 4. KVO
Key-value Observing，即键值监听，可以用于监听某个对象属性的变化。

### 4.1 KVO 的基本原理
先上结论：利用 Runtime 动态创建一个支持 KVO 的子类，修改该实例对象的 `isa` 指针，让其指向刚刚创建的子类，从而实现 KVO。

下面来看一下具体过程如何。假设我们有一个 `Person` 的实例对象，那么它查找方法的过程如下图所示：
![原实例方法查找过程](https://user-gold-cdn.xitu.io/2018/4/21/162e65aff55e142f?imageView2/0/w/1280/h/960/ignore-error/1)
一旦我们通过下面的方法：

    [person addObserver: forKeyPath: options: context:]
向实例对象 `person` 添加监听，Runtime 便动态生成一个 `Person` 类的子类 `NSKVONotifying_Person `，这个子类会重写 `setter`、`class`、`dealloc` 以及 isKVOA 等相关方法。并修改被监听实例的 `isa` 指针，让其指向这个新生成的子类 `NSKVONotifying_Person`。因为这个新增的子类实现了 KVO 的相关方法，所以当我们再对该实例发送消息时，会通过 `isa` 指针先到这个新的子类中查找相应方法，从而得到通知。

同理，当我们调用下面的方法：

    [Object removeObserver: forKeyPath:]
移除一个对象实例的监听时，Runtime 会将该实例的 `isa` 指针恢复，同时移除生成的子类。

> 关于销毁 KVO 的动态子类还有待进一步研究

### 4.2 KVO 在多线程中的行为如何？
- KVO 是同步的，一旦对象的属性发生变化，只有用同步的方式，才能保证所有观察者的方法能够执行完成。KVO 监听方法中，不要有太耗时的操作。

- KVO 的方法调用，是在对应的线程中执行的。在子线程修改观察属性时，观察者回调方法将在子线程中执行。

- 在多个线程同时修改一个观察属性的时候，KVO 监听方法中会存在资源抢夺的问题，需要使用互斥锁。如果涉及到多线程，KVO 要特别小心，通常 KVO 只是做一些简单的观察和处理，千万不要搞复杂了，KVO的监听代码，一定要简单。

[参考](http://www.cnblogs.com/QianChia/p/5771074.html)

### 4.3 如何手动触发 KVO？
首先关闭默认：
``` Objective-C
+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key
{
    
    if ([key isEqualToString:@"想要手动控制的key"])return NO;
    
    return [super automaticallyNotifiesObserversForKey:key];
}
```
然后重写 setter ：

``` Objective-C
- (void)setTmpStr:(NSString *)tmpStr
{
    [self willChangeValueForKey:@"tmpStr"];
    
    _tmpStr = tmpStr;
    
    [self didChangeValueForKey:@"tmpStr"];
}
```

## 10. 其它
### 10.1 如何访问并修改一个类的私有属性

1. 通过 KVC 获取
2. 通过 runtime 访问并修改私有属性

### 10.2 weak实现机制？为什么对象释放后会自动置为nil？
运行时会维护一张 weak 哈希表， weak 对象的地址作为 key，该表记录了每个对象的引用计数，当计数为 0 时就会触发销毁机制，


### 10.3 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

1. 不可以，因为结构的偏移已经固定了
2. 可以，这是新建，当然可以声明并定义了