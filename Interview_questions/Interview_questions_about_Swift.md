# 面试题系列之 Swift

### 什么是 Optional 类型，它用来解决什么问题？

Swift 是严格类型安全的语言，Optional 表示某中可选类型，即它如果有值那么它就是 x，否则就是没有值，nil。它能避免很多因为意外nil而出现的错误。

### Optional 类型是如何实现的？

### 什么情况下不得不使用隐式拆包？为什么？
- 对象属性在初始化的时候不能nil,否则不能被初始化。典型的例子是Interface Builder outlet类型的属性，它总是在它的拥有者初始化之后再初始化。在这种特定的情况下，假设它在Interface Builder中被正确的配置——outlet被使用之前，保证它不为nil。
- 解决强引用的循环问题——当两个实例对象相互引用，并且对引用的实例对象的值要求不能为nil时候。在这种情况下，引用的一方可以标记为unowned,另一方使用隐式拆包。

### 可选类型解包的方式有哪些？安全性如何？

### 什么时候用 Structure，什么时候用 Class
值类型和引用类型的区别

### 什么是泛型？用来解决什么问题的？
通过泛型可以定义类型安全的数据结构（类型安全），而无需使用具体的数据类型（可扩展）。这能够显著提高性能并得到更高质量的代码（高性能），因为你可以重用数据处理算法，而无须复制类型特定的代码（可重用）。

### 闭包是引用类型吗？
是引用类型，不会发生复制。

### 搜索关键词 Swift Interview Questions

### what is the type of x? And what is its value?

``` Swift
let d = ["john": 23, "james": 24, "vincent": 34, "louis": 29]
let x = d.sorted{ $0.1 < $1.1 }.map{ $0.0 }
```
复习一些 Swift 的容器，尤其是源码，看看他们的实现方式。

### what's the differences between `unowned` and `weak`?

- unowned: the reference is assumed to always have a value during its lifetime - as a consequence, the property must be of non-optional type
- weak: at some point it's possible for the reference to have no value - as a consequence, the property must be of optional type.

### Swift 应用了面向协议编程？

#### Swift 中 Array 是值类型，他有一个 copy on wirte 的行为，具体怎么实现的？
对于简单值类型 (像是 Int 或 CGPoint) 来说，整个值直接存储在一个变量中，当初始化一个新变量，或是将新值赋给已经存在的变量时，复制都会自动发生。

然而，将一个数组赋给新变量并不会发生底层存储的复制，这只会创建一个新的引用，它指向同一块在堆上分配的缓冲区，所以该操作将在常数时间内完成。直到指向共享存储的变量中有一个值被更改了 (例如：进行 insert 操作)，这时才会发生真正的复制。不过要注意的是，只有在改变时底层存储是共享的情况下，才会发生复制存储的操作。如果数组对它自身存储所持有的引用是唯一的，那么直接修改存储缓冲区也是安全的。

当我们说 Array 实现了写时复制优化时，我们本质上是在对其操作性能进行一系列相关的保证，从而使它们表现得就像上面描述的一样

### 函数式编程，什么事“单子（Monad）”、“函子（functors）”？

一个“函子”是一种表示 Type 的类型，它：

- 封装了另一种类型（类似于封装了某个 `T` 类型的 `Array` 或 `Optinal`）
- 有一个具有 `(T->U) -> Type` 签名的 `map` 方法

一个“单子”是一种类型，它：

- 是一个函子（所以它封装了一个 `T` 类型，拥有一个 `map` 方法）
- 还又一个具有 `(T->Type) -> Type`签名的 `flatMap` 方法

简单来说：一个单子就是一种带有 flatMap 方法的类型，一个函子就是以中带有一个

### Swift和Objective-C混编，各自定义的静态变量（常量）如何排布？为什么？
在模拟器上测试的结果是Objective-C和Swift是各自独立的区域，Objective-C的静态变量（常量）会处于低区，Swift位于高区，它们之间并不连续。

为什么？暂时不清楚，有待进一步探究。
