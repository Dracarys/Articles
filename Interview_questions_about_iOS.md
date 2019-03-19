# Interview questions of iOS

## 1 Objective-C

### 1.1 Objective-C 对象
Objective-C 是基于 C 实现的，因此是 C 的超集，通过下面的一些定义可以发现：objc中所有关于类的实现都是基于结构体的。

下面是 `obj.h` 的定义：

``` Objective-C
#if !OBJC_TYPES_DEFINED
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
#endif

/// An opaque type that represents a method selector.
typedef struct objc_selector *SEL;

/// A pointer to the function of a method implementation. 
#if !OBJC_OLD_DISPATCH_PROTOTYPES
typedef void (*IMP)(void /* id, SEL, ... */ ); 
#else
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
#endif
```

#### 1.1.1关于 Class 的实现：

``` Objective-C
struct objc_class 
{
    Class isa;
    Class super_class;// 指向父类                                   
    const char *name;                                         
    long version;                                             
    long info;                                                
    long instance_size;                                       
    struct objc_ivar_list *ivars;// internal varables? 成员变量列表                             
    struct objc_method_list **methodLists; // 方法列表                  
    struct objc_cache *cache; // 方法缓存                              
    struct objc_protocol_list *protocols; // 协议                 
}
```

#### 1.1.2 isa 指针指向什么？有什么作用？

指向：实例的 `isa` 指针指向其类，其类指向元类，元类指向根元类，根元类指向自己。

作用：运行时借此可以查看实例的类，查找类定义了哪些方法，进而找到实现。

#### 1.1.3 `isKindOfClass` VS `isMemberOfClass`
#?

### 1.2 Objective-C 的关键字

#### 1.2.1 @property 属性

属性本质就是声明一个存取器（getter、setter），编译器会根据声明自动生成实现。声明时涉及很多关键字，这些关键字会影响到存取器最终的行为。

默认的关键字有：Strong/retain, assign, atomic, nonnull。

##### nonatomic、atomic区别？atomic为什么不是绝对线程安全的？
`atomic` 和 `nonatomic` 的区别向编译器表明，生成的 `getter` 和 `setter` 方法是否为原子操作，即是否需要多线程安全特性，默认是 `atomic`。

修饰一些可变集合时不是安全的。
##### @synthesize 和 @dynamic 分别有什么作用？有了自动合成属性实例变量之后， @synthersize还有哪些使用场景？

- synthesize 告知编译器，需要自动合成属性实例变量，2.0 之后已经可以自动合成了，
- dynamic 告知编译器不要自动合成，我自己来实现。

有了自动合成需要的地方：

- eadwrite property with custom getter and setter
- readonly property with custom getter
- when using @dynamic propertyName, the property won't be automatically synthesized (pretty obvious, since @dynamic and @synthesize are mutually exclusive)
- properties declared in a @protocol
- properties declared in a category
- overridden properties，when you override a property of a superclass, you must explicitly synthesize it

[参考](https://stackoverflow.com/questions/19784454/when-should-i-use-synthesize-explicitly)

##### ivar、getter、setter 是如何生成并添加到这个类中的？

是由编译器在编译时根据声明的属性自动合成的，同时会想类中插入一个以“_"开头，与属性名同名的实例变量。

#### 1.2.2 其他关键字

##### self、super 关键字作用与区别

- self 是类的隐藏参数，指向调用方法的实例，与 this 类似，区别是工厂方法也可以使用。
- supper 时一个 Magic 关键字，它本质上是一个编译器标识符，告诉编译器到该类的父类中查找该方法。

##### instancetype 和 id 区别？

instancetype 可以最大化的利用编译器的检查功能，协助排查一些类型错误问题，而 id 则不具备。

##### volatile 关键字

##### designated initializer，怎么用，注意内容？
即指定构造器，该宏是在 Swift 出现后新增的，用法与 SWift 的指定构造器用法相同：

1. 子类如果有指定构造器，那么该指定构造器必须调用直接父类的指定构造器
2. 如果子类有指定构造器，那么便利构造器必须调用自己的其它构造器（包括制定构造器及其它便利构造器），不能调用supper的普通构造器
3. 子类如何好实现了指定构造器，那么必须实现所有父类的指定构造器。

#### 1.3 聊聊 Class extension

- 可以将暴露给外面的只读属性改为读写属性，方便内部使用（但我个人更推荐在内部尽量直接使用变量）
- 可以添加一些私有的实例变量、私有的属性、私有方法等你不想暴露给外部的内容。

#### 多态




### 关于Block

Block的底层实现：

Block的生命周期：

Block的三种类型：

- \_NSGlobalBlock_
- \_NSStackBlock_
- \_NSMallocBlock_

Block循环引用问题：

Block如何修改外部变量：

### 函数指针



#### 在block里堆数组执行添加操作，这个数组需要声明成 __block吗？同理如果修改的是一个NSInteger，那么是否需要？

#### 对于Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？
最大的优点是它的运行时特性，不足是没有命名空间，对于命名冲突，可以使用长命名法或特殊前缀解决，如果是引入的第三方库之间的命名冲突，可以使用link命令及flag解决冲突。
#### buildSetting link flag 解决命名冲突。

#### 实现一个NSString类

#### Objective-C中类方法和实例方法有什么本质的区别和联系？

类方法只能由类来调用，不能访问成员变量，实例方法只能由实例来调用，可以访问成员变量。类方法为该类所有对象共享

#### _objc_msgForward 函数是做什么的，直接调用会发生什么？

消息转发，通常是在目标上无法找到相应方法时处罚；直接调用将跳过查找 IMP 的过程。操作不当容易引起崩溃。

#### 手动出发KVO

首先关系默认：
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

### 指针

#### 指针常量、常量指针区别？

指针常量：`int * const ptr` 指针本身是一个敞亮。在声明的时候初始化，里面的值（存放的地址）不能更改；
常量指针：`const int * ptr`指针本身是一个常量，通过常量地址初始化，它可以再指向另一个常量地址。

### 运行完Test函数后会有什么结果

``` C
void getMemory(char *p) {
	p = (char *)malloc(100);
}

void test(void) {
	char *str = NULL;
	getMemory(str);
	strcpy(str, "hello, world");
	printf(str);
}
```
*参考答案：程序崩溃，应为 `getMemory()` 并不能传递动态内存，Test 函数中的 str 一直都是 NULL，copy将发生错误，导致崩溃*

另一个

``` C
void getMemory(char *p) {
	char p[] = "hello world";
	return p;
}

void test(void) {
	char *str = NULL;
	str = getMemory();
	printf(str);
}
```
*参考答案：可能乱码。因为 `getMemory()` 返回的是 “栈内存” 指针，该指针的地址不是 NULL，但其原先的内容已经被清除了，新内容不可知*。

再一个：

``` C
void getMemory(char **p, int num) {
	*p = (char *)malloc(num);
}

void test(void) {
	char *str = NULL;
	getMemory(&str, 100);
	strcpy(str, "hello");
	printf(str);
}
```
*参考答案：输出 “hello”，单丝会导致内存泄漏*

还一个：

``` C
void test(void) {
	char *str = (char *)malloc(100);
	strcpy(str, "hello");
	free(str);
	if (str != NULL) {
		strcpy(str, "world");
		printf(str);
	}
}
```
参考答案：篡改动态内存区的内容，后果难以预料，非常危险，因为 `free` 之后，str成为野指针，条件语句将不能按预定逻辑运行。


## 二、关于 Swift

### 什么是 Optional 类型，它用来解决什么问题？

表示可能为空值，解决空值崩溃的问题

#### 什么情况下不得不使用隐式拆包？为什么？

#### 可选类型解包的方式有哪些？安全性如何？

### 什么时候用 Structure，什么时候用 Class
值类型和引用类型的区别

### 什么是泛型？用来解决什么问题的？


### 闭包是引用类型吗？
是引用类型，不会发生复制。

### 搜索关键词 Swift Interview Questions

### what is the type of x? And what is its value?

``` Swift
let d = ["john": 23, "james": 24, "vincent": 34, "louis": 29]
let x = d.sorted{ $0.1 < $1.1 }.map{ $0.0 }
```

### what's the differences between `unowned` and `weak`?

- unowned: the reference is assumed to always have a value during its lifetime - as a consequence, the property must be of non-optional type
- weak: at some point it's possible for the reference to have no value - as a consequence, the property must be of optional type.



### 内存管理

#### 引用计数是如何实现的

#### 有哪些导致崩溃的常见问题？如何进行预防？

- 取空值，自定义取值方法，添加到category中
- 越界，取值时始终进行验证，且永远不要相信后台反馈
- 不能识别到方法，向错误类型发送消息，预防方法同上，此外还可以通过runtime处理消息转发方法，来减少崩溃

#### 内存泄漏的原因有哪些？如何解决？

- 创建后未释放，通过正确释放解决，重点关注 new、create等方法名
- 单位时间开辟大量空间，添加 autoreleasepool
- Corefoundation方法使用不当，持有权转移错误
- 循环引用，添加weak中间代理

#### 深拷贝、浅拷贝

#### BAD_ACCESS在什么情况下出现？如何调试？

读取了一个不是你管控的内存地址，通常是读取了一个已释放的指针

启动僵尸对象，然后结合发生错误时的操作，逐步定位问题根源。


## Runtime

### 消息相关
#### Objective-C的反射机制
#### 为什么 Objective—C的方法不叫调用叫发消息：

#### 给一个对象发消息的过程如何，或者说如何通过 selector 找到对应的 IMP 地址：

#### 类对象也是如此吗：

#### 如果消息发送失败有哪些补救措施：

- 你是不是发错了，要不要转发给别人吗？
- 没人能处理，要不要添加一个吗？
- 全部内容都在这，你看着处理吧

### 类别（category）

#### 什么是类别？实现原理如何？

``` C++
typedef struct category_t *Category;

struct category_t {
    const char *name;	//category名称
    classref_t cls; 	//要拓展的类
    struct method_list_t *instanceMethods; //给类添加的实例方法的列表
    struct method_list_t *classMethods;  //给类添加的类方法的列表
    struct protocol_list_t *protocols;  //给类添加的协议的列表
    struct property_list_t *instanceProperties;  //给类添加的属性的列表
};
```

[参考](https://juejin.im/post/5a9d14856fb9a028e52d5568)

#### 类别有哪些作用，有什么局限性？为什么？

类别主要有3个作用：

- 将类的实现分散到多个不同文件或多个不同框架中。
- 创建对私有方法的前向引用。
- 向对象添加非正式协议。

有两方面局限性：

- 无法向类中添加新的实例变量，类别没有位置容纳实例变量。
- 名称冲突，即当类别中的方法与原始类方法名称冲突时，类别具有更高的优先级。类别方法将完全取代初始方法从而无法再使用初始方法。
无法添加实例变量的局限可以使用字典对象解决

受局限的原因：

如何突破局限：

#### 使用 runtime Associate 方法关联的对象，需要在主对象dealloc的时候释放吗？
无论在MRC下还是ARC下均不需要在主对象dealloc的时候释放，被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的object_dispose()方法中释放。

### class的载入过程

### 如何访问并修改一个类的私有属性

1. 通过 KVC 获取
2. 通过 runtime 访问并修改私有属性

### weak实现机制？为什么对象释放后会自动置为nil？
运行时会维护一张 weak 哈希表， weak对象的地址作为 key，该表记录了每个对象的引用计数，当计数为 0 时就会处罚销毁机制，

### autorealse 如何实现的

这个解释不完善，有待进一步研究
autoreleasePool 是一个Objective-C的义仲内存自动回收机制，它可以延时家如autoreleasePool中的变量释放的时机。
autoreleasePool在Runloop时间开始之前（push），释放是在一个 RunLoop 事件即将结束之前（pop）

#### 子线程中需要加autoreleasepool吗？什么时间释放？
每个线程都默认有一个autoreleasepool，在线程即将退出前释放。但是如果该线程会产生大量的内存碎片，那么建议创建runloop以及自己的释放池，以便可以及时释放。

#### autorelease 和 @autoreleasepool区别

### 如何hook一个对象的方法，而不影响其它对象。

methodswazzing，方法替换，有待进一步验证。

### 设计一个检测主线程卡顿的方案

- 基于runloop，验证一个循环是不是在1/60秒内完成；
- 我们启动一个worker线程，worker线程每隔一小段时间（delta）ping以下主线程（发送一个NSNotification），如果主线程此时有空，必然能接收到这个通知，并pong以下（发送另一个NSNotification），如果worker线程超过delta时间没有收到pong的回复，那么可以推测UI线程必然在处理其他任务了，此时我们执行第二步操作，暂停UI线程，并打印出当前UI线程的函数调用栈

### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

1. 不可以，因为结构的便宜已经固定了；
2. 可以，这是新建，当然可以声明并定义了

## 多线程

### Objective-C 中的锁

- @synchronized关键字锁
- NSLock对象锁
- NSCondition
- NSConditionLock 条件锁
- NSRecursiveLock 递归锁
- pthread_mutex 互斥锁 (通用互斥锁,类UNix都有的)
- pthread_rwlock
- dispatch_semaphore 信号量实现枷锁（GCD）
- OSSpinLock
- POSIX conditions
- os_unfair_lock

#### 互斥锁、自旋锁优缺点？

- 互斥锁的代价更高，因为线程有休眠，要唤醒，但是安全性更好，
- 自旋锁代价小，效率高，但是有优先级反转的可能，且比较耗资源

#### 互斥锁、自旋锁是如何实现的？ 

#### 分别用 C/C++ 和 Objective-C 实现互斥锁、自旋锁。

#### 死锁档四个条件、优先级翻转

1. 条件互斥；
2. 占有和等待条件；
3. 不可抢占；
4. 环路等待；

优先级反转是指：如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock
#### 什么时候处理多线程，几种方式，优缺点？

#### 系统有哪些在后台运行的Thread

#### 队列和线程的关系

#### 线程同步的方式

#### 线程池的结构？如何实现的？

#### 开启一条线程的方法？线程可以取消吗？

- NSThread
- pThread
- GCD
- NSOperation

一旦提交即不可取消，为提交执行的可以。

#### runloop和线程的关系？各个mode是做什么的？如何实现一个runloop

防止线程退出，

```
do {

} while()
```

#### 用过NSOperationQueue么？如果用过或者了解的话，为什么要使用 NSOperationQueue，实现了什么？跟 GCD 之间的区别和类似得地方

### Dispatch_p



### 数据库

#### CoreData的原理，与SQLite相比优劣？

#### CoreData的6个成员类

#### CoreData 多线程中处理大量数据同步时的操作？

#### NSpersistentStoreCoordinator，NSManagedObjectContext 和 NSManagedObject 中的哪些需要在线程中创建或者传递？你用过什么样的策略实现的？

#### SQLite中插入特殊字符的方法和接受的处理方法？

#### 如果不用数据库，只使用普通文件，如何设计亿量级别的日志系统？

#### 索引的作用、优缺点，与主键的区别

#### 乐观锁VS悲观锁


## 设计模式

### 什么是设计模式？

### MVC的缺点

### 其它架构

### MVVM如何实现绑定

### 单例

### Delegate、Notification、KOV的区别，优缺点

#### NSNotification和KVO的区别和用法是什么？什么时候应该使用通知，什么时候应该使用KVO，它们的实现上有什么区别吗？如果用protocol和delegate（或者delegate的Array）来实现类似的功能可能吗？如果可能，会有什么潜在的问题？如果不能，为什么？

### 设计一个方案来检测KVO的同步一步问题，willChange和didChange的不同点

### 如果现在要实现一个下载功能，如何设计，每个类都觉题做什么？

### KVC如何实现的
模型的性质是通过一个简单的键（通常是个字符串）来指定的。视图和控制器通过键来查找相应的属性值。在一个给定的实体中，同一个属性的所有值具有相同的数据类型。键-值编码技术用于进行这样的查找—它是一种间接访问对象属性的机制。
键路径是一个由用点作分隔符的键组成的字符串，用于指定一个连接在一起的对象性质序列。第一个键的
性质是由先前的性质决定的，接下来每个键的值也是相对于其前面的性质。键路径使您可以以独立于模型
实现的方式指定相关对象的性质。通过键路径，您可以指定对象图中的一个任意深度的路径，使其指向相
关对象的特定属性。

#### KVC keyPaht 的集合运算符如何使用？
#### KVC 和 KVO中的 KeyPath一定是属性吗？
#### 如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？

### 如何设计一个网络请求库

### 设计一个图片缓存

### 简单工厂模式、工厂模式以及抽象工厂模式？

#### 抽象工程模式在 Cocoa SDK 中的那些类中有体现？

### 项目组件化用过吗？怎么接耦的？[解答](http://www.code4app.com/blog-822715-1562.html)

### 你实现过一个框架或者库以供别人使用么？如果有，请谈一谈构建框架或者库时候的经验；如果没有，请设想和设计框架的public的API，并指出大概需要如何做、需要注意一些什么方面，来使别人容易地使用你的框架


## CocoTouch

#### keyWindow、UIWindow的layer、UIView的继承关系

#### CoreGraphic、CGPath、maskLayer

#### NSTimer准吗？有哪些替代方案

#### load和initialize方法的调用试机？

#### 为什么一定要在主线程里更新UI

### View哪些属性时 animatable 的？

### LayouSubviews 何时会被调用？

- 初始化时不会触发
- 滚动 UIScrollView 时会触发
- 旋转 UIScreen 时会触发
- 当 view 的 frame 发生变化时会触发
- 

### 为什么动画完成后，layer会恢复到原先的状态？

#### 给一个View设置圆角的方法有哪些，各有什么不同？

#### 响应链、如何扩大View的响应范围

view 有一个是否响应事件的方法，重写该方法，对指定范围内的方法返回yes即可。

#### 手触碰到屏幕的时候，响应机制是怎样的？第一响应者是谁？追问 UIView和UIResponse的关系是什么？

Window,然后依次向下传递，

uiview 继承自UIResponse 
#### 直接用UILabel和自己用DrawRect画UILabel，那个性能好，为什么？那个占用的内存少？为什么？
直接用更好，调用 coreGraphics 会导致离屏渲染，

#### iOS的应用程序有几种状态？推到后台代码是否可以执行哦？双击home键，代码是否可以执行

可以执行，
- 自己主动保活，如播放无音音频文件；
- 通过notification 推送唤醒。

可以，有短暂的保存时间。

#### 一般使用的图标内存为多大？200x300的图片，内存应该占用多少比较合理？

#### 自线程中调用connection方法，为什么不回调？
没加入runloop，子线程被销毁了

#### UI框架和CA、CG框架的关系是什么？做过哪些内容？

#### Quartz 框架


## iOS SDK

### 蓝牙的围栏功能

### homeKit

### CoreText

### CoreImage

### 静态库和动态库之间的区别

### iOS从什么时候开始支持动态库的？

### iOS 的签名机制如何？






## 工具类

### 如何检测应用是否卡顿

instrument，animation测试

### 如何编写单元测试，例如测试一个网络库，如何提高覆盖率

尽量覆盖方法的每一个分支

### images.xcassetsa和直接使用图片有什么不一样？

### 平时是怎么进行测试的，内存方面怎么测试

### lldb常用调试命令？

### 如何给一款App瘦身

### 从点击图标到应用启动的过程？

### DSYM文件是什么，你是如何分析的？

符号文件，可以通过Xcode解析后，直接定位到问题代码

### 如果进行网络、内存、性能优化？

### 如何把异步线程转换成同步任务进行单元测试？[参考](https://mp.weixin.qq.com/s?mpshare=1&scene=23&mid=100000048&sn=bafde424579a5cb57d7d88f12fc5791e&idx=1&__biz=MzUyNDM5ODI3OQ%3D%3D&chksm=7a2cba984d5b338ecb16e7c9f374244bdf483a1c5452ce0df7da73d236f4783f906eeeef41c9&srcid=1017YqCOZ1dePfHkgcFDmICp#rd)




### 操作系统

#### 堆和栈的区别，为什么要分堆和栈？如何优化？那种会造成内存碎片化？
- 栈区(stack)由编译器自动分配释放 ,存放方法(函数)的参数值, 局部变量的值等，栈是向低地址扩展的数据结构，是一块连续的内存的区域。即栈顶的地址和栈的最大容量是系统预先规定好的。

- 堆区(heap)一般由程序员分配释放, 若程序员不释放,程序结束时由OS回收，向高地址扩展的数据结构，是不连续的内存区域，从而堆获得的空间比较灵活。

- 碎片问题：对于堆来讲，频繁的new/delete势必会造成内存空间的不连续，从而造成大量的碎片，使程序效率降低。对于栈来讲，则不会存在这个问题，因为栈是先进后出的队列，他们是如此的一一对应，以至于永远都不可能有一个内存块从栈中间弹出.

- 分配方式：堆都是动态分配的，没有静态分配的堆。栈有2种分配方式：静态分配和动态分配。静态分配是编译器完成的，比如局部变量的分配。动态分配由alloca函数进行分配，但是栈的动态分配和堆是不同的，他的动态分配是由编译器进行释放，无需我们手工实现。

- 分配效率：栈是机器系统提供的数据结构，计算机会在底层对栈提供支持：分配专门的寄存器存放栈的地址，压栈出栈都有专门的指令执行，这就决定了栈的效率比较高。堆则是C/C++函数库提供的，它的机制是很复杂的。

- 全局区(静态区)(static),全局变量和静态变量的存储是放在一块 的,初始化的全局变量和静态变量在一块区域, 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。程序结束后有系统释放。

- 文字常量区—常量字符串就是放在这里的。程序结束后由系统释放。

- 程序代码区—存放函数体的二进制代码

#### 线程和进程的区别，内存共享、进程买哦树、写时复制、操作系统启动《深入理解计算机系统》

-  一个程序至少要有进城,一个进程至少要有一个线程.
进程:资源分配的最小独立单元,进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位.
- 线程:进程下的一个分支,是进程的实体,是CPU调度和分派的基本单元,它是比进程更小的能独立运行的基本单位,线程自己基本不拥有系统资源,只拥有一点在运行中必不可少的资源(程序计数器、一组寄存器、栈)，但是它可与同属一个进程的其他线程共享进程所拥有的全部资源。
- 进程和线程都是由操作系统所体会的程序运行的基本单元，系统利用该基本单元实现系统对应用的并发性。
- 进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。
- 但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。

#### 静态常量去访问的过程

#### 进程间如何通信，方式有哪些

#### 两个进程分别只想同一个地址空间并初始化一个值，分别输出什么？

#### 程序执行的过程

#### 一个进程有哪些区

代码区、数据区、堆区、共享库、栈区、

#### 内核态和用户态的区别，为什么要这样分

隔离，

#### 为什么要有page cache，操作系统怎么设计的Page cache

#### 介绍5中IO模型

#### 异步编程的时间循环

#### 项目采用64位，为什么用64位，怎么修改成64位，i386又是指什么，他们之间什么关系



### 网络协议

#### 七层模型

#### TCP/UDP区别，头部分别有多长？

#### TCP的三次握手四次挥手

#### TCP滑动窗口，窗口大小，如何确定的

#### HTTP有哪些部分？HTTPS的原理

#### HTTPS 密钥协商交还的过程

#### HTTP请求有哪些方法？如何选择？

#### 为什么要使用HTTP，而不直接用TCP

#### 如何保证HTTP传输到达

#### TCP的拥塞控制

#### ping是什么协议

#### 传输层和网络层分别是做什么的？

#### UDP可以实现一对多吗？怎么实现？

#### 发送一个HTTP请求的过程，那么HTTPS呢？

#### TCP是如何保证可靠的

#### cookie

#### 网络造成卡顿的原因

#### 断点续传怎么实现？需要设置什么？

#### 在杭州HTTP请求服务器响应快，可能离服务器距离近，而在深圳访问就很慢很慢，会是什么原因？如果用户投诉，怎么分析这个问题？



## 三方框架

### AFNetworking，是否支持IPv6

3.0之后支持

### AFNetwoking 为什么添加一条常驻线程？
网络请求是异步的，这会导致获取到请求数据时，线程已经退出，代理方法没有机会执行。因此，AFN 的做法是使用一个 runloop 来保证线程不死~
然而频繁的创建线程并启动runloop肯定会造成内存泄露(runloop 无法停止.线程无法退出)
所以AFN就创建了一个单例线程,并且保证线程不退出
[参考](https://www.jianshu.com/p/7170035a18e8)

### YYKit

#### YYAsyncLayer如何进行异步绘制的？

#### YYCache

#### YYModel

### SDWebImage

### Kingfisher

### Pod 的工作原理

#### Pod update和 Pod install的区别

#### malloc函数如何实现的



## 数据结构

### 哈希表

什么是哈希表：

原理如何：

你如何实现一个可变的哈希表：

#### 一张图片的内存占用大小是由什么决定的

- 色深
- 颜色空间；
- 分辨率

#### 实现一个可变数组

#### 如何实现一个 LRU 缓存

### 如何实现栈和队列？

#### 小根堆的插入时间负责度

#### 二分查找的时间复杂度怎么求的？


### 算法相关

#### 2个集合的交集

#### 找出数组中重复的数字

#### 百亿数据中查找相同的数字及出现次数

#### 在一个10G的数据中找到最大的100个数

#### 1000瓶药，10只狗，找出毒药。

#### 排序算法，各时间复杂度如何？

#### 堆排，有时间杂度为O(n)的排序算法吗

#### 快排的时间复杂度，为什么是nlogn，最坏的情况是什么，最坏时间复杂度如何？

#### 字符串转浮点数

#### atoi函数实现

#### 二叉树迭代中序遍历

#### 二叉树层序遍历，按层输出

#### 深度有限和广度优先的使用场景

#### 两个链表查找相同节点，交叉

#### 链表环判断

#### 合并K个链表 用小根堆维护

#### 字符串旋转

#### 找链表倒数第K个节点

#### 把一个链表比某个值 大的放在左边，比它小的放在右边

#### n个整数的无序数组，找到每个元素后面比它大的第一个数，要求时间复杂度为O(N)

#### T9算法如何实现，全拼算法

#### 强连通量算法

#### 最短路径算法


## 编译原理

### 编译过程

#### 以 Clang、LLVM 为例简述一下编译过程

#### static关键字的作用

#### define 与 static的区别

### #import"" 和 #import<> 区别，还有#inlcude

### 如何防止别人反编译你的 App？

	以 Clang 和 LLVM 为例，什么是静态语言和动态语言的区别？
	
## Git

### 常用命令


[iOS-Developer-and-Designer-Interview-Questions](https://github.com/9magnets/iOS-Developer-and-Designer-Interview-Questions)

[developers](https://www.toptal.com/swift/interview-questions)
