How to decrypt a iOS App

### 为什么要decrypte


### 准备工作

一台Mac 设备，或者其它的linux设备

1. 一部已经越狱的iOS设备，具体适合设备取决于你的分析需求，如果你要分析一个64位的应用，那么必须选用64的设备，反之必须使用32位的设备。
下面是一张对应的设备列表。

2. 根据Cydia首页的教程，安装OpenSSH。注意按教程修改密码，以免被他人利用。

3. 在越狱设备上安装Cycritp。

4. 下载主角：dumpdecrypted 并编译。得到dumpdecrypted.dylib.

至此所有准备工作完成，后面逐步用到这里所涉及的工具。

### 流程

坚固的堡垒往往容易从内部被攻破，这里正是利用这点，将我们得到dumpdecrypted.dylib放置到需要砸壳的应用的Documents目录下，然后运行从内部将其攻破。

要实现这一点，我们就要先找打Documents目录，然后在从桌面端，将文件拷贝到手机端，之后在运行。得到破壳的应用文件后，在从手机端拷贝回桌面端，以供学习研究。

万事俱备，东风来吧！

### 开始

要找到手机上某个应用对应的Documents目录，我就需要先讲桌面端与手机端连接起来，你可以通过iExplorer、iTools、ifunbox，等可视化工具操作，也可以直接通过命令行操作，我们这里推荐采用命令行的模式，为什么？因为哪些图形化的工具都要收费，再者命令行操作看上去更酷一些不是吗。

这里就要请出我们之前在手机上安装的OpenSSH了，如果你很感兴趣，可以在这里了解更多关于OpenSSH的内容。首先在检查你的Mac和手机是不是在同一网段，例如：这里我的Mac是192.168.0.109，手机是192.168.0.129，其次打开Terminal（终端），输入一下命令：

``` Shell
ssh root@192.168.0.129
```
之后会要求你输入登录密码，注意如果你在准备阶段修改过密码，那么这是情输入新密码，如果没有，那么密码是：“alpan”
之后我们就远程登录到手机上了，很酷是不是？

登录之后要做什么呢？当然是找堡垒的位置，好安插我吗的“间谍”--dumpdecrypted.dylib，可目录众多怎么找呢？别慌，这是就要请出先前准备的Cycript工具了。

首先在手机上打开你要学习的应用，注意仅打开一个，把其它没有的全部关掉，否则打开的太多，还是会看花眼的。

接下来在终端中输入如下命令：

``` Shell
ps -e | grep var
```
之后可以在终端上看到：

``` Shell

127 ??         0:00.89 /usr/libexec/pkd -d/var/db/PlugInKit-Annotations
 1270 ??         0:00.23 /private/var/db/stash/_.r4WV27/Applications/MobileSafari.app/webbookmarksd
 1545 ??         0:05.46 /var/mobile/Containers/Bundle/Application/58E4482C-E4C1-413C-85C5-BBE9AE34E5A1/XXX.app/XXX
 1598 ttys000    0:00.01 grep var
 
 ```
这里的XXX就是我们要学习的那个应用。注意这里如果提示不识别ps 命令，请重新安装cycript，及其以来的avm 工具，然后按提示重启springboard。

接下来我们就要祭出大杀器，Cycript，在终端输入：

``` Shell
cycript -p XXX
```
注意XXX与前面列出的名称要保持一致哦

接下来我们就进入了Cycript脚本命令操作状态，Cycript可以做的事情很多，感兴趣的可以在这里进一步学习。解析来输入如下命令：

``` Shell
cy# [[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask][0]
```
注意cy# 是Cycript操作状态的标识，之后才是我们要输入的命令。如果你熟悉iOS开发，是不是感觉很眼熟，这不就是我们平时获取Documents目录的方法吗，是的，这也整是Cycript的魅力。
输入命令后就会显示出对应的Documents目录了，记下目录，退出cycipt（怎么退出呢）。（事实上没必要，只要不关掉终端就好。回头直接拷贝粘贴就是了）

既然准确的定位到Documents目录，那么接下来就是放间谍进去啦。

这里你可以通过之前介绍的几个图形化工具，将dumpdecrypted.dylib拷贝进去，也可以像我一样装一装，用命令操作。
退出SSH连接，怎么退？logout啊，在终端上进入我们之前准备好的dumpdecrypted.dylib所在目录。然后在终端上输入如下命令：

``` Shell
scp dumpdecrypted.dylib root@192.168.0.129:/var/mobile/Containers/Data/Application/9C376783-E2FA-4B5C-8167-538D5C2FE31A/Documents/
```
执行后，会要求你输入密码，输入正确的密码后，记开始向目标拷贝了。这里你可以看到拷贝的进度。

拷贝完成后，我们再次通过SSH连接到手机端，进入刚刚执行拷贝到目标Documents目录，先ls一下看看我们的文件是否拷贝正确。没问题，就在终端中输入如下命令：

``` Shell
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib /var/mobile/Containers/Bundle/Application/58E4482C-E4C1-413C-85C5-BBE9AE34E5A1/XXX.app/XXX
```
注意上面的路径，要正确，是app所在目录的路径，不是Documents的路径哦。

执行完毕后，我们在ls一下，看看是不是多了一个XXX.decrypted的文件，有就说明砸壳成功啦。

接下来就是怎么把它拷贝到我们的电脑上，以便研究学习了。怎么办呢，还是通过scp命令。

退出SSH连接，回到我们的桌面系统中，在终端上输入如下命令：

``` Shell
scp root@192.168.0.129:/var/mobile/Containers/Data/Application/9C376783-E2FA-4B5C-8167-538D5C2FE31A/Documents/wifikey.decrypted /Users/hahaha/Documents/cracktools/
```
同样，在拷贝进度完成后，我们就可以在Users/hahaha/Documents/cracktools/目录下找到我们的XXX.decrypted文件了。至此砸壳全部完成。


### 导出头文件

仅仅止步于砸壳是没有意义的，重点是我们要学习它内部是怎么组织架构的，那么如果能看到目标应用的所有头文件是不是很有帮助呢，答案自然不言而喻。接下来就从我们刚刚砸完壳的文件中把头文件导出来。

这里就要祭出class-dump这件法宝了，（如何安装呢）

在终端上进入XXX.decrypted所在目录，例如我这里是Users/hahaha/Documents/cracktools/，然后运行如下命令。

``` Shell
class-dump -H wifikey.decrypted -o outputHeaders
```
进入outputHeaders看看，怎么是空的，什么都没有。这里就要特别说明一下，我们需要指定需要导出的包架构，由于我这里用的是iPhone 4s，是armv7的(这里涉及到处理器架构和指令集的问题，由于我也还在学习中，所以不便多说，以免误导)，所以这里需要指定：

``` Shell
class-dump --arch armv7 -H wifikey.decrypted -o outputHeaders
```
再次查看outputHeaders文件夹，嗯，所有的 头文件都被dump出来了。

### 后记
其实iOS砸壳的教程已经很多了，但是今天本人实际操作下来，还是不是很顺利，别人的文章都不是少说了这里，就是少说了那里，不过好在最后自己尝试成功了。未来避免后人像我一样浪费时间，所以就把自己的过程记录下来。当然了，也不排除是本人自己笨或粗心，理解能力不强，不管怎么样，就算帮不了别人，至少也能增强自己的记忆。这就够了。

### 参考
[《iOS逆向 - dumpdecrypted工具砸壳》by 小木头](http://www.liuchendi.com/2015/12/23/iOS/24_dumpdecrypted/)

[《iOS逆向 - Cycript基本用法》 by 小木头](http://www.liuchendi.com/2015/12/19/iOS/23_Cycrip/)

[《iOS逆向之App脱壳》 by 龙马君](http://www.jianshu.com/p/47836c78eb0a)

[《iOS逆向工程之App脱壳》 by 青玉伏案](http://www.cnblogs.com/ludashi/p/5725743.html)


