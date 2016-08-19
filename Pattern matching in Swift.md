# Swift中的模式匹配

对任何编程语言来说，模式匹配都是最强大的功能之一，通过制定数值间匹配的规则，使你的代码变得更加简洁和灵活。

Apple 在 Swift 中开放了模式匹配，今天我们就一起来领略Swift 中的模式匹配技术。

本文设计模式如下：

- 元组模式（Tuple pattern）
- 类型转换模式（Type-casting patterns）
- 通配符模式（Wildcard pattern）
- 可选类型模式（Optional pattern）
- 枚举转换模式（Enumeration case pattern）
- 表达式模式（Expression pattern）

为了展示如何运用模式匹配，为你设计这样一个场景：作为[raywenderlich.com](https://www.raywenderlich.com/)的主编，你需要通过模式匹配来安排教程并发布到网站上。

注意：该教程需要Xcode 8 和 Swift 3，并假定您已具备Swift开发的基础。如果您才接触Swift不久，那么你可以查看我们为您提供的其它[Swit教程](https://www.raywenderlich.com/category/swift)。
	
### 开始

欢迎光临，代主编！您今天的主要任务是改进那些即将发布到网站上的教程编排。首先下载[起始playground](https://cdn4.raywenderlich.com/wp-content/uploads/2016/07/starter-project.playground-2.zip)并打开.

playground包含两部分：
- `random_uniform(value:)`函数，会生成一个介于0和＊之间的随机数，通过该随机数来指定编排的具体日子。
- 余下的代码，解析了**tutorials.json**文件，并返回一个包含字典元素的数组，接下来将通过这个数组提供的信息对教程进行编排。

注意：欲了解更多在Swift中如何解析JSON的知识，请访问[该教程](https://www.raywenderlich.com/120442/swift-json-tutorial)

虽然不需要了解解析过程，但要清楚其数据结构，在playground的**Resources**文件夹中找到并打开**tutorials.json**文件，
每个待编排的教程均含有两个属性：名称和排定日期。你的团队负责实施排定的教程，为每个教程的日期赋一个介于1（周一）到5（周五）的值，如果没有排定教程则为nil

你本想一周内每天一篇教程，但是查阅完教程安排表后发现，你的团队在同一天实施了两篇教程。你需要修复该问题，此外，你还想将教程进行特定排序。你该怎么完成这一切呢？如果你说“用模式”，那么你就说对了。

### 模式匹配类型

让我们先来了解下，你在该教程中会用到哪些模式：

- 元组模式（Tuple patterns）用于匹配正确的元组类型值
- 类型转换模式（Type-casting patterns）允许你去转换或匹配类型
- 通配符模式（Wildcard pattern）用于匹配或忽略任意值和类型
- 可选类型模式（Optional pattern）用于匹配可选值
- 枚举转换模式（Enumeration case pattern）用于匹配已有枚举类型
- 表达式模式（Expression pattern）允许你通过一个表达式去比较给定的值

### 元组模式（Tuple Pattern）

首先，通过一个元组模式去教程数组。在playground的最后添加如下代码：
``` Swift
enum Day: Int {
  case monday, tuesday, wednesday, thursday, friday, saturday, sunday
}
```
如此为一周中每天创建一个枚举。原始类型的**Int**，那么可以赋 0 到 6 任意值来表示星期一到星期日。在枚举的后面添加如下代码：

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
现在通过一个数组来保存所有的教程：

``` Swift
var tutorials: [Tutorial] = []
```

接下来，紧挨着刚才的代码，添加如下代码，将包含字典的数组转换为包含tutorial对象的数组：

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

这里通过`for-in`遍历整个 json 数组，同时又通过一个元组遍历了数组中每个字典的所有键值对，这就是元组模式的运用。
将tutorial实例添加到了数组中，但现在tutorial 还是空的，接下来我们将同哟
You add each tutorial to the array, but it is currently empty—you are going to set the tutorial’s properties in the next section with the type-casting pattern.
Type-Casting Patterns
To extract the tutorial information from the dictionary, you’ll use a type-casting pattern. Add this code inside the for (key, value) in dictionary loop, replacing the placeholder comment:

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

代码逐步说明:

1. 在Switch中传入一个包含键和值的元组，元组模式的又一次运用。
2. 测试教程的title是否为String，如果是，则通过类型转换模式将其抓换为String。
3. You test if the tutorial’s day is a string with the as type-casting pattern. If the test succeeds, you convert it into an Int first and then into a day of the week with the Day enumeration’s failable initializer init(rawValue:). You subtract 1 from the dayInt variable because the enumeration’s raw values start at 0, while the days in tutorials.json start at 1.
4. 为包含所有情况，添加了一个defalut ，并通过break 来退出Switch
5. 

在playground的最后添加如下代码，以将tutorial信息输出到控制台。

``` Swift
print(tutorials)
```

如你所见，现在数组中的每个教程都有了自己的title和day属性。 准备就绪，接下来就该解决教程排期问题：一周中每天仅一个教程。

### 通配符模式（Wildcard Pattern）
要通过通配符模式来排定教程，首先得先去取消每个教程的安排。在playground的最后添如下代码:

``` Swift
tutorials.forEach { $0.day = nil }
```

这里将所有无排定的教程的日期设为nil。为了排定所有教程，在palayground的最后添加如下代码:

``` Swift
// 1 
let days = (0...6).map { Day(rawValue: $0)! }
// 2
let randomDays = days.sorted { _ in random_uniform(value: 2) == 0 }
// 3
(0...6).forEach { tutorials[$0].day = randomDays[$0] }
```

代码说明:

1. 首先创建一个days数组，一周中的日期无重复。
2. You “sort” this array. The random_uniform(value:) function is used to randomly determine if an element should be sorted before or after the next element in the array. In the closure, you use an underscore to ignore the closure parameters, since you don’t need them here. Although there are technically more efficient and mathematically correct ways to randomly shuffle an array this shows the wildcard pattern in action!
3. 最后，将随机生成且不重复的日期，赋值给教程的day属性

在playground的最后添加如下代码，以便打印tutorial：

``` Swift
print(tutorials)
```

成功了！现在你已经为一周中的每天安排了一个教程，无重复，无空白。做得好！

### 可选类型模式（Optional Pattern）

教程日程搞定了，但是作为主编，你还需要对教程进行排序。接下来通过可选类型模式来解决该问题。

按升序对教程进行排序，未安排的教程通过title进行升序排序，已安排的教程按日期进行排序。在playground的最后添加如下代码：

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
1.通过数组的ort(_:) 方法对tutorials数组进行排序。该方法接受一个简洁的闭包，该闭包定意义了数组中任一两个tutorial间的排序规则。升序该方法返回true，否则返回false。

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

在Tutorial的extension中，你通过枚举来将Day类型转换为自定义的字符串。下面，添加一个计算型属性name，来取代这种绑定关系。在Playground的最后添加如下代码：

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

通过可能的枚举值在Switch中去匹配当前值（self)。这就是枚举类型匹配的应用。

令人阴险深刻吧？数字总是计算便捷，但名字则更加直观且便于理解。

### 表达式模式（Expression Pattern）
接下来，会添加一个用于描述教程安排顺序的属性。你本可以再一次向下面这样使用枚举来处理该问题（不要将这里的代码添加到Playground中）：

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

但同样事情做两次，是个差劲的主编，不是吗? ;] 我们使用另外一种类似的方式来解决。首先要重载模式匹配操作符，以改变其功能，能适用于日期。在playground的最后添加如下代码:

``` Swift
func ~=(lhs: Int, rhs: Day) -> Bool {
  return lhs == rhs.rawValue + 1
}
```

这段代码允许你使用 1 到 7 之间的任意整数与日期进行匹配。你可以通过该重载操作符以另外一种方式去写你的计算型属性。

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

感谢模式匹配操作符重载，先**day**对象已经可以跟整型表达式匹配了。这就是表达式模式的实际应用。

### 组合起来

定义好了日期名称，并且教程的排序也排好了，现在开始打印每个教程。在playground的最后添加如下代码块：


``` Swift
for (index, tutorial) in tutorials.enumerated() {
  guard let day = tutorial.day else {
    print("\(index + 1). \(tutorial.title) is not scheduled this week.")
    continue
  }
  print("\(index + 1). \(tutorial.title) is scheduled on \(day.name). It's the \(tutorial.order) tutorial of the week.")   
}
```

主要到for-in循环中的元组了吗？，这里又用到了元组模式。

哇哦！你做为代主编，今天的人物还真是不少，不过你做的很棒，现在你可以放松去享受下泳池了。

开玩笑的！主编的工作永远也做不完，怪怪滚回去工作吧！

### 接下来
这里下载[最终playground](https://cdn2.raywenderlich.com/wp-content/uploads/2016/07/patterns.playground-1.zip)。想亲自试试，可以到 [IBM Swift Sandbox](http://swiftlang.ng.bluemix.net/#/repl/57b2d2a68bbb01d512f4ca2b) 动手体验。

如果想了解更多Swift中关于模式匹配的知识，请访问 [Greg Heo](https://twitter.com/gregheo) 在[RWDevCon 2016](https://twitter.com/RWDevCon)上的[视频](https://www.raywenderlich.com/132574/rwdevcon-2016-session-202-programming-swift-style-2)**Programming in a Swift Style**。