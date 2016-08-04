# \[翻译]如何制作一个类似Uber的启动页

_本文翻译自_[How To Create an Uber Splash Screen](https://www.raywenderlich.com/133224/how-to-create-an-uber-splash-screen) _由 [Derek Selander](https://www.raywenderlich.com/u/lolgrep) 发表于Raywenderlich_ 。

受限于个译者个人英语水平及经验，翻译的内容难免有词不达意，甚至错误的地方，还望各位同行不吝指教。

此外由于该项目使用了PDF文件，所以在模拟器上运行会有一点小抽风，例如启动页，和应用到背景上的图型，在模拟器上都会发生扭曲，所以建议在真机上学习。原文中作者也提到了该问题，但是比较靠后。为了避免首次运行起始工程产生困惑，译者该问题提到文首说明。

－－－－－－－－－－－－－－－－－－－－－

Oh, the wonderful splash screen—a chance for developers to go wild with fun animations as the app frantically pings API endpoints for critical data needed to function. 为了让用户在等待应用启动的过程中始终保持高昂的兴趣，一个酷酷的启动页就变的尤为重要。

虽然涟漪式启动页已广泛应用，但是你很难找到一个如Uber这般出色的。在2016年的首季，Uber启动了一个由CEO领导的用户体验重塑计划，该计划的成果之一,便是一个炫酷的涟漪式启动页。

本教程以仿制Uber启动动效为目标。其中运用了大量的`CAlayer`和`CAAnimation`类，及其相应子类。相对于概念介绍，本文更着重于如何运用这些类去实现一个产品级的动效。如需了解动画相关概念，请访问 _Marin Todorov_ 的系列视频教程：
[**Intermediate iOS Animation**](https://www.raywenderlich.com/u/icanzilb)

### 开始吧

因为本文涉及的动画众多，所以这里有提供了一个已经通过`CALayer`实现了部分动画的起始工程。 [在这里下载](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/Fuber-starter.zip).

>原作者提供的起始工程是基于Swift2.x的，你还可以在[这里下载]()到由译者提供的基于Xcode 8 beta4 的Swift3.0起始工程，注意：译者提供的起始工程是在原作者工程的基础上迁移至Swift3.0的，并提供了中文注释，仅供学习交流。如有引用需求，请参照原作者和Raywenderlich的开源共享协议。

起始工程是一个叫做Fuber的App，Fuber提供驾乘共享服务，乘客可请求一位Segway驾驶员，搭载自己抵达城市的任何地方。Fuber发展迅速，已在60多个国家为用户提供服务，但也面临众多国家的反对和工会要求其必须与司机签订合同的要求。：］(原作者卖萌了)

<center>![Splash screen](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/fuber\_logo-480x161.png)</center>

最终，我们会创建一个如下炫酷的启动页:

<center>![Fuber Animation](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Animation.gif)</center>

打开起始工程，运行并简单浏览一下工程结构。

首先从视图控制器开始，应用通过父试图控制`RootContainerViewController`加载`SplashViewController`，并由其负责子视图控制器的切出工作. 父视图控制器从启动动画开始运行，直至应用的所有准备工作全部完成。This could happen when there is a handshake success to an API endpoint and the app has the necessary data to continue.需要指出的是，在这个小工程中启动页处于一个独立的模块。

RootContainerViewController中已经实现好两个方法：`showSplashViewController()`和 `showSplashViewControllerNoPing()`。 
由于教程中大部分时间，都在调用showSplashViewControllerNoPing()方法调试启动动画，所以我们先将精力放在SplashViewController的子视图动画上，然后再考虑如何模拟后台的请求过程，并跳转到主控制器上。

### 启动页及其构成

`SplashViewController`的`view`包含两个`subview`。 第一个`subview`是用于构成网格背景的`TileGridview`，它包含了一系列按网格排列的`TileView`实例。另一个`subview`由一个名为`AnimatedULogoView`的 U 字型动画图标构成。

![Splash Screen](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Fuber-View-Hierarchy-1.png)

`AnimatedULogoView`包含4个`CAShapeLayers`:

- **circleLayer** 用于实现一个 U 字型的白色背景
- **lineLayer** 用于实现一条从`circleLayer`中心到边缘的直线
- **squareLayer** 用于实现位于`circleLayer`中心位置的方块
- **maskLayer** 用于实现`view`的遮罩，通过改变它的边界来实现一个简单遮罩动画。

通过这几个`CAShapeLayers`动画的组合，实现了Fuber的 U 字型动画。

![RiderIconView](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/RiderIconView.gif)

现在我们已经了解这些图层是如何构成的，接下来我们添加一些动画来让`AnimatedULogoView`动起来吧。

### 添加圆形动画

创建复杂动画的关键，就在于排除干扰专注于我们正在实现的部分。 打开**AnimatedULogoView.swift**文件. 在`init(frame:)`方法中, 注释掉除**circleLayer**外其它向`view`中添加 _sublayer_ 的方法，完成动画后会再将其添加回来。注释完成后的代码如下:

``` Swift
override init(frame: CGRect) {
  super.init(frame: frame)
 
  circleLayer = generateCircleLayer()
  lineLayer = generateLineLayer()
  squareLayer = generateSquareLayer()
  maskLayer = generateMaskLayer()
 
//  layer.mask = maskLayer
  layer.addSublayer(circleLayer)
//  layer.addSublayer(lineLayer)
//  layer.addSublayer(squareLayer)
}
```

找到**generateCircleLayer()**方法，思考一下圆形是如何被创建的。事实上它只是一个通过`UIBezierPath`创建的`CAShapeLayer` 。 注意这行代码:

``` Swift
layer.path = UIBezierPath(arcCenter: CGPointZero, 
                             radius: radius/2, 
                         startAngle: -CGFloat(M_PI_2), 
                           endAngle: CGFloat(3*M_PI_2),
                          clockwise: true).CGPath
```

如果向`startAngle`参数传 0 或使用默认值, 贝塞尔弧线会从右侧(3点钟位置)开始。传入`-M_PI_2`即 -90 度, 则会从顶部开始，如果`endAngle`恰好是270度即 `3 * M_PI_2` ，弧线则再次回到顶点（形成一个圆形）。注意为了绘制的动画效果，我们使用圆形的半径作为`lineWidth`。

**circleLayer**的动画由三个**CAAnimation**类实现：一个用于**stokeEnd**的**CAKeyframeAnimation**动画，一个用于**transform**的**CABasicAnimation**动画，和一个负责将两部分动画组合起来的**CAAnimationGroup**。这些动画效果将一次性创建完毕。

向事先写好的**animateCircleLayer()**方法中添加如下代码:

``` Swift
  // strokeEnd
  let strokeEndAnimation = CAKeyframeAnimation(keyPath: "strokeEnd")
  strokeEndAnimation.timingFunction = strokeEndTimingFunction
  strokeEndAnimation.duration = kAnimationDuration - kAnimationDurationDelay
  strokeEndAnimation.values = [0.0, 1.0]
  strokeEndAnimation.keyTimes = [0.0, 1.0]
```

通过设置的0.0和1.0两个值，Core Animation便为我们绘制一个从startAngle到endAngle顺时针旋转到炫酷动画。伴随着strokeEnd值的增加，线条沿着圆环慢慢延长，圆形也渐渐填满。在这个例子中，如果我们将values属性的值设为[0.0, 0.5]，则会出现一个半圆，这是因为StrokeEnd在动画结束刚达到圆周的一半。

现在添加过渡动画:

``` Swift
  // transform
  let transformAnimation = CABasicAnimation(keyPath: "transform")
  transformAnimation.timingFunction = strokeEndTimingFunction
  transformAnimation.duration = kAnimationDuration - kAnimationDurationDelay
 
  var startingTransform = CATransform3DMakeRotation(-CGFloat(M_PI_4), 0, 0, 1)
  startingTransform = CATransform3DScale(startingTransform, 0.25, 0.25, 1)
  transformAnimation.fromValue = NSValue(CATransform3D: startingTransform)
  transformAnimation.toValue = NSValue(CATransform3D: CATransform3DIdentity)
```

这里同时实现了放大和沿 Z 轴旋转的动画。 这会让圆形在顺时针旋转45度的同时逐渐变大。这里的旋转很重要，因为当lineLayer与其它图层一起做动画时，圆形的旋转要与其位置和速度保持一致。
最后在animateCircleLayer()函数的最下方添加一个CAAnimationGroup。这个动画组将包含之前的两个动画，如此我们仅向circleLayer添加一次动画即可。ini

``` Swift
  // Group
  let groupAnimation = CAAnimationGroup()
  groupAnimation.animations = [strokeEndAnimation, transformAnimation]
  groupAnimation.repeatCount = Float.infinity
  groupAnimation.duration = kAnimationDuration
  groupAnimation.beginTime = beginTime
  groupAnimation.timeOffset = startTimeOffset
 
  circleLayer.addAnimation(groupAnimation, forKey: "looping")
```

这里我们修改了CAAnimationGroup的两个重要属性：beginTime and timeOffset。如果你对其中任何一个不熟悉，那么你都可以在[这里](http://ronnqvi.st/controlling-animation-timing/)找到关于这两个属性的介绍和使用说明。
将groupAnimation的beginTime设置为与父view相同。

对timeOffeset的设置是必要的，因为动画首次运行时实际上是从一半开始的。等完成所有动画后，可以尝试改变startTimeOffset的值，并观察动画的视觉效果上有什么不同。 

添加完动画组后，编译并运行，检查下动画效果.

![Splash Screen CircleIn Animation](https://cdn4.raywenderlich.com/wp-content/uploads/2016/05/CircleIn-Animation.gif)

>注意: 试着删除**groupAnimation.animations**数组中的**strokeEndAnimation**或**transformAnimation**，以确认每个动画具体实现了哪些视觉效果. 像这样再实验一下本文中的其它动画，你会惊讶于，仅仅改变动画的组合方式就可以产生令人难以预料的独特视觉效果.


### 添加Line动画

完成了**circleLayer**的动画, 接下来我们再来完成**lineLayer**动画。还是 **AnimatedULogoView.swift**文件, 找到**startAnimating()**方法并注释掉除**animateLineLayer()**外的所有动画调用。注释后的代码如下:

``` Swift
public func startAnimating() {
  beginTime = CACurrentMediaTime()
  layer.anchorPoint = CGPointZero
 
//  animateMaskLayer()
//  animateCircleLayer()
  animateLineLayer()
//  animateSquareLayer()
}
```

此外, 修改**init(frame:)**方法中的代码，只显示**circleLayer**和**lineLayer**两个图层:

``` Swift
override init(frame: CGRect) {
  super.init(frame: frame)
 
  circleLayer = generateCircleLayer()
  lineLayer = generateLineLayer()
  squareLayer = generateSquareLayer()
  maskLayer = generateMaskLayer()
 
//  layer.mask = maskLayer
  layer.addSublayer(circleLayer)
  layer.addSublayer(lineLayer)
//  layer.addSublayer(squareLayer)
}
```

注释掉图层和动画后, 转到**animateLineLayer()**方法并实现下面这组动画效果:

``` Swift
  // lineWidth
  let lineWidthAnimation = CAKeyframeAnimation(keyPath: "lineWidth")
  lineWidthAnimation.values = [0.0, 5.0, 0.0]
  lineWidthAnimation.timingFunctions = [strokeEndTimingFunction, circleLayerTimingFunction]
  lineWidthAnimation.duration = kAnimationDuration
  // Swift 3.0 keyTimes是一个NSNumber数组
  lineWidthAnimation.keyTimes = [0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]
```

这个动画使lineLayer的width呈现出先增后减的效果。

为接下来的动画添加如下代码:

``` Swift
  // transform
  let transformAnimation = CAKeyframeAnimation(keyPath: "transform")
  transformAnimation.timingFunctions = [strokeEndTimingFunction, circleLayerTimingFunction]
  transformAnimation.duration = kAnimationDuration
  transformAnimation.keyTimes = [0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]
 
  var transform = CATransform3DMakeRotation(-CGFloat(M_PI_4), 0.0, 0.0, 1.0)
  transform = CATransform3DScale(transform, 0.25, 0.25, 1.0)
  transformAnimation.values = [NSValue(CATransform3D: transform),
                               NSValue(CATransform3D: CATransform3DIdentity),
                               NSValue(CATransform3D: CATransform3DMakeScale(0.15, 0.15, 1.0))]
```

与circleLayer的形变动画非常相似, 这里我们定义了个一个沿 Z 轴顺时针旋转的动画。 然而，我们还为Line添加一个缩小到25%形变，恢复形变和一个最终缩小到15%的形变.

通过CAAnimationGroup将动画组合起来，并添加到lineLayer上：

``` Swift
  // Group
  let groupAnimation = CAAnimationGroup()
  groupAnimation.repeatCount = Float.infinity
  groupAnimation.removedOnCompletion = false
  groupAnimation.duration = kAnimationDuration
  groupAnimation.beginTime = beginTime
  groupAnimation.animations = [lineWidthAnimation, transformAnimation]
  groupAnimation.timeOffset = startTimeOffset
 
  lineLayer.addAnimation(groupAnimation, forKey: "looping")
```

编译运行，注意观察变化.

![Splash Screen Knockoutline Animation](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Knockoutline-Animation.gif)

注意我们设置了同样的-M_PI_4形变初始值，以便Line能够和circle的绘制对齐。我们还设置keyTimes为`[0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]`。 数组中首尾两个元素是确定: 0 代表动画开始那一刻，1.0代表动画结束那一刻，然后通过计算来获取圆形绘制刚刚结束、第二部分的动画即将开始的那一刻。通过 kAnimationDurationDelay出一kAnimationDuration来得到一个确切的比值，但由于它是一个延迟动画, 还需要从1.0中减去它， 这是因为当动画结束时再返回到起点（形成一个循环动画，否则会出现跳跃，造致使动画不连贯）。
circleLayer和lineLayer动画都已经完成，接下来我们该完成中间的方形动画了。

### 为Square添加动画

与之前类似。 在**startAnimating()**函数中注释掉除animateSquareLayer()外的其它动画调用。此外，像下面这样修改**init(frame:)**方法的代码：

``` Swift
override init(frame: CGRect) {
  super.init(frame: frame)
 
  circleLayer = generateCircleLayer()
  lineLayer = generateLineLayer()
  squareLayer = generateSquareLayer()
  maskLayer = generateMaskLayer()
 
//  layer.mask = maskLayer
  layer.addSublayer(circleLayer)
//  layer.addSublayer(lineLayer)
  layer.addSublayer(squareLayer)
}
```

完成后转到animateSquareLayer()方法实现如下动画代码:

``` Swift
  // bounds
  let b1 = NSValue(CGRect: CGRect(x: 0.0, y: 0.0, width: 2.0/3.0 * squareLayerLength, height: 2.0/3.0  * squareLayerLength))
  let b2 = NSValue(CGRect: CGRect(x: 0.0, y: 0.0, width: squareLayerLength, height: squareLayerLength))
  let b3 = NSValue(CGRect: CGRectZero)
 
  let boundsAnimation = CAKeyframeAnimation(keyPath: "bounds")
  boundsAnimation.values = [b1, b2, b3]
  boundsAnimation.timingFunctions = [fadeInSquareTimingFunction, squareLayerTimingFunction]
  boundsAnimation.duration = kAnimationDuration
  boundsAnimation.keyTimes = [0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]
```

这一部分动画用于改变CALayer的bounds. 它先变为原尺寸的2/3，在到全尺寸，再到0。
接下来，添加背景色的动画:

``` Swift
  // backgroundColor
  let backgroundColorAnimation = CABasicAnimation(keyPath: "backgroundColor")
  backgroundColorAnimation.fromValue = UIColor.whiteColor().CGColor
  backgroundColorAnimation.toValue = UIColor.fuberBlue().CGColor
  backgroundColorAnimation.timingFunction = squareLayerTimingFunction
  backgroundColorAnimation.fillMode = kCAFillModeBoth
  backgroundColorAnimation.beginTime = kAnimationDurationDelay * 2.0 / kAnimationDuration
  backgroundColorAnimation.duration = kAnimationDuration / (kAnimationDuration - kAnimationDurationDelay)
```

注意上面的fillMode属性. 从beginTime不为零开始, 动画就会在起始和结束时保持住当前颜色（CGColor）. 这使动画在被添加到父CAAnimationGroup时不会出现闪烁。（译者：翻译的不好:(。请试着改变该属性的设置，看看视觉效果上有什么不同，以加深理解。）
了解了这些，我们就动手来实现一下吧:

``` Swift
  // Group
  let groupAnimation = CAAnimationGroup()
  groupAnimation.animations = [boundsAnimation, backgroundColorAnimation]
  groupAnimation.repeatCount = Float.infinity
  groupAnimation.duration = kAnimationDuration
  groupAnimation.removedOnCompletion = false
  groupAnimation.beginTime = beginTime
  groupAnimation.timeOffset = startTimeOffset
  squareLayer.addAnimation(groupAnimation, forKey: "looping")
```

编译运行检查运行结果. 注意留意SquareLayer。

![Splash Screen Tutorial](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/KnockoutSquare-Animation.gif)

现在将所有的动画组合起来

>注意: 在电脑的GPU完成对iOS设备的模拟任务前，模拟器上的动画可能会有那么一点小抽。如果你的电脑带不动动画，可以试着将模拟器窗口调小或者转到真机开发。

###The Mask

首先，取消init(frame:)方法中所有添加的Layer方法的注释，以及startAnimating()所有动效方法的注释.
将所有动画组合起来运行。

![PreMask Animation](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/PreMask-Animation.gif)

看上去还是有点怪怪点，是不是？circleLayer在缩小时，它的边缘收缩地很突然。幸运地是, 遮罩动画可以解决该问题, 让所有子图层变的平滑.

在animateMaskLayer()方法中添加如下代码:

``` Swift
  // bounds
  let boundsAnimation = CABasicAnimation(keyPath: "bounds")
  boundsAnimation.fromValue = NSValue(CGRect: CGRect(x: 0.0, y: 0.0, width: radius * 2.0, height: radius * 2))
  boundsAnimation.toValue = NSValue(CGRect: CGRect(x: 0.0, y: 0.0, width: 2.0/3.0 * squareLayerLength, height: 2.0/3.0 * squareLayerLength))
  boundsAnimation.duration = kAnimationDurationDelay
  boundsAnimation.beginTime = kAnimationDuration - kAnimationDurationDelay
  boundsAnimation.timingFunction = circleLayerTimingFunction
```

这是一个边界动画. 注意，由于这是一个应用于所有子图层的遮罩，当边界发生变化时, 整个AnimatedULogoView都将消失。
Now implement a corner radius animation to keep the mask circular:

``` Swift
  // cornerRadius
  let cornerRadiusAnimation = CABasicAnimation(keyPath: "cornerRadius")
  cornerRadiusAnimation.beginTime = kAnimationDuration - kAnimationDurationDelay
  cornerRadiusAnimation.duration = kAnimationDurationDelay
  cornerRadiusAnimation.fromValue = radius
  cornerRadiusAnimation.toValue = 2
  cornerRadiusAnimation.timingFunction = circleLayerTimingFunction
```

将这两个动画添加到一个CAAnimationGroup中，已完成这个图层（的所有动画）：

``` Swift
  // Group
  let groupAnimation = CAAnimationGroup()
  groupAnimation.removedOnCompletion = false
  groupAnimation.fillMode = kCAFillModeBoth
  groupAnimation.beginTime = beginTime
  groupAnimation.repeatCount = Float.infinity
  groupAnimation.duration = kAnimationDuration
  groupAnimation.animations = [boundsAnimation, cornerRadiusAnimation]
  groupAnimation.timeOffset = startTimeOffset
  maskLayer.addAnimation(groupAnimation, forKey: "looping")
```

编译并运行。
![RiderIconView Animation](https://cdn4.raywenderlich.com/wp-content/uploads/2016/05/RiderIconView-Animation.gif)
看起来不错！

### 网格

A digital frontier. Try to picture clusters of UIViews as they move through the TileGridView instance. 它们看起来什么样呢，呃。。。这里就不引用[Tron](https://en.wikipedia.org/wiki/Tron:_Legacy#Sequel)了，请继续往往下看！
网格背景由一些列的附加到TileGridView类的TileView组成。为了便于理解这个概念, 我们打开TileView.swift文件，找到init(frame:)方法，在方法的最后添加如下代码:

``` Swift
layer.borderWidth = 2.0
```

编译并运行应用。

![Fuber-Grid-View](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Grid-View-180x320.png)

如果你所见，TileView被整齐地排列成一张网格。整个创建逻辑都集中在TileGridView.swift文件的renderTileViews()方法中。幸运的是，我们需要的布局逻辑已经实现好。接下来要做的就是让它动起来!

### 为TileView添加动画

TileGridView仅有一个直接子subview，containerView. 它负责添加所有的TileView。 此外,还有一个名为tileViewRows的属性, 它是一个二维数组，包含所有添加到containerView的TileView。
回到TileView中的init(frame:)方法. 删除我们刚才添加用于显示边界的代码，并取消向图层中添加chimeSplashImage方法的注释。完成后的方法如下:

``` Swift
override init(frame: CGRect) {
  super.init(frame: frame)
  layer.contents = TileView.chimesSplashImage.CGImage
  layer.shouldRasterize = true
}
```

编译并运行。
![Grid Starting](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/Grid-Starting.gif)

酷。。。。我们马上就要大功告成了。
然而, TileGridView (以及它的TileView们)还需要添加一些动画效果. 打开TileView.swift文件,转到startAnimatingWithDuration(_:beginTime:rippleDelay:rippleOffset:) 方法并添加如下动画代码:

``` Swift
  let timingFunction = CAMediaTimingFunction(controlPoints: 0.25, 0, 0.2, 1)
  let linearFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionLinear)
  let easeOutFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseOut)
  let easeInOutTimingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
  let zeroPointValue = NSValue(CGPoint: CGPointZero)
 
  var animations = [CAAnimation]()
```

这段代码设置了一系列我们即将用到的时间函数。继续添加下面的代码:

``` Swift
  if shouldEnableRipple {
    // Transform.scale
    let scaleAnimation = CAKeyframeAnimation(keyPath: "transform.scale")
    scaleAnimation.values = [1, 1, 1.05, 1, 1]
    scaleAnimation.keyTimes = TileView.rippleAnimationKeyTimes
    scaleAnimation.timingFunctions = [linearFunction, timingFunction, timingFunction, linearFunction]
    scaleAnimation.beginTime = 0.0
    scaleAnimation.duration = duration
    animations.append(scaleAnimation)
 
    // Position
    let positionAnimation = CAKeyframeAnimation(keyPath: "position")
    positionAnimation.duration = duration
    positionAnimation.timingFunctions = [linearFunction, timingFunction, timingFunction, linearFunction]
    positionAnimation.keyTimes = TileView.rippleAnimationKeyTimes
    positionAnimation.values = [zeroPointValue, zeroPointValue, NSValue(CGPoint:rippleOffset), zeroPointValue, zeroPointValue]
    positionAnimation.additive = true
 
    animations.append(positionAnimation)
  }
```

shouldEnableRipple是个布尔值，用于控制何时将形变动画和位置动画添加到我们刚刚创建的数组中。 Its value is set to true for all the TileViews that are not on the perimeter of the TileGridView. This logic is already done for you when the TileViews are created in the renderTileViews() method of TileGridView.
Add an opacity animation:

``` Swift
  // Opacity
  let opacityAnimation = CAKeyframeAnimation(keyPath: "opacity")
  opacityAnimation.duration = duration
  opacityAnimation.timingFunctions = [easeInOutTimingFunction, timingFunction, timingFunction, easeOutFunction, linearFunction]
  opacityAnimation.keyTimes = [0.0, 0.61, 0.7, 0.767, 0.95, 1.0]
  opacityAnimation.values = [0.0, 1.0, 0.45, 0.6, 0.0, 0.0]
  animations.append(opacityAnimation)
```

This is a pretty self-explanatory animation with some very specific keyTimes.
现在将这些动画添加到一个动画组中:

``` Swift
  // Group
  let groupAnimation = CAAnimationGroup()
  groupAnimation.repeatCount = Float.infinity
  groupAnimation.fillMode = kCAFillModeBackwards
  groupAnimation.duration = duration
  groupAnimation.beginTime = beginTime + rippleDelay
  groupAnimation.removedOnCompletion = false
  groupAnimation.animations = animations
  groupAnimation.timeOffset = kAnimationTimeOffset
 
  layer.addAnimation(groupAnimation, forKey: "ripple")
```

This will add groupAnimation to the instance of TileView. Note that the group animation could either have one or three animations in the group, depending on the value of shouldEnableRipple.
现在我们已经为每一个TileView实现了动画, 接下来将在TileGridView中调用它们. 打开TileGridView.swift文件将以下代码添加到startAnimatingWithBeginTime\(\_\:\)方法中:

``` Swift 
private func startAnimatingWithBeginTime(beginTime: NSTimeInterval) {
  for tileRows in tileViewRows {
    for view in tileRows {
      view.startAnimatingWithDuration(kAnimationDuration, beginTime: beginTime, rippleDelay: 0, rippleOffset: CGPointZero)
    }
  }
}
```

编译并运行。
![Grid-1](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Grid-1.gif)

呃。。。看上去已经好多了，但AnimatedULogoView的跳动应该通过TileViews向外产生一个类似水波的涟漪效果。 这就意味着应该创建一个与动画时长相乘的延迟系数，基于从中央到View到外围View之间的距离.
在startAnimatingWithBeginTime\(\_\:\)函数下, 添加一个新的函数:

``` Swift
private func distanceFromCenterViewWithView(view: UIView)->CGFloat {
  guard let centerTileView = centerTileView else { return 0.0 }
 
  let normalizedX = (view.center.x - centerTileView.center.x)
  let normalizedY = (view.center.y - centerTileView.center.y)
  return sqrt(normalizedX * normalizedX + normalizedY * normalizedY)
}
```

该方法可以便捷地获取到，指定view与位于中心的TileView之间的距离。
回到startAnimatingWithBeginTime(_:)函数，将其内容替换为如下代码:

``` Swift
  for tileRows in tileViewRows {
    for view in tileRows {
      let distance = self.distanceFromCenterViewWithView(view)
 
      view.startAnimatingWithDuration(kAnimationDuration, beginTime: beginTime, rippleDelay: kRippleDelayMultiplier * NSTimeInterval(distance), rippleOffset: CGPointZero)
    }
  }
```

通过刚刚添加的distanceFromCenterViewWithView\(\_\:\)函数，来计算每个子动画的延迟启动时间。
编译运行.

![Grid-2](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/Grid-2.gif)

好多了! 动画现在看上去已经有模有样了, 但还是少点什么。TileViews应该像水波一样，向四周逐渐扩散开来。

The best way to do this is to whip out your high-school math (don’t cringe—it will be over before you know it) and normalize the vector based upon the distance of the TileView from the center.

在distanceFromCenterViewWithView(\_\:\)函数下面添加如下函数:

``` Swift
private func normalizedVectorFromCenterViewToView(view: UIView)->CGPoint {
  let length = self.distanceFromCenterViewWithView(view)
  guard let centerTileView = centerTileView where length != 0 else { return CGPointZero }
 
  let deltaX = view.center.x - centerTileView.center.x
  let deltaY = view.center.y - centerTileView.center.y
  return CGPoint(x: deltaX / length, y: deltaY / length)
}
```

回到startAnimatingWithBeginTime(_:)，将代码修改如下:

``` Swift
private func startAnimatingWithBeginTime(beginTime: NSTimeInterval) {
  for tileRows in tileViewRows {
    for view in tileRows {
 
      let distance = self.distanceFromCenterViewWithView(view)
      var vector = self.normalizedVectorFromCenterViewToView(view)
 
      vector = CGPoint(x: vector.x * kRippleMagnitudeMultiplier * distance, y: vector.y * kRippleMagnitudeMultiplier * distance)
 
      view.startAnimatingWithDuration(kAnimationDuration, beginTime: beginTime, rippleDelay: kRippleDelayMultiplier * NSTimeInterval(distance), rippleOffset: vector)
    }
  }
}
```

这会计算TileView的移动向量，并将其应用于rippleOffset。
编译运行一下.

![Grid-3](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Grid-3.gif)

棒极了! 接下来的点睛之笔就是：添加一个放大的效果， a scale animation needs to occur right before there is a change in the mask’s bounds.
在startAnimatingWithBeginTime(_:)函数的顶端，添加如下代码：

``` Swift 
  let linearTimingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionLinear)
  
  let keyframe = CAKeyframeAnimation(keyPath: "transform.scale")
  keyframe.timingFunctions = [linearTimingFunction, CAMediaTimingFunction(controlPoints: 0.6, 0.0, 0.15, 1.0), linearTimingFunction]
  keyframe.repeatCount = Float.infinity;
  keyframe.duration = kAnimationDuration
  keyframe.removedOnCompletion = false
  keyframe.keyTimes = [0.0, 0.45, 0.887, 1.0]
  keyframe.values = [0.75, 0.75, 1.0, 1.0]
  keyframe.beginTime = beginTime
  keyframe.timeOffset = kAnimationTimeOffset
 
  containerView.layer.addAnimation(keyframe, forKey: "scale")
```

再编译运行一下。

![FuberFinal](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/FuberFinal.gif)

漂亮，你已经创建了一个产品的动画效果，会大一批的Fuber用户在Twitter上为此点赞！：］（作者又卖了个萌！）
注意：试着改变kRippleMagnitudeMultiplier和kRippleDelayMultiplier的值，看看会有什么有趣的事发生。

最后在RootContainerViewController.swift文件中，将viewDidLoad()最后一行代码showSplashViewControllerNoPing()改为showSplashViewController()。
最后在编译运行一次，检查一下自己的工作成果

![Fuber Animation](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Animation.gif)

给自己点个赞吧，这是一个非常炫酷的启动页。

### 接下来？

可以在这下载到[最终的Fuber工程](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Fuber-final.zip).
如果你想了解更多关于动画的知识，请访问这里的[iOS动画教程](https://www.raywenderlich.com/store/ios-animations-by-tutorials).