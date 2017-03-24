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
我们知道 Swift 通过其语言自身的特点，保证了全局常量和存储型属性，即使被多个线程交替存取，也仅初始化一次，但是这里有一个特例，就是 lazy 属性，当多个线程同时访问一个尚未初始化的 lazy 属性时，则不能保证仅初始化一次。

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
*来源《The Swift Programming Language (SWift 3.0.1)》Closure 一章Shorthand Argument Names 小节*

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
@objc 当 Objective-C 需要引用 Swift 代码，我们可以将需要暴露给 Objective-C 使用的任何地方 (包括类，属性和方法等) 的声明前面加上 @objc 修饰符。
@objc 修饰符的另一个作用是为 Objective-C 重新声明方法或者变量的名字
@dynamic 当需要用刀某些动态特性时，即可用此关键词，例如KVO。

### 12.@escaping & @noescape
摘录自[《What Do @escaping and @noescape Mean In Swift 3》](https://cocoacasts.com/what-do-escaping-and-noescaping-mean-in-swift-3/)

### 13.溢出运算符
当向一个整型常量或变量赋一个超出其所允许范围的数值时，默认情况下，Swift不会生成一个无效的数值，而是报错。该做法为过大或过小数值地操作提供了额外的安全性。

例如，Int16 整型能容纳从 -32768 到 32767 的有符号整型。如果尝试向 Int16 有符号整型的常量或变量赋超过其容纳范围的数值，则会报错：

``` Swift
var potentialOverflow = Int16.max
// potentialOverflow 被赋值为 Int16 整型所能容纳的最大值 32767。
potentialOverflow += 1
// 该操作会引发错误
```

在处理过大或过小值时提供相应的错误处理，可以让我们对边界的操作更加灵活。相对于报错，我们可以对数值进行截取。Swift提供了三种溢出运算符来让系统支持整型溢出运算。这些运算符均以 & 开头：
- 溢出加 &+
- 溢出减 &-
- 溢出乘 &*

### 14.特殊的字面量

在 C 中调试时我们经常会用到 `__FILE__`，`__FUNCTION__` 等特殊字面量，来获取一些信息，那么在 Swift 中怎么用呢？

```Swfit
    print(#file)// 打印当前文件

    func hello() {
        print(#function)//打印当前函数
    }
    hello()

    print(#line)//打印当前行号

    print(#column)// 打印所处的列，7
```

通过上面的例子可以看出，Swift 采用 `#` + 关键字小写的方式，很好的照顾了那些从其它语言转过来的朋友们。

# 15.用函数做参数以简化代码
摘录自：[@南峰子_老驴](http://m.weibo.cn/3321824014/4060979186247385)

```Swift
	let setInt: (Int, String) -> Void = UserDefaults.standard.set
	let getInt: (String) -> Int = UserDefaults.standard.integer

	setInt(10, "Ten")
	print(getInt("Ten"))
```

# 16.包名冲突

今天在尝试项目中引入“NetworkExtension”的时候，发现编译器提示“File 'NameOfCrrentFile.Swift is a part of module 'NetworkExtension' ; ignoring import”， 后来在苹果[官方论坛](https://forums.developer.apple.com/thread/45186)上找到了答案，原来是工程名“NetworkExtension”与要引入的包名冲突了，由于**.swift已经是“NetworkExtension”工程（实际也是一个包）一部分，所以Swift拒绝再在该包中引入一个同名的包。

Swift虽然支持命名空间，但是随即也带来了类似的问题，看来接下来有必要好好了解下Swift的包管理和命名空间问题。

# 17. #keyPath语法糖
摘录自：[@南峰子_老驴](http://m.weibo.cn/3321824014/4060979186247385)

在使用KVC或KVO时，经常会犯一些因KeyPath拼写不正确，从而导致应用崩溃的错误。为此，Swift 3中引入了 #keyPath()表达式，不多解释，直接看代码：

```Swift
class Person: NSObject {
    dynamic var firstName: String
    
    init(firstName: String) {
        self.firstName = firstName
    }
}

let chris = Person(firstName: "Chris")

let keyPath = #keyPath(Person.firstName)
chris.value(forKey: keyPath)
```
# 18. as、as?、as!

从字面理解`as`作为什么的意思，那也在Swift中它的作用就是类型转换（type casting）,那么三个as分别有什么不同呢？

`as` 直接转换，首先，其可用于从`sub class`到`super class`／`Any`的向上转换(upcast)，这种转换没有额外的风险，所以直接`as`:

``` Swift
class People {
}

class Child: People {
}

let john  = Child()
let people: People = johon as People

```
其次，其可用于消除歧义，指明类型；

``` Swift
let num1 = 7 as CGFloat

let num2 = 7 as Int

let num3 = 3.14 as Int

let num4 = (7 / 2) as Double
```

最后，其还可以在 Switch

``` Swift
class Boy: Child {
}

class Girl: Child {
}

let child: Child = Boy()

switch child {
    case let boy as Boy:
        //
        break
    case let girl as Girl:
        //
        break
    default:
        break
}
```

`as?`用于向下转换（downcast）。尝试去转换并返回一个可选类型，成功则包含一个值，失败则为`nil`，所以转换结果需要判断。

``` Swift
class People {
}

class Child: People {
}

let people: People  = People()

if let child = people as? Child {
    print("转换成功")
} else {
    print("转换失败")
}

```

`as!` 强制转换，成功返回可选类型，失败则导致运行时错误。所以在操作前，要求开发者非常清楚待转换对象的类型。

``` Swift
let someOne: Any = People()
let people = someOne as! People // 转换成功

let someOne: Any = "Who are you"
let people = someOne as! People // 转换失败，崩溃。

```
# 19.闭包的几种表达

闭包不是什么新生事物，但 Swift 仿佛对此情有独衷，提高了非常丰富的支持（效仿茴字吗？）：

``` Swift
let score = [1,3,40,32,2,53,77,13]

// Version 1
func sortAscending (_ i: Int , _ j: Int) -> Bool {
    return i < j
}
let scoreSorted1 = score.sorted(by: sortAscending)
// Version 2
let scoreSorted2 = score.sorted { (i, j) -> Bool in
    return i > j
}
// Version 3 
let scoreSorted3 = score.sorted {i, j in i < j}

// Version 4 
let scoreSorted4 = score.sorted(by: {$0 < $1})

// Version 5
let scoreSorted5 = score.sorted{$0 < $1}

// 以上五种表达方式是等价的。
``` 

	注意：不是所有的闭包都可以采用上面的任意一种写法。