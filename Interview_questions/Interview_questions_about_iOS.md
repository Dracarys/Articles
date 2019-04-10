# 面试题系列之iOS项目经验

## Cocoa
Cocoa 是 Apple 为 Mac 等设备打造的一套框架，以 NextSetp 为基础，所以命名多带有 “NS” 前缀，

### NSTimer准吗？有哪些替代方案

不准确，因为 Runloop mode 的切换有可能导致计时器暂停，从而不准确。

解决方案：

- 将计时器添加到自定义线程，这样就不收 Runloop mode的影响，但是要注意子线程的Timer 回调也是在子线程；
- 将计时器添加到特定的 Runloop Mode 中，例如：NSRunLoopCommonMode
- 另起炉灶，用 `mach_absolute_time()` 来实现更高精度的定时器
- CADisplayLink
- GCD timer

### 实现一个 NSString 类

## CocoaTouch
CocoaTouch 是 Apple 为 iPhone 等触屏移动设备专门打造的框架，继承了部分 Cocoa 框架的基础，命名多带有 “UI” 前缀。
### KeyWindow、UIWindow、UIView 之间的关系，Layer、UIView 什么关系？

UIWindow 继承自 UIView，KeyWindow 指当前处于前端，并能接受键盘以及其它非点击相关事件，同一时间只有能一个 KeyWindow。

简单说 UIView 就是 Layer 的 Agent。负责除绘图意外的任务。UIView 继承自 UIResponder，因此它可以对事件进行响应。例如多层级 View 的响应链传递，那都是通过 UIView 的 HitTest 实现的。而 Layer 则直接继承自 NSObject，承担与视图相关的绘制工作，例如修改 View 的大小，背景颜色等最终都是由 Layer 来最终完成。

### UIView 的哪些属性是 animatable 的？

- frame
- bounds
- center
- transform 仿射变换
- alpha 透明度
- contenStrech 拉伸，已弃用。

### UIView 的 LayouSubviews 方法何时会被调用？

- 初始化时不会触发
- 滚动 UIScrollView 时会触发
- 旋转 UIScreen 时会触发
- 当 view 的 frame 发生变化时会触发

### 为什么动画完成后，Layer 会恢复到原先的状态？

Layer 和 View 一样存在着一个层级树状结构：

- 模型树：主要用来记录 Layer 的相关属性状态。
- 呈现树：是对模型树的一份拷贝，记录的是 Layer 动画过程中的属性状态。
- 渲染树：是私有的，负责渲染相关。

了解了这一设计模型后，就可以看到动画只不过是呈现树所展示的一种假象，真正持有 Layer 状态的是模型树，因为呈现树只是一个拷贝，在动画过程中的变化不会影响到模型树，所以当动画结束时，Layer 的属性依然是模型树所记录的状态。

### 给一个 View 设置圆角的方法有哪些，各有什么不同？

离屏渲染绘制 layer tree 中的一部分到一个新的缓存里面（这个缓存不是屏幕，是另一个地方），然后再把这个缓存渲染到屏幕上面。一般来说，你需要避免离屏渲染。因为这个开销很大。在屏幕上面直接合成层要比先创建一个离屏缓存然后在缓存上面绘制，最后再绘制缓存到屏幕上面快很多。这里面有 2 个上下文环境的切换（切换到屏幕外缓存环境，和屏幕环境）

- 直接调用 Layer 的方法，会触发离屏渲染
- 设置 CALayer的mask，同样会触发离屏渲染
- 通过 Core Graphics 绘制带圆角的视图，同样会触发离屏渲染。
- 设置 View 的背景的 content mode

### 直接用 UILabel 和自己用 DrawRect 画 UILabel，那个性能好，为什么？那个占用的内存少？为什么？
直接用更好，调用 coreGraphics 会导致离屏渲染。原因就是解离屏渲染。

### 响应链、如何扩大 View 的响应范围
重写 `- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event` 方法，对那些位于制定范围内的点返回 `YES`，注意潜在的干扰。

### 手触碰到屏幕的时候，响应机制是怎样的？第一响应者是谁？追问 UIView 和 UIResponse 的关系是什么？

第一响应者是 KeyWindow，它通过调用 `- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event` 方法，返回最远的后代（包括自己），该方法是通过自顶向下的递归调用 `- (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event ` 方法，来确定具体应该是谁来响应点击事件的。

uiview 继承自UIResponse。

### iOS 的应用程序有几种状态？推到后台代码是否可以执行哦？双击home键，代码是否可以执行

可以执行，
- 自己主动保活，如播放无音音频文件；
- 通过notification 推送唤醒。

可以，有短暂的保存时间。

### 自线程中调用 connection 方法，为什么不回调？这个问题很老了，现在已经开始用 URLSession。
没加入runloop，子线程被销毁了

### UI框架和CA、CG框架的关系是什么？做过哪些内容？

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

### 如何 hook 一个对象的方法，而不影响其它对象。
methodswazzing，方法替换，有待进一步验证。

### 渲染 UI 为什么要在主线程
UIKit 并不是一个 线程安全 的类，UI 操作涉及到渲染访问各种 View 对象的属性，如果异步操作下会存在读写问题，而为其加锁则会耗费大量资源并拖慢运行速度。另一方面因为整个程序的起点 UIApplication 是在主线程进行初始化，所有的用户事件都是在主线程上进行传递（如点击、拖动），所以 view 只能在主线程上才能对事件进行响应。而在渲染方面由于图像的渲染需要以60帧的刷新率在屏幕上 同时 更新，在非主线程异步化的情况下无法确定这个处理过程能够实现同步更新

[参考](https://juejin.im/post/5c406d97e51d4552475fe178)



## iOS SDK

### 蓝牙

### homeKit

### CoreText

### CoreImage

### iOS 的签名机制如何？

[iOS 签名机制](http://www.cocoachina.com/ios/20181221/25913.html)





## 项目经验

### 设计一个监控 app 启动速度的模块，大体的设计思路如何？

### 如何检测应用是否卡顿
检查卡顿分两种方式：一种是利用 Instrument 的 animation 测试；另一种是 监听 RunLoop。这里重点介绍第二种。（该方法的介绍来自 戴铭--如何利用 RunLoop 原理去监控卡顿）

监听 RunLoop，首先要创建一个 CFRunLoopOberverContext 观察者。

```objc
CFRunLoopObserverContext context = {0,(__bridge void*)self,NULL,NULL};
runLoopObserver = CFRunLoopObserverCreate(kCFAllocatorDefault,kCFRunLoopAllActivities,YES,0,&runLoopObserverCallBack,&context);
```

将创建好的观察者 runLoopObserver 添加到主线程 RunLoop 的 common 模式下观察。然后创建一个持续的子线程专门用来监控主线程的 RunLoop 状态。

一旦发现进入睡眠前的 kCFRunLoopBeforeSources 状态，或者唤醒后的状态 kCFRunLoopAfterWaiting，在设置的时间阈值内一直没有变化，即可判定为卡顿。之后便可 dump 出堆栈的信息，以便进一步分析具体是哪个方法的执行时间过长。

开启子线程监控的代码如下：

```objc
// 创建子线程监控
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    // 子线程开启一个持续的 loop 用来进行监控
    while (YES) {
        long semaphoreWait = dispatch_semaphore_wait(dispatchSemaphore, dispatch_time(DISPATCH_TIME_NOW, 3 * NSEC_PER_SEC));
        if (semaphoreWait != 0) {
            if (!runLoopObserver) {
                timeoutCount = 0;
                dispatchSemaphore = 0;
                runLoopActivity = 0;
                return;
            }
            //BeforeSources 和 AfterWaiting 这两个状态能够检测到是否卡顿
            if (runLoopActivity == kCFRunLoopBeforeSources || runLoopActivity == kCFRunLoopAfterWaiting) {
                // 将堆栈信息上报服务器的代码放到这里
            } //end activity
        }// end semaphore wait
        timeoutCount = 0;
    }// end while
});
```
代码中的 NSEC_PER_SEC， 表示触发卡顿的时间阈值，单位是秒。这里设置成了 3 秒，其依据的是 WatchDog 的机制来确定的。WatchDog 在不同状态下设置的不同时间如下：

- 启动（Launch）20s；
- 恢复（Resume）10s；
- 挂起（Suspend）10s；
- 退出（Quit）6s；
- 后台（Background）3min（在 iOS 7 之前，每次申请 10min；之后改为每次申请 3min，可连续申请，最多申请 10min）。

该阈值设定在小于 WatchDog 的时间之余，尽量结合大多数用户的感受而定。

### 如何 dump 堆栈信息
第一种：通过系统函数获取

```objc
static int s_fatal_signals[] = {
    SIGABRT,
    SIGBUS,
    SIGFPE,
    SIGILL,
    SIGSEGV,
    SIGTRAP,
    SIGTERM,
    SIGKILL,
};

static int s_fatal_signal_num = sizeof(s_fatal_signals) / sizeof(s_fatal_signals[0]);

void UncaughtExceptionHandler(NSException *exception) {
    NSArray *exceptionArray = [exception callStackSymbols]; // 得到当前调用栈信息
    NSString *exceptionReason = [exception reason];       // 非常重要，就是崩溃的原因
    NSString *exceptionName = [exception name];           // 异常类型
}

void SignalHandler(int code)
{
    NSLog(@"signal handler = %d",code);
}

void InitCrashReport()
{
    // 系统错误信号捕获
    for (int i = 0; i < s_fatal_signal_num; ++i) {
        signal(s_fatal_signals[i], SignalHandler);
    }
    
    //oc 未捕获异常的捕获
    NSSetUncaughtExceptionHandler(&UncaughtExceptionHandler);
}

int main(int argc, char * argv[]) {
    @autoreleasepool {
        InitCrashReport();
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));

```
另一种：利用三方开源库 PLCrashReporter 来获取，其可以定位到问题代码的具体位置，且性能消耗不大。

```objc
// 获取数据
NSData *lagData = [[[PLCrashReporter alloc]
                                          initWithConfiguration:[[PLCrashReporterConfig alloc] initWithSignalHandlerType:PLCrashReporterSignalHandlerTypeBSD symbolicationStrategy:PLCrashReporterSymbolicationStrategyAll]] generateLiveReport];
// 转换成 PLCrashReport 对象
PLCrashReport *lagReport = [[PLCrashReport alloc] initWithData:lagData error:NULL];
// 进行字符串格式化处理
NSString *lagReportString = [PLCrashReportTextFormatter stringValueForCrashReport:lagReport withTextFormat:PLCrashReportTextFormatiOS];
// 将字符串上传服务器
NSLog(@"lag happen, detail below: \n %@",lagReportString);

```

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

### 应用内如何进行多语言切换？

### 如何给一款 App 瘦身
- 支持 AppThinning，启用图片AssetCatalog，以便针对不同设备进行图片的分割；
- 去除无用的图片资源，LSUnuserdResources，非常好用的一个工具；
- 图片资源压缩，利用 TinyPng 或 ImageOptimism 对 Png 资源进行重新压缩，超过100KB的图片可以采用 WebP 格式，但是会增加一倍的 CPU 解码消耗；
- 通过 AppCode 分析，去除陈旧未使用的类，除了能瘦身还能加快启动速度

### DSYM文件是什么，你是如何分析的？

符号表文件，可以通过Xcode解析后，直接定位到问题代码

### 如果进行网络、内存、性能优化？

### 如何把异步线程转换成同步任务进行单元测试？
[参考](https://mp.weixin.qq.com/s?mpshare=1&scene=23&mid=100000048&sn=bafde424579a5cb57d7d88f12fc5791e&idx=1&__biz=MzUyNDM5ODI3OQ%3D%3D&chksm=7a2cba984d5b338ecb16e7c9f374244bdf483a1c5452ce0df7da73d236f4783f906eeeef41c9&srcid=1017YqCOZ1dePfHkgcFDmICp#rd)

### 项目组件化用过吗？怎么接耦的？
[参考](http://www.code4app.com/blog-822715-1562.html)

### 你实现过一个框架或者库以供别人使用么？如果有，请谈一谈构建框架或者库时候的经验；如果没有，请设想和设计框架的 public 的API，并指出大概需要如何做、需要注意一些什么方面，来使别人容易地使用你的框架

### 如果现在要实现一个下载功能，如何设计，每个类都觉题做什么？

### 设计一个监控 App 启动速度的模块，说下大体的设计思路

### 如何设计一个网络请求库

### 设计一个图片缓存

### 如何捕捉Crash，设计思路。
[参考](https://blog.csdn.net/skylin19840101/article/details/50955808)

### 设计一个异步渲染的框架

YYAsynicDisplay

[iOS-Developer-and-Designer-Interview-Questions](https://github.com/9magnets/iOS-Developer-and-Designer-Interview-Questions)

[developers](https://www.toptal.com/swift/interview-questions)
