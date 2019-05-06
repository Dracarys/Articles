# 面试题系列之常用三方框架

## 1. 网络库

### 1.1 AFNetworking，是否支持IPv6
3.0 之后支持
### 1.2 AFNetwoking 为什么添加一条常驻线程？
网络请求是异步的，这会导致获取到请求数据时，线程已经退出，代理方法没有机会执行。因此，AFN 的做法是使用一个 runloop 来保证线程不死~
然而频繁的创建线程并启动runloop肯定会造成内存泄露(runloop 无法停止.线程无法退出)
所以 AFN 就创建了一个单例线程,并且保证线程不退出.

>[来源参考](https://www.jianshu.com/p/7170035a18e8)

> 这个问题是在 AFNetworking 未更新到 3.0 之前时存在的，主要原因在与 NSURLConnection。在 Apple 推出 NSURLSession 后，作者果断的摒弃了旧方式，转而使用 NSURLSession，因为它所带来的优化太多了，复用、多连接、管道等等。

### 1.3 AFNetworking 的 reachability 是如何检测到网络状态变化的？
要获取设备状态，必然少不了引入 `SystemConfiguration.framework` 框架。AFNetworking 自然也不例外，它创建一个 `SCNetworkReachabilityRef`，并将其添加到主线程的 RunLoop 中，与 `kCFRunLoopCommonModes` 关联。这样当状态发生变化时，就会收到回调。

## 2. 图片库
在 Swift 诞生之前最著名的图片库时 SDWebImage，在 Swift 诞生后，又出现了一个原生的 Kingfisher。

### 2.1 SDWebImage 图片编解码
SDWebImage 设置有单独的解码模块，当向缓存请求某个图片时，如果命中，那么此时就会对图片数据进行解码操作，然后在返回。因为请求是在后台线程中进行的，所以也就避免了在赋值时在主线程进行解码，今儿因为解码而出现阻塞的问题。如果不明中，会进入下载流程，下载完成后，在重复之前的查询过程。

### 2.2 Kingfisher 图片编解码

## 3. 模型库
日常开发中最长用到的模型库就是“缓存”，而提到缓存就不得不说到 YYCache。从各方面来讲，它都是一个十分优秀的开源库，非常值得学习。此外还有 YYMode、YYImage、YYAsyncLayer，也都非常值得深入学习。
### 3.1 YYCache

### 3.2 YYMode

### 3.3 YYAsyncLayer如何进行异步绘制的？

### 3.4 YYImage 图片编解码

## 4. 三方库管理工具

### 4.1 Pod 的工作原理

### 4.2 Pod update和 Pod install的区别

## 5. 其它

### 5.1 FB 的 Async 库都做了什么？
