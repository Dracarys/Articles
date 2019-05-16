# Xcode command line tools 

## 安装
工欲善其事，必先利其器。所以学习之前，先看看如何安装。首先你的有 Xcode，没安装请先移步 App Store 进行下载安装，一键式操作，非常便捷。之后，就可以安装 Command line tools 了，打开 Terminal，直接键入如下命令：

```shell
xcode-select --install
```
此时会弹出提示，确认安装，等进度条结束即可。结束后，进入 `/Library/Developer/CommandLineTools/usr/bin/` 查看一下，具体有哪些工具可用。

长长一串，有点眼晕啊，但是细心观察可以发现很多都是跟版本控制、Swift、metal相关的，这些我们不去讨论，重点集中在那些通用型较强的工具上。比较多，很多还牵涉到编译到相关知识，不足之处望斧正。

## ar
ar 工具应该是来自 architecture 的简写（猜的，未考证），可以通过它对二进制的静态库进行一些编辑操作，例如：对某个架构的静态库进行解包，删除其中的一些目标文件（.O 文件）等等。

>⚠️ 注意：必须是 Non-fat 静态库，否则会提示类型错误。

## lipo

## nm

## nmedit

## size

## ranlib

## as

## cmpdylib

## objdmp

## libtool

## segedit

## strip

## rebase

## otool