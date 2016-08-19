# Swift中的模式匹配

*本文翻译自 [Pattern Matching in Swift](https://www.raywenderlich.com/134844/pattern-matching-in-swift)，由 [Cosmin Pupaza](https://www.raywenderlich.com/u/shogunkaramazov) 发表于[Raywenderlich](https://www.raywenderlich.com)*。

*受限于译者英语水平及翻译经验，译文容难免有词不达意，甚至错误的地方，还望不吝赐教，予以指正*。

对任何编程语言来说，模式匹配都是其最强大的特性之一，它允许你制定数值间匹配的规则，可使代码变得更加简洁和灵活。

Apple 在 Swift 中提供了模式匹配，今天我们就一起来领略Swift 中的模式匹配技术。

本文涉及以下模式：

- 元组模式（Tuple pattern）
- 类型转换模式（Type-casting patterns）
- 通配符模式（Wildcard pattern）
- 可选类型模式（Optional pattern）
- 枚举转换模式（Enumeration case pattern）
- 表达式模式（Expression pattern）

为了展示如何运用模式匹配，为你设计了如下场景：作为[raywenderlich.com](https://www.raywenderlich.com/)的主编，你需要通过模式匹配来排定教程并按照排定好的日程将其发布到网站上。

注意：该教程需要 Xcode 8 和 Swift 3，并假定您已具备 Swift 开发的基础知识。如果才开始接触Swift，请访问我们为您提供的其它[Swit教程](https://www.raywenderlich.com/category/swift)。
	
### 开始

欢迎光临，代主编！您今天的主要任务是改进那些即将发布到网站上的教程排定日程。首先下载[起始playground](https://cdn4.raywenderlich.com/wp-content/uploads/2016/07/starter-project.playground-2.zip)并打开.

playground包含两部分：
- `random_uniform(value:)`函数，会生成一个介于0和给定值之间的随机数，通过它来随机地为教程排定日期。
- 余下的代码，通过解析**tutorials.json**文件，返回了一个包含字典元素的数组，接下来将通过该数组所包含信息对教程进行排期。

注意：欲了解更多在Swift中如何解析JSON的知识，请访问[教程](https://www.raywenderlich.com/120442/swift-json-tutorial)

虽然不需要了解解析的具体过程，但还是清楚其数据结构，在 playground 的 **Resources** 文件夹中找到并打开 **tutorials.json** 文件。可以看到每个待排定的教程均包含两个属性：title 和 day。每个排定的教程的 day 属性都会赋一个介于1（周一）到5（周五）的值，暂不排定的教程则设为 nil。

本想一周内每天一篇教程，但是查阅完教程安排表后发现，存在一天两篇教程的情况。你需要修复该问题，此外，还需对教程进行特定排序。该怎么解决这些问题呢？如果你能想到“用模式！”，那么你就算想对了。

### 模式匹配类型

让我们先来了解下本文即将用到的几种模式：

- 元组模式（Tuple patterns）用于匹配正确的元组类型值
- 类型转换模式（Type-casting patterns）允许你去转换或匹配类型
- 通配符模式（Wildcard pattern）用于匹配或忽略任意值和类型
- 可选类型模式（Optional pattern）用于匹配可选值
- 枚举模式（Enumeration case pattern）用于匹配已有枚举类型
- 表达式模式（Expression pattern）允许你通过一个指定的表达式去比较指定的值

### 元组模式（Tuple Pattern）

首先，通过元组模式创建一个包含所有教程的数组。在playground的最后添加如下代码：

``` Swift
enum Day: Int {
  case monday, tuesday, wednesday, thursday, friday, saturday, sunday
}
```
这里一周的划分情况创建了一个原始类型为 **Int**的枚举，这样便可以通过赋 0 到 6 之间的任意值来表示星期一到星期日。在枚举的后面添加如下代码：

``` SWift
class Tutorial {
 
  let title: String
  var day: Day?
 
  init(title: String, day: Day? = nil) {
    self.title = title
    self.day = day
  }
}
```
接下来通过一个数组来保存所有的 tutorial：

``` Swift
var tutorials: [Tutorial] = []
```

紧挨着刚才的代码，添加如下内容，将包含字典的数组转换为包含 tutorial 对象的数组：

``` Swift
for dictionary in json {
  var currentTitle = ""
  var currentDay: Day? = nil
 
  for (key, value) in dictionary {
    // todo: extract the information from the dictionary
  }
 
  let currentTutorial = Tutorial(title: currentTitle, day: currentDay)
  tutorials.append(currentTutorial)
}
```

这里通过`for-in`语句来遍历整个 json 数组，同时又通过元组去遍历字典中的键值对。这里展示了元组模式的应用。

已将 tutorial 添加到了数组中，但它现在还是空的，下一小节将通过类型转换模式来设置其属性。

### 类型转换模式（Type-Casting Patterns）
为了取出字典的信息，你将用到类型转换模式。在`for (key, value) in dictionary`循环体中，用如下代码替换掉注释：

``` Swift
// 1
switch (key, value) {
  // 2
  case ("title", is String):
    currentTitle = value as! String
  // 3
  case ("day", let dayString as String):
    if let dayInt = Int(dayString), let day = Day(rawValue: dayInt - 1) {
      currentDay = day
  }
  // 4
  default:
    break
}
```

代码说明:

1. 通过 switch 匹配元组中的键值对。
2. 测试 tutorial 的 title 是否为 String，如果是，则通过类型转换模式将其转换为 String 类型（译者注：这里有点String转String，但别迷糊，注意开始的 json 数组类型）。
3. 通过类型转换来测试day属性。如果测试通过，先将其转为整型，然后在通过 Day 枚举的可失败构造器 `init(rawValue:)`，将其构造成枚举 day。减 1 是因为 json 中的日期是从 1 开始， 而枚举是从 0 开始的。
4. switch语句要完整，这里添加一个 defalut 分支，并通过 break 语句来退出 switch。 

在playground的最后添加如下代码，以将 tutorial 信息输出到控制台。

``` Swift
print(tutorials)
```

如你所见，现在数组中的每个 tutorial 都拥有了自己的 title 和 day 属性。 前期准备已毕，接下来就该解决教程排定的问题：一周中每天仅安排一个教程。

### 通配符模式（Wildcard Pattern）

要通过通配符模式来排定教程，首先得取消每个教程现有的日期安排。在 playground 的最后添如下代码:

``` Swift
tutorials.forEach { $0.day = nil }
```

这里通过将 tutorial 的 day 设置为 nil 来取消所有教程安排。为了重新排定所有教程，在 palayground 的最后添加如下代码:

``` Swift
// 1 
let days = (0...6).map { Day(rawValue: $0)! }
// 2
let randomDays = days.sorted { _ in random_uniform(value: 2) == 0 }
// 3
(0...6).forEach { tutorials[$0].day = randomDays[$0] }
```

代码说明:

1. 首先创建一个 days 数组，其中无重复日期。
2. 对数组进行“排序”. 通过`random_uniform(value:)`方法对数组中两个相邻元素进行随机排序。由于不需要，所以闭包中使用下划线来忽略参数。虽然还有其他更高效的数学方法来随机打乱一个数组，但这里更好地展示了通配符模式的应用。
3. 最后，将随机生成的日期赋值给前7个 tutorial 的 day 属性。

在 playground 的最后添加如下代码，将 tutorial 的排定情况输出到控制台：

``` Swift
print(tutorials)
```

成功了！现在你已经为一周中的每天安排了一个教程，无重复，无空白。做得好！

### 可选类型模式（Optional Pattern）

教程日程安排虽然搞定了，但是作为主编，你还需要对教程进行排序。接下来通过可选类型模式来解决该问题。

对教程进行升序排序，未排定日期的 tutorial 按 title 排序，已排定日期的 tutorial 按 day 进行排序。在 playground 的最后添加如下代码：

``` Swift
// 1
tutorials.sort {
  // 2
  switch ($0.day, $1.day) {
    // 3
    case (nil, nil):
      return $0.title.compare($1.title, options: .caseInsensitive) == .orderedAscending
    // 4
    case (let firstDay?, let secondDay?):
      return firstDay.rawValue < secondDay.rawValue
    // 5
    case (nil, let secondDay?):
      return true
    case (let firstDay?, nil):
      return false
  }
}
```

代码说明:

1. 通过数组的`sort(_:)`方法对 tutorials 进行排序。该方法接受一个简洁的闭包，该闭包定意义了数组中任一两个 tutorial 间的排序规则。升序该方法返回true，否则返回false。

2. switch接受一个元组，该元组包含两个当前正在进行比较的tutorial的da属性。这是元组模式的又一次应用。

3. 如果两个为排定的教程日期为nil，那么将通过 compare(_:options:) 方法，按教程名称对其进行升序排序。

4. 为了测试两个教程是否均已排定，这里使用了两个可选类型。 该模式将仅匹配哪些可以解封的值。如果两个值都可以被解封，那么通过它们的原始值进行升序排序

5. 同样运用可选模式，测试是否仅有一个教程被排定，如果是，则将未排定的教程排到前面。

添加改行代码到playground的最后，以打印排好序的教程：

``` Swift
print(tutorials)
```

现在我们已经将教程按预先的想法排好了。

### 枚举模式（Enumeration Case Pattern）

现在我们通过枚举模式去侦测每个教程具体安排的日期。

在 Tutorial 的 extension 中，你通过枚举来将 Day 类型转换为自定义的字符串。下面，添加一个计算型属性 name，来取代这种绑定关系。在Playground的最后添加如下代码：

``` Swift
extension Day {
 
  var name: String {
    switch self {
      case .monday:
        return "Monday"
      case .tuesday:
        return "Tuesday"
      case .wednesday:
        return "Wednesday"
      case .thursday:
        return "Thursday"
      case .friday:
        return "Friday"
      case .saturday:
        return "Saturday"
      case .sunday:
        return "Sunday"
    }
  }
}
```

在 switch 语句通过枚举去匹配当前值（self）。这里展示了枚举类型在匹配中的应用。

一目了然，是不是？数字虽然计算便捷，但名称则更加直观且便于理解。

### 表达式模式（Expression Pattern）

接下来，会添加一个用于描述教程安排顺序的属性。你本可以向下面这样使在此使用枚举来处理该问题（不要将这里的代码添加到 Playground 中）：

``` Swift
var order: String {
  switch self {
    case .monday:
      return "first"
    case .tuesday:
      return "second"
    case .wednesday:
      return "third"
    case .thursday:
      return "fourth"
    case .friday:
      return "fifth"
    case .saturday:
      return "sixth"
    case .sunday:
      return "seventh"
  }
}
```

但同样事情做两次，是个差劲的主编，不是吗? 我们使用另外一种类似的方式来解决该问题。首先重载模式匹配操作符，改变其默认功能以便能适用于 Day 类型。在 playground 的最后添加如下代码:

``` Swift
func ~=(lhs: Int, rhs: Day) -> Bool {
  return lhs == rhs.rawValue + 1
}
```

这段代码允许你使用 1 到 7 之间的任意整数与日期进行匹配。你可以通过该重载操作符以另外一种方式去实现你的计算型属性。

在palyground的最后添加如下代码：

``` Swift
extension Tutorial {
 
  var order: String {
    guard let day = day else {
      return "not scheduled"
    }
    switch day {
      case 1:
        return "first"
      case 2:
        return "second"
      case 3:
        return "third"
      case 4:
        return "fourth"
      case 5:
        return "fifth"
      case 6:
        return "sixth"
      case 7:
        return "seventh"
      default:
        fatalError("invalid day value")
    }
  }
}
```

感谢模式匹配操作符重载，现在**day**对象已经可以跟整型表达式匹配了。这展示了表达式模式的应用。

### 组合起来

定义好了日期名称，并且教程的排序也排好了，现在开始打印每个教程。在 playground 的最后添加如下代码块：


``` Swift
for (index, tutorial) in tutorials.enumerated() {
  guard let day = tutorial.day else {
    print("\(index + 1). \(tutorial.title) is not scheduled this week.")
    continue
  }
  print("\(index + 1). \(tutorial.title) is scheduled on \(day.name). It's the \(tutorial.order) tutorial of the week.")   
}
```

注意到 for-in 语句中的元组了吗？，这里又用到了元组模式。

### 接下来
这里下载[最终playground](https://cdn2.raywenderlich.com/wp-content/uploads/2016/07/patterns.playground-1.zip)。想亲自试试，可以到 [IBM Swift Sandbox](http://swiftlang.ng.bluemix.net/#/repl/57b2d2a68bbb01d512f4ca2b) 动手体验。

如果想了解更多Swift中关于模式匹配的知识，请访问 [Greg Heo](https://twitter.com/gregheo) 在[RWDevCon 2016](https://twitter.com/RWDevCon)上的[视频](https://www.raywenderlich.com/132574/rwdevcon-2016-session-202-programming-swift-style-2)**Programming in a Swift Style**。