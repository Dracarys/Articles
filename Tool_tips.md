# 【整理】Tool tips

## 1、显示Info.plist原始键/值

默认情况下，为了便于阅读Info.plist中显示的键/值并不是真实的键/值名称。但如果想查看 actual key 怎么办呢？
在Info.plist的任意项目，点击鼠标右键->Show Raw keys/values

## 2、pod install vs. pod update

很多人都这样认为，第一次配置时用 `pod install`，之后就全用 `pod update`，还有一些博客说从 github 上 clone 下来的项目全用 `pod update`，这些都不完全对。

### pod install

初次运行 `pod install` 命令，除了会解析依赖，下载依赖外，还会生成一个 `Podfile.lock` 文件，对所有的依赖进行版本追踪。如果之后再次运行 `pod install` 命令，将会检查并下载安装 `Podfile.lock` 文件中记录的版本，跳过解析的过程，除非某依赖未包含在 `Podfile.lock` 中。所以为了保持依赖的一致性，务必将 `Podfile.lock` 文件纳入版本控制中。

### pod update

运行 `pod update PODNAME` 命令，则会直接查询较新的依赖版本，而不会检查 `Podfile.lock`文件，如果未指定版本，则会下载最新的可用版本。如果未指定具体的 pod，则会更新所有依赖。

由此可见，Cocoapods 即保证了多人下发版本依赖的一致性，又兼顾了个人的更新需求。
[参见](https://guides.cocoapods.org/using/pod-install-vs-update.html)