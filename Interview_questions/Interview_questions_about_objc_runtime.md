# 面试题系列之运行时

## 1. 载入过程

1. read all classes in all new images, add them all to unconnected_hashl;
2. read all categories in all new images,
3. try to connected all classes
4. resolve selector refs and class refs
5. fix up protocol object
6. call `+load()` for classes and categories
7. all classes are ready before any categories are ready.

### 1.1 load 和 initialize 方法的调用时机？

- load 当类被加载到 runtime 的时候运行，在 main 函数执行之前，也就是类加载器加载时，每个类默认只会调用一次；通常被用来进行 Method Swizzle，但是这会增加 App 启动时间。
- initialize 在类接收到第一条消息之前被用调用。每个类只会调用一次。子类未实现会向上查找。一搬用来初始化全局变量或静态变量。

## 2. 类别（category）

### 2.1 什么是 category 

[参考](https://juejin.im/post/5a9d14856fb9a028e52d5568)

下面是 category 的结构体：
``` C++
typedef struct category_t *Category;

struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
    // Fields below this point are not always present on disk.
    struct property_list_t *_classProperties;

    method_list_t *methodsForMeta(bool isMeta) {
        if (isMeta) return classMethods;
        else return instanceMethods;
    }

    property_list_t *propertiesForMeta(bool isMeta, struct header_info *hi);
};
```

 category 是 Objective-C 2.0 之后添加的语言特性，主要作用是为已经存在的类追加方法。此外 Apple 还推荐另外两个使用场景：
 
 1. 可以把类的实现分开在几个不同的文件里，这样可以减少单个文件的体积；可以对功能进行分类，组织到不同的 category 中；可由多人共同开发一个类，减少冲突；按需加载，方便配置；
 2. 声明私有方法；

### 2.2 category 的特点

- category 只能给某个已有的类追加方法，不能追加成员变量
- category 可以追加属性，但只能生成 setter 和 getter 的声明，不能生成 setter 和 getter 的实现即成员变量；
- 如果 category 中的方法和类已有方法同名，category 中的方法会覆盖掉类中的已有方法；
- 如果多个 category 中存在同名的方法啊，因为运行时加载时添加的顺序是无法保证的，所以覆盖的顺序也就不能确定，进而导致最终不知道会调用哪个。

参考里说由编译器决定，有待验证，因为这跟 runtime 里的注释不符。

### 2.3 category VS extension

- extension 运行在编译期，它是类的一部分，拓展的方法，属性和变量一起形成一个完整的类。而 category 是运行期决定的，此时对象的内存布局已经确定，无法再追加变量
- extension 一般用来隐藏类的私有信息，也就说你只能给已有源码的类添加 extension，而 category 则不存在该问题；
- category 不能添加成员变量，而 extension 则没这个限制。

### 2.4 category 原理

原理非常简单，运行时准备时，会将 category 中的方法循环添加到类的方法列表中去，包括 protocol 列表， property 列表等，也是同理。同名的方法会被覆盖。

### 2.5 使用 runtime Associate 方法关联的对象，需要在主对象 dealloc 的时候释放吗？
无论在MRC下还是ARC下均不需要在主对象 dealloc 的时候释放，被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的object_dispose()方法中释放。

### 2.6 关联属性如何显示 weak ？

利用中间对象包装一层？？？？

## 3. 消息转发（objc_mgSend）

### 3.1 为什么 Objective—C 的方法不叫调用叫发消息?
因为最终所有方法的调用都会转为下面这样的调用形式，所以被称为“发消息”。

```
id _Nullable objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```
> ⚠️注意参数

### 3.2 给一个对象发消息的过程如何，或者说如何通过 selector 找到对应的 IMP 地址：

1. 先在 cache 中查找
2. 在方法类表中查找
3. 到父类中查找
4. 查遍所有直到根类

### 3.3 类对象也是如此吗：

是的，只是实例对象要先通过 isa 指针取得该实例的类，然后就一样了。

### 3.4 如果消息发送失败有哪些补救措施：

- 你不能处理啊，那你要添加一个的吗？实例方法调用 `+(BOOL)resolveInstanceMethod:(SEL)selector`，如果是类方法，那么调用 `+(BOOL)resolveClassMethod:(SEL)selector`，如果要动态添加，就重写响应方法，在方法实现中添加，并返回 YES， 如果不，要记得调用 super；
- 添加不了啊，那要我转发给别人吗？`-(id)forwardingTargetForSelector:(SEL)selector`，如果有备用处理对象，那么在此返回，否则返回 nil。通过此方法，可以配合 composition（在对象内部封入子对象，将任务交给子对象处理） 来模拟“多重继承”的某些特性。
- 全部内容都在这里了（NSInvocation），你看着办吧。`-(void)forwardInvocation:(NSInvocation *)invocation`，此方法可以实现的很简单，只需该面调用目标，使消息在新目标上得以调用即可。这就与上一步类似了。还可以在触发消息前，先以某种方式改变消息内容，比如追加另外一个参数，或者该换选择子（selector），等等。

接受者在以上的每一步都有机会处理消息，越往后代价越大。最好能在第一步就处理完毕，这样的话，运行时可以将此方法缓存起来。

### 消息转发哪些步骤可以被利用

消息转发时可以结合组合来模拟多重继承的某些特性。

## 4. 其它

### 4.1 如何访问并修改一个类的私有属性

1. 通过 KVC 获取
2. 通过 runtime 访问并修改私有属性

### 4.2 weak实现机制？为什么对象释放后会自动置为nil？
运行时会维护一张 weak 哈希表， weak 对象的地址作为 key，该表记录了每个对象的引用计数，当计数为 0 时就会触发销毁机制，


### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

1. 不可以，因为结构的偏移已经固定了
2. 可以，这是新建，当然可以声明并定义了