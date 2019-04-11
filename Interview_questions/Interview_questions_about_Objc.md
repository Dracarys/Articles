# 面试题系列之 Objective-C

## 1. Objective-C 对象
Objective-C 是在 C 基础之上实现的面向对象功能，因此它是一个 C 的超集，通过下面的一些定义可以发现：objc 中所有关于类的实现都是基于结构体的。

下面是 `obj-private.h` 的定义：

```cpp
struct objc_class;
struct objc_object;

typedef struct objc_class *Class;
typedef struct objc_object *id;

namespace {
    struct SideTable;
};

#include "isa.h"

union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    struct {
        ISA_BITFIELD;  // defined in isa.h
    };
#endif
};


struct objc_object {
private:
    isa_t isa;

public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();

    // initIsa() should be used to init the isa of new objects only.
    // If this object already has an isa, use changeIsa() for correctness.
    // initInstanceIsa(): objects with no custom RR/AWZ
    // initClassIsa(): class objects
    // initProtocolIsa(): protocol objects
    // initIsa(): other objects
    void initIsa(Class cls /*nonpointer=false*/);
    void initClassIsa(Class cls /*nonpointer=maybe*/);
    void initProtocolIsa(Class cls /*nonpointer=maybe*/);
    void initInstanceIsa(Class cls, bool hasCxxDtor);

    // changeIsa() should be used to change the isa of existing objects.
    // If this is a new object, use initIsa() for performance.
    Class changeIsa(Class newCls);

    bool hasNonpointerIsa();
    bool isTaggedPointer();
    bool isBasicTaggedPointer();
    bool isExtTaggedPointer();
    bool isClass();

    // object may have associated objects?
    bool hasAssociatedObjects();
    void setHasAssociatedObjects();

    // object may be weakly referenced?
    bool isWeaklyReferenced();
    void setWeaklyReferenced_nolock();

    // object may have -.cxx_destruct implementation?
    bool hasCxxDtor();

    // Optimized calls to retain/release methods
    id retain();
    void release();
    id autorelease();

    // Implementations of retain/release methods
    id rootRetain();
    bool rootRelease();
    id rootAutorelease();
    bool rootTryRetain();
    bool rootReleaseShouldDealloc();
    uintptr_t rootRetainCount();

    // Implementation of dealloc methods
    bool rootIsDeallocating();
    void clearDeallocating();
    void rootDealloc();
};
```

### 1.1 关于 Class 的实现：

在 Objective-C 中 class 是一个指向 objc_class 结构体的指针，而 objc_class 是继承自 objc_object 结构，所以说 Objective-C 的类也是一个对象。

下面不是 `objc_class` 的实现：

```cpp
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
};
```

### 1.2 isa 指针指向什么？有什么作用？
对象实例的 `isa` 指针指向定义它的类，类的 `isa` 指针指向元类（meta class），这是因为在 Objective-C 中，类也是一个对象，这个类对象正是由元类定义的，每个类都有一个独一无二的元类。元类的 `isa` 指针指向根元类，根元类的 `isa` 指针指向自己。

所有元类都用基类作为自己的类，对于顶层基类的元类也是如此，只是它指向自己而已

元类总是会确保类对象和基类的所有实例和类方法。对于从 `NSObject` 继承下来的类，这意味着所有的 `NSObject` 实例和 `protocol` 方法在所有的类（和meta-class）中都可以使用

上面所描述的关系正式通过 `isa` 指针实现的，可以说正是通过 `isa` 实现了 Objective-C 面向对象的特性。

### 1.3 `isKindOfClass` VS `isMemberOfClass`

```objc
// 向上遍历查找，只要该对象的继承链中有目标类就返回 YES
+ (Bool)isKindOfClass:(Class)cls {
	for (Class tcls = object_getClass((id)self); tcls; tcls->superclass){
		if (tcls == cls) return YES;
	}
	return NO;
}

// 取目标的类，然后直接与目标类比较
+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}
```

## 2. Objective-C 的关键字

### 2.1 @property 属性

属性本质就是声明一个存取器（getter、setter），编译器会根据声明在编译过程中自动添加实现。声明时涉及很多关键字，这些关键字会影响到存取器最终的行为。如果手动实现了存取器，那么这些关键字则仅为指导意义。

默认的关键字有：`strong` `retain` `assign` `atomic`，此外还有一些编译器指示性的关键字，例如：`nonnull` `nullable` 等。

### 2.2 nonatomic、atomic 区别？atomic 为什么不是绝对线程安全的？
`atomic` 和 `nonatomic` 的区别向编译器表明，生成的 `getter` 和 `setter` 方法是否为原子操作，默认是 `atomic`。

什么是线程安全？

### 2.3 nonatomic、atomic 实现？

atomic 实际上相当于一个引用计数器，这个大家很熟悉，如果被标记了atomic，那么被标记了的内存本身就有了一个引用计数器，第一个占用这块内存的线程，会给这个计数器+1，在这个线程操作这块内存期间，其他线程在访问这个内存的时候，如果发现“引用计数器”不为0，则阻塞，实际上阻塞并不等于休眠，他是基于 cpu 轮询片，休眠除非被叫醒，否则无法继续执行，阻塞则不同，每个cpu 轮询片到这个线程的时候都会尝试继续往下执行，可见 阻塞相对于休眠来讲，阻塞是主动的，休眠是被动的，如果引用计数器为0，轮询片到来，则先给这块内存的引用计数器+1，然后再去操作

### 2.3 @synthesize 和 @dynamic 分别有什么作用？有了自动合成属性实例变量之后， @synthersize还有哪些使用场景？

- synthesize 告知编译器，需要自动合成属性实例变量，已经可以自动合成了，
- dynamic 告知编译器不要自动合成，相应实现会在运行时我自己来实现。用法与synthesize相同，都是写在 @implement 的下方，列明要动态生成的属性名即可。

有了自动合成需要的地方：

- 自定义了存取器的 readwrite 属性
- 同理，自定义了 getter 方法的 readonly 属性
- 声明为 @dynamic 的属性，因为 @dynamic 就是告诉编译器不要管了，自己处理。
- @protocol 中声明的属性
- category 中声明的属性
- 重写过的继承自父类的属性

[参考](https://stackoverflow.com/questions/19784454/when-should-i-use-synthesize-explicitly)

### 2.4 ivar、getter、setter 是如何生成并添加到这个类中的？

是由编译器在编译时根据声明的属性自动合成的，同时会向类中插入一个以“_"开头，与属性名同名的实例变量。

### 2.5 self、super 关键字作用与区别

- self 是类的隐藏参数，指向调用方法的实例，与 this 类似，区别是工厂方法也可以使用。
- supper 时一个 Magic 关键字，它本质上是一个编译器标识符，告诉编译器到该类的父类中查找该方法。

### 2.6 instancetype 和 id 区别？

`id` 数据类型可以存储任何类型的对象，编译器也就无法验证类型是否匹配；
`instancetype` 是 clang3.5 开始提供的一个关键字，表示一个未知的 Objective-C 对象，类似于id。
按照 Cocoa 的惯例，Objective-C里所有使用init，alloc等名称的方法都会返回一个接受类类型的实例。这些方法被称为“有一个关联的返回类型”的方法，也就是说发给这些方法中的任意一个的消息都会返回一个以相同的静态类型代替接收类类型的一个实例。

一个关联返回类型也可以通过一些方法推断出来。要确定一个方法是否有一个可以被推断出的关联的返回类型，首先要参考驼峰命名法命名的 `selector`中的第一个单词（如`initWithObjects` 中的 init ），其次要看其返回类型与自己的类的类型是否兼容，并且：

- 第一个单词是 `alloc` 或 `new`，并且方法是一个类方法(+开头)
- 第一个单词是 `autorelease`，`init`，`retain` 或者 `self`，且方法是一个实例方法(-开头)

如果一个拥有关联返回类型的方法被子类方法复写了，那么子类方法必须返回一个与子类类型兼容的类型。比如：

```objectivec
@interface NSString : NSObject
- (NSUnrelated *)init; // incorrect usage: NSUnrelated is not NSString or a superclass of NSString
@end
```

区别就是，前者会得到编译器的支持，协助检查类型，减少错误。

[参考](http://www.cnblogs.com/rossoneri/p/5100530.html)

### 2.7 designated initializer，怎么用，注意内容？
即指定构造器，该宏是在 Swift 出现后新增的，用法与 SWift 的指定构造器用法相同：

1. 子类如果有指定构造器，那么该指定构造器必须调用直接父类的指定构造器
2. 如果子类有指定构造器，那么便利构造器必须调用自己的其它构造器（包括制定构造器及其它便利构造器），不能调用supper的普通构造器
3. 子类如何好实现了指定构造器，那么必须实现所有父类的指定构造器。

## 3. 指针

### 3.1 指针常量、常量指针区别？

指针常量：`int * const ptr` 指针本身是一个敞亮。在声明的时候初始化，里面的值（存放的地址）不能更改；
常量指针：`const int * ptr`指针本身是一个常量，通过常量地址初始化，它可以再指向另一个常量地址。

### 3.2 运行完 Test 函数后会有什么结果

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
*参考答案：程序崩溃，`getMemory()` 指针 p 与 str 指针并不是同一个指针，修改了指针 p，并不影响指针 str，所以 Test 函数中的 str 一直都是 NULL，copy将发生错误，导致崩溃，如果要修正，可以将 p 修改为一个指向指针的指针，这样就可以修改 str 的指向，让它指向已分配的内存 *

另一个

``` C
char * getMemory(char *p) {
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

## 4. 关于Block

Block 是通过 structrue 实现的，结构如下：

```c
struct Descriptor{
	unsigned long int reserved;
	unsigned long int size;
	void (*) (void *, void *) copy;
	void (*) (void *, void *) dispose;
};
	
struct {
	void *isa; // 例如 _NSConcreteStackBlock
	int flags;
	int  reserved;
	void (*) (void *, ...) invoke;// 函数指针
	struct Descriptor *descriptor;
	// 捕获到的变量
}
```
block 与函数指针非常类似，可以说是“带有自动变量值的匿名函数”，但 block 可以使实现逻辑看上去更紧凑清晰，节省代码量。

- \_NSConcreteGlobalBlock_
- \_NSConcreteStackBlock_
- \_NSConcreteMallocBlock_


如果 block 是定义在栈上的，那么 block 只在定义它的那个范围内有效。例如：

```c
void (^block)();
if (/* some condition */) {
	block = ^{
		NSLog(@"Block a");
	};
} else {
	block = ^{
		NSLog(@"Block b");
	};
}
block();
```
上面的两个 block a、b 都分配在栈上。编译器会给每个 block 分配好栈空间，然而等离开了相应的范围后，编译器有可能吧分配给 block 的内存覆盖掉。所以，这两个 block 分别只在对应的语句范围内有效。此代码时而正确，时而错误，完全取决于编译器是否覆盖他们。

可以通过 copy 将其复制到堆上，一旦复制到堆上，那么之后在 copy 将只增加其引用计数，注意 ARC 环境会自动释放。分配在栈上的无此顾虑，编译器已经处理好了。

```objc
void (^block)();
if (/* some condition */) {
	block = [^{
		NSLog(@"Block a");
	} copy];
} else {
	block = [^{
		NSLog(@"Block b");
	} copy];
}
block();
```
此外还有一种全局 block，此中 block 不会捕捉任何状态（例如外围的变量等），运行时也无须有状态来参与。block 所使用的内存在编译器就已经确定。全局 block 的拷贝是空操作，因为它不会在运行时被系统回收。相当于单例。

block 总能修改实例变量，而无需像修改普通局部变量那样，必须通过添加 `__block` 修饰来实现。但是需要注意如果通过读取或写入捕获了实例变量，那么也会一并将 self 变量捕获，这是因为实例变量是与 self 所指代的实例关联在一起的，所要特别注意“循环持有”的问题。

### 4.1 仍然有疑问，就是 __block 到底是如何影响被捕获的变量的？

### 4.2 在 block 里对数组执行添加操作，这个数组需要声明成 __block 吗？同理如果修改的是一个 NSInteger，那么是否需要？

- 在 block 中对一个可变数组进行元素的修改不需要用 `__block`，因为不需要修改指针指向的地址；
- 修改 NSInteger 则需要，因为它是值类型，（需要结合上一题回答）

## 5. RunLoop
RunLoop 是线程的基础。一个 RunLoop 就是一个实现处理循环，以变对工作进行调度并且协调接收到的事件。其目的是，在有工作要做时让线程忙碌，空闲时则进入休眠。

### 5.1 剖析 RunLoop
RunLoop 对象，在 iOS 中是由 CFRunLoop 实现的。它会监听任务的输入源，一旦就绪就分配控制权以便进行处理。这里的输入源可以是输入设备、网络、周期或延时，以及异步回调。RunLoop 会接受两种类型的输入源：一种是来自另一个线程或不同应用的异步消息；另一种是来自预定时间或周期性的同步事件。

![Structure of a run loop and tis sources](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg)

![Run Loop 过程 by 戴铭](https://static001.geekbang.org/resource/image/5f/7d/5f51c5e05085badb689f01b1e63e1c7d.png)
这两种类型的输入源可以分为三种对象：

- sources(CFRunLoopSource)
- timers(CFRunLoopTimer)
- observers(CFRunLoopObserver)

要接受当这些对象就绪需要处理的回调，必须先通过 `CFRunLoopAddSource`，`CFRunLoopAddTimer` 或 `CFRunLoopAddObserver` 等方法，将这些对象添加到 RunLoop 中。事后还可以从 RunLoop 中移除（或使其失效），以不在接受它的回调。

### 5.2 Run Loop 模式

每个添加到 RunLoop 中的 source，timer，以及 observer 必须与一种或多种 RunLoop 模式关联。模式用来甄别 RunLoop 正在处理哪种事件。每次 RunLoop 执行，它都运行在一个指定的模式中，此时，它仅处理与次模式相关联的 source，timer，以及 observer。如果把 sources 添加到默认模式（`KCFRunLoopDefaultMode` 常量），那么通常仅当应用（或线程）空闲时才会处理这些事件。系统还定义了其它一些模式，以执行特定的 source，timer，以及 observer。当然由于模式类型只是一个简单的字符串，我们还可以自定自己的模式。

Core Foundation 中定义了一种特殊的“虚”模式，称为 `CommonModes`，允许你管理爱你一个以上的模式，用 `kCFRunLoopCommonModes` 做为对象模式即可。每个 RunLoop 都有自己独立的 common modes，其中默认的事默认模式（kCFRunLoopDefaultMode），可以通过 `CFRunLoopAddCommonMode` 方法向其中添加其它模式。

每个线程都有唯一的一个 RunLoop，你不能创建或销毁它。Core Foundation 会自动创建它。可以通过 `CFRunLoopGetCurrent` 获取它，通过 `CFRunLoopRun` 来启动当前线程的 RunLoop，以运行在默认模式，直至主动通过 `CFRunLoopStop` 停止。此外，还可以通过 `CFRunLoopRunInMode` 方法让当前线程的 RunLoop 运行在特定模式，但是该模式必须至少有一个需要监听的 sources 或 timer。


|Mode|Name|Description|
|:---|:---|:----------|
|Default|NSDefaultRunLoopMode (Cocoa) kCFRunLoopDefaultMode (Core Foundation)|默认模式用于大多数操作。大多数情况下，你使用该模式来启动你的运行循环并配置你的输入源|
|Connection|NSConnectionReplyMode (Cocoa)|Cocoa 结合 `NSConnection` 对象使用该模式来监控应答。你自己应该很少需要使用这个模式|
|Modal|NSModalPanelRunLoopMode (Cocoa)|Cocoa使用该模式来识别用于模态面板的事件|
|Event tracking|NSEventTrackingRunLoopMode (Cocoa)|Cocoa使用该模式在鼠标拖动期间来限制传入的事件和其他类型用户界面跟踪循环|
|Common modes|NSRunLoopCommonModes (Cocoa) kCFRunLoopCommonModes (Core Foundation)|只是一个常用模式的可配置组。将输入源与这种模式结合也将它与组中其他模式结合。对于 Cocoa 引用，这组默认包括默认、模态和时间跟踪模式。核心基础包括的只是默认模式。你可以使用 `CFRunLoopAddCommonMode` 函数添加自定义模式|

RunLoop 可以递归嵌套运行。 Cocoa 的应用建立在 CFRunLoop 的基础之上，以实现高级实现循环。

参考：

- [Run Loop](http://www.voidcn.com/article/p-eieycuxz-de.html)
- [Run Loop 原文](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW20)

### 5.1 自己实现一个 Runloop

```c
do {

} while()

```

## 6. KVC
KVC 是一种间接访问对象属性的机制，使用字符串识别属性，而不是通过调用一个 setter/getter 方法或直接访问实例变量。在本质上，KVC 定义类你的应用实现的访问器方法的模式和方法签名。

在应用中实现 KVC 兼容的访问是一个重要的设计原则。访问器帮助执行适当的数据封装并促进与其他技术的集成，例如：Key-value observing，Core Data，Cocoa bindings 和 AppleScript。KVC 方法在许多情况下，也被用来简化应用代码。

KVC 的基本方法声明在 NSKeyValueCoding 中，是 Objective-C 的非正式协议，由 NSObject 提供默认实现。

### 6.1 KVC 的实现原理

#### 6.1.1 `setValue:forKey:`的原理：

![setValue:forKey:](https://user-gold-cdn.xitu.io/2018/12/26/167ea605f5f2c483?imageView2/0/w/1280/h/960/ignore-error/1)

#### 6.1.2 `valueForKey:`的原理：

![valueForKey:](https://user-gold-cdn.xitu.io/2018/12/26/167ea61203a823f5?imageView2/0/w/1280/h/960/ignore-error/1)

### 6.2 KVC 的用法
#### 6.2.1 访问对象的 properties
对象的properties通常分为三类：

- Attributes 
- To-one relationships
- To-many realtionships

示例：

```objc
@interface BankAccount : NSObject
 
@property (nonatomic) NSNumber* currentBalance;              // An attribute
@property (nonatomic) Person* owner;                         // A to-one relation
@property (nonatomic) NSArray< Transaction* >* transactions; // A to-many relation
 
@end
```

通过 Key 获取属性值：

- `valueForKey:`。如果访问的属性不存在，那么该对象会收到一条 `valueForUndefinekey:` 的消息。该方法的默认实现是抛出一个 `NSUndefinedKeyException` 异常，子类可以重写该方法；
- `valueForKeyPath:`。如果路径上的任何对象出现找不到的情况，都会向上面一样，抛出异常；
- `dictionaryWithValueForKeys:`。返回包指定 keys 的字典。

通过 Key 设置属性值：

- `setValue:forKey:`。该方法的默认实现，会对 NSNumber 和 NSValue 对象进行默认解封；如果属性为定义，那么该对象会收到一条 `setValue:forUndefinedKey:` 的消息，该方法的默认实现与获取相同，也是抛出 `NSUndefinedKeyException` 异常，同样子类可以重写该方法；
- `setValue:forKeyPath:`。逻辑与获取相同；
- `setValuesForKeysWithDictionary:`。与获取逻辑相同，只是如果要设置 nil，请用 NSNull 代替，因为集合类不支持插入 nil。

#### 6.2.2 访问集合类属性

- `mutableArrayValueForKey:` 和 `mutableArrayValueForKeyPath:`。会返回一个代理对象，行为与 NSMutableArray 相同；
- `mutableSetValueForKey:` 和 `mutableSetValueForKeyPath:`。同上
- `mutableOrderedSetValueForKey:` 和 `mutableOrderedSetValueForKeyPath:`。同上。

对返回的代理对象进行插入、修改等操作，也会修改原结合，只是不是在原基础上改，而是现在代理对象上改，之后在想普通的 KVC 赋值一样，赋过去。

通过 KVC 方式取集合的值时，还可以使用集合运算符，格式如下：

![Operator key path format](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/art/keypath.jpg)

集合运算符大致分为三类：

- 聚集操作符：`@avg`，`@count`，`@max`，`@min`，`@sum`；
- 数组操作符：`@distinctUnionOfObjects`，`@unionOfObjects`;
- 嵌套操作符：集合中的元素也是集合。`@distinctUnionOfArrays`，`@unionOfArrays`，`@distinctUnionOfSets`。

示例：

```objc
NSNumber *amountSum = [self.transactions valueForKeyPath:@"@sum.amount"];

NSArray *distinctPayees = [self.transactions valueForKeyPath:@"@distinctUnionOfObjects.payee"];

NSArray* moreTransactions = @[<# transaction data #>];
NSArray* arrayOfArrays = @[self.transactions, moreTransactions];
NSArray *collectedDistinctPayees = [arrayOfArrays valueForKeyPath:@"@distinctUnionOfArrays.payee"];
```

如何适配 KVC？

参考：[《Key-value Coding Programming Guide》](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107-SW1)

### 6.3 KVC 和 KVO 中涉及的 Keypath 一定是属性吗？
- KVC 实例变量也可以
- KVO 如果不是自动合成的属性，需要自己手动触发

## 7. KVO
Key-value Observing，即键值监听，可以用于监听某个对象属性的变化。

### 7.1 KVO 的基本原理
利用 Runtime 动态修改 `isa` 指针的指向实现的，当我们通过下面的方法：

    [Object addObserver: forKeyPath:options:context:]
向一个对象实例添加监听时，Runtime 会动态生成一个 Object 类的子类 `NSKVONotifying_Object`，这个子类会重写 `setter`、`class`、`dealloc` 以及 isKVOA 等相关方法。并修改被监听实例的 `isa` 指针，让其指向这个新生成的子类 `NSKVONotifying_Object`。因为这个新增的子类实现了 KVO 的相关方法，所以当我们再对该实例发送消息时，会通过 `isa` 指针先到这个新的子类中查找相应方法，从而得到通知。

同理，当我们调用下面的方法：

    [Object removeObserver:forKeyPath:]
移除一个对象实例的监听时，Runtime 会将该实例的 `isa` 指针恢复，同时移除生成的子类。

### 7.2 KVO 的底层实现

### 7.3 KVO 在多线程中的行为如何？
- KVO 是同步的，一旦对象的属性发生变化，只有用同步的方式，才能保证所有观察者的方法能够执行完成。KVO 监听方法中，不要有太耗时的操作。

- KVO 的方法调用，是在对应的线程中执行的。在子线程修改观察属性时，观察者回调方法将在子线程中执行。

- 在多个线程同时修改一个观察属性的时候，KVO 监听方法中会存在资源抢夺的问题，需要使用互斥锁。如果涉及到多线程，KVO 要特别小心，通常 KVO 只是做一些简单的观察和处理，千万不要搞复杂了，KVO的监听代码，一定要简单。

[参考](http://www.cnblogs.com/QianChia/p/5771074.html)

### 7.4 如何手动触发 KVO？
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
## 10. 其它问题

### 对于 Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？
最大的优点是它的运行时特性，不足是没有命名空间，对于命名冲突，可以使用长命名法或特殊前缀解决，如果是引入的第三方库之间的命名冲突，可以使用OtherLinkFlag 参数来进行设置。

### Objective-C 中类方法和实例方法有什么本质的区别和联系？
类方法只能由类来调用，不能访问成员变量，实例方法只能由实例来调用，可以访问成员变量。

它们的调用形式是相同的，都是通过 isa 指针去查找实现，然后在发送消息。

但是实例的方法位于类对象中，类方法位于元类对象中。

### Delegate、Notification、KVO 的区别，优缺点

- Delegate 是单向的委托，你只能委托给一个代理对象；
- Notification 可以一对多的发送消息，但是需要你主动发送，接收方需要注册；
- KVO 也可以实现一对多，且不需要主动触发，

### NSNotification 和 KVO 的区别和用法是什么？什么时候应该使用通知，什么时候应该使用KVO，它们的实现上有什么区别吗？如果用 protocol 和delegate（或者 delegate 的 Array）来实现类似的功能可能吗？如果可能，会有什么潜在的问题？如果不能，为什么？

1. 观察者和被观察者都必须是 NSObject 的子类，因为 OC 中 KVO 的实现基于 KVC 和 runtime 机制，只有是 NSObject 的子类才能利用这些特性；
2. 观察的属性需要使用 dynamic 关键字修饰，表示该属性的存取都由 runtime 在运行时来决定，由于 Swift 基于效率的考量默认禁止了动态派发机制，因此要加上该修饰符来开启动态派发。

### 设计一个方案来检测 KVO 的同步异步问题，willChange 和 didChange 的不同点




