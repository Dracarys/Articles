# \[翻译\]如何制作一个类似Uber的启动页

_本文翻译自_[How To Create an Uber Splash Screen](https://www.raywenderlich.com/133224/how-to-create-an-uber-splash-screen) _由 [Derek Selander](https://www.raywenderlich.com/u/lolgrep) 发表于Raywenderlich_

Oh, the wonderful splash screen—a chance for developers to go wild with fun animations as the app frantically pings API endpoints for critical data needed to function. 为了让用户在等待应用启动的过程中始终保持高昂的兴趣，一个涟漪式启动页就变的尤为重要。

虽然涟漪式启动页已广泛应用，但是你很难找到一个如Uber这般出色的。在2016年的首季，Uber启动了一个由CEO领导的用户体验重塑计划，该计划的成果之一,便是一个炫酷的涟漪式启动页。

本教程以仿制Uber启动动效为目标。其中运用了大量的CAlayer和CAAnimation类，及其相应子类。相比这些类的概念介绍，本教程更着重于如何运用这些类去实现一个产品级的动效。如需了解动画相关概念，可以访问Marin Todorov的系列视频教程：
[_Intermediate iOS Animation_](https://www.raywenderlich.com/u/icanzilb)

### 开始


Since there are a significant number of animations to implement in this tutorial, you’ll start with a project that already has the CALayers created for all the lovely animations to come. Download the starter project here.
起始工程是一个叫做Fuber的App，Fuber提供驾乘共享服务，乘客可请求一位Segway驾乘人员，以便搭载，抵达城市的任何地方。Fuber发展迅速，已在60多个国家为用户提供服务，但也面临众多国家的反对和工会必须与司机签订合同的要求。：］(这里原作者卖了个萌)

![Splash screen](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/fuber\_logo-480x161.png)

最终，我们会创建一个如下图所示的涟漪式启动页:

![Fuber Animation](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Animation.gif)

打开起始工程，运行并简单浏览。

首先从视图控制器开始，应用通过父试图控制RootContainerViewController加载SplashViewController，并由其负责子视图控制器的切出工作. 父视图控制器从涟漪动画开始运行，直至应用的准备工作全部完成。This could happen when there is a handshake success to an API endpoint and the app has the necessary data to continue.需要指出的是，涟漪启动页在整个工程中是一个独立的模块。

其中 _RootContainerViewController\:_ _showSplashViewController\(\)_ 和 _showSplashViewControllerNoPing\(\)_两个函数已经实现. 



For the majority of this tutorial, you will call showSplashViewControllerNoPing() which only loops through the animations, so that you can focus on animating the subviews in SplashViewController and later on you will use showSplashViewController() to simulate an API delay then transition to the main controller.

### 启动视图及层构成

SplashViewController的view包含两个subview。 第一个subview是由细网格构成的TileGridview， which contains a grid-like layout of subview instances called TileView. 另一个subview由一个可动的U型图标构成，The other subview consists of the animating ‘U’ icon, known as the AnimatedULogoView.

![Splash Screen](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Fuber-View-Hierarchy-1.png)

AnimatedULogoView包含4个CAShapeLayers:

- circleLayer represents the circular white background for the ‘U’.
- lineLayer is the straight line that goes from the middle of the circleLayer layout to its outer edge.
- squareLayer is the square that lies in the center of the circleLayer layer.
- maskLayer is used as a mask for the view. It is used to collapse all the other layers in one easy animation when its bounds are changed through animation.

Combined, these CAShaperLayers create the Fuber ‘U’.

![RiderIconView](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/RiderIconView.gif)

Now that you know how these layers are composed, it’s time to generate the animations to make the AnimatedULogoView animatable.

###Animating the Circle

When working with animations, it’s best to eliminate visual “noise” and focus on the animation currently being implemented. Navigate to AnimatedULogoView.swift. In init(frame:), comment out all the sublayers being added to the view except the circleLayer layer. You’ll add them back in one at a time as you complete the animations. The code should now look like this:

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

Find generateCircleLayer() to understand how the circle is created. It’s a simple CAShapeLayer created with a UIBezierPath. Pay attention to this line:

``` Swift
layer.path = UIBezierPath(arcCenter: CGPointZero, 
                             radius: radius/2, 
                         startAngle: -CGFloat(M_PI_2), 
                           endAngle: CGFloat(3*M_PI_2),
                          clockwise: true).CGPath
```

By default and if you provide 0 as the startAngle, arc bezier paths start from the right (3 o’clock position). By providing -M_PI_2 which is -90 degrees, you start from the top and the endAngle would be 270 degrees or 3*M_PI_2 which again is the top of the circle. Also pay attention that because you want to animate the stroke, you are using the radius value as the lineWidth.
The circleLayer animation needs to be comprised of three CAAnimations: a CAKeyframeAnimation animating a strokeEnd, a CABasicAnimation animating a transform and a CAAnimationGroup used to wrap the two together. You’ll create these animations one at a time.
Navigate to the animateCircleLayer() placeholder and add the following:

``` Swift
  // strokeEnd
  let strokeEndAnimation = CAKeyframeAnimation(keyPath: "strokeEnd")
  strokeEndAnimation.timingFunction = strokeEndTimingFunction
  strokeEndAnimation.duration = kAnimationDuration - kAnimationDurationDelay
  strokeEndAnimation.values = [0.0, 1.0]
  strokeEndAnimation.keyTimes = [0.0, 1.0]
```

By providing 0.0 and 1.0 as the animation values you instruct the Core Animation framework to start from the startAngle and stroke the circle to the endAngle, creating the cool “clock-like” animation. So as the value of strokeEnd increases, the length of the line segment along the circumference increases, and the circle is gradually “filled in”. For this particular example, if you were to change the values property to [0.0, 0.5], only half of the circle would be drawn because the strokeEnd would only reach half-way around the circle’s circumference during the animation.
Now add the transform animation:

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

This animation performs both a scale transform and a rotational transform about the z-axis. This results in the circleLayer layer growing while rotating clockwise by 45 degrees. This rotation is important because it needs to match the position and speed of the lineLayer when it’s being animated along with the rest of the layers.
Finally, add a CAAnimationGroup to the bottom of animateCircleLayer(). This animation encapsulates the previous two animations, so that you only need to add one animation to the circleLayer layer.

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

This CAAnimationGroup has two notable properties being modified: beginTime and timeOffset. If you are unfamiliar with either one, a great description of these properties and how they’re used can be found here.
This groupAnimation‘s beginTime is being set in reference to the timing of its parent view.
The timeOffset is needed because the animation actually starts halfway through on its first run. When you have more animations completed, try changing the value of startTimeOffset and observe the visual differences.
Add the groupAnimation to circleLayer, then build and run the application to see what it looks like.

![Splash Screen CircleIn Animation](https://cdn4.raywenderlich.com/wp-content/uploads/2016/05/CircleIn-Animation.gif)

Note: Try removing either the strokeEndAnimation or transformAnimation in the groupAnimation.animations array to really get an idea of what each animation does. Try to experiment like this for each animation you create in this tutorial. You’ll be suprised how different combinations of animations can produce unique visuals you would never have anticipated.


### Animating the Line

With the animations of circleLayer complete, it’s time to address the lineLayer‘s animations. While still in AnimatedULogoView.swift, navigate to startAnimating() and comment out all the calls to animating methods except animateLineLayer(). The result should look like the code below:

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

In addition, change the content in init(frame:) so that circleLayer and lineLayer are the only CALayers being used:

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

With the CALayers/animations properly commented out, go to animateLineLayer() and implement the next group of animations:

``` Swift
  // lineWidth
  let lineWidthAnimation = CAKeyframeAnimation(keyPath: "lineWidth")
  lineWidthAnimation.values = [0.0, 5.0, 0.0]
  lineWidthAnimation.timingFunctions = [strokeEndTimingFunction, circleLayerTimingFunction]
  lineWidthAnimation.duration = kAnimationDuration
  lineWidthAnimation.keyTimes = [0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0]
```

This animation is responsible for increasing then decreasing the lineLayer‘s width.
For the next animation, add the following:

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

Much like the circleLayer transform animation, here you’re defining a clockwise rotation about the z-axis. For the line, however, you’re also performing a 25% scale transform, quickly followed by an identity transform before a final scale to 15% of its original size.
Group the animations together using a CAAnimationGroup and add it to lineLayer:

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

Build and run, and observe the prettiness.

![Splash Screen Knockoutline Animation](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/Knockoutline-Animation.gif)

Note that you used the same -M_PI_4 initial transform value to align the line and the stroke of the circle. You also used [0.0, 1.0-kAnimationDurationDelay/kAnimationDuration, 1.0] for keyTimes. The first and last elements of the array are obvious: 0 means start and 1.0 means end so to get the middle value you want to calculate when the circle stroke is complete and the second part (shrinking) will happen. Dividing kAnimationDurationDelay by kAnimationDuration gets you to the exact percentage but because it’s a delayed animation, you subtract it from 1.0 because you want to go back by the duration of the delay from when the animation ends.
You’ve now checked off the circleLayer and the lineLayer animations, so it’s time to move on to the square in the center.

###Animating the Square

The drill should be getting familiar by now. Go to startAnimating() and comment out the animation method calls except for animateSquareLayer(). In addition, change init(frame:) so it looks like this:

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

Once done, head over to animateSquareLayer() and get cracking on the next set of animations:

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

This particular animation changes the CALayer‘s bounds. A keyframe animation is created that goes from two-thirds the length, to the full length, then to zero.
Next, animate the background color:

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

Take note of the fillMode property. Since the beginTime is non-zero, the animation will clamp both the starting and ending CGColors to the animation. This results in no flickering from the animations when added to the parent CAAnimationGroup.
Speaking of which, it’s time to implement that:

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

Build and run to check your progress. You’ve now taken care of the squareLayer.
![Splash Screen Tutorial](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/KnockoutSquare-Animation.gif)

Time to combine the animations and see this animation come together!

Note: Remember that animations on the simulator could be a bit jagged since your computer is emulating work typically done on the GPU of your iOS device. If your computer can’t keep up with the animations, try switching to a simulator with a smaller screen or develop on an actual iOS device.

###The Mask

First, uncomment all the layers being added in init(frame:) and uncomment all the animations in startAnimating().
With all the animations put together, build and run Fuber.

![PreMask Animation](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/PreMask-Animation.gif)

Still looks a bit off, doesn’t it? There’s a sudden jump in the bounds when the circleLayer collapses in size. Fortunately, the mask animation will fix that, shrinking the sublayers all in one smooth go.
Go to animateMaskLayer() and add the following:

``` Swift
  // bounds
  let boundsAnimation = CABasicAnimation(keyPath: "bounds")
  boundsAnimation.fromValue = NSValue(CGRect: CGRect(x: 0.0, y: 0.0, width: radius * 2.0, height: radius * 2))
  boundsAnimation.toValue = NSValue(CGRect: CGRect(x: 0.0, y: 0.0, width: 2.0/3.0 * squareLayerLength, height: 2.0/3.0 * squareLayerLength))
  boundsAnimation.duration = kAnimationDurationDelay
  boundsAnimation.beginTime = kAnimationDuration - kAnimationDurationDelay
  boundsAnimation.timingFunction = circleLayerTimingFunction
```

This is the animation for the bounds. Remember that when the bounds change, the whole AnimatedULogoView will disappear since this layer is the mask that’s applied to all the sublayers.
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

Add these two animations to a CAAnimationGroup to complete this layer:

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

###The Grid

A digital frontier. Try to picture clusters of UIViews as they move through the TileGridView instance. What do they look like? Well … time to stop making references to Tron and take a look!
The background grid consists of a series of TileViews all attached to the parent TileGridView class. To get a quick visual understanding of this, open up TileView.swift and find init(frame:). Add the following to the bottom of this method:

``` Swift
layer.borderWidth = 2.0
```

Build and run the application.
![Fuber-Grid-View](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/Fuber-Grid-View-180x320.png)

As you can see, the TileViews are arranged so that they’re stacked together in a grid. The creation of all this logic happens in a method called renderTileViews() in TileGridView.swift. Fortunately, the logic is already created on your behalf for this grid layout. All you need to do is animate it!

### Animating the TileView

TileGridView has a single direct child subview called containerView. It adds all the child TileViews. In addition, there is a property called tileViewRows, which is a two-dimensional array containing all the TileViews added to the container view.
Navigate back to TileView‘s init(frame:). Remove the line you added to show the border width and enable the commented-out line that adds the chimeSplashImage to the layer’s contents. The method should now look like this:

``` Swift
override init(frame: CGRect) {
  super.init(frame: frame)
  layer.contents = TileView.chimesSplashImage.CGImage
  layer.shouldRasterize = true
}
```

编译并运行。
![Grid Starting](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/Grid-Starting.gif)

棒极了。。。。马上就要完成了。
However, TileGridView (and all of its TileViews) needs some animation. Open up TileView.swift, go to startAnimatingWithDuration(_:beginTime:rippleDelay:rippleOffset:) and plop down the next chunk of animations:

``` Swift
  let timingFunction = CAMediaTimingFunction(controlPoints: 0.25, 0, 0.2, 1)
  let linearFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionLinear)
  let easeOutFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseOut)
  let easeInOutTimingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut)
  let zeroPointValue = NSValue(CGPoint: CGPointZero)
 
  var animations = [CAAnimation]()
```

This code sets up a series of timing functions you’ll use right now. Add this code:
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

shouldEnableRipple是个布尔值， that controls when the transform and position animations are added to the animations array you just created. Its value is set to true for all the TileViews that are not on the perimeter of the TileGridView. This logic is already done for you when the TileViews are created in the renderTileViews() method of TileGridView.
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
Now that you’ve written the method to animate each TileView, it’s time to call it from TileGridView. Head over to TileGridView.swift and add the following code to startAnimatingWithBeginTime\(\_\:\)\:

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