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

### 2.7 volatile 关键字
volatile 提醒编译器它后面所定义的变量随时都有可能改变，因此编译后的程序每次需要存储或读取这个变量的时候，都会直接从变量地址中读取数据。如果没有volatile关键字，则编译器可能优化读取和存储，可能暂时使用寄存器中的值，如果这个变量由别的程序更新了的话，将出现不一致的现象

volatile的本意是“易变的”，由于访问寄存器的速度要快过RAM，所以编译器一般都会作减少存取外部RAM的优化

[参考](http://www.cnblogs.com/yc_sunniwell/archive/2010/06/24/1764231.html)

### 2.8 designated initializer，怎么用，注意内容？
即指定构造器，该宏是在 Swift 出现后新增的，用法与 SWift 的指定构造器用法相同：

1. 子类如果有指定构造器，那么该指定构造器必须调用直接父类的指定构造器
2. 如果子类有指定构造器，那么便利构造器必须调用自己的其它构造器（包括制定构造器及其它便利构造器），不能调用supper的普通构造器
3. 子类如何好实现了指定构造器，那么必须实现所有父类的指定构造器。

## 3. 聊聊 Class extension

- 可以将暴露给外面的只读属性改为读写属性，方便内部使用（但我个人更推荐在内部尽量直接使用变量）
- 可以添加一些私有的实例变量、私有的属性、私有方法等你不想暴露给外部的内容。

### 4. 关于Block

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

### 4.2 在 block 里堆数组执行添加操作，这个数组需要声明成 __block吗？同理如果修改的是一个 NSInteger，那么是否需要？

- 在 block 中对一个可变数组进行元素的修改不需要用 `__block`，因为不需要修改指针指向的地址；
- 修改 NSInteger 则需要，因为它是值类型，（需要结合上一题回答）

### 对于 Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？

最大的优点是它的运行时特性，不足是没有命名空间，对于命名冲突，可以使用长命名法或特殊前缀解决，如果是引入的第三方库之间的命名冲突，可以使用OtherLinkFlag 参数来进行设置。

### 实现一个 NSString 类

### Objective-C 中类方法和实例方法有什么本质的区别和联系？

类方法只能由类来调用，不能访问成员变量，实例方法只能由实例来调用，可以访问成员变量。类方法为该类所有对象共享

实例方法都要通过 isa 指针传递，而类方法则不需要。

### 指针

#### 指针常量、常量指针区别？

指针常量：`int * const ptr` 指针本身是一个敞亮。在声明的时候初始化，里面的值（存放的地址）不能更改；
常量指针：`const int * ptr`指针本身是一个常量，通过常量地址初始化，它可以再指向另一个常量地址。

### 运行完 Test 函数后会有什么结果

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
