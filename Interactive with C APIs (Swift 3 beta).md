# ［翻译］Swift与 C API的交互（Swift 3 beta）

_本文是《Using Swift with Cocoa and Objective-C》书中Interacting with C APIs一章的中文翻译。初次翻译，受限于个人知识水平，难免有词不达意，甚至错误的地方，望斧正。_

作为与Objective-C交互的一部分，Swift对 C 语言的类型和特性也提供了良好的兼容。Swift还提供了相应的交互方式，以便在需要时可以在代码中使用常见的 C 结构模式。

### 基本类型

虽然Swift提供了与 C 语言中char，int，float和double等基本类型等价的类型。但这些类型，诸如Int，不能与Swift核心类型进行隐式转换。因此除非代码中有明确要求（使用等价的 C 类型），否则都应使用Int（等Swift核心类型）。


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
定义在 C 和Objective-C源文件中的全局常量，会自动被Swift编译器引入为全局常量。

#### 常量的引入

在Objective-C中，常量通常用来为属性和函数参数提供一组可选值。使用NS\_STRING\_ENUM和NS\_EXTENSIBLE\_STRING\_ENUM宏标注的Ojbective-C的typedef声明，可被Swift以普通类型的成员的方式引入。

表示一组可用值的常量，可以通过添加NS\_STRING\_ENUM宏，来将其引入为枚举。例如，下面这段关于TraficLightColor的Objective-C字符串常量声明：

``` Objective-C
	typedef NSString * TrafficLightColor NS_STRING_ENUM;
	TrafficLightColor const TraficLightColorRed;
	TrafficLightColor const TraficLightColorYellow;
	TrafficLightColor const TraficLightColorGreen;
```

下面展示了Swift如何引入它们:

``` Swift
	enum TrafficLightColor: String {
		case red
		case yellow
		case green
	}
```
	
对于呈现一组可扩展的可用常量值来说，可通过添加NS\_EXTENSIBLE\_STRING\_ENUM宏，来使其以结构体的形式被引入。例如，下面这段关于StateOfMatter的Objective-C字符串常量声明：

``` Objective-C
	typedef NSString * StateOfMatter NS_EXTENSIBLE_STRING_ENUM;
	StateOfMatter const StateOfMatterSolid;
	StateOfMatter const StateOfMatterLiquid;
	StateOfMatter const StateOfMatterGas;
```

下面展示了Swift如何引入它们:

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
	
通过NS\_EXTENSIBLE\_STRING\_ENUM宏被引入的常量，在Swift代码中是可以扩展添加新值的。

``` Swift
	extension StateOfMatter {
		static var plasma: StateOfMatter {
			return StateOfMatter(rawValue: "plasma")
		}
	}
```
	
###函数

Swift可以把任何声明在 C 头文件中的函数作为全局函数引入。例如，下面的 C 函数声明：

``` C
	int product(int multiper, int multiplicand);
	int quotient(int dividend, int devisor, int *remainder);
	
	struct Point2D createPoint2D(float x, float y);
	float distance(struct Point2D from, struct Point2D to);
```

下面展示了Swift如何引入它们:
	
``` Swift
	func product(_ multiplier: Int32, _ multiplicand: Int32) -> Int32
	func quotient(_ dividen: Int32, _ devison: Int32, UnsafeMutablePointer<Int32>!) -> Int32
	
	func createPoint2D(_ x: Float, _ y: Float) -> Point2D
	func distance(_ from: Point2D, _ to: Point2D) -> Float
```
	
#### 可变参数函数(Variadic Functions)
在Swift中，可以通过getValist(\_:)或者withValist(\_:\_:)来调用 C 中诸如vaspritf的可变参数函数. getValist(\_:)函数接收一个包含CVarArg值的数组，并返回一个CVaListPointer，与直接返回不同withValist(\_:\_:)则通过接受一个闭包来实现。返回值CVaListPointer之后会被传递给 C 可变参数函数的va\_lsit参数。

例如，下面的代码展示了如何在Swift中调用vasprintf函数：

``` Swift
	func Swiftprintf(format: String, argument: CVarArg...) -> String? {
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
	
	print(Swiftprintf(format: "√2 ≅ %g", arguments: sqrt(2.0))!)
	// Prints "√2 ≅ 1.41421"
```

	注意
	可选类型指针不能传入withVaList(_:invoke:)函数，
	可以通过Int.init(bitPattern:)构造函数，来将可选类型指针转为Int，
	Which has the same C variadic calling conventions as pointer on all supported platforms(？？？ 尚待完善)。
	
### 结构体
Swift可以把任何头文件中声明的 C 结构体引入为Swift结构体，引入后的结构体会为每一个原 C 结构体成员生成一个存储型属性和一个该结构体的逐一成员构造器。如果被引入的成员均有默认值，那么Swift同时会生成一个无参数的默认构造器。例如下面这个 C 结构体：

``` C
	struct Color {
		float r, g, b;
	}:
	
	typedef struct Color Color;
```

下面是与之相应的Swift结构体：

``` Swift
	public struct Color {
 		var r: Float
 		var g: Float
 		var b: Float
 	
 		init()
 		init(r: Float, g: Float, b: Float)
	 }
```

#### 将函数引入为类型成员

CoreFoundation框架中的 C API，大都提供了用于创建、存取、或者修改 C 结构体的函数。可以通过在代码添加CF\_Swift\_NAME宏，来让Swift将这些 C 函数引入为结构体的成员函数。例如，下面这段C函数声明：

``` C
	Color ColorCreateWithCMYK(float c, float m, float y, float k) CF_SWFIT_NAME(Color.init(c:m:y:k:));

	float ColorGetHue(Color color) CF_Swift_NAME(getter:Color.hue(self:));

	void ColorSetHue(Color color, float hue) CF_Swift_NAME(setter:Color.hue(self:newvalue));

	Color ColorDarkenColor(Color color, flaot amout) CF_Swift_NAME(Color.Darken(self:amount:));

	extern const Color ColorBondiBlue CF_Swift_NAME(Color.boundiBlue);

	Color ColorGetCalibrationColor(void) CF_Swift_NAME(getter:Color.calibration());

	Color ColorSetCalibrationColor(Color color) CF_Swift_NAME(setter:Color.calibration(newValue:));
```
下面展示了Swift如何将它们以类型成员的方式引入：

``` Swift
	extension color {
		init(c: Float, m: Float, y: Float, k: Float)
	
		var hue: Float { get set }
	
		func darken(amount: Float) -> Color
	
		static var bondiBlue: Color
	
		static var calibration: Color
	}
```

传入CF\_Swift\_NAME宏的参数语法与#selector表达式相同。CF\_Swift\_NAME宏中的self，表示接收该方法的实例对象。

	注意
	使用CF_Swift_NAME宏时不能改变被引入成员函数的参数顺序和数量。
	如果想更Swift点，可以重写一个Swift函数，然后在其内部再调用所需的 C 函数
	（译者：用Swift再做一层封装）

### 枚举
Swift可以把任何NS\_ENUM标记的 C 枚举引入为Int类型的Swift枚举。无论是系统框架还是其它代码，引入后的枚举都会自动移除原命名前缀，
例如，下面这个通过NS\_ENUM宏声明的 C 枚举:

``` C
	typedef NS_ENUM(NSInteger, UITableViewCellStyle){
		UITableViewCellStyleDefault, 
		UITableViewCellStyleValue1, 
		UITableViewCellStyleValue2, 
		UITableViewCellStyleSubtitle
	}
```
在Swift中会被引入为如下形式：

``` Swift
	enum UITableViewCellStyle: Int {
		case 'default'
		case value1
		case value2
		case subtitle
	}
```

需要时，可通过._name_的方式来引用一个枚举值。

``` Swift
	let cellStyle: UITableViewCellStyle = .default
```

	注意
	Swift引入的 C 枚举，在构造时即使入参与声明不一致，也不会导致构造失败。
	这么处理是为了与 C 兼容，因为 C 枚举允许任意类型的值，即使这个值没有暴露在头文件中，
	而仅仅是供内部使用。
	
那些未通过NS\_ENUM和NS\_OPTIONS宏声明的 C 枚举会被引入为Swift结构体。C 枚举中的每个成员都会被引入为一个与结构体类型相同的全局只读计算型属性，而非结构体成员属性。

例如，下面这个未通过NS\_ENUM宏而声明的 C 结构体

``` C
	typedef enum {
		MessageDispositionUnread = 0,
		MessageDispositionRead = 1,
		MessageDispositionDeleted = -1
	} MessageDisposition;
```

在Swift中，它会被引入为以下形式：

``` Swfit
	struct MessageDisposition: RawRePresentable, Equatable{}
	
	var MessageDispositionUnread: MessageDisposition { get }
	var MessageDispositionRead: MessageDisposition { get }
	var MessageDispositionDeleted: MessageDisposition { get }
```
Swift会自动为引入的C枚举类型适配Equaltable协议。

#### 选项型枚举（Option sets）
Swift同样可以把NS\_OPTIONS宏标注的 C 选项型枚举引入为Swift的选项（Option set）。与先前枚举的引入类似，选项值的命名前缀也会被移除。
例如，下面这个Objective-C声明的选项：

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

在Swift中会被引入为以下形式：

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
在Objective-C中，选项型枚举实际上是整型位掩码。可通过位或运算符（｜）来组合可选值，也可以通过位与运算符（&)检查选项值。通过常量或表达式来创建选项型枚举，空选项型枚举用常量零（0）表示。
在Swift中，选项是以一个遵从OptionSet协议的结构体来实现，每个选项值都有一个静态变量。与枚举类似，可通过（.）语法获取一个选项值，也可以通过字面量数组来创建一个选项值。一个空的选项既可以通过字面量空数组（［］）来创建，也可以通过其默认构造函数创建。

	注意
	当引入NS_OPTIONS宏标记的 C 枚举时，数值为0的枚举成员会被Swift标记为无效，
	因为Swift中的空选项表示没有任何选择。（？？？不通顺）

选项与Swift中的Set集合类型相似，可以通过insert(\_:)或formUnion(\_:)函数添加选项值，也可以通过remove(\_:)或substract(\_:)函数删除一个选项值，还可以通过contains(\_:)来查验选项值。

``` Swift
	let options: Data.Base64EncodingOptions = [
		.lineLength64Characters,
		.endLineWithLineFeed
	]
	
	let string = data.base64EncodedString(options:options)
```

#### 联合体（Unions）
Swift仅部分支持 C 联合体类型，对于引入的 C 联合体，Swift无法存取不支持的域（fields）。但那些使用联合体作为参数或返回值的 C 和Objective-C API，可以被Swift正确调用。

#### 位域（Bit Fields）
Swift可以把结构体中的位域，诸如Foundation中的NSDecimal类型，引入为计算型存储属性。在对其进行读取时，Swift会自动将其值转换为Swift兼容类型。

#### 匿名的结构体和联合体域（Union Fields）
C 中的Struct和union类型在定义时可以作为域而不命名，Swift是不支持匿名结构体的，所以这些域会被引入为以__Unnamed_fieldName格式命名的嵌套类型。

例如，这个名为Pie的 C 结构体，它包含一个匿名结构体curst域和匿名联合体filling域：

``` C
	struct Pie {
		struct { bool flakey; } crust;
		union { int fruit; int meat; } filling;
	}
```

它们会分别被Swift引入为，一个Pie.__Unamed_crust类型的curst属性和一个Pie.__Unamed_filling类型的filling属性。


### 指针

Swift一直在尽力避免直接访问指针。但仍提供了丰富的指针类型以备不时之需。下表使用Type来指代不同语言的相应类型。

变量、参数、返回值的指针对照关系如下：

|C Syntax|Swift Syntax|
|:------|:----------|
|const Type \*|UnsafePointer\<Type\>|
|Type \*|UnsafeMutablePointer\<Type\>|

类类型指针对照关系如下：

|C Syntax|Swift Syntax|
|:------|:----------|
|Type \* const \*|UnsafePointer\<Type\>|
|Type \* __strong *|UnsafeMutablePointer\<Type\>|
|Type \*\*|AutoreleasingUnsafeMutablePointer\<Type\>|

如果Swift中没有与 C 指针所指内容相应的类型，例如，一个不完全的结构体类型，那么这个指针会被引入为OpaquePointer。

#### 常量指针
一个接受UnsafePointer\<Type\>类型参数的函数，同样可以接受下列类型参数：

- 一个UnsafePointer\<Type\>，UnsafeMutalbePointer\<Type\>，或AutoreleasingUnsafeMutablePointer\<Type\>类型的值，如有必要它会被转换为UnsafePointer\<Type\>类型。
- 如果Type是Int8或UInt8，则可接受一个String类型的值。该字符串会自动被转换为一个UTF8字符缓存，随之指向该缓存的指针被传入函数。
- 一个包含一个或多个变量、属性、Type类型下标引用的in-out表达式。表达式会以指向左起首位参数内存地址的指针形式被传入。
- ［Type］（一个含有Type类型元素的数组）,会以指向数组首地址的指针形式传入。

传入函数的指针，仅保证在函数调用期间有效。不要尝试持有或在函数返回后访问该指针。

例如，这样一个函数：

``` Swift
	func takesAPointer(_ p: UnsafePointer<Float>!) {
		// ...
	}
```
可以这样调用它：

``` Swift
	var x: Float = 0.0
	takesAPointer(&x)
	takesAPointer([1.0, 2.0, 3.0])
```

一个接受UnsafePointer\<Void\>类型参数的函数，可以把任意Type类型操作数以UnsafePointer\<Type\>形式接受。

例如下面这个函数：

``` Swift
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
一个接受UnsafeMutablePointer\<Type\>类型参数的函数，同样可以接受下列类型参数：

- 一个UnsafeMutablePointer\<Type\>类型的值
- 一个含有一个或多个变量、属性、Type类型下标引用的in-out表达式。表达式会以指向左起首位参数内存地址的指针形式被传入。
- inout［Type］的值，会以指向数组首地址的指针形式传入，与此同时其生命周期会被延长，持续于整个函数调用期间。

例如下面这个函数：

``` Swift
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

一个接受UnsafeMutablePointer\<Void\>类型参数的函数，可以把任意Type类型操作数以UnsafeMutablePointer\<Type\>形式接受

例如下面这个函数：

``` Swift
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
一个接受AutoreleasingUnsafeMutablePointer\<Type\>类型参数的函数，同样可以接受下列类型参数：

- 一个AutoreleasingUnsafeMutablePointer\<Type\>类型的值
- 一个含有一个或多个变量、属性、Type类型下标引用的in-out表达式，它会被按位拷贝到一个不持有的临时缓存，随之指向该缓存的指针会被传入，并且在返回时，缓存中的值会被加载，保持，并赋值到操作数中。

注意：上表与之前不同，_不包含数组_。

例如下面这个函数：

``` Swift
	func takesAnAutoreleasingPointer(_ p: AutoreleasingUnsafeMutablePointer<NSDate?>!) {
		// ...
	}
```

它可以这样被调用：

``` Swift
	var x: NSDate? = nil
	AutoreleasingUnsafeMutablePointer(&x)
```	

引入时，指针所指的类型不会被等价转换（bridged）。例如，NSString \*\* 会被Swift引入为AutoreleasingUnsafeMutablePointer\<NSString?\>，而不是AutoreleasingUnsafeMutablePointer\<String?\>。

#### 函数指针
通过@convention(c)标注，Swift会根据 C 函数指针调用规则将其引入为结构体。例如，一个int (x) (void)类型的 C 函数指针，会以 @convertion(c) () -> Int32 的形式引入Swift。当调用一个接受函数指针类型参数的函数时，可以直接传入一个顶级的Swift函数，一个字面量闭包，或者nil。还可以传入一个泛型闭包属性，或者一个闭包参数列表和者闭包体中都没有引用泛型参数的泛型函数。例如，Core Foundation中的CFArrayCreateMutable(\_:\_:\_:)函数。CFArrayCreateMutable(\_:\_:\_:)函数，接受一个初始化为函数指针的CFArrayCallBacks结构体：

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
在Objective-C中，指针类型的声明可以通过标注\_Nullable和\_Nonnull来表明是否可以接受空值nil或NULL。在Swift中，空指针是通过一个值为nil的可选类型指针来实现。透过一个整数内存地址来构造指针，是可失败的。一个非可选的指针类型不能被赋值为nil。

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
当处理未知数据类型时，可能会用到不安全的指针操作。在Swift中，可通过运算符对一个指针的值进行位运算，以此来创建一个指定偏移量的新指针。

``` Swift
	let pointer: UnsafePointer<Int>
	let offsetPointer = pointer + 24
	// offsetPointer 是一个相对向前偏移了24位的新指针
```

#### 数据类型空间计算
在 C 语言中，通过sizeof操作符来获取各种变量或数据类型的空间大小，而Swift则分别通过sizeof(\_:)和sizeofValue(\_:)来获取指特定类型或值所占的内存空间大小。然而与 C 语言中的sizeof不同，Swift中的sizeof(\_:)和sizeofValue(\_:)函数未将因内存对齐而增加的额外空间计算在内。例如，在Darwin中以 C 方式来获取timeval结构体的空间是16字节，而通过Swift获取则是12字节。

``` Swift
	printf(sizeof(timeval.self))
	// Prints "12"
```

类似功能由trideof(\_:)和strideofValue(\_:)函数代替，这两个函数返回的内存空间大小与 C 的sizeof返回相同。

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
	仅有哪些符合 C 函数调用规则的Swift函数类型可以作为函数指针参数。
	与 C 函数指针相同，带有@convertion(c)属性的Swift函数类型不会捕获其周边的代码环境。
	更多内容，请至《Swift编程语言》（Swift 3）中类型属性一章。

### 单次初始化
在 C 语言中，POSIX的pthread_once()函数和 Grand Central Dispatch中的dipatch_once()、dispatch_once_f()函数都有保证代码仅被初始化一次的机制。在Swift中，全局常量和存储型属性即使被多个线程同时交替存取，也能保证仅初始化一次。这是由语言自身特点来实现的。相应的POSIX和Grand Central Dispatch C 函数不能在Swift中调用。

### 预处理命令
Swift没有预处理程序。相应地，它通过编译属性，条件编译block，和语言特性来实现相同功能。因此，预处理命令不会被引入到Swift中。

### 简单宏命令
在Swift中可通过全局常量来代替，在 C 和Objective-C中由#define定义的常量。例如，#define FADE\_ANNOTATION\_DURATION 0.35，可以在Swift中被更好的表示为 let FADE\_ANNOTATION\_DURATION = 0.35。由于宏定义的常量可以被直接映射为Swift的全局变量，所以编译器会自动引入那些定在 C 和Objective—C源文件中的简单宏定义。

### 复杂宏命令
Swift不支持 C 或者Objective-C中的复杂宏命令，这里的复杂宏是指那些带有括号，与函数类似，却未用于定义常量的宏。C 和Objective-C中的复杂宏命令通常被用来规避类型检查限制，或者充当被大量使用的代码的模板。与此同时宏也让debuging和重构变困难。Swfit中可以通过函数和泛型来更好地达到这一目的。综上，C 和Objective-C中的复杂宏命令在Swift代码中是无效。

### 条件编译Block
Swift和Objective-C通过不同的方式实现了条件编译。Swift通过_条件编译block_来实现。例如，

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
	
通过逻辑与 && 和逻辑或 || 符号可以混合判断条件，通过逻辑否 ！可以做假条件判断，还可以通过 #elseif 和 #else 来添加条件判断分支，此外在一个选择编译block中还能嵌套另一个选择编译block。

``` Swift
	#if arch(arm) || arch(arm64)
	#if Swift(>=3.0)
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

与 C 语言的预编译不同，Swift的条件编译block必须完整且语法正确，这是因为Swift代码即使尚未被编译，也会进行语法检查。
特例，如果条件编译block包含swift()判断，那么这个表达式仅在Swift版本与判断条件相匹配的时候才会去解析该表达式。这是为了确保旧版编译器不会去尝试解析较新版本的Swift语法。










