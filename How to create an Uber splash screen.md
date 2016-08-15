# \[译文]如何制作一个类似Uber的溅落式启动屏

*本文翻译自 [How To Create an Uber Splash Screen](https://www.raywenderlich.com/133224/how-to-create-an-uber-splash-screen)*， *由 [Derek Selander](https://www.raywenderlich.com/u/lolgrep) 发表于Raywenderlich*。

*受限于译者英语水平及翻译经验，译文容难免有词不达意，甚至翻译错误的地方，还望不吝赐教予以指正* 。

一个好的溅落式启动页（别被毫无动画效果的静态启动页迷惑），使开发人员有机会在展示动画期间，从后端获取必要的数据。同时它在应用启动期间让用户始终保持高昂兴趣方面也发挥了重要作用。

虽然溅落式启动页已广泛存在，但是你很难找到一个如Uber这般出色的。在2016年的首季，Uber释出一个由CEO领导的品牌重塑战略，品牌重塑的成果之一，便是一个非常炫酷的溅落式启动页。

本文以仿制Uber启动动画为目标。其中运用了大量的**CAlayer**和**CAAnimation**类，及其相应子类。相对于概念介绍，本文更着重于如何运用这些类去实现一个产品级的动画效果。如需了解动画背后的相关知识，请访问 *Marin Todorov* 的系列视频教程：
[**Intermediate iOS Animation**](https://www.raywenderlich.com/u/icanzilb)

## 开始

鉴于本文涉及的动画众多，这里提供一个已为后续动画创建好所有CALayer的[起始工程](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/Fuber-starter.zip)。

起始工程是一个叫做Fuber的应用，Fuber提供（Segway）驾乘共享服务，乘客通过向Segway驾驶员发出请求，来邀请其搭载自己抵达城市的任何地方。Fuber发展迅速，已在60多个国家为用户提供服务，但也面临众多国家的反对和工会要求其必须与司机签订合同的问题。:](原作者卖萌了)

<center>![Splash screen](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/fuber\_logo-480x161.png)</center>

最终，我们会创建一个如下的非常炫酷的溅落式启动页:

<center>![Fuber Animation](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Animation.gif)</center>

打开并运行起始工程，简单浏览一下工程结构。

首先从视图控制器开始，应用通过负责子视图（切入）切出任务的**RootContainerViewController**加载**SplashViewController**。父视图控制器从启动页开始运行，直至应用的所有准备工作全部完成。这期间应用会连接到后端，获取后续所需数据。需要指出的是，在这个简单的项目中启动页被设计成了一个独立的模块。

在**RootContainerViewController**中已经实现好了两个方法：`showSplashViewController()`和 `showSplashViewControllerNoPing()`。 
由于教程中大部分时间，都在调用`showSplashViewControllerNoPing()`方法（调试启动动画），所以我们先将精力放在**SplashViewController**的子视图动画创建上，然后在通过`showSplashViewController()`模拟一个访问API的延迟效果，并随即跳转到主视图控制器。

## 溅落式启动页视图及其图层结构

**SplashViewController**的视图（view）包含两个子视图（subview）。 第一个子视图是用于构成波纹网格背景的**TileGridview**，它包含了一系列按网格排列的**TileView**实例。另一个子视图名为**AnimatedULogoView**，它构成了 U 字型的动画图标。

<center>![Splash Screen](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Fuber-View-Hierarchy-1.png)</center>

**AnimatedULogoView**包含4个**CAShapeLayer**:

- **circleLayer** 用于实现字母 U 的白色背景
- **lineLayer** 用于实现从**circleLayer**的中心到边缘的一条线段
- **squareLayer** 用于实现位于**circleLayer**中心位置的方块
- **maskLayer** 用作视图遮罩，通过改变其**bounds**的动画效果，来将其它所有图层的动画效果整齐划一地混合起来。

通过组合这几个**CAShaperLayer**动画，共同实现了**Fuber**中字母 **U** 的动画效果。

<center>![RiderIconView](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/RiderIconView.gif)</center>

了解了图层的构成之后，接下来我们就来添加一些动画让**AnimatedULogoView**动起来吧。

## 让圆形动起来

创建复杂动画的关键，在于排除视觉干扰专注于我们正在实现的部分。 打开**AnimatedULogoView.swift**文件。找到`init(frame:)`方法，注释掉除**circleLayer**外其它向视图中添加子图层（sublayer）的方法，完成动画后会再将其全部添加回来。注释完成后的代码如下:

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

找到`generateCircleLayer()`方法，了解下圆形是如何被创建的。其实只是简单地通过 *UIBezierPath* 创建了一个 *CAShapeLayer* (图层)。 注意看这行代码:

``` Swift
layer.path = UIBezierPath(arcCenter: CGPointZero, 
                             radius: radius/2, 
                         startAngle: -CGFloat(M_PI_2), 
                           endAngle: CGFloat(3*M_PI_2),
                          clockwise: true).CGPath
```

向 *startAngle* 传入 0 或使用默认值, 弧线会从右侧（3点钟位置）开始。传入 **-M\_PI\_2** 即 -90度, 则会从顶部开始，如果 *endAngle* 恰好是270度即 **3 * M\_PI\_2**，弧线则再次回到顶点（形成一个圆形）。注意为了绘制的动画效果，我们使用圆形的半径作为**lineWidth**。

**circleLayer**的动画需要三个**CAAnimation**子类来实现：一个作用于**stokeEnd**的**CAKeyframeAnimation**动画，一个作用于**transform**的**CABasicAnimation**动画，和一个负责将两部分动画组合起来的**CAAnimationGroup**。这将一次性同时创建所有动画。

在事先写好的`animateCircleLayer()`方法中添加如下代码:

``` Swift
  // strokeEnd
  let strokeEndAnimation = CAKeyframeAnimation(keyPath: "strokeEnd")
  strokeEndAnimation.timingFunction = strokeEndTimingFunction
  strokeEndAnimation.duration = kAnimationDuration - kAnimationDurationDelay
  strokeEndAnimation.values = [0.0, 1.0]
  strokeEndAnimation.keyTimes = [0.0, 1.0]
```

通过向动画的**Values**属性提供的 0.0 和 1.0，我们便透过Core Animation框架生成了一个从 *startAngle* 到 *endAngle* 沿顺时针旋转的动画。随着 *strokeEnd* 属性值的增加，弧线沿着圆周慢慢伸展，圆形也渐渐被"填满"。在这个例子中，如果我们将**values**属性的值设为[0.0, 0.5]，则仅会画半个圆，这是因为 *StrokeEnd* 在动画结束时刚达好到圆周的一半。

> 译者注：“圆形也渐渐被‘填满’”一句的填满是引起来的，并不是真的被填满，而是描边的 *lineWidth* 与圆形半径相同，从而产生了填满的视觉效果。可参考`generateCircleLayer()`方法中`layer.fillColor = UIColor.clear.cgColor`这段代码，事实上填充色被设置为透明，

现在添加形变（transform）动画:

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

该动画同时实现了放大和沿 Z 轴旋转的两个形变。这使得圆形在沿顺时针旋转45度的同时逐渐变大。这里的旋转很重要，因为圆形的旋转要与**lineLayer**和其它图层一块动起来时的位置和速度保持一致。

最后在`animateCircleLayer()`方法的最下面添加一个**CAAnimationGroup**。这个动画组将包含之前的两个动画，这样我们仅向**circleLayer**图层添加一次动画即可。

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

这里我们修改了CAAnimationGroup的两个重要属性：**beginTime** 和 **timeOffset**。如果你对其中任何一个不熟悉，那么你都可以在[这里](http://ronnqvi.st/controlling-animation-timing/)找到关于该属性的介绍和使用说明。
将 **groupAnimation** 的 **beginTime** 设置为与父视图相同。

对**timeOffeset**的设置是必要的，因为动画首次运行时实际上是从一半开始的。当完成更多动画效果后，你可以试着改变**startTimeOffset**的值，并观察动画在视觉效果上的不同。 

将动画组添加到**circleLayer**之后，编译并运行应用，检查下动画效果.

<center>![Splash Screen CircleIn Animation](https://cdn4.raywenderlich.com/wp-content/uploads/2016/05/CircleIn-Animation.gif)</center>

>注意: 试着删除**groupAnimation.animations**数组中的**strokeEndAnimation**或**transformAnimation**，以确认每个动画具体实现了哪些视觉效果. 可以按该方法再去验证一下文中的其它动画，你会惊讶于，仅仅改变动画的组合方式就可以产生如此令人难以预料的独特视觉效果.


## 让线段动起来

完成了**circleLayer**的动画, 接下来我们再来完成**lineLayer**动画。还是在 **AnimatedULogoView.swift**文件中, 找到`startAnimating()`方法并注释掉除`animateLineLayer()`外的所有动画调用。注释后的代码如下:

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

此外, 修改`init(frame:)`方法中的代码，只显示**circleLayer**和**lineLayer**两个图层:

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

注释掉图层和动画后, 转到`animateLineLayer()`方法并实现下面这组动画:

``` Swift
  // lineWidth
  let lineWidthAnimation = CAKeyframeAnimation(keyPath: "lineWidth")
  lineWidthAnimation.values = [0.0, 5.0, 0.0]
  lineWidthAnimation.timingFunctions = [strokeEndTimingFunction, circleLayerTimingFunction]
  lineWidthAnimation.duration = kAnimationDuration
  // Swift 3.0 keyTimes是一个NSNumber数组
  lineWidthAnimation.keyTimes = [0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]
```

该动画会使**lineLayer**的宽度（width）呈现出先增后减的效果。

再为接下来的动画添加如下代码:

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

与**circleLayer**的形变动画非常相似, 这里我们定义了个一个沿 Z 轴顺时针旋转的动画。 此外我们还为线段添加了一个先缩小到25%，再恢复到原有尺寸，最后再缩小到15%的形变动画.

通过**CAAnimationGroup**将动画组合起来，并添加到**lineLayer**上：

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

编译并运行，注意观察变化.

<center>![Splash Screen Knockoutline Animation](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Knockoutline-Animation.gif)</center>

注意我们设置了相同的初始形变值**-M\_PI\_4**，以便线段（line）和圆形（circle）在绘制时能对齐。为此我们还将**keyTimes** 设置为`[0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]`。 数组中首尾两个元素是确定的: 0 代表动画开始那一刻，1.0 代表动画结束那一刻，然后通过计算来获取圆形绘制刚刚结束、第二部分的动画即将开始时的那一刻。由于它是一个延迟的动画效果，所以我们还需要从 1.0 中减去通过**kAnimationDurationDelay**除以**kAnimationDuration**而得到的确切百分比，这是因为我们想让动画在结束后的延迟过程中再返回到起点。（译者：形成一个循环动画，否则会出现跳跃，致使动画不连贯）

**circleLayer**和**lineLayer**动画都已完成，接下来我们该完成中间的方块动画了。

## 让方块动起来

与之前类似。 在`startAnimating()`函数中注释掉除`animateSquareLayer`外的其它动画方法调用。然后在像下面这样修改`init(frame:)`方法的代码：

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

完成后转到`animateSquareLayer()`方法实现如下动画代码:

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

这一部分动画用于改变**CALayer**的大小（bounds）。创建一个先将其边长缩小到2/3，再恢复，最终在缩小到零的关键帧动画。

接下来，为背景色添加动画效果:

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

注意上面的**fillMode**属性。一旦**beginTime**不为零时, 动画就会在起始点和结束点保持住当前颜色（CGColor）。这避免了动画在被添加到父CAAnimationGroup时出现闪烁。（译者：这里译的不好:(。请试着改变该属性的设置，看看视觉效果上有什么不同，以加深理解。）

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

编译并运行检查动画效果。注意观察方块的变化。

<center>![Splash Screen Tutorial](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/KnockoutSquare-Animation.gif)</center>

现在将所有的动画组合起来看看效果如何！

>注意: 在电脑的GPU完成对iOS设备的模拟任务前，模拟器上的动画可能会有那么一点小抽。如果你的电脑带不动动画，可以试着将模拟器窗口调小或者转到真机开发。

## 遮罩

首先，取消`init(frame:)`方法中对所有添加图层方法的注释，以及`startAnimating()`方法中对所有动画调用的注释.

组合好所有动画后，再次编译并运行。

<center>![PreMask Animation](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/PreMask-Animation.gif)</center>

看上去还是有点怪怪的，是不是？圆形在缩小时，它的边缘会有一个小跳跃。幸运地是, 遮罩动画可以解决该问题，让所有子图动画平滑整齐划一.

转到｀animateMaskLayer()｀方法并添加如下代码:

``` Swift
  // bounds
  let boundsAnimation = CABasicAnimation(keyPath: "bounds")
  boundsAnimation.fromValue = NSValue(CGRect: CGRect(x: 0.0, 
                                                     y: 0.0, 
                                                 width: radius * 2.0, 
                                                height: radius * 2))
  boundsAnimation.toValue = NSValue(CGRect: CGRect(x: 0.0, 
                                                   y: 0.0, 
                                               width: 2.0/3.0 * squareLayerLength,
                                              height: 2.0/3.0 * squareLayerLength))
  boundsAnimation.duration = kAnimationDurationDelay
  boundsAnimation.beginTime = kAnimationDuration - kAnimationDurationDelay
  boundsAnimation.timingFunction = circleLayerTimingFunction
```

这是一个边界(bounds)动画。记住，由于这是一个应用于所有子图层的遮罩，当边界发生变化时, 整个**AnimatedULogoView**都将消失，直至遮罩被应用到所有子图层。

现在在添加一个让方块变圆的圆角动画:

``` Swift
  // cornerRadius
  let cornerRadiusAnimation = CABasicAnimation(keyPath: "cornerRadius")
  cornerRadiusAnimation.beginTime = kAnimationDuration - kAnimationDurationDelay
  cornerRadiusAnimation.duration = kAnimationDurationDelay
  cornerRadiusAnimation.fromValue = radius
  cornerRadiusAnimation.toValue = 2
  cornerRadiusAnimation.timingFunction = circleLayerTimingFunction
```

将这两个动画添加到一个**CAAnimationGroup**中，以完成这个图层（的所有动画）：

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

<center>![RiderIconView Animation](https://cdn4.raywenderlich.com/wp-content/uploads/2016/05/RiderIconView-Animation.gif)</center>

看起来好多了！

## 网格

试想一下有一系列以**TileGridView**实例的方式来移动的 _UIView_。 它们看起来会是什么样呢？呃。。。这里就不引用[创](https://zh.wikipedia.org/wiki/創：光速戰記)并展开说明了！（译者：《创》是一部科幻电影。这里翻译的不好，见谅！）

网格背景由一些列附加到**TileGridView**类的**TileView**组成。为了便于从视觉上理解这个概念, 我们打开**TileView.swift**文件，找到`init(frame:)`方法，在方法的最后添加如下代码:

``` Swift
layer.borderWidth = 2.0
```

编译并运行应用。

<center>![Fuber-Grid-View](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Grid-View-180x320.png)</center>

如果你所见，**TileView**被整齐地排成一张网格。整个创建逻辑都集中在**TileGridView.swift**文件的`renderTileViews()`方法内。幸运的是，我们所需的布局逻辑（起始工程）已经实现好。接下来要做的就是让它动起来!

## 让瓦片视图（TileView）动起来

**TileGridView**仅有一个直接的子视图（subview）**containerView**。它负责添加所有的**TileView**。 此外，还有一个名为**tileViewRows**的属性, 它是一个二维数组，包含所有添加到containerView中的**TileView**。

回到**TileView**中的`init(frame:)`方法. 删除我们刚才添加的用于显示边界的代码，并取消向图层中添加**chimeSplashImage**方法的注释。完成后的方法如下:

``` Swift
override init(frame: CGRect) {
  super.init(frame: frame)
  layer.contents = TileView.chimesSplashImage.CGImage
  layer.shouldRasterize = true
}
```

编译并运行。

<center>![Grid Starting](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/Grid-Starting.gif)</center>

酷。。。。我们就要大功告成了。

然而，**TileGridView**（以及它的**TileView**们）还需要添加一些动画效果。打开**TileView.swift**文件，找到`startAnimatingWithDuration(_:beginTime:rippleDelay:rippleOffset:)` 方法并添加如下动画代码:

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
    scaleAnimation.timingFunctions = [linearFunction, 
                                      timingFunction, 
                                      timingFunction, 
                                      linearFunction]
    scaleAnimation.beginTime = 0.0
    scaleAnimation.duration = duration
    animations.append(scaleAnimation)
 
    // Position
    let positionAnimation = CAKeyframeAnimation(keyPath: "position")
    positionAnimation.duration = duration
    positionAnimation.timingFunctions = [linearFunction, 
                                         timingFunction, 
                                         timingFunction, 
                                         linearFunction]
    positionAnimation.keyTimes = TileView.rippleAnimationKeyTimes
    positionAnimation.values = [zeroPointValue, 
                                zeroPointValue, 
                                NSValue(CGPoint:rippleOffset), 
                                zeroPointValue, 
                                zeroPointValue]
    positionAnimation.additive = true
 
    animations.append(positionAnimation)
  }
```

**shouldEnableRipple**是个布尔值，用于控制何时将形变动画和位置动画添加到我们刚刚创建的数组中。在通过`renderTileViews()`方法创建时，所有未处在**TileGridView**外围边缘的**TileView**，就已将**shouldEnableRipple**设为**true**。

添加一个不透明动画:

``` Swift
  // Opacity
  let opacityAnimation = CAKeyframeAnimation(keyPath: "opacity")
  opacityAnimation.duration = duration
  opacityAnimation.timingFunctions = [easeInOutTimingFunction, 
                                      timingFunction, 
                                      timingFunction, 
                                      easeOutFunction, 
                                      linearFunction]
  opacityAnimation.keyTimes = [0.0, 0.61, 0.7, 0.767, 0.95, 1.0]
  opacityAnimation.values = [0.0, 1.0, 0.45, 0.6, 0.0, 0.0]
  animations.append(opacityAnimation)
```

该动画简单明了，只是设置了一些非常特殊的的**keyTimes**。

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

这会将**groupAnimation**添加到**TileView**实例上。注意，动画组会因**shouldEnableRipple**值的不同而可能包含一个或三个动画。

现在我们已经为每一个**TileView**实现了动画, 接下来需要在**TileGridView**中去调用它们。打开**TileGridView.swift**文件将以下代码添加到`startAnimatingWithBeginTime(_:)`方法中:

``` Swift 
private func startAnimatingWithBeginTime(beginTime: NSTimeInterval) {
  for tileRows in tileViewRows {
    for view in tileRows {
      view.startAnimatingWithDuration(kAnimationDuration, beginTime: beginTime, 
      						                                 rippleDelay: 0, 
      						                                rippleOffset: CGPointZero)
    }
  }
}
```

编译并运行。

<center>![Grid-1](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Grid-1.gif)</center>

嗯。。。看上去已经好多了，但**AnimatedULogoView**的跳动应该通过**TileView**向外产生一个类似水波的涟漪效果。这就意味还需要创建一个，基于从中央视图（view）到外围视图之间距离的，用于与一个常数相乘的延迟系数。

紧挨着`startAnimatingWithBeginTime(_:)`函数下面, 添加如下的一个新函数:

``` Swift
private func distanceFromCenterViewWithView(view: UIView)->CGFloat {
  guard let centerTileView = centerTileView else { return 0.0 }
 
  let normalizedX = (view.center.x - centerTileView.center.x)
  let normalizedY = (view.center.y - centerTileView.center.y)
  return sqrt(normalizedX * normalizedX + normalizedY * normalizedY)
}
```

该方法可以便捷地获取到，指定视图与位于中心的视图，两个视图（_TileView_）中心点之间的距离。

回到`startAnimatingWithBeginTime(_:)`函数，将其内容替换为如下代码:

``` Swift
  for tileRows in tileViewRows {
    for view in tileRows {
      let distance = self.distanceFromCenterViewWithView(view)
 
      view.startAnimatingWithDuration(kAnimationDuration, beginTime: beginTime, rippleDelay: kRippleDelayMultiplier * NSTimeInterval(distance), rippleOffset: CGPointZero)
    }
  }
```

这里通过刚刚添加的`distanceFromCenterViewWithView(_:)`函数，来计算（每个子视图）动画的延迟启动时间。

编译运行.

<center>![Grid-2](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/Grid-2.gif)</center>

好多了! 动画现在看上去已经有模有样了, 但还是少点什么。**TileView**应该像水波一样，向四周逐渐扩散开来。

解决该问的最好方法就是拿出自己的高中数学知识，然后根据**Tileview**与中心点间距离来得到一个量化的顶点。

在`distanceFromCenterViewWithView(_:)`函数下面再添加一个函数:

``` Swift
private func normalizedVectorFromCenterViewToView(view: UIView)->CGPoint {
  let length = self.distanceFromCenterViewWithView(view)
  guard let centerTileView = centerTileView where length != 0 else { return CGPointZero }
 
  let deltaX = view.center.x - centerTileView.center.x
  let deltaY = view.center.y - centerTileView.center.y
  return CGPoint(x: deltaX / length, y: deltaY / length)
}
```

回到`startAnimatingWithBeginTime(_:)`方法，将代码修改如下:

``` Swift
private func startAnimatingWithBeginTime(beginTime: NSTimeInterval) {
  for tileRows in tileViewRows {
    for view in tileRows {
 
      let distance = self.distanceFromCenterViewWithView(view)
      var vector = self.normalizedVectorFromCenterViewToView(view)
 
      vector = CGPoint(x: vector.x * kRippleMagnitudeMultiplier * distance, 
                       y: vector.y * kRippleMagnitudeMultiplier * distance)
 
      view.startAnimatingWithDuration(kAnimationDuration, 
                                      beginTime: beginTime, 
                                      rippleDelay: kRippleDelayMultiplier * NSTimeInterval(distance), 
                                      rippleOffset: vector)
    }
  }
}
```

这会通过 _rippleOffset_ 计算位于每个顶点（vector）的 _TileView_ 的偏移量。

编译运行一下.

<center>![Grid-3](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Grid-3.gif)</center>

太棒了! 接下来是点睛之笔：添加一个放大的效果，这个放大的动画效果要刚好在遮罩边界（bounds）发生改变之前。

在`startAnimatingWithBeginTime(_:)`函数的开始位置，添加如下代码：

``` Swift 
  let linearTimingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionLinear)
  
  let keyframe = CAKeyframeAnimation(keyPath: "transform.scale")
  keyframe.timingFunctions = [linearTimingFunction, 
                              CAMediaTimingFunction(controlPoints: 0.6, 0.0, 0.15, 1.0), 
                              linearTimingFunction]
  keyframe.repeatCount = Float.infinity;
  keyframe.duration = kAnimationDuration
  keyframe.removedOnCompletion = false
  keyframe.keyTimes = [0.0, 0.45, 0.887, 1.0]
  keyframe.values = [0.75, 0.75, 1.0, 1.0]
  keyframe.beginTime = beginTime
  keyframe.timeOffset = kAnimationTimeOffset
 
  containerView.layer.addAnimation(keyframe, forKey: "scale")
```

再次编译并运行。

<center>![FuberFinal](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/FuberFinal.gif)</center>

漂亮，我们已经创建了一个产品级的动画效果，会有大一批的Fuber用户在微博（Twitter）上为此点赞的！：］（作者又卖了个萌！）

>注意：试着改变**kRippleMagnitudeMultiplier**和**kRippleDelayMultiplier**的值，看看会有什么有趣的事发生。

接下收尾，在**RootContainerViewController.swift**文件中，将`viewDidLoad()`最后一行代码`showSplashViewControllerNoPing()`改为`showSplashViewController()`。

最后在编译运行一次，欣赏下自己的工作成果吧

<center>![Fuber Animation](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Animation.gif)</center>

给自己点个赞吧，这是一个非常炫酷的溅落式启动页。

## 接下来

可以在这下载到[最终的Fuber工程](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Fuber-final.zip).

您还可以在这里找到由译者更新至Swift 3.0的[最终的Fuber工程](https://github.com/Dracarys/Fuber-final)，请使用Xcode 8.0 beta4 或更新版本打开。

如果想了解更多关于动画的知识，请访问这里的[iOS动画教程](https://www.raywenderlich.com/store/ios-animations-by-tutorials).