苹果的推送服务（TO：合作方）

### 推送通知发展史
iOS 的推送服务起始于 iOS 3.x（来源：https://en.wikipedia.org/wiki/Apple_Push_Notification_Service#cite_note-appledev-2） ，最初它仅作为服务端提醒客户端的一种方式，承载的信息也非常有限，知道iOS 8.x 开始，Apple才真正发力，引入更多的交互功能，下表，列举推送历代的发展。iOS 5.x 开始引入通知中心，iOS 6.x 本地推送， payload增至 256byte；iOS 7.x 开始引入静默推送，payload增至1KB；iOS 8.x 开始引入自定义提示音、用户权限、增强交互，提供更多的按钮懂多，本地推送提供地理围栏，payload增至2KB。iOS 9.x 开始引入手表，推送将同时推送只手表上。同时引入了文本输入功能。远程推送更新协议为HTTP／2，将多种证书合为一种。payload增至4KB。iOS 10.x开始整合，推送UserNotifications framework，将以前的功能统统整合进来，并进一步增强了交互。丰富内容，允许自定义UI，允许应用内展示，扩展载体，氛围Title，Subtitle，body三部分。通知可以队列添加，移除，更新。在推送协议中添加新的头部表示：apns-collapse-id。 iOS 11.x 隐藏内容，更丰富的自定义UI，自定义输入，支持分类。


###功能展示
结合公司App的实际情况，以前的版本因为市场占有率已经非常低，与收益相比，适配的成本很好，得不偿失。所以这里从 iOS 8.x开始介绍，

#### iOS 8.x

#### iOS 9.x

#### iOS 10.x

#### iOS 11.x

###推送的流程

###后端的发展

1. 早起TCP协议

2. 过渡到http协议
