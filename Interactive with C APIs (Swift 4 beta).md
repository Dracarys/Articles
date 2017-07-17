# ［翻译］Swift与 C API的交互（Swift 4 beta）

*本文译自《Using Swift with Cocoa and Objective-C》书中 Interacting with C APIs 一章*。
*受限于译者英语水平及翻译经验，译文难免有词不达意，甚至错误的地方，还望不吝赐教，予以指正*

－－－－－－－－－－－－－－－－－－－



作为与 Objective-C 交互的一部分，Swift 对 C 语言的类型和特性也提供了良好的兼容。Swift还提供了相应的交互方式，以便在需要时可以在代码中使用常见的 C 结构模式。

### 基本类型

虽然 Swift 提供了与 C 语言中 char，int，float 和 double 等基本类型等价的类型。但这些类型，如 Int，不能与 Swift 核心类型进行隐式转换。因此除非代码中有明确要求（使用等价的 C 类型），否则都应使用 Int（等 Swift 核心类型）。


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
定义在 C 和 Objective-C 源文件中的全局常量，会自动被 Swift 编译器引入为全局常量。

#### 导入的枚举和结构体

在 Objective-C 中，常量通常用来为属性和函数参数提供一组可选值。Ojbective-C 中那些被 NS\_STRING\_ENUM 或 NS\_EXTENSIBLE\_STRING\_ENUM 宏标注的 typedef 声明，会被 Swift 以普通类型的成员的方式导入。通过 NS\_STRING\_ENUM 声明的枚举不可以在添加额外的值进行扩展，而通过 NS\_EXTENSIBLE\_STRING\_ENUM 声明的枚举，则可以在 Swift 的扩展（extension）中进行扩展。

通过 NS\_STRING\_ENUM 宏声明的一组常量，会被导入为结构体。例如，下面这段在 Objective-C 中 被声明为字符串常量的 TraficLightColor 类型：

``` Objective-C
typedef NSString * TrafficLightColor NS_STRING_ENUM;
TrafficLightColor const TraficLightColorRed;
TrafficLightColor const TraficLightColorYellow;
TrafficLightColor const TraficLightColorGreen;
```

下面是 Swift 导入后结果: 

``` Swift
struct TrafficLightColor: RawRepresentable {//注意区别，Swift 3 中是导入为枚举的
	typealias RawValue = String

	init(rawValue: RawValue)
	var rawValue: RawValue { get }
	static var red: TrafficLightColor { get }
	static var yellow: TrafficLightColor { get }
 	static var green: TrafficLIghtColor { get }
}
```
	
通过 NS\_EXTENSIBLE\_STRING\_ENUM 宏声明的一组可扩展的常量值，同样会被导入为结构体。例如，下面这段在 Objective-C 中被声明为字符串常量的 StateOfMatter 类型：

``` Objective-C
typedef NSString * StateOfMatter NS_EXTENSIBLE_STRING_ENUM;
StateOfMatter const StateOfMatterSolid;
StateOfMatter const StateOfMatterLiquid;
StateOfMatter const StateOfMatterGas;
```

下面是 Swift 导入后结果:

``` Swift
struct StateOfMatter: RawRepresentable {
	typealias RawValue = String
	
	init(_ rawValue: RawValue) //注意该构造器在 Swift 3中是没有的。
	init(rawValue: RawValue)
	var rawValue: RawValue { get }
	
	static var solid: StateOfMatter { get }
	static var liquid: StateOfMatter { get }
	static var gas: StateOfMatter { get }
}
```

以可扩展形式声明的常量，在被导入后，会额外添加一个可忽略参数标签的构造器，以便扩展添加新值。
	
通过 NS\_EXTENSIBLE\_STRING\_ENUM 宏声明的常量，导入后，可在 Swift 代码中扩展添加新值。

``` Swift
extension StateOfMatter {
	static var plasma: StateOfMatter {
		return StateOfMatter(rawValue: "plasma")
	}
}
```
	
###函数

Swift可以把任何声明在 C 头文件中的函数作为全局函数导入。例如，下面的 C 函数声明：

``` C
int product(int multiplier, int multiplicand);
int quotient(int dividend, int devisor, int *remainder);

struct Point2D createPoint2D(float x, float y);
float distance(struct Point2D from, struct Point2D to);
```

下面是 Swift 导入后结果:
	
``` Swift
func product(_ multiplier: Int32, _ multiplicand: Int32) -> Int32
func quotient(_ dividen: Int32, _ devison: Int32, UnsafeMutablePointer<Int32>!) -> Int32

func createPoint2D(_ x: Float, _ y: Float) -> Point2D
func distance(_ from: Point2D, _ to: Point2D) -> Float
```
	
#### 变参函数(Variadic Functions)

在 Swift 中，可以通过 `getValist(\_:)` 或者 `withValist(\_:\_:)` 来调用 C 中诸如 vaspritf 这样的变参函数。 `getValist(\_:)` 函数接收一个包含 CVarArg 值的数组，并返回一个 CVaListPointer，与直接返回不同的是 `withValist(\_:\_:)` 通过接受一个闭包来实现。其结果 CVaListPointer 最后会作为 va\_lsit 参数传递给 C 变参函数。

例如，下面的代码展示了如何在 Swift 中调用 vasprintf 函数：

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
	可选类型指针不能传入 `withVaList(_:invoke:)` 函数，
	可以通过 `Int.init(bitPattern:)` 构造函数，来将可选类型指针转为 Int，
	在所有支持的平台上该指针的调用规则与 C 相同。
	
### 结构体
Swift 可以把任何头文件中声明的 C 结构体引入为 Swift 结构体，引入后的结构体会为每一个原 C 结构体成员生成一个存储型属性和一个逐一成员构造器。如果被引入的成员均有默认值，那么 Swift 同时会生成一个无参数的默认构造器。例如下面这个 C 结构体：

``` C
struct Color {
	float r, g, b;
}:

typedef struct Color Color;
```

下面是与之相应的 Swift 结构体：

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

CoreFoundation 框架中的 C API，大都提供了用于创建、存取、以及修改 C 结构体的函数。可以通过在代码添加CF\_SWIFT\_NAME 宏标记，来让 Swift 将这些 C 函数引入为结构体的成员函数。例如，下面这段C函数声明：

``` C
Color ColorCreateWithCMYK(float c, float m, float y, float k) CF_SWFIT_NAME(Color.init(c:m:y:k:));

float ColorGetHue(Color color) CF_SWIFT_NAME(getter:Color.hue(self:));

void ColorSetHue(Color color, float hue) CF_SWIFT_NAME(setter:Color.hue(self:newvalue));

Color ColorDarkenColor(Color color, flaot amout) CF_SWIFT_NAME(Color.Darken(self:amount:));

extern const Color ColorBondiBlue CF_SWIFT_NAME(Color.boundiBlue);

Color ColorGetCalibrationColor(void) CF_SWIFT_NAME(getter:Color.calibration());

Color ColorSetCalibrationColor(Color color) CF_SWIFT_NAME(setter:Color.calibration(newValue:));
```
下面展示了 Swift 如何将它们以类型成员的方式导入：

``` Swift
extension color {
	init(c: Float, m: Float, y: Float, k: Float)

	var hue: Float { get set }

	func darken(amount: Float) -> Color

	static var bondiBlue: Color

	static var calibration: Color
}
```

传入 CF\_SWIFT\_NAME 宏的参数语法与 `#selector` 表达式相同。CF\_SWIFT\_NAME 宏中的 self，表示接收该方法的实例对象。

	注意
	使用 CF_SWIFT_NAME 宏时不能改变被引入成员函数的参数顺序和数量。
	如果想更Swift点，可以重写一个Swift函数，然后在其内部再调用所需的 C 函数
	（译者：用Swift再做一层封装）

### 枚举
Swift 可以把任何使用 NS\_ENUM 宏标记的 C 枚举引入为 Int 类型的 Swift 枚举。无论是系统框架还是其它代码，引入后的枚举都会自动移除原命名前缀，
例如，下面这个通过 NS\_ENUM 宏声明的 C 枚举:

``` C
typedef NS_ENUM(NSInteger, UITableViewCellStyle){
	UITableViewCellStyleDefault, 
	UITableViewCellStyleValue1, 
	UITableViewCellStyleValue2, 
	UITableViewCellStyleSubtitle
}
```
在 Swift 导入后，会呈现为以下形式：

``` Swift
enum UITableViewCellStyle: Int {
	case 'default'
	case value1
	case value2
	case subtitle
}
```

通过点语法（译者注：.valueName）来引用一个枚举值。

``` Swift
let cellStyle: UITableViewCellStyle = .default
```

	注意：
	Swift 引入的 C 枚举，在构造时即使入参与声明不一致，也不会导致构造失败。
	这么处理是为了与 C 兼容，因为 C 枚举允许任意类型的值，即使这个值没有暴露在头文件中，
	而仅仅是供内部使用。
	
那些使用 NS\_ENUM 或 NS\_OPTIONS 宏声明的 C 枚举会被导入为 Swift 结构体。C 枚举中的每个成员都会被导入为一个与结构体类型相同的只读的全局计算型属性，而非结构体成员属性。

例如，下面这个未使用 NS\_ENUM 宏声明的 C 结构体

``` C
typedef enum {
	MessageDispositionUnread = 0,
	MessageDispositionRead = 1,
	MessageDispositionDeleted = -1
} MessageDisposition;
```

在 Swift 中，它会被导入为以下形式：

``` Swfit
struct MessageDisposition: RawRePresentable, Equatable{}

var MessageDispositionUnread: MessageDisposition { get }
var MessageDispositionRead: MessageDisposition { get }
var MessageDispositionDeleted: MessageDisposition { get }
```
Swift 会自动为导入的 C 枚举类型适配 Equaltable 协议。

#### 选项型枚举（Option sets）

Swift 同样可以把 NS\_OPTIONS 宏标注的 C 选项型枚举导入为 Swift 的选项（Option set）。与先前枚举的引入类似，选项的命名前缀也会被移除。
例如，下面这个在 Objective-C 声明的选项：

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

在 Swift 中会被导入为以下形式：

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
在 Objective-C 中，选项型枚举实际上是整型位掩码。既可以通过位或运算符（｜）来组合选项值，又可以通过位与运算符（&)检查选项值。我们还可以通过常量或表达式来创建选项，空选项用常量零（0）表示。

在 Swift 中，选项是以一个遵从 OptionSet 协议的结构体来实现的，每个选项都是该结构体的一个静态变量。与枚举类似，我们既可以通过字面量数组来创建一个选项，又可以通过点（.）语法获取选项的一个值。一个空的选项既可以通过字面量空数组（［］）创建，也可以通过调用其默认构造函数创建。

	注意
	当引入 NS_OPTIONS 宏标记的 C 枚举时，任何值为 0 的选项成员都会被 Swift 标记为无效，
	因为 Swift 中使用空选项来表示没有选项。

选项与 Swift 中的 Set 集合类型相似，可以通过 `insert(\_:)` 或 `formUnion(\_:)` 函数添加选项值，也可以通过 `remove(\_:)` 或 `substract(\_:)` 函数删除一个选项值，还可以通过 `contains(\_:)` 来查验选项值。

``` Swift
let options: Data.Base64EncodingOptions = [
	.lineLength64Characters,
	.endLineWithLineFeed
]
	
let string = data.base64EncodedString(options:options)
```

#### 联合体（Unions）（译者注：该部分为新增加内容个，Swift 3 是不支持的）

Swift 会把 C 联合体导入为 Swift 的结构体。虽然 Swift 不支持联合体，但是被导入后其使用仍与 C 联合体非常相似。例如，下面这个拥有 isAlive 和 isDead 两个域的联合体 SchroedingersCat:

```C
union SchroedingersCat {
	bool isAlive;
	bool isDead;
}
```

在 Swift 中会将其导入为一下形式：

```Swift
sturct SchroedingsCat {
	var isAlive: Bool { get set }
	var isDead: Bool { get set }

	init(isAlive: Bool)
	init(isDead: Bool)

	init()
}
```

由于 C 中的联合体所有域共享一段内存，所以被 Swift 导入后生成的所有计算型属性也具有相同的内存结构。因此，当改变任何一个该结构体实例的计算型属性时，那么该实例的其它属性也会发生相应改变。

例如下面这个例子，当改变 SchroedingersCat 实例上的 isAlive 计算型属性时，其另一个计算型属性 isDead 也发生来相应的变化：

```Swift
var mittens = SchroedingersCat(isAlive: fasle)

print(mittens.isAlive, mittens.isDead)
// Prints "fasle false"

mittens.isAlive = true
print(mittens.isDead)
// Prints "true"
```

#### 位域（Bit Fields）

Swift 可以把结构体中的位域，例如 Foundation 中的 NSDecimal 类型，导入为计算型属性。在对其进行访问时，Swift 会自动将其值转换为与 Swift 相兼容的类型。

#### 匿名的结构体和联合体域（Union Fields）

(译者注：这里变化很大，且受限译者理解水平，翻译的可能不够准确)

C 中的 Struct 和 union 类型可定义成匿名或者匿名类型，匿名域由具名的相邻结构题或联合体类型组成。

例如，这个名为 Cake 的 C 结构体，它包含一个拥有 layers 和 height 两个相邻域的匿名联合体，以及一个域为 toppings的匿名结构题类型：

``` C
struct Cake {
	union {
		int layers;
		double height;
	}

	struct {
		bool icing;
		bool sprinkles;
	} toppings;
}
```

在 Cake 被导入后，可以这样初始化并调用它：

```Swift
var simpleCake = Cake()
simpleCake.layers = 5
print(simpleCake.toppings.icing)
//	Prints "false"
```

Cake 在被导入后，会生成一个逐一成员构造器，可为其域传自定义值来构造该结构体，例如：

```Swift
let cake = Cake( .init(layers: 2), toppings: .init(icing: true, sprinkles: false))

print("The cake has \(cake.layers) layers")
// Prints "The cake has 2 layers."
print("Does it have sprinkes?", cake.toppings.sprinkles ? "Yes." : "No.")
// Prints "Does it have sprinkles? No."
```

由于 Cake 结构体的第一个域是匿名的，所以它的构造器的第一个参数也没有标签。由于 Cake 结构体的域含有匿名类型，所以使用 .init 构造器，通过类型推断的方式来为结构体的每个匿名域设置初始值。

### 指针

Swift 一直在尽力避免直接访问指针。但仍提供了丰富的指针类型以备不时之需。下表使用 Type 来指代不同语言的相应类型。

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

无类型指针与原始内存（指针）的对照关系表：(译者注：swift 3中是不存在的)

|C Syntax|Swift Syntax|
|:------|:----------|
|const void \*|UnsafeRawPointer\<Type\>|
|void \*|UnsafeMutableRawPointer\<Type\>|

Swift 还提供来用于操作缓存的指针类型，请参见 缓存指针(Buffer Pointers)。

如果 Swift 中没有与 C 指针所指内容相应的类型，例如，一个不完全的结构体（incomplete struct）类型，那么这个指针会被导入为 OpaquePointer。

#### 常量指针

一个接受 UnsafePointer\<Type\> 类型参数的函数，同样可以接受下列类型参数：

- 一个 UnsafePointer\<Type\>，UnsafeMutalbePointer\<Type\>，或 AutoreleasingUnsafeMutablePointer\<Type\> 类型的值，如有必要它会被转换为 UnsafePointer\<Type\> 类型。
- 如果 Type 是 `Int8` 或 `UInt8` ，则可接受一个 String 类型的值。该字符串会自动被转换为一个 UTF8 字符缓存（buffer），然后将指向该缓存（buffer）的指针传入该函数。
- 一个包含一个或多个变量、属性、Type 类型下标引用的 in-out 表达式。表达式会以指向左起首位参数内存地址的指针形式被传入。
- ［Type］（一个含有Type类型元素的数组）,会以指向数组首地址的指针形式传入。

传入函数的指针，仅保证在函数调用期间有效。不要尝试持有或在函数返回后访问该指针。

例如，这样一个函数：

``` Swift
func takesAPointer(_ p: UnsafePointer<Float>) {
	// ...
}
```
可以这样调用它：

``` Swift
var x: Float = 0.0
takesAPointer(&x)
takesAPointer([1.0, 2.0, 3.0])
```

一个接受 UnsafePointer\<Void\> 类型参数的函数，同样可以接受相同操作数的任意 Type 类型的 UnsafePointer\<Type\> 指针。

例如下面这个函数：（译者注：注意无类型指针的变动，由Void变成了Raw）

``` Swift
func takesARawPointer(_ p: UnsafeRawPointer?) {
	// ...
}
```
它可以这样被调用：

``` Swift
var x: Float = 0.0, y: Int = 0
takesARawPointer(&x)
takesARawPointer(&y)
takesARawPointer([1.0, 2.0, 3.0] as [Float])
let intArray = [1, 2, 3]
takesARawPointer(intArray)
```

#### 可变指针（Mutable Pointers）
一个接受 UnsafeMutablePointer\<Type\>类型参数的函数，同样可以接受下列类型参数：

- 一个 UnsafeMutablePointer\<Type\> 类型的值
- 一个含有一个或多个变量、属性、Type类型下标引用的 in-out 表达式，以指向多个值地址的指针的形式被传入。
- 一个包含多个变量、属性、或下标引用的［Type］类型的 in-out表达式，会以指向该数组起始地址的指针形式传入，与此同时其生命周期会被延长，持续于整个函数调用期间。

例如下面这个函数：

``` Swift
func takesAMutablePointer(_ p: UnsafeMutablePointer<Float>) {
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

一个接受 UnsafeMutableRawPointer\<Void\> 类型参数的函数，同样可以接受相同操作数的任意 Type 类型 的 UnsafeMutablePointer\<Type\> 指针。

例如下面这个函数：

``` Swift
//译者注：注意无类型指针的变化。
func takesAMutableRawPointer(_ p: UnsafeMutableRawPointer?) {
	// ...
}
```

它可以这样被调用：

``` Swift
var x: Float = 0.0, y: Int = 0
var a: [Float] = [1.0, 2.0, 3.0], b: [Int] = [1, 2, 3]
takesAMutableRawPointer(&x)
takesAMutableRawPointer(&y)
takesAMutableRawPointer(&a)
takesAMutableRawPointer(&b)
```	

#### 自释放指针（autoreleasing Pointers）

一个接受 AutoreleasingUnsafeMutablePointer\<Type\> 类型参数的函数，同样可以接受下列类型参数：

- 一个 AutoreleasingUnsafeMutablePointer\<Type\> 类型的值
- 一个含有一个或多个变量、属性、Type类型下标引用的 in-out 表达式，它会被按位拷贝到一个非持有的临时缓存，随之指向该缓存的指针会被传入，并且在返回时，缓存中的值会被加载，持有，并赋值到操作数中。

注意：上表 _不包含数组_。

例如下面这个函数：

``` Swift
func takesAnAutoreleasingPointer(_ p: AutoreleasingUnsafeMutablePointer<NSDate?>) {
	// ...
}
```

它可以这样被调用：

``` Swift
var x: NSDate? = nil
AutoreleasingUnsafeMutablePointer(&x)
```	

指针所指的类型不会被转换（bridged）。例如，NSString \*\* 会被 Swift 导入为 AutoreleasingUnsafeMutablePointer\<NSString?\>，而不是 AutoreleasingUnsafeMutablePointer\<String?\>。

#### 函数指针

通过 @convention(c) 标注，Swift 会根据 C 函数指针调用规则将其引入为闭包。例如，一个 `int (x) (void)`类型的 C 函数指针，在 Swift 中会被导入为 `@convertion(c) () -> Int32` 。当调用一个接受函数指针类型参数的函数时，可以直接传入一个顶级的 Swift 函数，一个 closure literal，或者nil。还可以传入一个泛型闭包属性，或者一个闭包参数列表和者闭包体中都没有引用泛型参数的泛型函数。例如，Core Foundation中的CFArrayCreateMutable(\_:\_:\_:)函数。CFArrayCreateMutable(\_:\_:\_:)函数，接受一个初始化为函数指针的CFArrayCallBacks结构体：

``` Swift
func customCopyDescription(_ p: UnsafeRawPointer?) -> Unmanaged<CFString>? {
	// return an Unmanaged<CFString>? value
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

上面的例子中，CFArrayCallBacks的构造函数，把 nil 分别赋值给 retain 和 release 参数，把 `customCopyDescription(_:)` 函数作为参数赋给 copyDescription，并把一个闭包体作为参数赋值 equal。

	注意：
	Only Swift function types with C function reference calling convention may be used for function pointer arguments. Like a C function pointer, a Swift function type with the @convertions(c) attribute does not capture the context of its surrounding scope.

	For more information, see Type Attributes in _The Swift Programming Language (Swift 4)

#### 缓存指针（Buffer Pointers）

缓存指针通常用于访问一个低级的内存区域。例如，使用缓存指针来高效地处理应用与服务器之间的数据交换。

Swift 有一下几种缓存指针类型：

- UnsafeBufferPointer
- UnsafeMutableBufferPointer
- UnsafeRawBufferPointer
- UnsafeMutableRawBufferPointer

类型明确的缓存指针，UnsafeBufferPointer 和 UnsafeMutableBufferPointer 允许你查看或者改变一个相邻的内存块儿。该类型允许你把内存作为一个集合来访问，where each item is an instance of the buffer type's Element generic type parameter.

原始缓存指针类型，UnsafeRawBufferPointer 和 UnsafeMutableRawBufferPointer，允许你把一整块儿的内存作为一个包含 Uint8 值的集合进行查看和修改，在这块内存中，每个值与一个字节的内存相对应。These types let you use low-level programming patterns, such as operating on raw memory without compiler-checked type safety, or switching between several different typed interpretations of the same memory.


#### 空指针
在 Objective-C 中，指针类型的声明可以通过标注 \_Nullable 和 \_Nonnull 来表明是否可以接受空值 nil 或 NULL。在 Swift 中，空指针是通过一个值为 nil 的可选类型指针来实现。透过一个整数内存地址来构造指针，是可失败的。一个非可选的指针类型不能被赋值为 nil。

对应关系如下：

|  Objective-c Syntax   |   Swift Syntax       |
|:----------------------|:---------------------|
|const Type \* \_Nonnull|UnsafePointer\<type\> |
|const Type * \_Nullable|UnsafePointer\<type\>?|
|const Type \* _Null\_unspecified|UnsafePointer\<type\>!|

	注意(译者注：在Swift 4版本中该注意内容以被移除)
	在Swift 3之前，nullable和non-nullable指针均通过一个非可选（non-optional）指针类型实现。
	将现有代码迁移至Swift最新版本时，可能需要手动修改所有通过nil初始化的指针为可选类型。
	
#### 指针运算
当处理未知数据类型时，可能会用到不安全的指针操作。在Swift中，可通过运算符对一个指针的值进行位运算，以此来创建一个指定偏移量的新指针。

``` Swift
let pointer: UnsafePointer<Int8>
let offsetPointer = pointer + 24
// offsetPointer is 24 strides ahead of pointer
// offsetPointer 是一个向前偏移了24位的新指针
```
	注意
	如欲了解更多关于 Swift 是如何计算不同数据类型和值的空间大小的，请参考 数据类型空间计算（Data Type Size Calculation）。


#### 数据类型空间计算（Data Type Size Calculation）（译者注：注意该部分内容变动很大）

在 C 语言中，可以通过 sizeof 和 alignof 操作符来获取任意变量或数据类型的内存占用大小及对齐情况。在 Swift 中可以通过访问 MemoryLayout<T> 的 size, stride 和 alignment 属性来了解 T 类型的相应情况。例如，timeval 结构体在 Darwin 系统上所占的空间大小和步长是 16, 对齐为 8:

```Swift
print(MemoryLayout<timeval>.size)
// Prints "16"
print(MemoryLayout<timeval>.stride)
// Prints "16"
print(MemoryLayout<timeval>.alignment)
// Prints "8"
```

当在 Swift 中调用的 C 函数需要传入类型大小或值大小的时候，这些就派上用场了。例如，setsockopt(\_:\_:\_:\_:\_:)函数，可以通过接受一个 timeval 指针和指针所指值的大小来设置sokect接收超时选项（SO_RCVTIMEO）：

``` Swift
let sokfd = socket(AF_INET, SOCK_STREAM, 0)
var optval = timeval(tv_sec: 30, tv_usec: 0)
let optlen = socklen_t（MemoryLayout<timeval>.size)
if setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &optavl, optlen) == 0 {
	// ...
}
```

欲了解更多信息，请参见 MemoryLayout

### 仅一次的初始化（One-Time Initialization）

在 C 语言中，POSIX 的 `pthread_once()` 函数和 Grand Central Dispatch 中的 `dipatch_once()`、`dispatch_once_f()` 函数都有保证代码仅被初始化一次的机制。在 Swift 中，全局常量和存储型属性即使被多个线程同时交替存取，也能保证仅初始化一次。这是由语言自身特点来实现的, 因此 POSIX 和 Grand Central Dispatch 中相应的 C 函数未在 Swift 开放。

### 预处理命令
Swift 编译器没有预处理程序。相应地，它通过编译属性，条件编译 block，和语言特性来实现相同功能。因此，预处理命令不会被导入到 Swift 中。

### 简单宏命令

在 Swift 中可通过全局常量来代替，在 C 和 Objective-C 中由 #define 定义的常量。例如，#define FADE\_ANNOTATION\_DURATION 0.35，可以在 Swift 中被更好的表示为 let FADE\_ANNOTATION\_DURATION = 0.35。由于宏定义的常量可以被直接映射为 Swift 的全局变量，所以编译器会自动引入那些定在 C 和 Objective—C 源文件中的简单宏定义。

### 复杂宏命令

Swift 没有与 C 或 Objective-C 中的复杂宏命令相应的构功能特性，这里的复杂宏是指那些带有括号，与函数类似，却未用于定义常量的宏。C 和 Objective-C 中的复杂宏命令通常被用来规避类型检查限制，或者充当被大量使用的代码的模板。与此同时宏也让debuging和重构变困难。Swfit中可以通过函数和泛型来更好地达到这一目的。综上，C 和Objective-C中的复杂宏命令在Swift代码中是无效。

### 条件编译代码块（Conditional Compilation Blocks)

Swift 和 Objective-C 通过不同的方式实现了代码的条件编译。Swift 通过_条件编译代码块_来实现。例如，如果通过 swift -D DEBUG_LOGGING 设置 DEBUG_LOGGING 条件编译标识，那么编译器就会引入位于条件代码块中的代码。

``` Swift
#if DEBUG_LOGGING
print("Flag enabled.")
#endif
```

编译条件判断中可以包含 true 和 false 字面值，自定义条件判断标识（通过 -D <#flag#>指定)，和下表所列的平台判断标识。

|   Function    | Valid arguments|
|:--------------|:---------------|
|os()|OSX, iOS, watchOS, tvOS, Linux|
|arch()|x86_64, arm, arm64, i386|
|swift()|>=followed by version number|

	注意
	通过arch(arm)来判断ARM 64设备，不会返回true。当代码以32位iOS模拟器为目标编译时，arch(i386)会返回true。
	
通过逻辑与 && 和逻辑或 || 符号可以混合判断条件，通过逻辑否 ！可以做假条件判断，还可以通过 #elseif 和 #else 来添加条件判断分支，此外在一个选择编译代码块儿中还能嵌套另一个选择编译代码块儿。

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

与 C 语言的预编译不同，Swift 的条件编译代码块儿必须完整且语法正确，这是因为 Swift 代码即使尚未被编译，也会进行语法检查。
特例，如果条件编译代码块儿包含 swift() 判断，那么这个表达式仅在 Swift 版本与判断条件相匹配的时候才会去解析该表达式。这是为了确保旧版编译器不会去尝试解析较新版本的 Swift 语法。










