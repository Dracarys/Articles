# 面试题系列之iOS

## CocoTouch
#### KeyWindow、UIWindow、UIView 之间的关系，Layer、UIView 什么关系？

UIWindow 继承自 UIView，KeyWindow 指当前处于前端，并能接受键盘以及其它非点击相关事件，同一时间只有能一个 KeyWindow。

简单说 UIView 就是 Layer 的 Agent。负责除绘图意外的任务。UIView 继承自 UIResponder，因此它可以对事件进行响应。例如多层级 View 的响应链传递，那都是通过 UIView 的 HitTest 实现的。而 Layer 则直接继承自 NSObject，承担与视图相关的绘制工作，例如修改 View 的大小，背景颜色等最终都是由 Layer 来最终完成。

#### NSTimer准吗？有哪些替代方案

不准确，因为 Runloop mode 的切换有可能导致计时器暂停，从而不准确。

解决方案：

- 将计时器添加到自定义线程，这样就不收 Runloop mode的影响，但是要注意子线程的Timer 回调也是在子线程；
- 将计时器添加到特定的 Runloop Mode 中，例如：NSRunLoopCommonMode
- 另起炉灶，用 `mach_absolute_time()` 来实现更高精度的定时器
- CADisplayLink
- GCD timer

#### load 和 initialize 方法的调用时机？

- load 当类被加载到 runtime 的时候运行，在 main 函数执行之前，也就是类加载器加载时，每个类默认只会调用一次；通常被用来进行 Method Swizzle，但是这会增加 App 启动时间。
- initialize 在类接收到第一条消息之前被用调用。每个类只会调用一次。子类未实现会向上查找。一搬用来初始化全局变量或静态变量。

### View 哪些属性时 animatable 的？

- frame
- bounds
- center
- transform 仿射变换
- alpha 透明度
- contenStrech 拉伸，已弃用。

### LayouSubviews 何时会被调用？

- 初始化时不会触发
- 滚动 UIScrollView 时会触发
- 旋转 UIScreen 时会触发
- 当 view 的 frame 发生变化时会触发

### 为什么动画完成后，layer会恢复到原先的状态？

Layer 和 View 一样存在着一个层级树状结构：

- 模型树：主要用来记录 Layer 的相关属性状态。
- 呈现树：是对模型树的一份拷贝，记录的是 Layer 动画过程中的属性状态。
- 渲染树：是私有的，负责渲染相关。

了解了这一设计模型后，就可以看到动画只不过是呈现树所展示的一种假象，真正持有 Layer 状态的是模型树，因为呈现树只是一个拷贝，在动画过程中的变化不会影响到模型树，所以当动画结束时，Layer 的属性依然是模型树所记录的状态。

#### 给一个View设置圆角的方法有哪些，各有什么不同？

离屏渲染绘制 layer tree 中的一部分到一个新的缓存里面（这个缓存不是屏幕，是另一个地方），然后再把这个缓存渲染到屏幕上面。一般来说，你需要避免离屏渲染。因为这个开销很大。在屏幕上面直接合成层要比先创建一个离屏缓存然后在缓存上面绘制，最后再绘制缓存到屏幕上面快很多。这里面有 2 个上下文环境的切换（切换到屏幕外缓存环境，和屏幕环境）

- 直接调用 Layer 的方法，会触发离屏渲染
- 设置 CALayer的mask，同样会触发离屏渲染
- 通过 Core Graphics 绘制带圆角的视图，同样会触发离屏渲染。
- 设置 View 的背景的 content mode

### 响应链、如何扩大View的响应范围

重写 `- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event` 方法，对那些位于制定范围内的点返回 `YES`，注意潜在的干扰。

### 手触碰到屏幕的时候，响应机制是怎样的？第一响应者是谁？追问 UIView和UIResponse的关系是什么？

第一响应者是 KeyWindow，它通过调用 `- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event` 方法，返回最远的后代（包括自己），该方法是通过自顶向下的递归调用 `- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event ` 方法，来确定具体应该是谁来响应点击事件的。

uiview 继承自UIResponse 
### 直接用 UILabel 和自己用 DrawRect 画 UILabel，那个性能好，为什么？那个占用的内存少？为什么？
直接用更好，调用 coreGraphics 会导致离屏渲染。原因就是解离屏渲染。

### iOS的应用程序有几种状态？推到后台代码是否可以执行哦？双击home键，代码是否可以执行

可以执行，
- 自己主动保活，如播放无音音频文件；
- 通过notification 推送唤醒。

可以，有短暂的保存时间。

#### 自线程中调用 connection 方法，为什么不回调？这个问题很老了，现在已经开始用 URLSession。
没加入runloop，子线程被销毁了

#### UI框架和CA、CG框架的关系是什么？做过哪些内容？

UIKit 构建在 CoreAnimation 框架之上，CA 框架构建在 Core Graphics 和 OpenGL ES 之上。

### 哪些操作会导致离屏渲染
设置一下属性时都会触发离屏渲染：

- shouldResterize （光栅化）
- masks
- shadows
- edge antialiasing （抗锯齿）
- group opacity （不透明)
- corner radious
- Gradient

### 怎么判断是否存在离屏渲染
Instruments 的 Core Animation 工具中有几个和离屏渲染相关的检查选项：

- Color Offscreen-Rendered Yellow 开启后会把那些需要离屏渲染的图层高亮成黄色，这就意味着黄色图层可能存在性能问题，模拟器也存在该选项。
- Color Hits Green and Misses Red
如果shouldRasterize被设置成YES，对应的渲染结果会被缓存，如果图层是绿色，就表示这些缓存被复用；如果是红色就表示缓存会被重复创建，这就表示该处存在性能问题了。 

### 应用内如何进行多语言切换？


## iOS SDK

### 蓝牙

### homeKit

### CoreText

### CoreImage

### 静态库和动态库之间的区别

- 动态库 \*.dylib 和 \*.framework，连接时不复制，程序运行时由系统动态加载到内存，
- 静态库 \*.a 和 \*.framework。连接时，静态会被复制到输出目标中

### iOS从什么时候开始支持动态库的？

Xcode 6 之后支持创建动态库工程。

### iOS 的签名机制如何？

[iOS 签名机制](http://www.cocoachina.com/ios/20181221/25913.html)




## 工具类

### 设计一个监控 app 启动速度的模块，大体的设计思路如何？

### 如何检测应用是否卡顿

instrument，animation测试

### 如何编写单元测试，例如测试一个网络库，如何提高覆盖率

尽量覆盖方法的每一个分支

### images.xcassetsa和直接使用图片有什么不一样？

1. 除图标和lancher 外支持多种图片格式
2. 图片支持[UIImage imagedName:""]的方式实例化，不能从 Bundle 加载
3. 在编译时，Image.xcassets 中的所有文件hi被打包为 Assets.car 文件，再解包取图片相对增加了一点难度。
4. 减少 App 包的大小
5. 支持 PDF 格式的矢量图

### 平时是怎么进行测试的，内存方面怎么测试

Instrument memory 相关测试。 

### lldb常用调试命令？

- p
- watchpoint

### 如何给一款 App 瘦身

- 去除不必要的图片；
- 去除陈旧未使用的类，除了能瘦身还能加快启动速度
- 

### DSYM文件是什么，你是如何分析的？

符号表文件，可以通过Xcode解析后，直接定位到问题代码

### 如果进行网络、内存、性能优化？

### 如何把异步线程转换成同步任务进行单元测试？
[参考](https://mp.weixin.qq.com/s?mpshare=1&scene=23&mid=100000048&sn=bafde424579a5cb57d7d88f12fc5791e&idx=1&__biz=MzUyNDM5ODI3OQ%3D%3D&chksm=7a2cba984d5b338ecb16e7c9f374244bdf483a1c5452ce0df7da73d236f4783f906eeeef41c9&srcid=1017YqCOZ1dePfHkgcFDmICp#rd)

### 算法相关

#### 不实用 temp 交换 a、b 值

#### 银行家算法

#### 数组滑动窗口窗口的中位数？

#### 小根堆的插入时间负责度

#### 二分查找的时间复杂度怎么求的？

#### 2个集合的交集

#### 找出数组中重复的数字

#### 百亿数据中查找相同的数字及出现次数

#### 在一个10G的数据中找到最大的100个数

#### 1000瓶药，10只狗，找出毒药。

#### 排序算法，各时间复杂度如何？

#### 堆排，有时间杂度为O(n)的排序算法吗

#### 快排的时间复杂度，为什么是nlogn，最坏的情况是什么，最坏时间复杂度如何？

#### 字符串转浮点数

#### atoi函数实现

#### 二叉树迭代中序遍历

#### 二叉树层序遍历，按层输出

#### 平衡二叉树判断

#### 子树判断

#### 深度有限和广度优先的使用场景

#### 两个链表查找相同节点，交叉，

两条链表相交，形状成 Y 字形，找到第一个相同节点。

#### 链表环判断

#### 合并K个链表 用小根堆维护

#### 字符串旋转

#### 找链表倒数第K个节点

#### 把一个链表比某个值 大的放在左边，比它小的放在右边

#### n个整数的无序数组，找到每个元素后面比它大的第一个数，要求时间复杂度为O(N)

#### T9算法如何实现，全拼算法

#### 强连通量算法

#### 最短路径算法

#### Z 型打印二叉树

#### 打印螺旋矩阵

#### 百万数据找出前 K 个





[iOS-Developer-and-Designer-Interview-Questions](https://github.com/9magnets/iOS-Developer-and-Designer-Interview-Questions)

[developers](https://www.toptal.com/swift/interview-questions)
