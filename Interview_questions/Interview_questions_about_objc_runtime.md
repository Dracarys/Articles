# 面试题系列之Runtime

## 1. 运行时基础
### 1.1 运行时的载入过程如何

1. read all classes in all new images, add them all to unconnected_hashl;
2. read all categories in all new images,
3. try to connected all classes
4. resolve selector refs and class refs
5. fix up protocol object
6. call `+load()` for classes and categories
7. all classes are ready before any categories are ready.

### 1.2 load 和 initialize 方法的调用时机？

- load 当类被加载到 runtime 的时候运行，在 main 函数执行之前，也就是类加载器加载时，每个类默认只会调用一次；通常被用来进行 Method Swizzle，但是这会增加 App 启动时间。
- initialize 在类接收到第一条消息之前被用调用。每个类只会调用一次。子类未实现会向上查找。一搬用来初始化全局变量或静态变量。

### 1.3 ISA指针
isa 指针实际上是一个联合体，其结构如下：

```cpp
// 精简过的isa_t共用体
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;

# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
    struct {
    // 0代表普通的指针，存储着Class，Meta-Class对象的内存地址。
    // 1代表优化后的使用位域存储更多的信息。
    uintptr_t nonpointer        : 1; 
   // 是否有设置过关联对象，如果没有，释放时会更快
    uintptr_t has_assoc         : 1;
    // 是否有C++析构函数，如果没有，释放时会更快
    uintptr_t has_cxx_dtor      : 1;
    // 存储着Class、Meta-Class对象的内存地址信息
    uintptr_t shiftcls          : 33; // MACH_VM_MAX_ADDRESS 0x1000000000
    // 用于在调试时分辨对象是否未完成初始化
    uintptr_t magic             : 6;
    // 是否有被弱引用指向过。
    uintptr_t weakly_referenced : 1;
    // 对象是否正在释放
    uintptr_t deallocating      : 1;
    // 引用计数器是否过大无法存储在isa中
    // 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中
    uintptr_t has_sidetable_rc  : 1;
    // 里面存储的值是引用计数器减1
    uintptr_t extra_rc          : 19;
    
#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };
#endif
};
```

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
- category 可以追加属性，但只能生成 setter 和 getter 的声明，不能生成 setter 和 getter 的实现；
- 如果 category 中的方法和类已有方法同名，category 中的方法会“覆盖”掉类中的已有方法（并不是真的覆盖，而是插入到原有方法之前，所以会被优先查询并返回）；
- 如果多个 category 中存在同名的方法啊，因为运行时加载时添加的顺序是无法保证的，所以“覆盖”的顺序也就不能确定，进而导致最终不知道会调用哪个。

### 2.3 category VS extension

- extension 运行在编译期，它是类的一部分，拓展的方法，属性和变量一起形成一个完整的类。而 category 是运行期决定的，此时对象的内存布局已经确定，无法再追加变量
- extension 一般用来隐藏类的私有信息，也就说你只能给已有源码的类添加 extension，而 category 则不存在该问题；
- category 不能添加成员变量，而 extension 则没这个限制。

### 2.4 category 原理
分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。
Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量

> 引自掘金 xx_cc 的[《Category 的本质》](https://juejin.im/post/5aef0a3b518825670f7bc0f3)

### 2.5 category 不能添加成员变量，那么如果要添加有什么办法吗？
有，两种方式：

- 利用静态变量，category 可以添加属性，通过属性来操作该静态变量。但是这样，即便类被销毁了，该静态变量也持续存在，且依然保留原有的值；
- 利用关联对象 Associate Object；

### 2.6 如何使用关联对象呢？
```objc
-(void)setName:(NSString *)name
{
    objc_setAssociatedObject(self, @"name",name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
-(NSString *)name
{
    return objc_getAssociatedObject(self, @"name");    
}
```
添加属性：

```objc
objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
```
关联对象的属性设置：

```objc
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,  // 指定一个弱引用相关联的对象
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 指定相关对象的强引用，非原子性
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 指定相关的对象被复制，非原子性
    OBJC_ASSOCIATION_RETAIN = 01401,  // 指定相关对象的强引用，原子性
    OBJC_ASSOCIATION_COPY = 01403     // 指定相关的对象被复制，原子性   
};
```
获取属性：

```objc
objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
```
移除属性：

```objc
- (void)removeAssociatedObjects
{
    // 移除所有关联对象
    objc_removeAssociatedObjects(self);
}

```
### 2.7 关联对象原理
一个实例对象就对应一个 ObjectAssociationMap，而ObjectAssociationMap 中存储着多个此实例对象的关联对象的 key 以及 ObjcAssociation，为 ObjcAssociation 中存储着关联对象的 value 和 policy 策略。
由此我们可以知道关联对象并不是放在了原来的对象里面，而是自己维护了一个全局的map用来存放每一个对象及其对应关联属性表格

![关联对象原理](https://user-gold-cdn.xitu.io/2018/5/14/1635a628a228e349?imageView2/0/w/1280/h/960/ignore-error/1)
> 引自掘金 cc_xx [《关联对象实现原理》](https://juejin.im/post/5af86b276fb9a07aa34a59e6)

### 2.8 使用 runtime Associate 方法关联的对象，需要在主对象 dealloc 的时候释放吗？
无论在 MRC 下还是 ARC 下均不需要在主对象 dealloc 的时候释放，被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的 object_dispose() 方法中释放。

### 2.9 关联属性如何显示 weak ？
利用中间对象包装一层？？？？

```objc
-(void)setWeakvalue:(NSObject *)weakvalue {
    __weak typeof(weakvalue) weakObj = weakvalue;
    typeof(weakvalue) (^block)() = ^(){
        return weakObj;
    };
    objc_setAssociatedObject(self, weakValueKey, block, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
-(NSObject *)weakvalue {
    id (^block)() = objc_getAssociatedObject(self, weakValueKey);
    return block();
}
```

## 3. 消息转发（objc_mgSend）
![方法调用的流程](https://user-gold-cdn.xitu.io/2018/7/2/16456e432e51af79?imageView2/0/w/1280/h/960/ignore-error/1)

### 3.1 为什么 Objective—C 的方法不叫调用叫发消息?
因为 Objective-C 中的方法调用其实都是转成了 objc_msgSend 函数的调用，给 receiver（方法调用者）发送了一条消息（selector方法名）。方法调用过程中也就是 objc_msgSend 底层实现分为三个阶段：消息发送、动态方法解析、消息转发

```
id _Nullable objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```
> ⚠️注意参数

### 3.2 给一个对象发消息的过程如何，或者说如何通过 selector 找到对应的 IMP 地址：

1. 先在 cache 中查找
2. 在方法类表中查找
3. 到父类的 cache，方法列表中查找
4. 查遍所有直到根类

### 3.3 类对象也是如此吗：
是的，只是实例对象要先通过 isa 指针取得该实例的类，然后就一样了。如果是类方法，因为类方法定义在原类中，则先通过 ISA 指针得到元类，之后就一样了。

### 3.4 如果消息发送失败有哪些补救措施：

- 动态解析：你不能处理啊，那你要添加一个的吗？实例方法调用 `+(BOOL)resolveInstanceMethod:(SEL)selector`，如果是类方法，那么调用 `+(BOOL)resolveClassMethod:(SEL)selector`，如果要动态添加，就重写响应方法，在方法实现中添加，并返回 YES， 如果不，要记得调用 super；
- 消息转发：添加不了啊，那要我转发给别人吗？`-(id)forwardingTargetForSelector:(SEL)selector`，如果有备用处理对象，那么在此返回，否则返回 nil。通过此方法，可以配合 composition（在对象内部封入子对象，将任务交给子对象处理） 来模拟“多重继承”的某些特性。
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
向实例对象 `person` 添加监听，Runtime 便动态生成一个 `Person` 类的子类 `NSKVONotifying_Person `，这个子类会重写 `setter`、`class`、`dealloc` 以及 `_isKVOA` 等相关方法（之所以要重写 `Class` 方法是为了尽量保持与原类相同，避免因为替换了类的实现而导致异常）。并修改被监听实例的 `isa` 指针，让其指向这个新生成的子类 `NSKVONotifying_Person`。因为这个新增的子类实现了 KVO 的相关方法，所以当我们再对该实例发送消息时，会通过 `isa` 指针先到这个新的子类中查找相应方法，从而得到通知。

![动态子类的方法查找过程](https://user-gold-cdn.xitu.io/2018/4/21/162e65b0293d6569?imageView2/0/w/1280/h/960/ignore-error/1)

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