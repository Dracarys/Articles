# 面试题系列之 Objective-C

## 1. Objective-C 对象
通过对 objc runtime 源码对剖析，可以发现 objc 中所有关于类的实现都是基于结构体的。

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

在 Objective-C 中 class 是一个 `objc_class` 类型的结构体指针，而 objc_class 是继承自 `objc_object` 结构，所以说 Objective-C 的类也是一个对象。

下面不是 `objc_class` 的部分实现：

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

原子操作有许多种，有纯粹用于做同步的（即作为锁的用途），有用于做基本运算操作的，也有可将这两者相结合的。

对于早期的多核处理器，有不少提供了数据交换（swap）、标志测试与设置（flag test and set）等基于“锁”的原子操作。比如8086上的XCHG指令，ARMv7架构之前的SWP指令，这些都属于SWAP原子操作；而Blackfin 561 Duo-Core DSP上则提供了flag test and set原子操作……这些原子操作的实现比较简单，不过都是基于“锁”，也就是说如果你要用原子操作来同步某一共享存储对象，那么必须先针对它定义一个原子变量作为锁去同步。这些原子操作所引发的最大问题就是如果某一线程在利用这些锁做“自旋等待”，而另一个线程在释放该原子锁之前就被销毁了，那么等待该锁的线程就倒霉成为僵尸线程了～尽管一般对于应用层来说，我们不会轻易自己去杀线程，但对于操作系统层还有学术界而言，这是一个必须解决的问题。目前一般常见的解决方案就是添加一个重试次数，如果重试了比如1000次，这个原子锁还没有被释放，那么就强行解锁，或者抛出异常等。所以这里要提醒各位的是，用了原子锁之后，后续相关的操作得尽量快，然后马上释放锁，否则的话宁可直接调用系统所提供的同步原语API。下面给出一份使用SWAP原子操作流程的伪代码，当然flag test and set跟这个流程其实也差不多～

```c
// 在执行多任务前，原子锁对象初始值为0
volatile int g_atomic_lock = 0;

// 这里是多个线程可能共同的代码
void AtomicModify(void)
{
    // SWAP的第一个参数指向某一原子对象的地址。
    // SWAP操作一般是将第二个实参的值写入到原子对象中去，
    // 然后返回该原子对象在SWAP操作之前的值。
    while(SWAP(&g_atomic_lock, 1) == 1)
    {
        // 如果SWAP操作返回1，说明之前已经有线程对该对象上了锁，
        // 此时只能等待该原子对象重新变为0。
        CPU_PAUSE();    // 这里暗示CPU可以做对其他线程的调度
    }

    // SWAP操作成功之后，g_atomic_lock的值变为1了，
    // 此时对多个线程所共享的对象进行操作
    DoModificationToSharedObject();

    // 对共享对象操作完之后，释放原子锁
    SWAP(&g_atomic_lock, 0);
}
```

随着科技的进步，现代多核处理器纷纷引入了无锁（Lock-Free）原子操作，比如比较与交换（Compare and Swap，简称CAS），加载时锁定/有条件存储（Load-locked Store-conditional，简称LL-SC）。x86处理器使用前者，Alpha、Power、MIPS、RISC-V、ARMv7、ARMv8等则使用后者，而从ARMv8.1开始也引入了CAS原子操作。LL-SC形式上虽然与CAS有些不同，但逻辑都是互通的。其主要思路就是在当前线程先加载共享对象的值，然后对它做任意修改操作，最后写的时候是原子的——先判定之前所加载的共享对象的值与当前共享对象的值是否完全相同，如果相同，则把新修改的值写回去；否则交换失败，程序可以做循环从而再次从该共享对象中加载值。这种无锁机制可以把之前的原子锁去掉，而直接对目标共享对象进行操作。

> 引自[《C11标准的原子操作详解》](https://www.jianshu.com/p/ae1f912a7607) 作者：zenny_chen

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
2. 如果子类有指定构造器，那么便利构造器必须调用自己的其它构造器（包括指定构造器及其它便利构造器），不能调用supper的普通构造器
3. 子类如果实现了指定构造器，那么必须实现所有父类的指定构造器。

## 3. 指针

### 3.1 指针常量、常量指针区别？

指针常量：`int * const ptr` 指针本身是一个常量。在声明的时候初始化，里面的值（存放的地址）不能更改；
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
	// 此处的 p 是声明在栈上的，注意与 malloc 进行区别。
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
	// str 使用完毕之后需要释放
}
```
*参考答案：输出 “hello”，但是会导致内存泄漏*

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
假设有如下代码：

```c
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int age = 10;
        void (^block)(int, int) = ^(int a, int b) {
            NSLog(@"This is block, a= %d, b = %d", a, b);
            NSLog(@"This is block, age = %d", age);
        };
        block(3, 5);
    }
    
    return 0;
}
```

其重写展开后如下：

```c++
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int age;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _age, int flags=0) : age(_age) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself, int a, int b) {
  int age = __cself->age; // bound by copy

            NSLog((NSString *)&__NSConstantStringImpl__var_folders__2_jlhfyq854qb4j25pwnx17m1w0000gn_T_main_61a3ee_mi_0, a, b);
            NSLog((NSString *)&__NSConstantStringImpl__var_folders__2_jlhfyq854qb4j25pwnx17m1w0000gn_T_main_61a3ee_mi_1, age);
        }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        int age = 10;
        void (*block)(int, int) = ((void (*)(int, int))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, age));
        ((void (*)(__block_impl *, int, int))((__block_impl *)block)->FuncPtr)((__block_impl *)block, 3, 5);
    }

    return 0;
}
```
其中各结构体关系如下图：

![Block 结构体关系图](https://user-gold-cdn.xitu.io/2018/5/20/1637de343b05ffaa?imageView2/0/w/1280/h/960/ignore-error/1)

> 引自：掘金[<探寻block的本质>](https://juejin.im/post/5b0181e15188254270643e88)

### 4.1 Block 的变量捕获
先上结论：

![变量捕获对比图](https://user-gold-cdn.xitu.io/2018/5/20/1637de344d83a2f8?imageView2/0/w/1280/h/960/ignore-error/1)

下面看一些详细解析：

#### 4.1.1 局部变量
先来看一段代码：

```c
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        auto int a = 10;
        static int b = 11;
        void(^block)(void) = ^{
            NSLog(@"hello, a = %d, b = %d", a,b);
        };
        a = 1;
        b = 2;
        block();
    }
    return 0;
}
// log : block本质[57465:18555229] hello, a = 10, b = 2
// block 中 a 的值没有被改变而 b 的值随外部变化而变化。
```
重写展开的代码如下：

```c++
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int a;
  int *b; // 注意这里对 b 的捕获，声明类一个同名指针
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, int *_b, int flags=0) : a(_a), b(_b) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int a = __cself->a; // bound by copy
  int *b = __cself->b; // bound by copy

            NSLog((NSString *)&__NSConstantStringImpl__var_folders__2_jlhfyq854qb4j25pwnx17m1w0000gn_T_main_b50e27_mi_0, a,(*b));
        }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        auto int a = 10;
        static int b = 11;
        // 传入的是 b 的地址
        void(*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, a, &b));
        a = 1;
        b = 2;
        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    }
    return 0;
}
```
因为 a 是局部变量，碍于其生命周期，所以 block 必须对其进行值拷贝，因为它随时可能因为栈地弹出而结束。而 b 是静态变量，即使栈弹出其生命周期依然不会结束，所以这里不需要拷贝，只要生命一个指针，指向其地址即可。也正因为是指针，所以当 b 更新后，block 执行时读取 b 的值也是新值。

#### 4.1.2 全局变量
再来看一段代码：

```c
int a = 10;
static int b = 11;
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        void(^block)(void) = ^{
            NSLog(@"hello, a = %d, b = %d", a,b);
        };
        a = 1;
        b = 2;
        block();
    }
    return 0;
}
// log hello, a = 1, b = 2
```
重写展开的代码如下：

```c++
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  // 注意这里根本没有对全局变量 a，b 进行捕获
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, int *_b, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
 			  // 直接调用 a，b。因为他们的访问范围是全局的，根本不需要捕获
            NSLog((NSString *)&__NSConstantStringImpl__var_folders__2_jlhfyq854qb4j25pwnx17m1w0000gn_T_main_b50e27_mi_0, a, b);
        }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        auto int a = 10;
        static int b = 11;
        void(*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
        a = 1;
        b = 2;
        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    }
    return 0;
}

```

### 4.2 Block 对类对象的捕获


### 4.3 Block 的类型
下面通过代码验证一下：

```objc
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // 1. 内部没有调用外部变量的block
        void (^block1)(void) = ^{
            NSLog(@"Hello");
        };
        // 2. 内部调用外部变量的block
        int a = 10;
        void (^block2)(void) = ^{
            NSLog(@"Hello - %d",a);
        };
       // 3. 直接调用的block的class
        NSLog(@"%@ %@ %@", [block1 class], [block2 class], [^{
            NSLog(@"%d",a);
        } class]);
    }
    return 0;
}
```
可以看到输出结果：

```shell
Block_playground[1154:182941] __NSGlobalBlock__ __NSMallocBlock__ __NSStackBlock__
Program ended with exit code: 0
```
可以看到，跟之前的重写展开的代码貌似不太一样，是不是运行时捣的鬼，有待进一步验证。

#### 4.3.1 block 在内存中的存储

![Block在内存中的存储](https://user-gold-cdn.xitu.io/2018/5/20/1637de34c0579805?imageView2/0/w/1280/h/960/ignore-error/1)

由图中可见，不同类型的 Block 存储的内存区域也不尽相同，可以分为三类：

- \_\_NSGlobalBlock__ 全局型，直到程序结束才会回收；
- \_\_NSStackBlock__ 栈型，从名字可知其最位于栈上，自然生命周期也随栈地弹出而终结；
- \_\_NSMallocBlock__ 堆型，需要我们自己控制其生命周期。

#### 4.3.2 什么情况下 ARC 自动对 block 进行 copy 操作？

1. block 作为函数返回值时
2. 将 block 赋值给 __strong 指针时
3. block 作为 Cocoa API 中方法名含有 UsingBlock 的方法参数时
4. block 作为 GCD API 的方法参数时

### 4.4 __block 到底是如何影响被捕获的变量的？

#### 4.4.3 在 block 里对数组执行添加操作，这个数组需要声明成 __block 吗？同理如果修改的是一个 NSInteger，那么是否需要？

- 在 block 中对一个可变数组进行元素的修改不需要用 `__block`，因为不需要修改指针指向的地址；
- 修改 NSInteger 则需要，因为它是值类型，（需要结合上一题回答）


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


## 10. 其它问题

### 10.1 对于 Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？
最大的优点是它的运行时特性，不足是没有命名空间，对于命名冲突，可以使用长命名法或特殊前缀解决，如果是引入的第三方库之间的命名冲突，可以使用OtherLinkFlag 参数来进行设置。

### 10.2 Objective-C 中类方法和实例方法有什么本质的区别和联系？
- 类方法只能由类来调用，不能访问成员变量，实例方法只能由实例来调用，可以访问成员变量。
- 它们的调用形式是相同的，都是通过 isa 指针去查找实现，然后在发送消息。
- 实例的方法位于类对象中，类方法位于元类对象中。

### 10.3 什么是多态
OOP 的核心多态性（polymorphism）。多态性这个词源自希腊语，其含义是“多种形式”。我们把具有继承关系的多个类型称为多态类型，因为我们能使用这些类型的“多种形式”而无须在意他们的差异。

当我们使用积累的引用或指针调用基类中定义的一个函数时，我们并不知道该函数真正作用的对象是什么类型，因为它可能是一个基类对象也可能是一个派生类的对象。只能在运行时根据指针绑定的对象的真实类型而定。





