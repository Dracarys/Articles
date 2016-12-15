# Swift 学习小记
*本文主要记录一些自己在Swift的学习过程中遇到的一些小知识点，以便加深记忆*

[TOC]

### 1. 如何给代码分段
熟悉Objective-C的同学都知道可以通过`#pragma mark escription` 宏标记，在代码的方法导航中添加描述语句。此外，还可以在描述语句前添加一条中划线，例如`＃pragma mark － Description`，这样在方法导航中，会显示一条横线将代码段分隔开，令分段显示更加清晰。

那么怎么在Swift中使用呢？可以通过`//MARK: description`,`//????: description `,`//FIXME: description `这样的注释来添加分段说明，同样的，也可以在冒号后添加中划线，例如：`//MARK: - description`,`//????: - description `,`//FIXME: - description `，将代码分段用横线隔开。

### 2.如何在Swift中使用保留关键字
通常一种编程语言都是不允许使用关键字作为变量名的，那么在SWift中如果非要用到关键字怎么办呢？可以通过用\`\`将关键字括起的方式来使用，例如：\`self\`，当然，这是最后的手段，但有一线希望都不应该用关键字来作为变量名。

### 3.几个常见数值字面量
- 十进制 `let decimalInteger = 17` 
- 二进制 `let binaryInterger = 0b10001` 等价于17
- 八进制 `let octalInteger = 0o21` 等价于17
- 十六进制 `let hexadecimalInteger = 0x11` 等价于17
- 科学计数法 `1.25e2` 等价于`125.0`；`1.25e-2`等价于`0.0125`
- 十六进制指数式 `0xFp2`表示$15 * 2^2$，`0xFp-2`表示$15 * 2^-2$
- 十进制格式增强 `let paddedDobule = 000123.456`，`let oneMillion = 1_000_000`，`let justOverOneMillion = 1_000_000.000_000_1`

### 4.lazy属性的线程安全问题
我们知道Swift通过其语言自身的特点，保证了全局常量和存储型属性，即使被多个线程交替存取，也仅初始化一次，但是这里有一个特例，就是lazy属性，当多个线程同时访问一个尚未初始化的lazy属性时，则不能保证仅初始化一次。

### 5.如何便捷的在Swift中获取指针
Swift虽然在极力避免指针，但是为了 Object-C 和 C 兼容，还是保留了指针，但是我们不能像在 C 中一样通过`&`便捷的获取某个常量或变量的指针，例如：

``` Swift
	let a = 8
	
	let b = &a //编译错误

```
查看下错误信息："Type ‘inout Int’ of variable is not materializable"，这里编译器将`&a`识别为 inout Int类型，并不是我们期望的指针类型，所以是不是我们只要把它明确指定为指针类型就可以了呢？我们来试一下：

``` Swift
	func converToUnsafePointer(_ pointer: UnsafePointer<Int>) -> UnsafePointer<Int> {
    	return pointer
	}

	func converToUnsafeMutablePointer(_ pointer: UnsafeMutablePointer<Int>) -> UnsafeMutablePointer<Int> {
    		return pointer
		}

	var a = 8

	var b = converToUnsafePointer(&a)

	var c = converToUnsafeMutablePointer(&a)//顺利通过编译
```
哈哈，成功了，顺利通过编译。

### 6.Swift错误处理原则
*来源:[Magical Error Handling in Swift](https://www.raywenderlich.com/130197/magical-error-handling-swift)，由  [Gemma Barlow](https://www.raywenderlich.com/u/gemmakbarlow) 发表于Raywenderliche*

- 错误类型的命名要清晰无歧义
- 单个错误尽量用可选类型来处理
- 当可能出现多种错误时，用自定义的错误类型来处理
- 不要让抛除的错误传播过远

### 7.Swift数组指针的妙用

``` Swift
	let numbers = [1, 2, 3, 4, 5]
let sum = numbers.withUnsafeBufferPointer { buffer -> Int in
    var result = 0
    for i in stride(from: buffer.startIndex, to: buffer.endIndex, by: 2) {
        result += buffer[i]
    }
    return result
}
// 'sum' == 9
```

### 8.简写闭包参数名
*来源《The Swift Programming Language (SWift 3 Beta)》Shorthand Argument Names 小节*

Swift 自动为内联闭包提供了依次代表代表参数值的`$0`,`$1`,`$2`等参数简写。

如果您在闭包表达式中使用参数名称简写，您可以在定义闭包时省略其参数列表，相应参数简写的类型会通过函数类型对其进行推断。此外`in`关键字也可以被省略，因为此时闭包表达式完全由闭包函数体构成：

``` Swift
	reversed = sorted(names, { $0 > $1 } )

```
在这个例子中，`$0`和`$1`分别表示闭包中第一个和第二个String类型的参数。

### 9.绘制虚线

```Swift
		if let context = UIGraphicsGetCurrentContext() {
            CGContextSetLineWidth(context, 2.0);
            CGContextSetStrokeColorWithColor(context, UIColor.greenColor().CGColor);
            CGContextBeginPath(context);
            
            // 该数组用于指定虚线的重复样式，这里指定绘制10，然后跳过5，依次重复。
            let lengths: [CGFloat] = [10.0, 5.0]
            // 第4个参数指定传入的数组包含有几个元素。
            CGContextSetLineDash(context, 0.0, lengths, 2)
            
            CGContextMoveToPoint(context, 0, rect.size.height/2)
            CGContextAddLineToPoint(context, rect.width, rect.size.height/2)
            CGContextStrokePath(context)
        }
```

### 10.访问控制
Swift 3.0更新了访问控制，添加了open和fileprivate两个新控制权限，这里摘录了[《The Swift Programming Language (Swift 3)》](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html#//apple_ref/doc/uid/TP40014097-CH41-ID3)书中部分内容：

Swift 为代码中的实体提供了5种不同的访问权限。访问权限同时取决于源文件中的实体定义及源文件所属模块。

- open、public 访问权限定义的实体，在同一模块内可于任意源文件内访问，即使该源文件属于另一个被引入的模块。open 和 public 的区别稍后介绍。
- internal 访问权限定义的实体，可以被其定义的模块内的任意源文件访问，模块外的源文件则不能。通常，定义一个 app 或 Framework 的内部结构时，会使用internal权限。
- fileprivate 访问权限，限制实体仅能在其定义的源文件内访问。当某个功能仅用于某个源文件时，我们就可以通过添加 fileprivate 来隐藏其具体的实现细节。
- private 访问权限定义的实体，仅能在其所定义的作用域内访问。当某个功能仅用于某个声明时，我们就可以通过 private 访问权限，隐藏其具体的实现细节。

open是最高访问权限（最少限制），private是最低访问权限（限制最多）。

open访问权限仅适用于类和类成员，它与public访问权限的区别如下：

- 定义为 public，或其它更严格访问权限的类，仅能在其所定义的模块内被继承。
- 定义为 public，或其它更严格访问权限的类成员，仅能在其所定义的模块内被override(覆盖、重写)。
- 定义为 open 访问权限的类，既可以在其所定义的模块内被继承，也可以在其它引入其被定义模块的模块中被继承。
- 定义为 open 访问权限的类成员，既可以在其所定义的模块内override(覆盖、重写)，也可以在其它引入其被定义模块的模块中override(覆盖、重写)。 
(译者：在a模块中用 open 定义了一个A类，那么A类既可以在a模块中被继承，也可以在任何引入了a模块的b，c...等模块中被继承)

将一个类标记为 open 访问权限，即表明你已充分考虑到该类会被其它模块当作父类继承，并为此妥善的编写了该类的代码。

### 11.@objc & dynamic
关键字，摘录自王巍 [《Swift tips》](http://swifter.tips/objc-dynamic/)

### 12.@escaping & @noescape
摘录自[《What Do @escaping and @noescape Mean In Swift 3》](https://cocoacasts.com/what-do-escaping-and-noescaping-mean-in-swift-3/)