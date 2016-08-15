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

```
	let a = 8
	
	let b = &a //编译错误

```
查看下错误信息："Type ‘inout Int’ of variable is not materializable"，这里编译器将`&a`识别为 inout Int类型，并不是我们期望的指针类型，所以是不是我们只要把它明确指定为指针类型就可以了呢？我们来试一下：

```
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
哈哈，成功了，顺利通过编译。（更深层的原因，笔者受限于个人知识水平，未能深究，有知晓者还望不吝赐教。）

### 6.Swift错误处理原则
*来源:[Magical Error Handling in Swift](https://www.raywenderlich.com/130197/magical-error-handling-swift)，由  [Gemma Barlow](https://www.raywenderlich.com/u/gemmakbarlow) 发表于Raywenderliche*

- 错误类型的命名要清晰无歧义
- 单个错误尽量用可选类型来处理
- 当可能出现多种错误时，用自定义的错误类型来处理
- 不要让抛除的错误传播过远