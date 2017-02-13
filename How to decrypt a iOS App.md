#iOS应用砸壳

**本人不能，也没有能力保证这里所涉及工具的安全性，请自行辨别。越狱有风险，应用需谨慎**

### 为什么要脱壳
作为一个iOS开发者，经常会看到别人的应用多么多么炫酷，想学习学习吧，它又不是开源的，怎么办呢？只能自己掰开内部瞧一瞧。说是瞧瞧，可真正要打开就没那么简单了，从Appstore下载的App都是经过Apple加密的，也就是俗称的“加壳”，而我们要想一窥究竟，面临的第一个障碍就是，如何去掉这层壳，俗称“砸壳”，“脱壳”。

### 准备工作
正所谓工欲善其事，必先利其器。没有工具是万万不行的，下面列出了这次“砸壳”过程中所涉及的工具：

1. 一部运行 Linux，或者 macOS 的电脑；

2. 一部已经越狱的 iOS 设备，并且以安装OpenSSH（在Cydia首页有详细的教程，此处不再赘述），Cycript，avd-cmds；

3. 下载主角：[dumpdecrypted](https://github.com/stefanesser/dumpdecrypted) ，并按照Github上的说明进行编译，得到“dumpdecrypted.dylib”。

至此所有工具准备就绪，后面会逐步用到它们。

### 砸壳流程

坚固的堡垒往往容易从内部被攻破，这里也正是利用了这点，将我们得到 dumpdecrypted.dylib 放置到需要砸壳的应用的 Documents 目录下，然后运行从“内部”将其攻破。

要实现这一点，我们就要先找到 Documents 目录，然后从桌面端将 dumpdecrypted.dylib 拷贝到手机上，之后在运行之。得到破壳的应用文件后，在从手机端拷贝回桌面端，以供学习研究。

### 开始

要找到手机上某个应用对应的 Documents 目录，需要先将桌面端与手机端连接起来，你可以通过iExplorer、iTools、iFunBox 等可视化工具操作，也可以直接通过命令行操作，我们这里推荐采用命令行的模式，为什么？因为哪些图形化的工具都要收费，再者命令行操作看上去更酷一些不是吗。

是时候请出我们之前准备的OpenSSH了，如果你很感兴趣，可以在这里了解更多关于OpenSSH的内容。首先在检查你的桌面端和手机端是不是在同一网段，例如：这里我的 Mac IP 是192.168.0.109，手机 IP 是192.168.0.129。确定后打开终端（Terminal），输入如下命令并执行：

``` Shell
ssh root@192.168.0.129
```
执行后会要求你输入登录密码，注意如果你在手机上安装 OpenSSH 时修改过密码，那么这里请输入新密码，如果没有，那么默认的密码是：“alpine”。
输入密码后后就远程登录到了手机端，很酷是不是？

登录之后要做什么呢？当然是找“堡垒”的位置，以便安插我们的“间谍”--dumpdecrypted.dylib啊，可目录众多怎么找呢？别慌，有办法。首先打开那个我们要借鉴学习的应用，然后把它应用全部关闭，保证只有我们选中的那个应用处于活动状态（在后台也可以。

接下来在终端中输入如下命令：

``` Shell
ps -e | grep var
```
之后可以在终端上看到：

``` Shell

127 ??         0:00.89 /usr/libexec/pkd -d/var/db/PlugInKit-Annotations
 1270 ??         0:00.23 /private/var/db/stash/_.r4WV27/Applications/MobileSafari.app/webbookmarksd
 1545 ??         0:05.46 /var/mobile/Containers/Bundle/Application/58E4482C-E4C1-413C-85C5-BBE9AE34E5A1/target.app/target
 1598 ttys000    0:00.01 grep var
```
 
这里的 “target” 就是我们要学习研究的应用。注意这里如果提示不识别 ps 命令，请重新安装 avd-cmds 工具。

接下来就要祭出大杀器 Cycript 的时候了，在终端输入：


```Shell
cycript -p /var/mobile/Containers/Bundle/Application/58E4482C-E4C1-413C-85C5-BBE9AE34E5A1/target.app/target
```

执行后如果出现 `cy#` 提示符，就说明我们已经成功进入 Cycript 命令操作状态，Cycript可以做的事情很多，感兴趣的可以参考这篇[文章](http://www.liuchendi.com/2015/12/19/iOS/23_Cycrip/)。

接下来在终端输入如下命令：

``` Shell
[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask][0]
```
如果你熟悉 iOS 开发，是不是感觉很眼熟，这不就是我们平时获取 Documents 目录的方法吗，是的。这也正是 Cycript 的魅力。
输入命令后就会显示出对应的Documents目录了，记下目录，按 `Control+D` 退出Cycipt。

既然已经准确地定位到 Documents (堡垒)目录，那么接下来就是放“间谍”进去啦。

这里你可以通过之前介绍的几个图形化工具，将 dumpdecrypted.dylib 直接拷贝进去，也可以像我一样用命令操作。
退出SSH连接，怎么退？`logout` 啊。在终端上进入我们之前准备好的dumpdecrypted.dylib 所在目录。然后在终端上输入如下命令：

``` Shell
scp dumpdecrypted.dylib root@192.168.0.129:/var/mobile/Containers/Data/Application/9C376783-E2FA-4B5C-8167-538D5C2FE31A/Documents/
```
执行后，会要求你输入密码，输入正确的密码后，即开始向目标目录拷贝。这里你可以看到拷贝的进度。

拷贝完成后，我们再次通过 SSH 连接到手机端，进入目标应用的 Documents 目录，先 `ls`一下看看我们的文件是否拷贝正确。没问题，就在终端中输入如下命令：

``` Shell
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/58E4482C-E4C1-413C-85C5-BBE9AE34E5A1/target.app/target
```
注意上面的路径，要正确，是待砸壳 App 所在路径，不是 Documents 的路径。

执行完毕后，我们在 `ls` 一下，看看是不是多了一个XXX.decrypted的文件，有就说明砸壳成功啦。

接下来就是怎么把它拷贝到我们的电脑上，以便研究学习了。怎么办呢，还是通过scp命令。

退出SSH连接，回到我们的桌面系统，在终端上输入如下命令：

``` Shell
scp root@192.168.0.129:/var/mobile/Containers/Data/Application/9C376783-E2FA-4B5C-8167-538D5C2FE31A/Documents/wifikey.decrypted /Users/hahaha/Documents/cracktools/
```
同样，在拷贝进度完成后，我们就可以在 `Users/hahaha/Documents/cracktools`目录下找到我们的 “target.decrypted” 文件了。至此砸壳全部完成。


### 导出头文件

仅仅止步于砸壳是没有意义的，重点是我们要学习它内部是怎么组织的，那么如果能看到目标应用的所有头文件是不是很有帮助呢，答案自然不言而喻。接下来就从我们刚刚砸完壳的文件中把头文件导出来。

这里就用到 class-dump 这件法宝了， [github地址](https://github.com/nygard/class-dump)

在终端上进入 target.decrypted 所在目录，例如我这里是Users/hahaha/Documents/cracktools/，然后在终端上执行如下命令：

``` Shell
class-dump -H wifikey.decrypted -o outputHeaders
```
如果提示找不到class-dump命令，请检查是否拷贝到了 `/usr/local/bin`目录。

进入 outputHeaders 看看，如果你和我一样，用的是越狱 iPhone 4s，那么将会是空的，这是因为我们没有指定正确的指令集。下面是iOS对应的指令集：

|指令集|设备|
|:---|:---|
|armv6|iPhone, iPhone2, iPhone3G, 第一代、第二代 iPod Touch|
|armv7|iPhone3GS, iPhone4, iPhone4S, iPad, iPad2, iPad3(The New iPad), iPad mini, iPod Touch 3G, iPod Touch4|
|armv7s| iPhone5, iPhone5C, iPad4(iPad with Retina Display)|
|arm64| iPhone5S, iPad Air, iPad mini2(iPad mini with Retina Display)|

接下来我们通过指定指定指令集在试一次：

``` Shell
class-dump --arch armv7 -H wifikey.decrypted -o outputHeaders
```
看看 outputHeaders 文件夹，嗯，所有的 头文件都被dump出来了。

### 后记
其实iOS砸壳的教程已经很多了，但是今天本人实际操作下来，还是不是很顺利，别人的文章都不是少说了这里，就是少说了那里，不过好在最后自己尝试成功了。为了避免后人像我一样浪费时间，还是在写一篇的好。当然了，也不排除是本人自己笨，理解能力不强，不管怎么样，就算是重复，至少也能增强自己的记忆不是吗。这就够了。

### 参考
[《iOS逆向 - dumpdecrypted工具砸壳》by 小木头](http://www.liuchendi.com/2015/12/23/iOS/24_dumpdecrypted/)

[《iOS逆向 - Cycript基本用法》 by 小木头](http://www.liuchendi.com/2015/12/19/iOS/23_Cycrip/)

[《iOS逆向之App脱壳》 by 龙马君](http://www.jianshu.com/p/47836c78eb0a)

[《iOS逆向工程之App脱壳》 by 青玉伏案](http://www.cnblogs.com/ludashi/p/5725743.html)


