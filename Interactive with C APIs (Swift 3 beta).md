# Swift中与 C API的交互（Swift 3 beta）

_本文是《Using Swift with Cocoa and Objective-C》书中Interactiing with C APIs一章的中文翻译。初次翻译，受限于个人知识水平，难免有词不达意，甚至错误的地方，望斧正。_

作为与Objective-C交互的一部分，Swift对C语言的类型和特性也提供了良好的兼容。Swift还提供了相应的交互方式，以便在需要时可以在代码中使用常见的C结构模式。

### 基本类型

虽然Swift提供了与 C 语言中 char，int，float和double等基本类型相应的类型，但这些类型与Swift核心类型之间不能进行隐式转换。除非代码中有明确需求，否则都应该使用Int而不是int。


|C|Swift|
|:-----|:-----|
|bool|CBool|
|char, signed char|CChar|
|unsigned char|CUnsignedChar|
|short| CShort|
|unsigned short|CUnsignedShort|
|int|CInt|
|unsigned int|CUnsignedInt|
|long|CLong|
|unsigned long|CUnsignedLong|
|long long|CLongLong|
|unsigned long long|UnsignedLongLong|
|wchar_t|CWideChar|
|char16_t|CChart16|
|char32_t|CChart32|
|float|CFloat|
|double|CDouble|

### 全局常量
定义在 C 和Objective－C源文件中的全局常量，会自动被Swift编译器导入为全局常量。

#### 常量的导入

在Objective－C中，常量通常被用来为属性和函数参数提供一组可选值。通过使用NS_STRING_ENUM或者NS_EXTENSIBLE_STRING_ENUM宏标注一个Ojbective－C的typedef声明，来让Swift把它们以普通类型的成员的方式导入。

表示一组可用值的常量，可以通过添加NS_STRING_ENUM宏，来使其导入为枚举。例如，下面这段关于TraficLightColor的Objective-C字符串常量声明：

``` Objective-C
	typedef NSString * TrafficLightColor NS_STRING_ENUM;
	TrafficLightColor const TraficLightColorRed;
	TrafficLightColor const TraficLightColorYellow;
	TrafficLightColor const TraficLightColorGreen;
```

下面展示了SWift是如何导入的这段代码:

``` Swift
	enum TrafficLightColor: String {
		case red
		case yellow
		case green
	}
```
	
对于呈现一组可扩展的可用常量值来说，可通过添加NS_EXTENSIBLE_STRING_ENUM宏标注，来使其以结构体的形式被导入。例如，下面这段关于StateOfMatter的Objective-C字符串常量声明：

``` Objective-C
	typedef NSString * StateOfMatter NS_EXTENSIBLE_STRING_ENUM;
	StateOfMatter const StateOfMatterSolid;
	StateOfMatter const StateOfMatterLiquid;
	StateOfMatter const StateOfMatterGas;
```

下面展示了SWift是如何导入的这段代码:

``` Swift
	struct StateOfMatter: RawRepresentable {
		typealias RawValue = String
		
		init(rawValue: RawValue)
		var rawValue: RawValue { get }
		
		static var solid: StateOfMatter { get }
		static var liquid: StateOfMatter { get }
		static var gas: StateOfMatter { get }
	}
```
	
通过NS_EXTENSIBLE_STRING_ENUM宏被导入的常量，在Swift代码中可以扩展添加新值。

``` Swift
	extension StateOfMatter {
		static var plasma: StateOfMatter {
			return StateOfMatter(rawValue: "plasma")
		}
	}
```
	
###函数

Swift可以把任何声明在C头文件中的函数作为全局函数导入。例如，下面的 C 函数声明：

``` C
	int product(int multiper, int multiplicand);
	int quotient(int dividend, int devisor, int *remainder);
	
	struct Point2D createPoint2D(float x, float y);
	float distance(struct Point2D from, struct Point2D to);
```

下面展示了SWift是如何导入的这段代码:
	
``` Swift
	func product(_ multiplier: Int32, _ multiplicand: Int32) -> Int32
	func quotient(_ dividen: Int32, _ devison: Int32, UnsafeMutablePointer<Int32>!) -> Int32
	
	func createPoint2D(_ x: Float, _ y: Float) -> Point2D
	func distance(_ from: Point2D, _ to: Point2D) -> Float
```
	
#### 可变参数函数(Variadic Functions)
在Swift中，可以通过getValist(\_:)或者withValist(\_:_:)来调用 C 中诸如vaspritf的可变参数函数. getValist(\_:)函数通过接收一个CVarArg值的数组，来返回一个CVaListPointer，相对于直接反回withValist(\_:\_:)则通过接受一个闭包来实现。接下来返回值CVaListPointer在传递给 C 可变参数函数的va_lsit参数。

例如，下面的代码展示了如何在Swift中调用vasprintf函数：

``` Swift
	func swiftprintf(format: String, argument: CVarArg...) -> String? {
		return withValist(arguments) { va_list in
			var buffer: UnsafeMutablePointer<Int8>? = nil
			return format.withCstring { CString in 
				guard vasprintf(&buffer, CString, va_list) != 0 else {
					return nil
				}
				
				return String(validatingUTF8: buffer!)
			}
		}
	}
	
	print(swiftprintf(format: "√2 ≅ %g", arguments: sqrt(2.0))!)
	// Prints "√2 ≅ 1.41421"
```

	注意
	可选类型指针不能传入withVaList(_:invoke:)函数，
	可以通过Int.init(bitPattern:)构造函数，来将可选类型指针转为Int，
	Which has the same C variadic calling conventions as pointer on all supported platforms(？？？ 尚待完善)。
	
### 结构体
Swift可以把任何头文件中声明的C结构体导入为Swift结构体，导入后的结构体会为每一个原C结构体成员生成一个存储型属性和一个逐一成员构造器。如果被导入的成员均有默认值，那么Swift同时会生成一个无参数的默认构造器。例如下面这个C结构体：

``` C
	struct Color {
		float r, g, b;
	}:
	
	typedef struct Color Color;
```

下面是相应的Swift结构体：

``` Swift
	public struct Color {
 		var r: Float
 		var g: Float
 		var b: Float
 	
 		init()
 		init(r: Float, g: Float, b: Float)
	 }
```

#### 导入函数为类型成员

例如CoreFoundation框架中的C API，大都提供了用于创建、存取、或者修改C结构体的函数。你可以通过在代码引入CF_SWIFT_NAME宏，来让Swift把这些C函数导入为结构体的成员函数。例如，下面这段C函数声明：

``` C
	Color ColorCreateWithCMYK(float c, float m, float y, float k) CF_SWFIT_NAME(Color.init(c:m:y:k:));

	float ColorGetHue(Color color) CF_SWIFT_NAME(getter:Color.hue(self:));

	void ColorSetHue(Color color, float hue) CF_SWIFT_NAME(setter:Color.hue(self:newvalue));

	Color ColorDarkenColor(Color color, flaot amout) CF_SWIFT_NAME(Color.Darken(self:amount:));

	extern const Color ColorBondiBlue CF_SWIFT_NAME(Color.boundiBlue);

	Color ColorGetCalibrationColor(void) CF_SWIFT_NAME(getter:Color.calibration());

	Color ColorSetCalibrationColor(Color color) CF_SWIFT_NAME(setter:Color.calibration(newValue:));
```
下面是SWift如何向他们导入为类型成员：

``` Swift
	extension color {
		init(c: Float, m: Float, y: Float, k: Float)
	
		var hue: Float { get set }
	
		func darken(amount: Float) -> Color
	
		static var bondiBlue: Color
	
		static var calibration: Color
	}
```

CF_SWIFT_NAME宏的参数与#selector相同。CF_SWIFT_NAME宏中用到的self，表示接收该方法的实例对象。

	注意
	使用CF_SWIFT_NAME宏时不能改变被导入成员函数的参数个数和顺序。
	如果要更Swift点，可以重写一个Swift函数，然后在其内部调用所需C函数（译者：用swift再做一层封装）

### 枚举
Swift可以把任何NS_ENUM标记的C枚举类型导入为以Int为值类型的SWift枚举。无论是系统框架还是其它代码，导入后的枚举都会自动移除原命名前缀，
例如，下面这个通过NS_ENUM宏声明的C枚举:

``` C
	typedef NS_ENUM(NSInteger, UITableViewCellStyle){
		UITableViewCellStyleDefault, 
		UITableViewCellStyleValue1, 
		UITableViewCellStyleValue2, 
		UITableViewCellStyleSubtitle
	}
```
在Swift中会被导入为如下形式：

``` Swift
	enum UITableViewCellStyle: Int {
		case 'default'
		case value1
		case value2
		case subtitle
	}
```
通过.name方式来引用一个枚举值。

``` Swift
	let cellStyle: UITableViewCellStyle = .default
```

	注意
	Swift导入的 C 枚举，在构造时即使入参与声明不一致，也不会导致构造失败。这么处理是为了与C兼容，
	因为C枚举允许任意类型值，即使这个值没有暴露在头文件中而仅仅供内部使用。
	
那些未通过NS_ENUM和NS_OPTIONS宏声明的C枚举会被导入为SWift结构体。C枚举中的每个成员都会被导入为一个与结构体类型相同的全局只读计算型属性，非结构体成员属性。

例如，下面这个非通过NS_ENUM宏声明的C结构体

``` C
	typedef enum {
		MessageDispositionUnread = 0,
		MessageDispositionRead = 1,
		MessageDispositionDeleted = -1
	} MessageDisposition;
```

在SWift中，它会被导入为以下形式：

``` Swfit
	struct MessageDisposition: RawRePresentable, Equatable{}
	
	var MessageDispositionUnread: MessageDisposition { get }
	var MessageDispositionRead: MessageDisposition { get }
	var MessageDispositionDeleted: MessageDisposition { get }
```
Swift会自动为导入的C枚举类型适配Equaltable协议。

####可选集合
Swift同样可以把NS_OPTIONS宏标记的C枚举导入为SWift可选集合（Option set）。与枚举类型的导入类似，可选值的命名前缀也会被移除。
例如，下面这个Objective－C声明的可选类型：

``` Objective-C
	typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing){
		UIViewAutoresizingNone = 0,
		UIViewAutoresizingFlexibleLeftMargin = 1 << 0,
		UIViewAutoresizingFlexibleWidth = 1 << 1, 
		UIViewAutoresizingFlexibleRightMargin = 1 << 2,
		UIViewAutoresizingFlexibleTopMargin = 1 << 3,
		UIViewAutoresizingFlexibleHeight = 1 << 4,
		UIViewAutoresizingFlexibleBottomMargin = 1 << 5
	};
```

在SWift中会被导入为以下形式：

``` Swift
	public struct UIViewAutoresizing : OptionSet {
		public init(rawValue: UInt)
		
		public static var flexibleLeftMargin: UIViewAutoresizing { get }
		public static var flexibleWidth: UIViewAutoresizing { get }
		public static var flexibleRightMargin: UIViewAutoresizing { get }
		public static var flexibleTopMargin: UIViewAutoresizing { get }
		public static var flexibleHeight: UIViewAutoresizing { get }
		public static var flexibleBottomMargin: UIViewAutoresizing { get }
	}
```
在Objective－C中，可选集合实际上是整型位掩码。可以通过位或（｜）来组合可选值，也可以通过位与（&)检查（？？？，不准确）可选值。可用通通过常量或表达式来创建可选集合，空集合用常量零（0）来表示。
在Swift中，可选集合是一个遵从了OptionSet协议的结构体，每个可选值都有一个静态变量。与枚举类似，支持字面量数组创建和点（.）语法读取。一个空的可选集合既可以通过字面量数组（［］）来创建，也可以通过其默认构造函数创建。

	注意
	当导入NS_OPTIONS宏标记的 C 枚举时，数值为0的枚举成员会被Swift标记为无效，
	因为Swift中的空可选集合表示没有任何选择。（？？？不通顺）

可选集合与Swift中的Set类型相似，可以通过insert(\_:)或formUnion(\_:)函数添加可选值，也可以通过remove(\_:)或substract(\_:)函数删除一个可选值，还可以通过contains(\_:)来查验可选值。

``` Swift
	let options: Data.Base64EncodingOptions = [
		.lineLength64Characters,
		.endLineWithLineFeed
	]
	
	let string = data.base64EncodedString(options:options)
```

#### 联合体（Unions）
Swift近部分支持C联合体类型，对于导入的C联合体，Swift无法存取不支持的域（fields）。但那些含有联合体参数或返回值的C和Objective－C API，则可以被Swift正确调用。

#### 位域（Bit Fields）
Swift可以把结构体中的位域，诸如Foundation中的NSDecimal类型，到入为计算型存储属性。在对其进行读取时，Swift会自动将其值转换为Swift兼容类型。

#### 匿名的结构体和联合体域（Union Fields）
C中的Struct和union类型在定义时可以作为域而不命名，Swift是不支持匿名结构体的，所以这些域会被导入为__Unnamed_fieldName命名格式的嵌套类型。

例如，这个名为Pie的 C 结构体，它包含一个匿名结构体curst域和匿名联合体filling域：

``` C
	struct Pie {
		struct { bool flakey; } crust;
		union { int fruit; int meat; } filling;
	}
```

它们会分别被SWift导入为，一个Pie.__Unamed_crust类型的curst属性和一个Pie.__Unamed_filling类型的filling属性。


### 指针

Swift一直在尽力避免直接存取指针。但仍提供了丰富的指针类型以备不时之需。下表使用Type来指代不同语言的对应类型。

变量、参数、返回值的指针对照关系如下：

|C Syntax|Swift Syntax|
|:------|:----------|
|const Type \*|UnsafePointer<Type>|
|Type \*|UnsafeMutablePointer<Type|

类指针对照关系如下：

|C Syntax|Swift Syntax|
|:------|:----------|
|Type \* const \*|UnsafePointer<Type>|
|Type \* __strong *|UnsafeMutablePointer<Type|
|Type \*\*|AutoreleasingUnsafeMutablePointer<Type|

如果Swift中没有C指针所指内容的相应类型，例如一个不完整的结构体类型，那么这个指针会被导入为一个OpaquePointer。

#### 常量指针
接受UnsafePointer<Type>类型参数的函数，同样可以接受：

- UnsafePointer<Type>，UnsafeMutalbePointer<Type>，或AutoreleasingUnsafeMutablePointer<Type>类型的值，如有必要它会转换为UnsafePointer<Type>类型
- 如果Type是Int8或UInt8，那么可以接受String类型。自动把该字符串转换为一个UTF8字符缓存，之后将指向该缓存的指针传递给函数。
- 一个含有一个或多个变量、属性、Type类型下标引用的in-out表达式。它会以指向左起首参数地址的指针被传递。（？？？不准确）
- ［Type］（含有Type类型元素的数组）,会以指向数组首位的指针被传递。

传入函数的指针，仅保证在函数调用期间有效。不要尝试持有或在函数返回后存取指针。

例如这样一个函数：

``` SWift
	func takesAPointer(_ p: UnsafePointer<Float>!) {
		// ...
	}
```
我们可以这样调用它：

``` Swift
	var x: Float = 0.0
	takesAPointer(&x)
	takesAPointer([1.0, 2.0, 3.0])
```

一个接受UnsafePointer<Void>类型参数的函数，可以接受指向任意Type的UnsafePointer<Type>类型的指针。

例如下面这个函数：
``` SWift
	func takesAVoidPointer(_ p: UnsafePointer<Void>!) {
		// ...
	}
```
它可以这样被调用：
``` Swift
	var x: Float = 0.0, y: Int = 0
	takesAVoidPointer(&x)
	takesAVoidPointer(&y)
	takesAVoidPointer([1.0, 2.0, 3.0] as [Float])
	let intArray = [1, 2, 3]
	takesAVoidPointer(intArray)
```

#### 可变指针（Mutable Pointers）
接受UnsafeMutablePointer<Type>类型参数的函数，同样可以接受：

- 一个UnsafeMutablePointer<Type>类型的值
- 一个含有一个或多个变量、属性、Type类型下标引用的in-out表达式。它会以指向左起首参数地址的指针被传递。（？？？不准确）
- inout［Type］的值,会以指向数组首位的指针被传递，其生命周期会持续整个调用期间。

例如下面这个函数：

``` SWift
	func takesAMutablePointer(_ p: UnsafeMutablePointer<Float>!) {
		// ...
	}
```

它可以这样被调用：

``` Swift
	var x: Float = 0.0
	var a: [Float] = [1.0, 2.0, 3.0]
	takesAMutablePointer(&x)
	takesAMutablePointer(&a)
```	

一个接受UnsafeMutablePointer<Void>类型参数的函数，可以接受指向任意Type的UnsafeMutablePointer<Type>类型的指针。

例如下面这个函数：

``` SWift
	func takesAMutableVoidPointer(_ p: UnsafeMutablePointer<Void>!) {
		// ...
	}
```

它可以这样被调用：

``` Swift
	var x: Float = 0.0, y: Int = 0
	var a: [Float] = [1.0, 2.0, 3.0], b: [Int] = [1, 2, 3]
	takesAMutableVoidPointer(&x)
	takesAMutableVoidPointer(&y)
	takesAMutableVoidPointer(&a)
	takesAMutableVoidPointer(&b)
```	

#### 自释放指针（autoreleasing Pointers）
接受AutoreleasingUnsafeMutablePointer<Type>类型参数的函数，同样可以接受：

- 一个AutoreleasingUnsafeMutablePointer<Type>类型的值
- 一个含有一个或多个变量、属性、Type类型下标引用的in-out表达式，它会被按位拷贝到一个非持有的临时缓存。
- An in-out expression that contains a mutable variable, property, or subscript reference of type Type, which is copied bitwise into a temporary nonowning buffer. The address of that buffer is passed to the callee, and on return, the value in the buffer is loaded, retained, and reassigned into the operand.

注意这里不包含数组。

例如下面这个函数：

``` SWift
	func takesAnAutoreleasingPointer(_ p: AutoreleasingUnsafeMutablePointer<NSDate?>!) {
		// ...
	}
```

它可以这样被调用：

``` Swift
	var x: NSDate? = nil
	AutoreleasingUnsafeMutablePointer(&x)
```	

指针所指的类型不会被桥接。例如，NSString \*\* 会被SWift导入为AutoreleasingUnsafeMutablePointer<NSString?>，而不是AutoreleasingUnsafeMutablePointer<String?>。

#### 函数指针
通过@convention(c)关键字标示来C函数指针调用转换作为闭包导入到SWift中（？？？不通顺）。例如，一个int (x) (void)类型的C函数指针，可以以@convertion(c) () -> Int32 形式导入。当调用你个接受函数指针参数的函数时，可以直接传入SWift函数，闭包或nil。You can also pass a closure property of a generic type or a generic method as long as no generic type parameters are reference in the closure's argument lis or body. 例如，Core Foundation中的CFArrayCreateMutable(\_:\_:\_:)函数。CFArrayCreateMutable(\_:\_:\_:)函数，接受一个初始化为函数指针的CFArrayCallBacks结构体：

``` Swift
	func customCopyDescription(_ p: UnsafePointer<Void>!) -> Unmanaged<CFString>! {
		// ...
	}
	
 	var callBacks = CFArrayCallBacks(
 		version: 0,
 		retain: nil,
 		release: nil,
 		copyDescription: customCopyDescription,
 		equal: { (p1, p2) -> DarwinBoolean in 
 			// return Bool value
 		}
 	)
 	
 	var mutableArray = CFArrayCreateMutable(nil, 0, &callbacks)
```	

上面的例子中，CFArrayCallBacks在初始化时，分别把nil赋值给retain和release，把customCopyDescription(_:)函数作为参数赋给copyDescription，并把一个闭包体作为参数赋值equal。

#### 空指针
在Objective－C中，指针类型的声明可以通过_Nullable和_Nonnull表示表明是否可以接受空值nil或NULL。在SWift中，空指针通过nil值或这个可选指针类型来实现。通过接受一个表示内存地址的整型初始化指针，是可失败的。一个非可选的指针类型不能被赋值为nil。

对应关系如下：

|  Objective-c Syntax   |   Swift Syntax       |
|:----------------------|:---------------------|
|const Type \* \_Nonnull|UnsafePointer\<type\> |
|const Type * \_Nullable|UnsafePointer\<type\>?|
|const Type \* _Null\_unspecified|UnsafePointer\<type\>!|

	注意
	在Swift 3之前，nullable和non-nullable指针均通过一个非可选（non-optional）指针类型实现。
	将现有代码迁移至Swift最新版本时，可能需要手动修改所有通过nil初始化的指针为可选类型。
	
#### 指针运算
当处理未知数据类型时，可能会涉及不安全的指针操作。在Swift中，可以通过对一个指针的值进行位运算来创建一个指定偏移量的新指针。

``` SWift
	let pointer: UnsafePointer<Int>
	let offsetPointer = pointer + 24
	// offsetPointer 是一个相对向前偏移了24位的新指针
```

	注意
	关于Swift中如何对取类型和值的内存空间大小，请转至_数据类型空间计算_(Ddata Type Size Calculation)部分。

#### 类型空间计算
在C语言中，通过sizeof来获取各种变量或数据类型的空间大小，在SWift中，通过sizeof(\_:)来获取指定类型的空间大小，或通过 sizeofValue(\_:)来获取指定值所占空间大小。然而与C语言中的sizeof不同，Swift中的sizeof(\_:)和sizeofValue(\_:)函数未将因内存对齐而增加的额外空间计算在内。例如，在Darwin中通过C来获取timeval结构体的空间是16字节，而通过SWift获取则是12字节。

``` Swift
	printf(sizeof(timeval.self))
	// Prints "12"
```

可以trideof(\_:) strideofValue(\_:)函数来代替，这两个函数返回的内存空间大小与C的sizeof返回想同。

``` Swift
	printf(strideof(timeval.self))
	// Prints "16"
```

例如，setsockopt(\_:\_:\_:\_:\_:)函数，可以通过接受一个timeval指针和指针所指值的大小来设置sokect接收超时时间，这就需要用strideof()来计算值的长度：

``` Swift
	let sokfd = socket(AF_INET, SOCK_STREAM, 0)
	var optval = timeval(tv_sec: 30, tv_usec: 0)
	let optlen = socklen_t(stridof(timeval.self))
	if setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &optavl, optlen) == 0 {
		// ...
	}
```

	注意
	Only Swfit function types with C function reference calling convertion may be user for function pointer arguments. Like a C funcion pointer, a Swift function type with the @convertion(c) attribute does not capture the context of its surrounding scope.
	For more information, see Type Attributes in _The SWift Programming Language (Swift 3).

### 单次初始化
在C语言中，POSIX的pthread_once()函数和 Grand Central Dispatch中的dipatch_once()、dispatch_once_f()函数都可以保证代码仅被初始化一次（？？？这里不准确）。在SWift中，全局常量和存储型属性即使被多个线程同时交替存取，也能保证仅初始化一次。这是有语言自身特点来实现的。相应功能的POSIX和Grand Central Dispatch C函数不能在Swift中调用。

### 预处理命令
Swift没有预处理程序。相应地，它通过编译属性，条件编译block，和语言特性来实现相同功能。因此，预处理命令不会被导入到Swift中。

### 简单宏命令
在Swift中通过使用全局常量来代替，在C和Objective－C中由#define定义的常量。例如，#define FADE_ANNOTATION_DURATION 0.35常量，可以在SWift中被更好的表示为 let FADE_ANNOTATION_DURATION = 0.35。由于简单常量宏定义可以被直接映射为SWift的全局变量，所以编译器会自动导入那些定在C和Objective—C源文件中的简单宏定义。

### 复杂宏命令
Swift不支持C或者Objective－C中的复杂宏命令，这里的复杂宏是指那些带有括号，与函数类似，却未定义常量的宏。C和Objective－C中的复杂宏命令通常被用来规避类型检查限制，或者充当大量使用的代码模板。与此同时宏也让debuging和重构变困难。Swfit中可以通过函数和generics不必妥协地达到相同目的。综上，C和Objective－C中底复杂宏命令在Swift代码中是无效。

### 条件编译Block
Swift和Objective－C通过不同的方式实现了条件编译。SWift通过_条件编译Block_来实现。例如，

``` Swift
	#if DEBUG_LOGGING
	print("Flag enabled.")
	#endif
```

编译条件判断中可以包含true和false字面值，自定义条件判断flag（通过 -D <#flag#>指定)，和下表所列的平台判断方法。

|   Function    | Valid arguments|
|:--------------|:---------------|
|os()|OSX, iOS, watchOS, tvOS, Linux|
|arch()|x86_64, arm, arm64, i386|
|swift()|>=followed by version number|

	注意
	通过arch(arm)来判断ARM 64设备，不会返回true。当代码以32位iOS模拟器为目标编译时，arch(i386)会返回true。
	
通过逻辑与 && 和逻辑或 || 符号可以混合判断条件，通过逻辑否 ！可以做假条件判断，还可以通过 #elseif 和 #else 来添加条件判断分支，此外在一个选择编译Block中还能嵌套另一个Block。

``` Swift
	#if arch(arm) || arch(arm64)
	#if swift(>=3.0)
	print("Using Swift 3 ARM code")
		#else
		print("Using Swift 2.2 ARM code")
	#endif
	#elseif arch(x86_64)
	print("Using 64-bit x86 code.")
		#else
		print("Using general code.")
	#endif
```

与 C 语言的预编译不同，Swift的条件编译block必须完整且语法正确，这是因为SWift代码即使尚未被编译，也会进行语法检查。
特例，如果条件编译block包含 swift()  判断，那么这个表达式仅在 Swift 版本与判断条件相匹配的时候才会去解析该表达式。这是为了确保旧版编译器不会去尝试解析由较新版本的Swift语法。










