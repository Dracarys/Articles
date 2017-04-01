# 通过代码更新应用图标

今天iOS 10.3 发布了，新系统为用户带来更好的体验的同时，也为开发者提供了一个新的API可以允许开发者自己指定应用在主屏幕上的图标，虽然还要给用户弹确认，虽然还不能自动定时更新，但谢天谢地总算是能更新了不是吗，终于可以不用再为换个图标而急急忙忙地去审核了。

既然有了新功能，那就试试呗，先看看新API:

``` Swift
extension UIApplication {

    // If false, alternate icons are not supported for the current process.
    @available(iOS 10.3, *)
    open var supportsAlternateIcons: Bool { get }

    
    // Pass `nil` to use the primary application icon. The completion handler will be invoked asynchronously on an arbitrary background queue; be sure to dispatch back to the main queue before doing any further UI work.
    @available(iOS 10.3, *)
    open func setAlternateIconName(_ alternateIconName: String?, completionHandler: ((Error?) -> Swift.Void)? = nil)

    
    // If `nil`, the primary application icon is being used.
    @available(iOS 10.3, *)
    open var alternateIconName: String? { get }
}
```
好了，接下来准备两种样式的图标，拖入项目中，然后再写一小段代码：

``` Swift
func changeIcon() {
        let application = UIApplication.shared
        
        //首先判断是否设备支持，其次如果已经替换，
        if #available(iOS 10.3, *), application.alternateIconName != nil {
        	  //如果已经替换就换回主图标
            application.setAlternateIconName(nil, completionHandler: nil);
        } else if #available(iOS 10.3, *), application.supportsAlternateIcons {
            application.setAlternateIconName("AlternateIcon") {
                if let error = $0 {
                    print(error.localizedDescription)
                } else {
                    print("Change alternate Icon successed")
                }
            }
        }
    }
```
运行一下，我们可以看到Console的打印信息提示：“The file doesn’t exist.”。为什么会找不到呢，明明把图标文件添加到项目中了。再回顾下API文档，又这样一段话：

>The name of the alternate icon, as declared in the CFBundleAlternateIcons key of your app's Info.plist file. Specify nil if you want to display the app's primary icon, which you declare using the CFBundlePrimaryIcon key. Both keys are subentries of the CFBundleIcons key in your app's Info.plist file.

意思是我们还要修改一下“Info.plist”文件，在其中添加一些键值。想了解Info.plist文件所有键值可以点[这里](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/TP40009249-SW14)

按文档说明添加键值如下：

``` xml
<key>CFBundleIcons</key>  
<dict>  
    <key>CFBundleAlternateIcons</key>  
    <dict>  
         <key>CFBundleIconFiles</key>  
         <array>  
            <string>AlternateIconFileName</string>  
         </array>  
         <key>UIPrerenderedIcon</key>  
         <false/>  
    </dict>  
    <key>CFBundlePrimaryIcon</key>  
    <dict>  
        <key>CFBundleIconFiles</key>  
        <array>  
            <string>PrimaryIconFileName</string>  
        </array>  
    </dict>  
</dict>  
```

>注意：如果你的Info.plist文件看上去和我的不一样，请选中它，然后右键“Open as”->"Source code"

OK，再次运行，还是找不到文件，why？是时候祭出谷歌大法了。终于在苹果官方论坛里找到了[答案](https://forums.developer.apple.com/thread/71463)。看来很多人都在这里被[文档](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/TP40009249-SW14)给误导了。

正确的做法是：

``` xml
<key>CFBundleIcons</key>  
<dict>  
    <key>CFBundleAlternateIcons</key>  
    <dict>  
        <key>AlternateIconFileName</key>  
        <dict>  
            <key>CFBundleIconFiles</key>  
            <array>  
                <string>AlternateIconFileName</string>  
            </array>  
            <key>UIPrerenderedIcon</key>  
            <false/>  
        </dict>  
    </dict>  
    <key>CFBundlePrimaryIcon</key>  
    <dict>  
        <key>CFBundleIconFiles</key>  
        <array>  
            <string>PrimaryIconFileName</string>  
        </array>  
    </dict>  
</dict>  
```
可以看出“CFBundleAlternateIcons”字典还需要在嵌一个键与图标文件名相同的字典。

修改完成后，在此运行，终于可以更新了。

参考：[苹果官方论坛 giveme5 的回复](https://forums.developer.apple.com/thread/71463)





