# 面试题系列之 - iOS内存管理

### 引用计数是如何实现的
auto reference counting，

### 有哪些导致崩溃的常见问题？如何进行预防？

- 取空值，自定义取值方法，添加到 category 中
- 越界，取值时始终进行验证，且永远不要相信后台反馈
- 不能识别到方法，向错误类型发送消息，预防方法同上，此外还可以通过runtime处理消息转发方法，来减少崩溃

### 内存泄漏的原因有哪些？如何解决？

- 创建后未释放，通过正确释放解决，重点关注 new、create等方法名
- 单位时间开辟大量空间，添加 autoreleasepool
- Corefoundation方法使用不当，持有权转移错误
- 循环引用，添加 weak 中间代理

### 常用集合类的哪些操作是深拷贝？
在集合类对象中，对 immutable 对象进行 copy，是指针复制，mutableCopy 是内容复制；对mutable 对象进行 copy 和 mutableCopy 都是内容复制。但是，集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。用代码简单表示如下：

```objc
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //单层深复制
[mutableObject copy] //单层深复制
[mutableObject mutableCopy] //单层深复制
```
### autorealse 如何实现的

这个解释不完善，有待进一步研究
autoreleasePool 是一个Objective-C的一种内存自动回收机制，它可以延时家如autoreleasePool中的变量释放的时机。
autoreleasePool在Runloop时间开始之前（push），释放是在一个 RunLoop 事件即将结束之前（pop）

### 子线程中需要加 autoreleasepool 吗？什么时间释放？
每个线程都默认有一个autoreleasepool，在线程即将退出前释放。但是如果该线程会产生大量的内存碎片，那么建议创建runloop以及自己的释放池，以便可以及时释放。

### autoreleasepool 在 ARC 和 MRC 下有什么区别

### block 在 ARC 和 MRC 下有什么区别，使用注意事项

#### BAD_ACCESS 在什么情况下出现？如何调试？

读取了一个不是你管控的内存地址，通常是读取了一个已释放或者为初始化的指针

启动僵尸对象，然后结合发生错误时的操作，逐步定位问题根源。