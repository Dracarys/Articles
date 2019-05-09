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

## 3、安装 Network Link Conditioner
该工具现在包含在 Aditional Tool For Xcode 包中，需登陆开发者网站下载。详情不多讲，这里主要介绍一下安装。

在 macOS Mojava 10.14.4 目前最新的系统上，直接双击是无法安装的，会提示你 “Network Link Conditioner 是跟随系统的，无法被替换”，什么鬼？系统明明没有自带，却告诉我无法安装？直接手动磕，打开目录 `～/Library/PreferencePanes`，直接将 Conditioner 拖进去，重启。Done!

> [参考来源](https://stackoverflow.com/questions/52414375/cannot-install-xcode-10-network-link-conditioner-in-macos-mojave)