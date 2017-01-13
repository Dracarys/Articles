#【译】 Unsafe Swift: Using Pointers And Interacting With C

*本文翻译自 [Unsafe Swift: Using Pointers And Interacting With C](https://www.raywenderlich.com/148569/unsafe-swift)*， *由 [Ray Fix](https://www.raywenderlich.com/u/rayfix) 发表于[Raywenderlich](https://www.raywenderlich.com)*。

*受限于译者英语水平及翻译经验，译文难免有词不达意，甚至错误的地方，还望不吝赐教，予以指正*

－－－－－－－－－－－－－－－－－－－

默认情况下，Swift 是内存安全的，这意味着它禁止直接对内存进行操作，并确保所有对象在使用前都已初始化。既然有“默认情况”，那么就意味如果有需要，Swift也允许你通过指针对内存进行直接操作。

本文将带你领略Swift的“不安全”特性。这里的“不安全“可能会引起误解，它并不意味着你写的代码及危险又糟糕，相反它是提醒你要格外的留意自己的代码，因为编译器将不在帮你进行一些必要的审查。

如果你有与诸如 C 这样的不安全语言进行交互的需求，那么在使用这些 Swift 特性前，你可能还需要了解一些额外的与运行时相关的知识。这是一个更深入的话题，如果你熟悉 Swift，那么这些知识可以帮助你更好的理解，C 语言经验也会有所助益，但不是必须的。 

### 开始（Getting Started）

本文由三个 playgrounds 构成。首先，我们会创建几个小段代码以认识内存布局和不安全的指针操作。其次，我们将一个执行数据流的底层的 C API封装成 Swift 样式。最后, you will create a platform independent alternative to arc4random that, while using unsafe Swift, hides that detail from users.

先来新建一个playground, 命名为 *UnsafeSwift* . 平台任意, 本文所涉的代码均全平台通用. 确认导入了 Foundation framework.

###内存布局（Memory Layout）

![Sample memory](https://koenig-media.raywenderlich.com/uploads/2017/01/memory-480x214.png)

不安全的Swift直接与系统内存打交道。内存可以被看做是一系列排列整齐的盒子（事实上有数十亿之多），每个里面都有一个数字。每个盒子都有一个唯一的内存地址与之关联。最小的存储单元叫 *字节（byte）*，它由8个连续的比特位（bit）构成。8位构成的字节可以存储0-255的任意值。处理器可以高效地存取内存中不只一个字节的单词。以64位系统为例，一个字母是8位，长度为64个比特。

Swift 有个 MemoryLayout 的函数，可以查看内存中对象的大小和对齐情况。

在你的 playground 中添加如下代码:

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

MemoryLayout<Type> 估算函数可以在编译期侦测指定类型的大小，对齐和步长。返回值以字节为单位。例如，Int16 占用两个字节，且恰好与两个字节对齐。也就是说，都是从两段地址的起始。也就是说，从地址100开始开辟一个合法的Int16，注意不是101，因为那样会违反对齐规则。

When you pack a bunch of Int16s together, they pack together at an interval of stride. 因此，基础类型的大小与步长相同。

接下来，向playground中添加如下代码，我们看看用户定义的结构体在内存中对布局情况:

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

The empty structure has a size of zero. It can be located at any address since alignment is one. (i.e. All numbers are evenly divisible by one.) The stride, curiously, is one. This is because each EmptyStruct that you create has to have a unique memory address despite being of zero size.
For SampleStruct, the size is five but the stride is eight. This is driven by its alignment requirements to be on 4-byte boundaries. Given that, the best Swift can do is pack at an interval of eight bytes.
Next add:

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

由于类是引用类型，所以通过 MemoryLayout 得到的大小是： 8 字节。

如果想了解更多关于内存布局方面的知识，可以观看这段Mike Ash 的[视频](https://realm.io/news/goto-mike-ash-exploring-swift-memory-layout/)。

###指针（Pointers）

一个指针包含了一个内存地址。为了与内存直接操作的“不安全”相应，指针被称为 UnsafePointer. 虽然打起来有点烦，但是它可以提醒你正在脱离编译器的帮助，错误的处理可能引起不可预料的行为（并不限于崩溃）。

Swift的设计者，本可以只提供一种与 C 语言中 char \* 等价，可任意访问内存的 UnsafePointer 类型。但是他们没有，相反Swift 包含了将近一打的指针类型，以使用不同的场景和目的。选择合适的指针类型至关重要，可以帮助你远离那些未预料的行为。

Swift 指针采用了非常鲜明的命名方式，以便人们通过类型即可清楚其特点。可变还是不可变，类型不明确还是类型明确的，连续还是不连续。下面这张表罗列了这8中指针。

![unsafe swift pointers](https://koenig-media.raywenderlich.com/uploads/2016/12/pointers-650x444.png)

接下来一节，我们将学习这些指针类型。

### 裸指针应用 （Using Raw Pointers）

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
该例中我们使用不安全的Swift 指针去存取2个整数。

代码解释如下:

1. These constants hold often used values:
	- count holds the number of integers to store
	- stride holds the stride of type Int
	- alignment holds the alignment of type Int
	- byteCount holds the total number of bytes needed

2. A do block is added, to add a scope level, so you can reuse the variable names in upcoming examples.

3. The method UnsafeMutableRawPointer.allocate is used to allocate the required bytes. This method returns an UnsafeMutableRawPointer. The name of that type tells you the pointer can be used to load and store (mutate) raw bytes.

4. A defer block is added to make sure the pointer is deallocated properly. ARC isn’t going to help you here – you need to handle memory management yourself! You can read more about defer here.

5. The storeBytes and load methods are used to store and load bytes. The memory address of the second integer is calculated by advancing the pointer stride bytes.
Since pointers are Strideable you can also use pointer arithmetic as in (pointer+stride).storeBytes(of: 6, as: Int.self).

6. An UnsafeRawBufferPointer lets you access memory as if it was a collection of bytes. This means you can iterate over the bytes, access them using subscripting and even use cool methods like filter, map and reduce. The buffer pointer is initialized using the raw pointer.

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
- Memory is allocated using the method UnsafeMutablePointer.allocate. The generic parameter lets Swift know the pointer will be used to load and store values of type Int.
- Typed memory must be initialized before use and deinitialized after use. This is done using initialize and deinitialize methods respectively. Update: as noted by user atrick in the comments below, deinitialization is only required for non-trivial types. That said, including deinitialization is a good way to future proof your code in case you change to something non-trivial. Also, it usually doesn’t cost anything since the compiler will optimize it out.
- Typed pointers have a pointee property that provides a type-safe way to load and store values.
- When advancing a typed pointer, you can simply state the number of values you want to advance. The pointer can calculate the correct stride based on the type of values it points to. Again, pointer arithmetic also works. You can also say (pointer+1).pointee = 6
- The same holds true for typed buffer pointers: they iterate over values, instead of bytes.

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

该示例与前一个类似，不同处在于它先创建了一个裸指针。之后通过将内存绑定给制定的类型来得到类型指针。由于绑定了内存，所以可以以类型安全的方式存取。内存绑定是在创建类型指针时在后台完成的。余下的内容与前一个示例相同。一旦转换成类型指针，就可以调用'pointee'了。

###获取示例的字节（Getting The Bytes of an Instance）

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

This prints out the raw bytes of the SampleStruct instance. The withUnsafeBytes(of:) method gives you access to an UnsafeRawBufferPointer that you can use inside the closure.
withUnsafeBytes is also available as an instance method on Array and Data.

###求和校验（Computing a Checksum）

Using withUnsafeBytes(of:) you can return a result. An example use of this is to compute a 32-bit checksum of the bytes in a structure.
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

便携不安全代码的时候务必小心，以避免那些不可预料的行为。这里罗列了一些糟糕的例子。

####不要通过 `withUnsafeBytes` 返回指针。

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

不要在 withUnsafeBytes(of:) 作用域外调用指针。或许一时可用，但…

####同一时间只绑定一种类型！

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

绝不要将内存同时绑定到两个不相关的类型。这被称为类型双关而Swift不喜欢双关。

Never bind memory to two unrelated types at once. This is called Type Punning and Swift does not like puns. Instead, you can temporarily rebind memory with a method like withMemoryRebound(to:capacity:). Also, the rules say it is illegal to rebind from a trivial type (such as an Int) to a non-trivial type (such as a class). Don’t do it.
####Don’t walk off the end… whoops!

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
在已出现的差一错误中，尤数不安全代码最糟糕。有一务必小心审查，测试你的代码!

###不安全的Swift 示例 1: 压缩（Unsafe Swift Example 1: Compression）

是时候整理之前的知识点，封装一个C API了。Coca框架中包含了一些C 模块，其实现了一些常用的压缩算法。LZ4压缩速度最快，LZ4A压缩比最高，但相对速度较慢，ZLIB相对平衡了时间和压缩比，还有新（开源）LZFSE算法，更好的平衡了空间和压缩速度。

创建一个新的playground，命名为 Compression（压缩）。默认设置即可。然后用下面的代码替换原有内容：

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

The function that does the compression and decompression is perform which is currently stubbed out to return nil. You will add some unsafe code to it shortly.
Next add the following code to the end of the playground:

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

The Compressed structure stores both the compressed data and the algorithm that was used to create it. That makes it less error prone when deciding what decompression algorithm to use.
Next add the following code to the end of the playground:

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

The main entry point is an extension on the Data type. You’ve added a method called compressed(with:) which returns an optional Compressed struct. This method simply calls the static method compress(input:with:) on Compressed.
There is an example usage at the end but it is currently not working. Time to start fixing that!
Scroll back up to the first block of code you entered, and begin the implementation of the perform(_:on:using:workingBufferSize:) function as follows:

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

This converts from your Swift types to the C types required by the compression library, for the compression algorithm and operation to perform.
Next, replace return nil with:

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

This is what is happening here:
1. Allocate a compression_stream and schedule it for deallocation with the defer block.
2. Then, using the pointee property you get the stream and pass it to the compression_stream_init function. The compiler is doing something special here. By using the inout & marker it is taking your compression_stream and turning it into a UnsafeMutablePointer<compression_stream> automatically. (You could have also just passed streamPointer and not needed this special conversion.)
3. Finally, you create a destination buffer that will act as your working buffer.

Finish the perform function by replacing return nil with:

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

This is where the work really happens. And here’s what it’s doing:
1. Create a Data object which is going to contain the output – either the compressed or decompressed data, depending on what operation this is.
2. Set up the source and destination buffers with the pointers you allocated and their sizes.
3. Then you keep calling compression_stream_process as long as it continues to return COMPRESSION_STATUS_OK.
4. The destination buffer is then copied into output that is eventually returned from this function.
5. When the last packet comes in, marked with COMPRESSION_STATUS_END only part of the destination buffer potentially needs to be copied.
In the example usage you can see that the 10,000-element array is compressed down to 153 bytes. Not too shabby.

###Unsafe Swift Example 2: Random Generator

Random numbers are important for many applications from games to machine learning. macOS provides arc4random (A Replacement Call 4 random) that produces great (cryptographically sound) random numbers. Unfortunately this call is not available on Linux. Moreover, arc4random only provides randoms as UInt32. However, the file /dev/urandom provides an unlimited source of good random numbers.
In this section, you will use your new knowledge to read this file and create completely type safe random numbers.

![hexdump](https://koenig-media.raywenderlich.com/uploads/2016/12/hexdump-480x202.png)

Start by creating a new playground, calling it RandomNumbers. Make sure to select the macOS platform this time.
Once you’ve created it, replace the default contents with:

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

The file variable is declared static so only one will exist in the system. You will rely on the system closing it when the process exits. Since it is possible that multiple threads will want random numbers, you need to protect access to it with a serial GCD queue.
The get function is where the work happens. First you create some unallocated storage that is one beyond what you need because fgets is always 0 terminated. Next, you get the data from the file, making sure to do so while operating on the GCD queue. Finally, you copy the data to a standard array by first wrapping it in a UnsafeMutableBufferPointer that can act as a Sequence.
So far this will only (safely) give you an array of Int8 values. Now you’re going to extend that.
Add the following to the end of your playground:

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

This adds a static randomized property to all subtypes of the Integer protocol (see protocol oriented programming for more on this!). You first get the random numbers, and with the bytes of the array that is returned, you rebind (as in C++’s reinterpret_cast) the Int8 values as the type being requested and return a copy. Simples! :]
And that’s it! Random numbers in a safe way, using unsafe Swift under the hood.

### to Go From Here?

Here are the completed playgrounds. There many additional resources you can explore to learn more:
- Swift Evolution 0107: UnsafeRawPointer API gives a detailed overview of the Swift memory model and makes reading the API documents more understandable.
- Swift Evolution 0138: UnsafeRawBufferPointer API talks extensively about working with untyped memory and has links to open source projects that benefit from using them.
- If you are converting unsafe code to Swift 3 check out The Migration Guide. Even if you aren’t migrating, it contains a number of interesting examples.
- Interacting with C APIs will give you insights in how Swift interacts with C.
Mike Ash has an excellent presentation on Exploring Swift Memory Layout.
I hope you have enjoyed this tutorial. If you have questions or experiences you would like to share, I am looking forward to hearing about them in the forums!