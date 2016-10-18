# Swift 3.0 Unsafe World

本文是[《Swift 3.0 Unsafe World》](http://technology.meronapps.com/2016/09/27/swift-3-0-unsafe-world-2/?utm_source=Swift_Developments&utm_medium=email&utm_campaign=Swift_Developments_Issue_58) 的中文译文，由作者：[Roberto Perez](http://technology.meronapps.com/author/rob/) 发布于 [technology.meronapps.com](http://technology.meronapps.com)

*受限于译者英语水平及翻译经验，译文难免有词不达意，甚至错误的地方，还望不吝赐教，予以指正*

与大多数现代编程语言一样，在Swift的环境中我们都很从容，因为内存被额外的元素来管理，可能是编译器或者是Swift的运行时，也可能是糟糕的垃圾回收器。这一切的处理都隐藏的语言背后，很少需要我们来干预。

然而，鉴于Swift的兼容性，还是会涉及到诸如：OpenGL 或 POSIX 函数等 C Api的调用，这时我们就不得不出处理那个让大多数人都头疼的问题，对，就是指针和堆内存的手动管理。

在3.0之前，Swift的unsafe API有点让人困惑，有多种方式获得相同的结果，这导致我们在复制粘贴 stackoverflow 上的代码时，并没有真正理解它。到了3.0 发生了变化，这些API变得更加完善。

本文不会涉及如何将代码从Swift 2.x 迁移到 3.0，会重点讨论 3.0的变化，偶尔会与 C 进行比较，主要目的就是能正确的使用不安全引用与低级 C APIs进行交互。

让我们通过一个简单的例子，创建并持有一个整型指针。

在 C 语言中：

``` C
int *a = malloc(sizeof(int));
*a = 42;

printf("a's value: %d", *a);

free(a)
```

同样的在Swift中：

``` Swift
let a = UnsafeMutablePointer<Int>.allocate(capacity:1)
a.pointee = 42

print("a's value: \(a.pointee)") //42

a.dealocate(capacity:1)

```

我们在Swift中看到的第一个类型是 `UnsafeMutablePointer<T>`，该结构体代表一个指向T类型的指针，如你所见，他还有一个静态方法，`allocate`，它可转换`capacity`个数的元素。

真如所预想那样，还存在一个`UnsafeMutablePointer `的变体`UnsafePointer `，它不允许更该其所指向的元素，不仅如此，不可变的`UnsafePointer `甚至都没有`allocate`方法。

在Swift中，还有另外一种方式去生成一个`Unsafe[Mutable]Pointer `，那就是 `&` 操作符。当向一个`block`或`function`传参时，可以通过 `&` 来指定其接受一个指针。例如：

``` Swift
func receive(pointer: UnsafePointer<Int> ) {
	pirnt("param value is: \(pointer.pointee)") // 42
}

var a: Int = 42
receive(pointer: &a)

```

`&`操作只能应用于`var`声明的变量，任何时候我们都可以对其进行任意操作。例如，可以接受一个可变引用并修改引用值：

``` Swift
func receive(mutablePointer: UnsafeMutablePointer<Int>) {
	mutablePointer.pointee *= 2
}

var a = 42
receive(mutablePointer: &a)
print("A's value has changed in the function: \(a)") // 84

```

这里与第一个例子最大的不同在于，在第一个例子中我们手动开辟内存，（）and in the sample with a function, we are creating a pointer from a Swift allocated memory。显然，管理内存和接受指针是两个不同的点。在本文最后我们会讨论内存管理。

如果要接受一个由Swift管理内存的指针，且不通过创建一个函数的方式该怎么办？那么可以使用`withUnsafeMutablePointer`, 该方法接受一个Swift类型的指针和一个以该指针作为其参数的 block。例子：

``` Swift
var a = 42
withUnsafeMutablePointer(to: &a) { $0.pointee *= 2}

print("a's value is: \(a)") //84

```

现在我们了解了 如何通调用 带有指针参数的 C API，接下里我们看一个POSIX 的例子，通过 `opendir / readdir`来列出当前目录的内容。

``` Swift
var dirEnt: UnsafeMutablePointer<dirent>?
var dp: UnsafeMutablePointer<DIR>?

let data = ".".data(using: .ascii)
data?.withUnsafeBytes({(ptr: UnsafePointer<Int8>) in 
	dp = opendir(ptr)
})

repeat {
	dirEnt = readdir(dp)
	if let dir = dirEnt {
		withUnsafePointer(to: &dir.pointee.d_name, { ptr in 
			let ptrStr = unsafeBitCase(ptr, to: UnsafePointer<CChar>.self)
			let name = String(cString: ptrStr)
			print("\(name)")
		})
	}
} while dirEnt != nil

```

### Cast between pointers(指针间转换)

在操作 C API 时，有时会需要从一种指针转换为另一种指针。这在 C 中处理起来非常简单（这非常危险也非常容易出错），在Swift中，所有指针都必需指定类型，这也就意味着一个`UnsafePointer<Int>`不能用于一个接受`UnsafePointer<UInt8>`类型的参数，这在对于代码的安全性而言很有帮助，但这也意味着与那些需要类型转换的 C API进行交换变为不可能，例如，socket 的 `bind()`函数。这种情况，我们就需要用到`withMemoryRebound` ，该函数可将指针从一种类型转化为其它类型，接下来我们来看看在bind函数中(TODO:这里翻译不完整)

``` Swift
var addrIn = sockaddr_in()
// Fill sockaddr_in fields
withUnsafePointer(to: &addrIn) { ptr in 
	ptr.withMemoryRebound(to: sockaddr.self, capactiy: 1, { ptrSockAddr in 
		bind(socketFd, UnsafePointer(ptrSockAddr), socklen_t(MemoryLayout<sockaddr>.size))
	})
}
```

在转换指针类型时，有一种特殊情况，一些 C API 要求传入一个 void * 指针。在Swift 3.0 之前，我们可以用UnsafePointer<Void>，然而，3.0中添加了一个新的类型，来出来该类型指针：UnsafeRawPointer。该结构体不普通（这里翻译部通顺），意味着她不会持有任何绑定于任何特殊类型的信息，这简化了我们的代码。要创建一个UnsafeRawPointer，我们可以通过它的构造器包装一个现有指针来实现。如果想尝试不同的方法，讲一个UnsafeRawPointer转换一个指定类型的指针，我们可以使用我们之前用到的withMemoryRebound的一个特别版，assumingMemoryBound.

``` Swift
let intPtr = UnsafeMutablePointer<Int>.allocate(capacity: 1)
let voidPtr = UnsafeRawPointer(intPtr)
let intPtrAgain = VoidPtr.assumingMemoryBound(to: Int.self)

```

### Pointers as arrays

至此，我们已经讨论指针的典型应用场景，已经可以处理大多数的 C API调用。然而指针应用非常广泛，其中之一便是，内存数据遍历。在Swift中有几种方法来实现该功能。事实上，UnsafePointer 有一个advanced(by:) 方法，允许你遍历内存，advanced(by:) 会返回另一个UnsafePointer，可借此进行相应的存取操作。

``` Swift
let size = 10
var a = UnsafeMutablePointer<Int>.allocate(capacity: size)
for idx in 0..<10 {
	a.advanced(by: idx).pointee = idx
}
a.deallocate(capacity: size)
```

除此之外，Swift还有一个结构体，使用起来更加便捷，那就是UnsafeBufferPointer。该结构体就是Swift 数组和指针间的桥梁。如果我嘛从一个UnsafePointer构建一个UnsafeBufferPointer，那么我们就获得了大部分像Swift原生类型一样操作数组的能力，UnsafeBufferPointer实现了 Collection, Indexable 和 RandomAccessCollection等Swift协议。例子：

``` Swift
// Using a and size frome previous code
var b = UnsafeBufferPointer(start: a, count: size)
b.forEach({
	print("\($0)" // prints 0 to 9 that we fill previously)
})
```

之所以我们说UnsafeBufferPointer是Swift 数组的桥梁，还因为从一个现有数组非常容易得到一个指向它的UnsafeBufferPointer，例如：

``` Swift
var a = [1, 2, 3, 4, 5, 6]
a.withUnsafeBufferPointer({ ptr in 
	ptr.forEach({ print("\($0)")}) // 1, 2, 3...
})
```

### Memory management dangers

我们已经看到很多引用原始内存的方法，但有一点不能忘记，我们正在进入一个危险的领域。或许那个一再出现的unsafe，正警醒我们小心使用。不仅于此，我们正在混淆两个词，当使用Unsafe引用时，下面通过一个例子来看一看隐藏在其光环下的危险。

``` Swift
var collectionPtr: UnsafeMutableBufferPointer<Int>?

func duplicateElements(inArray: UnsafeMutableBufferPointer<Int>){
	for i in 0..<inArray.count {
		inArray[i] *= 2
	}
}

repeat {
	var collection = [1, 2, 3]
	collection.withUnsafeMutableBufferPointer({ collectionPtr = $0 })
} while false

duplicateElements(inArray: collectionPtr!) // crash due to EXC_BAD_ACCESS
```
		
虽然这只是个例子，但是它展示了当在代码中使用指针指向由Swift管理的变量时可能出现的问题。例子中collection在block中创建，所以在block结束时对它的引用也将被释放。所以我们在CollectionPtr保存的Collection引用也不在有效，当我们尝试在duplicateElements(inArray:)尝试使用一个无效引用就会造成崩溃。如果我们想使用一个指向受Swift内存管理元素的指针时，必须要保证在使用时它是有效的。始终要记住ARC会自动为那些俩开有效区域的引用添加release，如果它没有被strong引用，那么他将被释放。

避免让SWift来管理内存的办法就是我们自己来管理，像之前的几个例子那样，这就能排除无效内存引用的问题，但会引入另一个问题，如果我们没有手动销毁我们开辟的内存，会造成内存泄漏。

### bitPattern for pointers with fixed values

最后我们想介绍两个在Swift中的指针用法。首先是 void * 指针，它在 C API非常有用，用来指代一段内存地址。通常当一个函数可以接受不同类型的参数时，会用其对不同的值进行封装。例如：

``` Swift
void generic_function(int value_type, void* value);

generic_function(VALUE_TYPE_INT, (void *)2);
struct function_data data;
generic_function(VALUE_TYPE_STRUCT, (void *)&data);

```

如果我们想在swift中调用第一个函数，那么我们就需要使用特殊的构造器，它允许我们创建一个指向地址的指针。之前我们遇到的所有函数都不允许改变指针的地址，所以这里我们将用到`UnsafePointer(bitPattern:)`：

``` Swift
generic_function(VALUE_TYPE_INT, UnsafeRawPointer(bitPattern: 2))
var data = function_data()
withUnsafePointer(to: &data, { generic_function(VALUE_TYPE_STRUCT, UnsafeRawPointer($0)) })

```

### Opaque Pointer

最后，我们来讨论一下 Opaque pointer 转 Swift 类型的用法。它在一些包含用户数据参数的 C API 中非常常见，这些用户数据以一个指向未知内存地址的 void* 指针的形式呈现，随后会被调用。一个常见的使用场景就是处理那些由事件触发的设置有回调的函数。这时向Swift对象传递一个引用就变得非常有用，因为可以从 C 回调它的方法。

我们可以使用如下所用到的 UnsafeRawPointer，然而这会带来内存管理问题。如果我们向一个 C 函数传递一个非自己持有的对象指针，那么这个对象有个能被释放，早成应用崩溃。

Swift有一个非常实用的功能，可以接受一个指向对象的指针，并视需要对其进行持有或非持有。它们是非管理结构体的静态函数。通过`passRetained()`我们可以创建一个持有该对象的引用，这样我们就能确定在 C 环境调用时，他不会被释放掉。如果该对象已经在整个回调周期被持有，那么我们还可以调用`passUnretained()`。这两个方法都会产生的实力，都可以通过调用 `toOpaque()` 方法转换为UnsafeRawPointer

此外，我们还可以通过`fromOpaque()` 、 `takeRetained()` 、 `takeUnretained()`这几个转换方法，将一个UnsafeRawPointer，转换类或者结构体的实例。

``` Swift
void set_callback(void (* functionPtr)(void *), void* userData));
```

``` Swift
struct CallbackUserData {
	func sayHello() { print("hello world!") }
}

func callback(userData: UnsafeMutableRawPointer) {
	let callbackUserData = Unmanaged.fromOpaque(userData).takeRetainedValue()
	
	callbackUserData.sayHello() // "hello world!"
}

var userData = CallbackUserData()
let reference = Unmanaged.passRetained(userData).toOpaque()
set_callback(callback, reference)
```

### Conclusions

如你所见，在Swift中调用 C 代码时完全可行的，只要了解现有工具，实现起来很简单，也不需要大量的代码。虽然本文大量篇幅都在讨论 Unsafe 和 Unmanaged API，但我希望你能能把它作为你进一步学习和了解的基础。
