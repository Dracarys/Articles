# 面试题系列之常用三方框架

## 1. 网络库

### 1.1 AFNetworking，是否支持IPv6

3.0之后支持

### 1.2 AFNetwoking 为什么添加一条常驻线程？
网络请求是异步的，这会导致获取到请求数据时，线程已经退出，代理方法没有机会执行。因此，AFN 的做法是使用一个 runloop 来保证线程不死~
然而频繁的创建线程并启动runloop肯定会造成内存泄露(runloop 无法停止.线程无法退出)
所以AFN就创建了一个单例线程,并且保证线程不退出
[参考](https://www.jianshu.com/p/7170035a18e8)

### 1.3 AFNetworking 的 reachability是如何检测到网络状态变化的？

### 1.4 AFNetworking 与 MKNetworking 区别，优劣？

## 2. 图片库

### 2.1 SDWebImage 图片编解码

### 2.2 Kingfisher 图片编解码

## 3. 模型库

### 3.1 YYCache

### 3.2 YYMode

### 3.3 YYAsyncLayer如何进行异步绘制的？

### 3.4 YYImage 图片编解码

## 4. 三方库管理工具

### 4.1 Pod 的工作原理

### 4.2 Pod update和 Pod install的区别

## 5. 其它

### 5.1 FB 的Async库都做了什么？
