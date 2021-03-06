#【粗译完成】 Unsafe Swift: Using Pointers And Interacting With C

*本文翻译自 [Unsafe Swift: Using Pointers And Interacting With C](https://www.raywenderlich.com/148569/unsafe-swift)*， *由 [Ray Fix](https://www.raywenderlich.com/u/rayfix) 发表于[Raywenderlich](https://www.raywenderlich.com)*。

*受限于译者英语水平及翻译经验，译文难免有词不达意，甚至错误的地方，还望不吝赐教，予以指正*

－－－－－－－－－－－－－－－－－－－

默认情况下，Swift 是内存安全的，这意味着禁止对内存进行直接访问，并确保所有对象在使用前都已被正确初始化。既然是“默认”，那么就意味着如果有需要，Swift 还是允许通过指针对内存进行直接访问的。

本文将带你领略 Swift 的 “Unsafe” 特性。这里的 “Unsafe“ 可能有点让人困惑，它并不意味着你写的代码即危险又糟糕，相反它是在提醒你，提醒你要格外小心地编写自己的代码，因为编译器将不在帮你进行一些必要的审查。

在运用 Swift 一些非安全特性与诸如 C 这类非安全语言交互之前，我们还需要了解一些额外的与运行相关的知识。本文讨论的是一个进阶性的话题，如果有 Swift 基础，那么你会更好的理解本文所涉内容。C 语言开发经验也会有所帮助，但不是必须的。 

### 开始（Getting Started）

本文由三个 playgrounds 构成。首先，会通过几段代码来熟悉一些内存排布和非安全指针操作的相关知识。其次，会把一个用于数据流压缩的 C API 封装成 SWift API。最后, 创建一个全平台的随机数生成器用于取代**arc4random**, 并通过 Unsafe Swift 相关技术向用户隐藏细节.

首先新建一个 playground, 命名为 **UnsafeSwift**。 平台任意, 本文所涉的代码均无平台限制。接下来导入 Foundation 框架。

###内存排布（Memory Layout）
*【译者注】这一小节翻译的特别不好，尤其是“内存对齐”，为了避免误导，如果你从未了解过内存排布知识，请先参考其它书籍了解相关知识，如《计算机系统》*

![Sample memory](https://koenig-media.raywenderlich.com/uploads/2017/01/memory-480x214.png)

Swift 的非安全操作直接与系统内存打交道。内存可以被看做是一系列排列整齐的盒子（实际上有数以亿计之多），每个里面都有一个数字。每个盒子都有一个唯一的**内存地址**与之关联。最小的存储地址单元被称为一个 **字节（byte）**，它通常由 8 个连续的**比特（bit）**构成。一个 8 位的字节可以存储 0-255 的任意数值。虽然一个单词占据的内存通常不只一个字节，但是处理器同样可以对其进行高效存取。以64位系统为例，一个字母是 8 字节 64 个比特。

通过 Swift 提供的 **MemoryLayout** 表达式可以查看内存中对象的大小和对齐情况。

在 playground 中添加如下代码:

```Swift
MemoryLayout<Int>.size          // returns 8 (on 64-bit)
MemoryLayout<Int>.alignment     // returns 8 (on 64-bit)
MemoryLayout<Int>.stride        // returns 8 (on 64-bit)
 
MemoryLayout<Int16>.size        // returns 2
MemoryLayout<Int16>.alignment   // returns 2
MemoryLayout<Int16>.stride      // returns 2
 
MemoryLayout<Bool>.size         // returns 1
MemoryLayout<Bool>.alignment    // returns 1
MemoryLayout<Bool>.stride       // returns 1
 
MemoryLayout<Float>.size        // returns 4
MemoryLayout<Float>.alignment   // returns 4
MemoryLayout<Float>.stride      // returns 4
 
MemoryLayout<Double>.size       // returns 8
MemoryLayout<Double>.alignment  // returns 8
MemoryLayout<Double>.stride     // returns 8
```

`MemoryLayout<Type>` 用于在编译期侦测指定类型的 **size**，**alignmen** 和 **stride**，其返回值以字节为单位。例如，**Int16** 在 **size** 上占用两个字节，且恰好与两个字节 **对齐**，占满两个字节。

举个例子，在地址100上开辟一个 **Int16** 是合法的，101则不可以，因为这违反内存对齐原则（译者注：0偏移）。

当有一串 **Int16** 时，那么她们每个之间以一个 **stride** 串联在一起，基础类型的 **size** 和 **stirde** 相同。（译者注：类似链表？）

接下来，在playground中添加如下代码，看看用户定义的结构体在内存中的排布情况:

```Swift
struct EmptyStruct {}
 
MemoryLayout<EmptyStruct>.size      // returns 0
MemoryLayout<EmptyStruct>.alignment // returns 1
MemoryLayout<EmptyStruct>.stride    // returns 1
 
struct SampleStruct {
  let number: UInt32
  let flag: Bool
}
 
MemoryLayout<SampleStruct>.size       // returns 5
MemoryLayout<SampleStruct>.alignment  // returns 4
MemoryLayout<SampleStruct>.stride     // returns 8
```

这个空结构体的 size 为 0，由于它可以被定为在任意内存地址上，所以 alignment 是 1，同样 stride 也是 1，这是因为任何一个 **EmptyStruct** 在创建时都有一个唯一的内存地址，虽然大小为 0。

而 **SampleStruct** 的 **size** 是 5 **stride** 是 8 。这是因为它必须要对齐到 4 个字节（即4的倍数），所以 Swift 会以 8 字节的间隔（译者注：**stride**）去访问。

接下来添加:

```Swift
class EmptyClass {}
 
MemoryLayout<EmptyClass>.size      // returns 8 (on 64-bit)
MemoryLayout<EmptyClass>.stride    // returns 8 (on 64-bit)
MemoryLayout<EmptyClass>.alignment // returns 8 (on 64-bit)
 
class SampleClass {
  let number: Int64 = 0
  let flag: Bool = false
}
 
MemoryLayout<SampleClass>.size      // returns 8 (on 64-bit)
MemoryLayout<SampleClass>.stride    // returns 8 (on 64-bit)
MemoryLayout<SampleClass>.alignment // returns 8 (on 64-bit)
```

由于类是引用类型，所以 **MemoryLayout** 返回的是“引用”自身的大小： 8 字节。

如果想了解更多关于内存布局方面的知识，可以看看这段 Mike Ash 的精彩[讲解](https://realm.io/news/goto-mike-ash-exploring-swift-memory-layout/)。

###指针（Pointers）

一个指针包含了一个内存地址。为了与内存直接操作的“不安全”相应，指针被冠以“Unsafe”前缀，该指针类型称为 UnsafePointer. 虽然要多打点字，但是它可以提醒你正在脱离编译器的帮助，错误的处理可能引起[不可预料的行为](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html)（并不限于崩溃）。

Swift的设计者，本可以只提供一种与 C 语言中 char \* 等价的，可任意访问内存的 **UnsafePointer** 类型。但是他们没有，相反 Swift 提供了近乎一打的指针类型，以适用不同的场景和目的。选择合适的指针类型至关重要，可以帮助你避免一些不可预料的行为。

Swift 指针采用了非常鲜明的命名方式，以便人们通过类型即可清楚其特点。可变还是不可变的, 类型明确还是类型未知的, 连续还是不连续。下面这张表罗列了这 8 中指针。

![unsafe swift pointers](https://koenig-media.raywenderlich.com/uploads/2016/12/pointers-650x444.png)

下面，我们将学习这些指针类型。

### 裸指针的应用 （Using Raw Pointers）

在playground中添加如下代码:

```Swift
// 1
let count = 2
let stride = MemoryLayout<Int>.stride
let alignment = MemoryLayout<Int>.alignment
let byteCount = stride * count
 
// 2
do {
  print("Raw pointers")
 
  // 3
  let pointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
  // 4
  defer {
    pointer.deallocate(bytes: byteCount, alignedTo: alignment)
  }
 
  // 5
  pointer.storeBytes(of: 42, as: Int.self)
  pointer.advanced(by: stride).storeBytes(of: 6, as: Int.self)
  pointer.load(as: Int.self)
  pointer.advanced(by: stride).load(as: Int.self)
 
  // 6
  let bufferPointer = UnsafeRawBufferPointer(start: pointer, count: byteCount)
  for (index, byte) in bufferPointer.enumerated() {
    print("byte \(index): \(byte)")
  }
}
```
该例中我们使用不安全的 Swift 指针去存取2个整数。

代码解释如下:

1. 这些常量保存一些常用的值:
	- **count** 表明需要存储几个整形数值
	- **stride** 表示保存Int类型的步长
	- **alignment** 用于存储Int类型对齐所需空间大小
	- **byteCount** 表示总占用内存空间大小

2. **do** 代码块，用于指定一个作用域，以便在后面的例子中还可以继续使用相同的变量名。

3. `UnsafeMutableRawPointer.allocate` 用于开辟指定字节数的内存，该方法返回一个 `UnsafeMutableRawPointer` 指针. 类型名称已经清楚的表明，该指针可以用于装载或存储（可变的）类型未知的裸字节.

4. `defer` 代码块，可以确保指针在使用后能够得到正确地释放。在这里 ARC 是无效，你需要自己手动管理内存。可以从[这里](https://www.raywenderlich.com/130197/magical-error-handling-swift)了解到更多有关 `defer` 的知识。

5. `storeBytes` 和 `load` 方法用于存储和装载字节。 第二个整数的内存地址，可以通过指针的 **stride** 计算取得。

	既然指针可以步进，那么就可以对指针进行类似这样的运算`(pointer+stride).storeBytes(of: 6, as: Int.self)`。

6. `UnsafeRawBufferPointer`可以对存有一串连续字节的内存进行访问。也就是说可通过它遍历所有字节，可以通过下标，或者以 **filter**, **map** **reduce** 等更酷的方式去访问。buffer pointer需要通过裸指针来初始化。

###类型指针的应用（Using Typed Pointers）

前面的例子可以通过类型指针简化. 在playground中添加如下代码:

```Swift
do {
  print("Typed pointers")
 
  let pointer = UnsafeMutablePointer<Int>.allocate(capacity: count)
  pointer.initialize(to: 0, count: count)
  defer {
    pointer.deinitialize(count: count)
    pointer.deallocate(capacity: count)
  }
 
  pointer.pointee = 42
  pointer.advanced(by: 1).pointee = 6
  pointer.pointee
  pointer.advanced(by: 1).pointee
 
  let bufferPointer = UnsafeBufferPointer(start: pointer, count: count)
  for (index, value) in bufferPointer.enumerated() {
    print("value \(index): \(value)")
  }
}
```

注意以下不同:

- 通过`UnsafeMutablePointer.allocate`方法开辟了一块内存. 而传入的参数是告诉swift该指针用于装载和存储 **Int** 类型
- 类型明确了的内存，在使用和销毁前都必需初始化，初始化和销毁可通过`initialize` 和 `deinitialize` 方法来完成。 Update: as noted by user atrick in the comments below, deinitialization is only required for non-trivial types. That said, including deinitialization is a good way to future proof your code in case you change to something non-trivial. Also, it usually doesn’t cost anything since the compiler will optimize it out.
- 类型明确的指针提供了一个 **pointee** 属性，通过它可以以类型安全方式装载和存储相应类型的值。
- 当移动类型指针时，可以简单地通过指定需要移动的距离来移动指针。指针会根据其所指向的类型自动计算正确的步长。指针运算又一次发挥了作用。如`(pointer+1).pointee = 6`。
- 与之前的 buffer pointer类似：只是一个遍历的是值，另一个遍历的是字节。

###裸指针到类型指针的转换（Converting Raw Pointers to Typed Pointers）

类型指针除了直接初始化得到，还可以通过裸指针取得。

在playground中添加如下代码:

```Swift
do {
  print("Converting raw pointers to typed pointers")
 
  let rawPointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
  defer {
    rawPointer.deallocate(bytes: byteCount, alignedTo: alignment)
  }
 
  let typedPointer = rawPointer.bindMemory(to: Int.self, capacity: count)
  typedPointer.initialize(to: 0, count: count)
  defer {
    typedPointer.deinitialize(count: count)
  }
 
  typedPointer.pointee = 42
  typedPointer.advanced(by: 1).pointee = 6
  typedPointer.pointee
  typedPointer.advanced(by: 1).pointee
 
  let bufferPointer = UnsafeBufferPointer(start: typedPointer, count: count)
  for (index, value) in bufferPointer.enumerated() {
    print("value \(index): \(value)")
  }
}
```

该示例与前一个类似，不同处在于它先创建了一个裸指针。之后通过将内存 **binding** 给指定的 **Int** 类型来得到类型指针。由于绑定了内存，所以可以以类型安全的方式访问。内存绑定是在创建类型指针时在后台完成的。

余下的内容与前一个示例相同。一旦转换成类型指针，就可以调用 `pointee` 了。

###获取一个实例的字节（Getting The Bytes of an Instance）

通常，可以通过`withUnsafeBytes(of:)`函数来获取某个类型示例所占字节的大小。

在playground添加如下代码:

```Swift
do {
  print("Getting the bytes of an instance")
 
  var sampleStruct = SampleStruct(number: 25, flag: true)
 
  withUnsafeBytes(of: &sampleStruct) { bytes in
    for byte in bytes {
      print(byte)
    }
  }
}
```

这里打印了 **SampleStruct** 实例的裸字节。`withUnsafeBytes(of:) `让你可以在闭包中访问一个 `UnsafeRawBufferPointer `。

`withUnsafeBytes` 在 **Array** 和 **Data** 实例上同样有效。

###求和校验（Computing a Checksum）

`withUnsafeBytes(of:)` 方法可以返回一个值，下面的例子以此来计算一个结构体的字节校验和。

在playground中添加如下代码:

```Swift
do {
  print("Checksum the bytes of a struct")
 
  var sampleStruct = SampleStruct(number: 25, flag: true)
 
  let checksum = withUnsafeBytes(of: &sampleStruct) { (bytes) -> UInt32 in
    return ~bytes.reduce(UInt32(0)) { $0 + numericCast($1) }
  }
 
  print("checksum", checksum) // prints checksum 4294967269
}
```

The reduce call adds up all of the bytes and ~ then flips the bits. Not a particularly robust error detection, but it shows the concept.

###非安全操作三原则（Three Rules of Unsafe Club）

编写不安全代码的时候务必小心，以避免那些不可预料的行为。这里罗列了一些糟糕的例子。

**不要通过 `withUnsafeBytes` 返回指针（Dont't return the pointer from withUnsafeBytes）!**

```Swift
 // Rule #1
do {
  print("1. Don't return the pointer from withUnsafeBytes!")
 
  var sampleStruct = SampleStruct(number: 25, flag: true)
 
  let bytes = withUnsafeBytes(of: &sampleStruct) { bytes in
    return bytes // strange bugs here we come ☠️☠️☠️
  }
 
  print("Horse is out of the barn!", bytes)  /// undefined !!!
}
```

不要在 `withUnsafeBytes(of:)` 作用域外调用指针。或许一时可用，但…

**一时间只绑定一种类型（Only bind to oen type at a time）!**

```Swift

// Rule #2
do {
  print("2. Only bind to one type at a time!")
 
  let count = 3
  let stride = MemoryLayout<Int16>.stride
  let alignment = MemoryLayout<Int16>.alignment
  let byteCount =  count * stride
 
  let pointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
 
  let typedPointer1 = pointer.bindMemory(to: UInt16.self, capacity: count)
 
  // Breakin' the Law... Breakin' the Law  (Undefined behavior)
  let typedPointer2 = pointer.bindMemory(to: Bool.self, capacity: count * 2)
 
  // If you must, do it this way:
  typedPointer1.withMemoryRebound(to: Bool.self, capacity: count * 2) {
    (boolPointer: UnsafeMutablePointer<Bool>) in
    print(boolPointer.pointee)  // See Rule #1, don't return the pointer
  }
}
```

![badpun](https://koenig-media.raywenderlich.com/uploads/2016/12/badpun-480x175.png)

绝不要将内存同时绑定给两个不相关的类型。这被称为类型双关而 Swift 不喜欢双关。但可以通过 `withMemoryRebound(to:capacity:)` 方法暂时地重新绑定内存。同样，将一个普通类型（例如：Int）重新绑定到一个非普通类型（例如：类）也是不合法的。所以千万别这么做。

**千万别越界（Don’t walk off the end… whoops）!**

```Swift
// Rule #3... wait
do {
  print("3. Don't walk off the end... whoops!")
 
  let count = 3
  let stride = MemoryLayout<Int16>.stride
  let alignment = MemoryLayout<Int16>.alignment
  let byteCount =  count * stride
 
  let pointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
  let bufferPointer = UnsafeRawBufferPointer(start: pointer, count: byteCount + 1) // OMG +1????
 
  for byte in bufferPointer {
    print(byte)  // pawing through memory like an animal
  }
}
```
在已出现的 off-by-one 错误中，尤以不安全代码最糟糕。所以务必小心审查，测试你的代码!

###不安全的Swift 示例 1: 压缩（算法）（Unsafe Swift Example 1: Compression）

接下来我们运用之前所讲的知识来对一个 C API 进行封装。Coca 框架中包含一个 C 模块，其实现了一些常用的压缩算法。例如 LZ4压缩算法速度最快，LZ4A算法的压缩比最高，但相对速度较慢，ZLIB算法在时间和压缩比上比较均衡，此外还有一个新的（开源）LZFSE算法，更好的平衡了空间和压缩速率。

创建一个新的 playground，命名为 Compression（压缩）。默认设置即可。然后用下面的代码替换原有内容：

```Swift
import Foundation
import Compression
 
enum CompressionAlgorithm {
  case lz4   // speed is critical
  case lz4a  // space is critical
  case zlib  // reasonable speed and space
  case lzfse // better speed and space
}
 
enum CompressionOperation {
  case compression, decompression
}
 
// return compressed or uncompressed data depending on the operation
func perform(_ operation: CompressionOperation,
             on input: Data,
             using algorithm: CompressionAlgorithm,
             workingBufferSize: Int = 2000) -> Data?  {
  return nil
}
```
用来执行压缩和解压缩操作的 **perform** 函数还是空的，只是简单地返回 nil。稍后我们会添加一些“非安全代码”。

在 playground 中代码的最后添加如下代码:

```Swift
// Compressed keeps the compressed data and the algorithm
// together as one unit, so you never forget how the data was
// compressed.
 
struct Compressed {
 
  let data: Data
  let algorithm: CompressionAlgorithm
 
  init(data: Data, algorithm: CompressionAlgorithm) {
    self.data = data
    self.algorithm = algorithm
  }
 
  // Compress the input with the specified algorithm. Returns nil if it fails.
  static func compress(input: Data,
                       with algorithm: CompressionAlgorithm) -> Compressed? {
    guard let data = perform(.compression, on: input, using: algorithm) else {
      return nil
    }
    return Compressed(data: data, algorithm: algorithm)
  }
 
  // Uncompressed data. Returns nil if the data cannot be decompressed.
 func decompressed() -> Data? {
    return perform(.decompression, on: data, using: algorithm)
  }
 
}
```

**Compressed** 结构体存有压缩后的数据和相应的压缩算法。这可以在解压时减少因算法侦测不正确而导致的错误。

在 Playground 的代码末尾添加如下内容:

```Swift
// For discoverability, add a compressed method to Data
extension Data {
 
  // Returns compressed data or nil if compression fails.
  func compressed(with algorithm: CompressionAlgorithm) -> Compressed? {
    return Compressed.compress(input: self, with: algorithm)
  }
 
}
 
// Example usage:
 
let input = Data(bytes: Array(repeating: UInt8(123), count: 10000))
 
let compressed = input.compressed(with: .lzfse)
compressed?.data.count // in most cases much less than orginal input count
 
let restoredInput = compressed?.decompressed()
input == restoredInput // true
```

主入口是一个 **Data** 类型的扩展。我们已经添加了一个名为 `compressed(with:)` 的函数，它返回一个可选类型的 **Compressed** 结构体。该方法只是简单地调用了 **Compressed** 的静态方法 `compress(input:with:)`。

滚动到首次进入时的代码块，`perform(_:on:using:workingBufferSize:)` 函数实现如下：

```Swift
func perform(_ operation: CompressionOperation,
             on input: Data,
             using algorithm: CompressionAlgorithm,
             workingBufferSize: Int = 2000) -> Data?  {
 
  // set the algorithm
  let streamAlgorithm: compression_algorithm
  switch algorithm {
  case .lz4:   streamAlgorithm = COMPRESSION_LZ4
  case .lz4a:  streamAlgorithm = COMPRESSION_LZMA
  case .zlib:  streamAlgorithm = COMPRESSION_ZLIB
  case .lzfse: streamAlgorithm = COMPRESSION_LZFSE
  }
 
  // set the stream operation and flags
  let streamOperation: compression_stream_operation
  let flags: Int32
  switch operation {
  case .compression:
    streamOperation = COMPRESSION_STREAM_ENCODE
    flags = Int32(COMPRESSION_STREAM_FINALIZE.rawValue)
  case .decompression:
    streamOperation = COMPRESSION_STREAM_DECODE
    flags = 0
  }
 
  return nil /// To be continued
}
```
从 Swift 类型转换为 C 类型需要用到 compression 库中提供的压缩算法和操作。

接下来用如下代码替换`return nil`:

```Swift
// 1: create a stream
var streamPointer = UnsafeMutablePointer<compression_stream>.allocate(capacity: 1)
defer {
  streamPointer.deallocate(capacity: 1)
}
 
// 2: initialize the stream
var stream = streamPointer.pointee
var status = compression_stream_init(&stream, streamOperation, streamAlgorithm)
guard status != COMPRESSION_STATUS_ERROR else {
  return nil
}
defer {
  compression_stream_destroy(&stream)
}
 
// 3: set up a destination buffer
let dstSize = workingBufferSize
let dstPointer = UnsafeMutablePointer<UInt8>.allocate(capacity: dstSize)
defer {
  dstPointer.deallocate(capacity: dstSize)
}
 
return nil /// To be continued
```

代码释义如下:

1. 创建一个compression_stream并且通过defer代码块儿，确保其能够及时释放。
2. 接下来，通过访问 pointee 属性得到 steam，并且将其传递给compression_stream_init方法.编译器会做一些特殊的处理（必要的初始化）。 通过输入输出标识符 & 将接收的 compression_stream自动转换为UnsafeMutablePointer<compression_stream>。 (当然直接传递streamPointer也可以，这样就不需要转换了)
3. 左后，创建一个目标缓存，作为工作输出的缓冲区。

用如下代码替换掉`return nil`，以完成 `perform` 函数:

```Swift
// process the input
return input.withUnsafeBytes { (srcPointer: UnsafePointer<UInt8>) in
  // 1
  var output = Data()
 
  // 2
  stream.src_ptr = srcPointer
  stream.src_size = input.count
  stream.dst_ptr = dstPointer
  stream.dst_size = dstSize
 
  // 3
  while status == COMPRESSION_STATUS_OK {
    // process the stream
    status = compression_stream_process(&stream, flags)
 
    // collect bytes from the stream and reset
    switch status {
 
    case COMPRESSION_STATUS_OK:
      // 4
      output.append(dstPointer, count: dstSize)
      stream.dst_ptr = dstPointer
      stream.dst_size = dstSize
 
    case COMPRESSION_STATUS_ERROR:
      return nil
 
    case COMPRESSION_STATUS_END:
      // 5
      output.append(dstPointer, count: stream.dst_ptr - dstPointer)
 
    default:
      fatalError()
    }
  }
  return output
}
```

这里才是真正执行压缩任务的代码. 下面是代码释义:

1. 创建一个Data对象，用于存储输出数据，具体是压缩后的数据还是解压后的数据取决与当前操作。
2. 设置输入输出指针，及其大小。
3. 持续掉用compression_stream_process 直至完成 即状态COMPRESSION_STATUS_OK.
4. 目标 buff 被拷贝到 output 其从中会被返回
5. 当传入最后一个包时, marked with COMPRESSION_STATUS_END only part of the destination buffer potentially needs to be copied.

在这个例子中，我们可以看到含有10000个元素的数组被压缩到153字节。

###不安全的Swift 示例2:随机数生成器（Unsafe Swift Example 2: Random Generator）

无论是游戏还是机器学习类的应用，随机数至关重要。macOS 提供了 `arc4random` 函数用于生成随机数。糟糕的是 Linux 上不可用。此外 `arc4random` 仅提供 UInt32类型的随机数。而 `/dev/urandom` 提供了一个无限的随机数源。

这一小节，我们会运用新学到的知识读取这个文件，并创建类型安全的随机数。

![hexdump](https://koenig-media.raywenderlich.com/uploads/2016/12/hexdump-480x202.png)

先创建一个新的playground, 命名为“RandomNumbers”。确保平台选择macOS.
创建完毕后，用一下代码替换原有默认内容:

```Swift
import Foundation
 
enum RandomSource {
 
  static let file = fopen("/dev/urandom", "r")!
  static let queue = DispatchQueue(label: "random")
 
  static func get(count: Int) -> [Int8] {
    let capacity = count + 1 // fgets adds null termination
    var data = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)
    defer {
      data.deallocate(capacity: capacity)
    }
    queue.sync {
      fgets(data, Int32(capacity), file)
    }
    return Array(UnsafeMutableBufferPointer(start: data, count: count))
  }
}
```
将 file 声明为静态变量，以保证系统中仅存在一个实例。当线程退出时系统会关闭打开的文件。由于存在多个进程获取随机数的可能，所以通过一个串行的 GCD 队列来保证它的存取顺序。所有实现都在 get 方法中。首先创建capacity，因为 fgets 总是以 0 结束，所以这里要额外的加 1。从file中得到数据，确保操作是运行在 GCD 队列上，最后，将数据拷贝到一个由 `UnsafeMutableBufferPointer` 指针指向起始位的标准的数组中。现在我们得到一个含有 Int8 数值的数组。

接下来在 playground 末尾添加如下代码：

```Swift
extension Integer {
 
  static var randomized: Self {
    let numbers = RandomSource.get(count: MemoryLayout<Self>.size)
    return numbers.withUnsafeBufferPointer { bufferPointer in
      return bufferPointer.baseAddress!.withMemoryRebound(to: Self.self, capacity: 1) {
        return $0.pointee
      }
    }
  }
 
}
 
Int8.randomized
UInt8.randomized
Int16.randomized
UInt16.randomized
Int16.randomized
UInt32.randomized
Int64.randomized
UInt64.randomized
```

这里在通过扩展 Integer 协议，为其所有子类型添加了一个静态的 randomized 属性(可查阅[protocol oriented programming](https://www.raywenderlich.com/148448/introducing-protocol-oriented-programming)了解更多)。

###接下来（ Where to Go From Here?）

这是完整的[playgrounds](https://koenig-media.raywenderlich.com/uploads/2017/01/Unsafe.zip)。下面还提供了一些额外的一些资源，以便加深了解:

- [Swift Evolution 0107: UnsafeRawPointer API](https://github.com/apple/swift-evolution/blob/master/proposals/0107-unsaferawpointer.md) 更详细的介绍了Swift内存模型，可以帮助你更好的读懂API文档.
- [Swift Evolution 0138: UnsafeRawBufferPointer API](https://github.com/apple/swift-evolution/blob/master/proposals/0138-unsaferawbufferpointer.md)更多的讲述了如何与untyped memory的交互，并提供了一些开源工程连接，以便更好的运用。
- 如果你正在向Swift 3迁移非安全代码，可以看看这篇[The Migration Guide](https://swift.org/migration-guide/se-0107-migrate.html)。即使你还没开始前一，文中也提供了大量有趣的例子，值得一看。
- [Interacting with C APIs](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithCAPIs.html) 可以帮助更深入的了解与C API的交互。
Mike Ash 在[Exploring Swift Memory Layout](https://realm.io/news/goto-mike-ash-exploring-swift-memory-layout/)分享了一些非常棒的经验。
希望大家能喜欢这篇文章. 如果你有什么问题，或者有什么经验要分享，欢迎大家将其发布到论坛上，我会时刻关注的!