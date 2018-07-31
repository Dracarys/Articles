# \[译]Objective-C 中的元类（meta-class）

*本文翻译自 [What is a meta-class in Objective-C](https://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)*， *由 Matt Gallagher 发表于 [cocoawithlove](https://www.cocoawithlove.com)*。

*受限于译者英语水平及翻译经验，译文难免有词不达意，甚至错误的地方，还望不吝赐教，予以指正*

－－－－－－－－－－－－－－－－－－－

本文我们将剖析一个在 Objective-C 中比较陌生的概念——元类（meta-class）。Objective-C 中的每个类都有和自己相关联的元类，但你可能从来没有直接使用过它，它始终罩着一层神秘的面纱。为了一探究竟我们首先看看怎么在运时（runtime）创建一个类。然后通过创建的“class pair”（这里的pair做成双理解），我会解释什么是元类，然后探讨它对于 Objective-C 中对象和类的意义。

### 在运行时创建一个类

下面的代码在运行时创建了一个 NSError 的子类，并且给它添加了一个方法：

``` Objective-C
Class newClass = objc_allocateClassPair([NSError class], "RuntimeErrorSubclass", 0);
class_addMethod(newClass, @selector(report), (IMP)ReportFunction, "v@:");
objc_registerClassPair(newClass);
```
上面添加的方法，以 ReportFunction 函数作为它的实现，具体定义如下：

``` Objective-C
void ReportFunction(id self, SEL _cmd)
{
    NSLog(@"This object is %p.", self);
    NSLog(@"Class is %@, and super is %@.", [self class], [self superclass]);
    
    Class currentClass = [self class];
    for (int i = 1; i < 5; i++)
    {
        NSLog(@"Following the isa pointer %d times gives %p", i, currentClass);
        currentClass = object_getClass(currentClass);
    }

    NSLog(@"NSObject's class is %p", [NSObject class]);
    NSLog(@"NSObject's meta class is %p", object_getClass([NSObject class]));
}
```
表面上看来，这相当简单。在运行时创建一个类只需要简单三步:

1. 为”class pair”分配内存 (使用`objc_allocateClassPair`);
2. 添加所学的方法和变量到类中 (我已经通过 class_addMethod 添加了一个方法);
3. 注册类以便它能使用 (使用`objc_registerClassPair`)

然而，随之而来的问题是：“class pair”是个什么鬼啊？`objc_allocateClassPair`函数怎么就返回了一个值：the class。不是pair（成对的）吗，说好的另一半呢？

我相信你已经猜到了，另一半就是元类（也就是本文的主题）。为了解释它是什么和我们为什么需要它，还需要交代下 Objective-C 的对象和类的相关背景。

### 对一个数据结构而言到底怎么才能称之为对象呢？

每个对象都有类。这是面向对象的基本概念，在Objective-C中，它对数据结构也一样。任何含有一个指向其准确类地址的指针的数据结构，都可以被视作为对象。

在 Objective-C 中，对象的类是`isa`指针决定的.`isa`指针指向对象所属的类。

实际上，Objective-C 中对象最基本的定义是这样的：

``` Ojbective-C
typedef struct objc_object {
    Class isa;
} *id;
```

也就是说：任何一个以指向`Class`结构的指针为开始的结构体都可以被视作`objc_object`。

Objective-C 中对象最重要的特点就是，你可以发送消息给它们：

``` Objective-C
[@"stringValue" writeToFile:@"/file.txt" atomically:YES encoding:NSUTF8StringEncoding error:NULL];
```

之所以能发送成功是因为 Objective-C 对象（例如上面的`NSCFString`）在发送消息时，是因为运行时可以沿着对象的`isa`指针找到其所属的类（这里是`NSCFString`类）。该类包含一个可以适用所有该类实例对象的方法列表，和一个指向`父类（superclass）`的指针。运行时通过检查这个方法类标以及通过指针检查其父类的方法列表，从而找到一个匹配这条消息的方法（在上面的代码里，是`NSString`类的`writeToFile:atomically:encoding:error`方法）。之后运行时会调用相应的实现（`IMP`）。

关键点就在于`Class`定义了你可以给一个对象发送哪些方法。

### 什么是元类（meta-class）？

现在，可能你已经知道了，Objective-C 的一个类也是一个对象。这意味着你可以发送消息给一个类。

``` Objective-C
NSStringEncoding defaultStringEncoding = [NSString defaultStringEncoding];
```

在这个示例里，`defaultStringEncoding`被发送给了`NSString`类。

之所以能成功是因为 Objective-C 中每个类本身也是一个对象。如上面所看到的，这意味着类结构也必须以一个isa指针开始，从而可以和`objc_object`在二进制层面兼容，之后这个结构的下一字段必须是一个指向父类的指针（对于基类则为nil）。

正如我上周展示的，定义一个`Class`有很多种方式，取决于你的运行时库版本，但有一点，它们都以`isa`字段开始，并且仅跟着一个`superclass`字段。

``` Objective-C
typedef struct objc_class *Class;
struct objc_class {
    Class isa;
    Class super_class;
    /* followed by runtime specific details... */
};
```

为了调用`Class`里的方法，该`Class`的`isa`指针也必须指向一个包含了该`Class`方法列表的`Class`。

这就引出了元类的定义：元类是`Class`的类。

简单来说就是：
- 当你给对象发送消息时，消息是在寻找这个对象的类的方法列表;
- 当你给类发消息时，消息是在寻找这个类的元类的方法列表。

元类是必不可少的，因为它存储了类的类方法。每个类都必须有独一无二的元类，因为每个类都有独一无二的类方法。

### 元类的类是什么？

元类，就像之前的类一样，它也是一个对象。你也可以调用它的方法。自然的，这就意味着他必须也有一个类。

所有的元类都使用根元类（继承体系中处于顶端的类的元类）作为他们的类。这就意味着所有`NSObject`的子类（大多数类）的元类都会以`NSObject`的元类作为他们的类

根据这个规则，所有的元类使用根元类作为他们的类，根元类的元类则就是它自己。也就是说基类的元类的isa指针指向他自己。

###类和元类的继承

类用`super_class`指针指向了父类，同样的，元类用`super_class`指向类的`super_class`的元类。

说的更拗口一点就是，根元类把它自己的基类设置成了`super_class`。

在这样的继承体系下，所有实例、类以及元类都继承自一个基类。

这意味着对于继承于`NSObject`的所有实例、类和元类，他们可以使用`NSObject`的所有实例方法，类和元类可以使用NSObject的所有类方法

这些文字看起来莫名其妙难以理解。[Greg Parker](http://www.sealiesoftware.com/blog/)给出了一份精彩的[关系图](http://www.sealiesoftware.com/blog/class%20diagram.pdf)：

### 验证实验

为了验证，让我们看看我在文章开始写的`ReportFunction`函数的输出。这个函数的目的是跟随`isa`指针并打印出它的路途。

为了运行`ReportFunction`，我们需要创建一个动态实例来创建类调用`report`方法。

``` Objective-C
id instanceOfNewClass = [[newClass alloc] initWithDomain:@"someDomain" code:0 userInfo:nil];
[instanceOfNewClass performSelector:@selector(report)];
[instanceOfNewClass release];
```

这里没有声明`report`方法，但我使用`performSelector:`调用它，所以编译器不会给出警告。
然后`ReportFunction`函数会沿着isa进行检索，来告诉我们class，meta-class以及meta-class的class是什么样的情况：

```
得到对象的类：ReportFunction 函数使用object_getClass跟踪isa指针，因为isa指针是类的保护成员（你不能直接接收其他对象的isa指针）。ReportFunction不使用类方法，因为在类对象里调用类方法不能返回元类，它会再次返回这个类（因此[NSString class]会返回NSString 类而不是NSString元类）
```
这是程序运行时的输出（省略了`NSlog`前缀）：

``` Objective-C
This object is 0x10010c810.
Class is RuntimeErrorSubclass, and super is NSError.
Following the isa pointer 1 times gives 0x10010c600
Following the isa pointer 2 times gives 0x10010c630
Following the isa pointer 3 times gives 0x7fff71038480
Following the isa pointer 4 times gives 0x7fff71038480
NSObject's class is 0x7fff710384a8
NSObject's meta class is 0x7fff71038480
```

观察`isa`到达过的地址的值：

- 对象的地址是 `0x10010c810`
- 类的地址是`0x10010c600`
- 元类的地址是`0x10010c630`
- 根元类（`NSObject`的元类）的地址是`0x7fff71038480`
- `NSObject`元类的类是它本身

这些地址的值并不重要，重要的是它们说明了文中讨论的从类到元类到`NSObject`的元类的整个流程。

###最后

元类是`Class`的类。每个`Class`都有自己独一无二的元类（每个类都有自己独一无二的方法列表）。这意味着所有的类对象都不同。

元类总是会确保类对象和基类的所有实例和类方法。对于从`NSObject`继承下来的类，这意味着所有的`NSObject`实例和`protocol`方法在所有的类（和meta-class）中都可以使用。

所有元类都用基类作为自己的类，对于顶层基类的元类也是如此，只是它指向自己而已（译者注:看文中的图，一目了然）。