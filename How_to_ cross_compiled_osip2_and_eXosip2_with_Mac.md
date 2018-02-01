# 在Mac上为iOS交叉编译osip2和eXosip2

关于osip2和eXosip2是什么这里不比多说，相信看到找到此片文章的都已十分清楚sip协议栈。本文的重点在与如何在Mac平台上以iOS为目标平台交叉编译osip2和eXosip2。

下面以iOS模拟器和iOS真实设备为目标平台，分别列出编译配置：

## 编译配置

### iOS 模拟器

- osip2

```
$ ./configure CC=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang CFLAGS="-O2 -m32 -mios-simulator-version-min=5.0 -DPJ_SDK_NAME="\"iPhoneSimulator7.1.sdk\"" -arch i386 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator7.1.sdk" --host=arm-apple-darwin9 --target=arm-apple-darwin9 --prefix=/Users/noone/Documents/siplibs/i386
```
- eXosip2

```
$./configure CC=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang CFLAGS="-O2 -m32 -mios-simulator-version-min=5.0 -DPJ_SDK_NAME="\"iPhoneSimulator7.1.sdk\"" -arch i386 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator7.1.sdk" LDFLAGS=" -framework MobileCoreServices -framework CFNetwork -framework CoreFoundation" --disable-openssl --host=arm-apple-darwin9 --target=arm-apple-darwin9 --prefix=/Users/noone/Documents/siplibs/i386
```


### iOS 设备
- osip2

```
$ ./configure CC=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang CFLAGS="-DPJ_SDK_NAME="\"iPhoneOS7.1.sdk\"" -arch armv7 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.1.sdk" --host=arm-apple-darwin9 --target=arm-apple-darwin9 --prefix=/Users/noone/Documents/siplibs/arm
```
- eXosip2

```
$ ./configure CC=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang CFLAGS="-DPJ_SDK_NAME="\"iPhoneOS7.1.sdk\"" -arch armv7 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.1.sdk" LDFLAGS=" -framework MobileCoreServices -framework CFNetwork -framework CoreFoundation" --disable-openssl --host=arm-apple-darwin9 --target=arm-apple-darwin9 --prefix=/Users/noone/Documents/siplibs/arm
```

##  开始编译

 通过上文的配置，生成相应的**make**文件，然后在终端中执行如下命令：
 
 ```
 $make
 $make install
 ```
 待编译完毕，即可在我们刚刚通过*--prefix*参数配置的输出目录中找到编译好的哭文件。
 
## 合并静态库

仅为真实设备编译一个静态库，会导致项目不能在模拟器上运行，这个项目调试带来很多不便，怎么才能既能在模拟器运行，又能在真机上运行呢？可以分别编译两个目标静态库，然后再将两个库合二为一。合并方法如下：

```
$lipo -create /Users/noone/Desktop/arm/lib/libeXosip2.a /Users/noone/Desktop/i386/lib/libeXosip2.a -output /Users/noone/Desktop/libs/libeXosip2_for_ios.a
```

以上命令中所涉及的两个libeXosip2.a就是我们刚刚编译的分别对应模拟器和真机的静态库，'-output'参数用于指明输出位置和文件名。

## 最后
将静态库引入项目时，注意还需引入**libresolv.9.dylib**这个依赖库，方能编译成功。
 