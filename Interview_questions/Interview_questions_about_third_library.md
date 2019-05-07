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
YYCache 采用最近最少使用（Least Recently Used，LRU）算法。它综合了链表的快速编辑，以及哈希表的快速查询特性，以实现 LRU 算法。

### 3.4 YYImage 图片编解码
与 SDWebImage 的解码逻辑类似，也是放到后代队列进行预先解码，以避免在赋值后由主线程队列解码。

## 4. 三方库管理工具

### 4.1 Pod 的工作原理
CocoaPods的工作主要是通过ProjectName.xcworkspace来组织的，在打开ProjectName.xcworkspace文件后，发现Xcode会多出一个Pods工程。

1. 库文件引入及配置：库文件的引入主要由Pods工程中的Pods-ProjectName-frameworks.sh脚本负责，在每次编译的时候，该脚本会帮你把预引入的所有三方库文件打包的成ProjectName.a静态库文件，放在我们原Xcode工程中Framework文件夹下，供工程使用。
如果Podfile使用了use_frameworks!,这是生成的是.framework的动态库文件。引入方式也略有不同。
2. Resource文件：Resource资源文件主要由Pods工程中的Pods-ProjectName-resources.sh脚本负责，在每次编译的时候，该脚本会帮你将所有三方库的Resource文件copy到目标目录中。
3. 依赖参数设置：在Pods工程中的的每个库文件都有一个相应的SDKName.xcconfig，在编译时，CocoaPods就是通过这些文件来设置所有的依赖参数的，编译后，在主工程的Pods文件夹下会生成两个配置文件，Pods-ProjectName.debug.xcconfig、Pods-ProjectName.release.xcconfig。

### 4.2 Pod update和 Pod install 的区别
**结论**

- `pod install` 用于向你的工程安装新的 pods，那怕 `Podfile` 已经存在，且已运行过 `pod install`；即使，你仅仅是通过 CocoaPods 对 pods 进行增/删（编辑）。
- `pod update [PODNAME]` 只有在你想要升级 pods 到新版本时，才应用使用它。

**pod install**

根本区别就在于 `Podfile.lock` 文件，它会在首次调用 `pod install` 时生成。该文件用于跟踪、锁定已安装的每个 pod 的版本，所以版本控制时最好也将它一并提交，这样就能保证所有分发到副本都能安装到一致的依赖。

当再次调用 `pod install` 命令时，它将只解析那些尚未被 `Podfile.lock` 跟踪的依赖。对于已经存在于 `Podfile.lock` 中的依赖，则只是下载对应的版本，而不会检查是否有更新。对于新增的依赖，则会根据你在 `Podfile` 中的描述下载对应版本。（例如：pod 'MyPod', '~>1.2'）

**pod outdated**

理解了 `Podfile.lock`，这个命令容易了，它会列出那些相对于 `Podfile.lock` 现存记录有更新的依赖库。也就是说，如果此时你运行 `pod update PODNAME`，相应的依赖将更新到新版本，新版本与你在 `Podfile` 中指定的版本相对应，例如：pod 'MyPod', '~>x.y'。

**pod update**

如果运行 `pod update PODNAME`，CocoaPods 会忽略 `Podfile.lock` 中的版本记录，而直接尝试查询并更新对应的依赖。它会将依赖更新到尽可能新的版本（不会高过 Podfile 中的限制，如果有的话）。

如果运行时为制定具体依赖，那么它将更新 Podfile 中列明的所有依赖。

## 5. 其它

### 5.1 FB 的 Async 库都做了什么？
