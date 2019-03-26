# 面试题系列之iOS

## CocoTouch
#### keyWindow、UIWindow 的 layer、UIView 的继承关系

#### NSTimer准吗？有哪些替代方案

不准确，因为 Runloop mode 的切换有可能导致计时器暂停，从而不准确。

解决方案：

- 将计时器添加到自线程，这样就不收 Runloop mode的影响，但是要注意回调的线程问题
- 将计时器添加到特定的 Runloop Mode 中，例如：NSRunLoopCommonMode
- 另起炉灶，用 `mach_absolute_time()` 来实现更高精度的定时器
- CADisplayLink
- GCD timer

#### load 和 initialize 方法的调用时机？

- load 当类被加载到 runtime 的时候运行，在 main 函数执行之前，也就是类加载器加载时，每个类只会调用一次；通常被用来进行 Method Swizzle，但是这会增加 App 启动时间。
- initialize 在类接收到第一条消息之前被用调用。每个类只会调用一次。子类未实现会向上查找。一搬用来初始化全局变量或静态变量。

### View哪些属性时 animatable 的？

### LayouSubviews 何时会被调用？

- 初始化时不会触发
- 滚动 UIScrollView 时会触发
- 旋转 UIScreen 时会触发
- 当 view 的 frame 发生变化时会触发
- 

### 为什么动画完成后，layer会恢复到原先的状态？

因为这些产生的动画只是假象,并没有对layer进行改变.那么为什么会这样呢,这里要讲一下图层树里的呈现树.呈现树实际上是模型图层的复制,但是它的属性值表示了当前外观效果,动画的过程实际上只是修改了呈现树,并没有对图层的属性进行改变,所以在动画结束以后图层会恢复到原先状态

复习 layer 的知识

#### 给一个View设置圆角的方法有哪些，各有什么不同？

离屏渲染绘制 layer tree 中的一部分到一个新的缓存里面（这个缓存不是屏幕，是另一个地方），然后再把这个缓存渲染到屏幕上面。一般来说，你需要避免离屏渲染。因为这个开销很大。在屏幕上面直接合成层要比先创建一个离屏缓存然后在缓存上面绘制，最后再绘制缓存到屏幕上面快很多。这里面有 2 个上下文环境的切换（切换到屏幕外缓存环境，和屏幕环境）

- 直接调用 Layer 的方法，会触发离屏渲染
- 设置 CALayer的mask，同样会触发离屏渲染
- 通过 Core Graphics 绘制带圆角的视图，同样会触发离屏渲染，此外还将绘图任务转移给了 CPU
- 设置 View 的背景的 content mode

#### 响应链、如何扩大View的响应范围

hittest，在预计的范围内返回 yes 即可。
view 有一个是否响应事件的方法，重写该方法，对指定范围内的方法返回yes即可。

#### 手触碰到屏幕的时候，响应机制是怎样的？第一响应者是谁？追问 UIView和UIResponse的关系是什么？

Window,然后依次向下传递，

uiview 继承自UIResponse 
#### 直接用UILabel和自己用DrawRect画UILabel，那个性能好，为什么？那个占用的内存少？为什么？
直接用更好，调用 coreGraphics 会导致离屏渲染，

#### iOS的应用程序有几种状态？推到后台代码是否可以执行哦？双击home键，代码是否可以执行

可以执行，
- 自己主动保活，如播放无音音频文件；
- 通过notification 推送唤醒。

可以，有短暂的保存时间。

#### 一般使用的图标内存为多大？200x300的图片，内存应该占用多少比较合理？

#### 自线程中调用connection方法，为什么不回调？
没加入runloop，子线程被销毁了

#### UI框架和CA、CG框架的关系是什么？做过哪些内容？

UIKit 构建在 CoreAnimation 框架之上，CA 框架构建在 Core Graphics 和 OpenGL ES 之上。

#### Quartz 框架


## iOS SDK

### 蓝牙的围栏功能

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

#### 两个链表查找相同节点，交叉

#### 链表环判断

#### 合并K个链表 用小根堆维护

#### 字符串旋转

#### 找链表倒数第K个节点

#### 把一个链表比某个值 大的放在左边，比它小的放在右边

#### n个整数的无序数组，找到每个元素后面比它大的第一个数，要求时间复杂度为O(N)

#### T9算法如何实现，全拼算法

#### 强连通量算法

#### 最短路径算法





[iOS-Developer-and-Designer-Interview-Questions](https://github.com/9magnets/iOS-Developer-and-Designer-Interview-Questions)

[developers](https://www.toptal.com/swift/interview-questions)
