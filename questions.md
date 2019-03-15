# iOS 面试题

## 一、关于 Objective-C

### 1、Ojective-C的对象模型

#### 一个 Objective-C 对象的结构？

#### 一个 Objective-C 对象是如何进行内存布局的？有父类的情况呢？

#### 一个 Objective-C 对象的 isa 指针指向什么？有什么作用？

### 2、nonatomic、atomic区别？atomic为什么不是绝对线程安全的？

### self、super 关键字作用与区别

### volatile 关键字

### 3、@property 相关

#### 本质是什么？

#### ivar、getter、setter 是如何生成并添加到这个类中的？

#### @proptery 默认的属性关键字是什么？

#### @synthesize 和 @dynamic 分别有什么作用

#### 在有了自动合成属性实例变量之后， @synthersize还有哪些使用场景？

### 聊聊 Class extension

### isKindOfClass VS isMemberOfClass

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

### designated initializer，怎么用，注意内容？

### instancetype 和 id 区别？

#### 多态

#### Objective-C的反射机制

#### 在block里堆数组执行添加操作，这个数组需要声明成 __block吗？同理如果修改的是一个NSInteger，那么是否需要？

#### 对于Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？

#### buildSetting link flag 解决命名冲突。

#### 实现一个NSString类

#### Objective-C中类方法和实例方法有什么本质的区别和联系？

#### _objc_msgForward 函数是做什么的，直接调用会发生什么？

#### 手动出发KVO

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
参考答案：程序崩溃，应为 `getMemory()` 并不能传递动态内存，Test 函数中的 str 一直都是 NULL，copy将发生错误，导致崩溃

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
参考答案：可能乱码。因为 `getMemory()` 返回的是 “栈内存” 指针，该指针的地址不是 NULL，但其原先的内容已经被清除了，新内容不可知。

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
参考答案：输出 “hello”，单丝会导致内存泄漏

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

#### 什么情况下不得不使用隐式拆包？为什么？

#### 可选类型解包的方式有哪些？安全性如何？

### 什么时候用 Structure，什么时候用 Class

### 什么是泛型？用来解决什么问题的？

### 闭包是引用类型吗？

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

#### 内存泄漏的原因有哪些？如何解决？

#### 深拷贝、浅拷贝

#### BAD_ACCESS在什么情况下出现？如何调试？

读取了一个不是你管控的内存地址，通常是由于野指针导致的

启动僵尸对象，然后结合发生错误时的操作，逐步定位问题根源。


## Runtime

### 消息相关

#### 为什么 Objective—C的方法不叫调用叫发消息：

#### 给一个对象发消息的过程如何，或者说如何通过 selector 找到对应的 IMP 地址：

#### 类对象也是如此吗：

#### 如果消息发送失败有哪些补救措施：

### 类别（category）

#### 什么是类别？实现原理如何？

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

#### 使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放吗？

### class的载入过程

### 如何访问并修改一个类的私有属性

1. 通过 KVC 获取
2. 通过 runtime 访问并修改私有属性

### weak实现机制？为什么对象释放后会自动置为nil？

### autorealse 如何实现的

这个解释不完善，有待进一步研究
autoreleasePool 是一个Objective-C的义仲内存自动回收机制，它可以延时家如autoreleasePool中的变量释放的时机。
autoreleasePool在Runloop时间开始之前（push），释放是在一个 RunLoop 事件即将结束之前（pop）

#### 子线程中需要加autoreleasepool吗？什么时间释放？

#### autorelease 和 @autoreleasepool区别

### 如何hook一个对象的方法，而不影响其它对象。

### 设计一个检测主线程卡顿的方案

### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？


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

#### 互斥锁、自旋锁是如何实现的？ 

#### 分别用 C/C++ 和 Objective-C 实现互斥锁、自旋锁。

#### 死锁档四个条件、优先级翻转

#### 什么时候处理多线程，几种方式，优缺点？

#### 系统有哪些在后台运行的Thread

#### 队列和线程的关系

#### 线程同步的方式

#### 线程池的结构？如何实现的？

#### 开启一条线程的方法？线程可以取消吗？

#### runloop和线程的关系？各个mode是做什么的？如何实现一个runloop

#### 用过NSOperationQueue么？如果用过或者了解的话，为什么要使用 NSOperationQueue，实现了什么？跟 GCD 之间的区别和类似得地方



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

#### 手触碰到屏幕的时候，响应机制是怎样的？第一响应者是谁？追问 UIView和UIResponse的关系是什么？

#### 直接用UILabel和自己用DrawRect画UILabel，那个性能好，为什么？那个占用的内存少？为什么？

#### iOS的应用程序有几种状态？推到后台代码是否可以执行哦？双击home键，代码是否可以执行

#### 一般使用的图标内存为多大？200x300的图片，内存应该占用多少比较合理？

#### 自线程中调用connection方法，为什么不回调？没加入runloop，子线程被销毁了

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

### 如何编写单元测试，例如测试一个网络库，如何提高覆盖率

### images.xcassetsa和直接使用图片有什么不一样？

### 平时是怎么进行测试的，内存方面怎么测试

### lldb常用调试命令？

### 如何给一款App瘦身

### 从点击图标到应用启动的过程？

### DSYM文件是什么，你是如何分析的？

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

#### 内核态和用户态的区别，为什么要这样分

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

### AFNetwoking 为什么添加一条常驻线程？

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
