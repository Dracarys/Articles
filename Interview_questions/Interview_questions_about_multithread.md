# 面试题系列之多线程

## 1. 按读写特征，锁的分类？

- 排他锁，在访问共享资源前进行加锁操作，访问结束后解锁。一旦加锁将阻塞其它线程对共享资源的访问，直至解锁。
- 共享锁，又称读锁，可以并发的读取数据，但是在加锁期间不能对共享资源进行修改，直至解锁。
- 读写锁，读写锁是综合了互斥锁和共享锁的一种锁，即读取时是共享的，写入时时互斥（排他）的。它有三种状态：读加锁状态、写加锁状态和解锁状态。

## 2. Objective-C 中的锁

常用锁的效率对比：

![常用锁的效率对比](https://upload-images.jianshu.io/upload_images/153594-e1675e4e198b883b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

非常感谢[bestswift](https://bestswifter.com/ios-lock/)的总结。

### 2.1 SpinLock 自旋锁

OSSpinLock：自旋锁的一个数据类型，自旋锁广义上也属于拍他锁，它与互斥锁的区别是，在等待期不会放弃 CPU 资源，而事不断测试锁的状态，这样就一直占着 CPU，所以效率最高。但是也存在着优先级翻转的问题。

优先级翻转：如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock

> 时间片，现代操作系统在管理普通线程时，通常采用时间片轮转算法（Round Robin，简称RR）。每个线程会被分配一段时间片（quantum），通常在10～100毫秒左右。当线程用完属于自己的时间片以后，就会被操作系统挂起，放入等待队列中。直到下一次被分配时间片。

自旋锁的伪代码实现：

```objc
// 下面这一步要求不许为原子操作才可以。
BOOL lockk = NO;// 开始是没有锁，任何线程都可以申请锁
do {
	while(lock);// 如果处于锁状态就死循环，直至解锁
	lock = YES;// 加锁，让其它线程进入忙等状态
		Critical section //临界区
	lock = NO;// 解锁，让其它线程可以进入临界区
		Reminder section // 不需锁保护的代码
}
```

### 2.2 信号量（semaphore）

信号量是具有非负整数值的全局变量，只能有两种特殊的操作来处理：

- P(s)：如果 s 是非零的，那么 P 将 s 减 1，并且立即返回。如果 s 为零，那么就挂起这个线程，直到 s 变为非零，而一个 V 操作会重启这个线程。在重启之后，P 操作将 s 减 1，并将控制返回给调用者
- V(s)：V 操作将 s 加 1。如果有任何线程阻塞在 P 操作等待 s 编程非零，那么 V 操作会重启这些线程中的一个，然后该线程将 s 减 1， 完成它的 P 操作。

上述的 P、V 对 s 的操作必须为原子操作，V 操作重启对线程是不可预期的。

信号量是一种高级的同步机制，mutex 可以说是 semaphore 在仅取值 0/1 时的特例，Semaphore可以有更多的取值空间，用来实现更加复杂的同步，而不单单是线程间互斥。

### 2.3 pthread_mutex

互斥锁在申请锁时，调用了 pthread_mutex_lock 方法，它在不同的系统上实现各有不同，有时候它的内部是使用信号量来实现，即使不用信号量，也会调用到 lll_futex_wait 函数，从而导致线程休眠。

上文说到如果临界区很短，忙等的效率也许更高，所以在有些版本的实现中，会首先尝试一定次数(比如 1000 次)的 testandtest，这样可以在错误使用互斥锁时提高性能。

另外，由于 pthread_mutex 有多种类型，可以支持递归锁等，因此在申请加锁时，需要对锁的类型加以判断，这也就是为什么它和信号量的实现类似，但效率略低的原因。

>引用自[bestswifter](https://bestswifter.com/ios-lock/)

### 2.4 NSLock

通过 POSIX threads 实现的锁行为。加解锁必须在同一线程，否则将导致无法预料的问题。加解锁必须成对出现，未锁先解会给予错误提醒（这是相对 pthread_mutex 增加的功能），但不会崩溃。

不能通过它去实现递归锁，因为在一个线程中调用两次 lock 将导致永久锁定，如需递归锁，请使用 NSRecursiveLock 代替。

### 2.5 NSCondition

在一条线程中，NSCondition 对象即是锁，又是观察点。在线程执行之前会先判断条件是否为真，为真才可执行受到保护的代码，否则将被阻塞，直到其它线程通知条件对象。

使用方法如下：

1. 锁定条件对象
2. 进行条件判断，该条件是保证代码安全正确运行的前提
3. 如果条件为否，那么调用条件对象的 `wait` 或 `waitUntilDate:` 方法以阻塞线程。根据具体情况确定是否返回 2 ，还是继续等待
4. 条件为真，执行受到保护的任务
5. 根据具体任务选择更新或者给其它条件发现信号
6. 任务结束，解锁条件对象

伪代码如下：

```objc
lock the condition
while(!(boolean_predicate)) {
	wait on condition
}
do protected work
(optionally, signal or broadcast the condition again or change a predicate value)
unlock the condition
```

### 2.6 NSRecursiveLock

与 NSLock 类似，区别在于内部封装的 pthread_mutext_t 对象类型不同，这里使用的类型是 `PTHREAD_MUTEXT_RECURSIVE`，允许递归使用，不会出现 NSLock 在同一线程中连续两次 `lock` 永久锁定的问题。

### 2.7 NSConditionLock
NSConditionLock 借助 NSCondition 来实现，它的本质就是一个生产者-消费者模型。“条件被满足”可以理解为生产者提供了新的内容。NSConditionLock 的内部持有一个 NSCondition 对象，以及 _condition_value 属性，在初始化时就会对这个属性进行赋值:

```objc
// 简化版代码
- (id) initWithCondition: (NSInteger)value {
    if (nil != (self = [super init])) {
        _condition = [NSCondition new]
        _condition_value = value;
    }
    return self;
}
```
它的 lockWhenCondition 方法其实就是消费者方法:
```objc
- (void) lockWhenCondition: (NSInteger)value {
    [_condition lock];
    while (value != _condition_value) {
        [_condition wait];
    }
}
```

对应的 unlockWhenCondition 方法则是生产者，使用了 broadcast 方法通知了所有的消费者:
```objc
- (void) unlockWithCondition: (NSInteger)value {
    _condition_value = value;
    [_condition broadcast];
    [_condition unlock];
}
```

### 2.8 @synchronized(){}

`@synchronized` block 在被保护的代码上暗中添加了一个异常处理。为的是同步某对象时如若抛出异常，锁会被释放掉。

```objc
@synchronized(objc) {
	//do work
}
```
上面的代码会被转化为：

```objc
@try {
	objc_sync_enter(obj);
	// do work
} @finally {
	objc_sync_exit(obj);
}
```

> 引自：[关于 @synchronized，这儿比你想知道的还要多](http://yulingtianxia.com/blog/2015/11/01/More-than-you-want-to-know-about-synchronized/)

### 分别用 C/C++ 和 Objective-C 实现互斥锁、自旋锁。


### 死锁档四个条件、优先级翻转

1. 条件互斥；
2. 占有和等待条件；
3. 不可抢占；
4. 环路等待；

优先级反转是指：如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock

### volatile 关键字
Volatile 变量适用于独立变量的另一个内存限制类型。编译器优化代码通过加载这些变量的值进入寄存器。对于本地变量，这通常不会有什么问题。但是如果一个变量对另外一个线程可见，那么这种优化可能会阻止其他线程发现变量的任何变化。在变量之前加上关键字 volatile 强制编译器每次使用变量的时候都从内存里面加载。如果一个变量的值随时可能给编译器无法检测的外部源更改，那么你可以把该变量声明为 volatile 变量。
因为内存屏障和 volatile 变量降低了编译器可执行的优化，因此你应该谨慎使用它们，只在有需要的地方时候，以确保正确性。

### 什么时候处理多线程，几种方式，优缺点？

当有耗时的任务需要处理，在保证流畅的情况下主线程不能满足处理需求时，就需要开辟一个或多个线程处理该任务。

- NSThread
- NSOperation、NSOperationQueue
- GCD

[参考](http://www.cnblogs.com/andy-zhou/p/5321842.html)

#### 系统有哪些在后台运行的 Thread

### 队列和线程的关系
队列以运行方式来分有串行、并行，从功能上分有主队列和全局队列，队列由分为不同的优先级。可以通过队列来管理线程，线程也可以通过队列来管理多个任务。

### 线程的 Sleep 是如何 实现的？

### 线程同步的方式

同步多线程（SMT）,

- 事件
- 临界区
- 互斥器
- 信号量

### 线程池的结构？如何实现的？

```java
package mine.util.thread;  
  
import java.util.LinkedList;  
import java.util.List;  
  
/** 
 * 线程池类，线程管理器：创建线程，执行任务，销毁线程，获取线程基本信息 
 */  
public final class ThreadPool {  
    // 线程池中默认线程的个数为5  
    private static int worker_num = 5;  
    // 工作线程  
    private WorkThread[] workThrads;  
    // 未处理的任务  
    private static volatile int finished_task = 0;  
    // 任务队列，作为一个缓冲,List线程不安全  
    private List<Runnable> taskQueue = new LinkedList<Runnable>();  
    private static ThreadPool threadPool;  
  
    // 创建具有默认线程个数的线程池  
    private ThreadPool() {  
        this(5);  
    }  
  
    // 创建线程池,worker_num为线程池中工作线程的个数  
    private ThreadPool(int worker_num) {  
        ThreadPool.worker_num = worker_num;  
        workThrads = new WorkThread[worker_num];  
        for (int i = 0; i < worker_num; i++) {  
            workThrads[i] = new WorkThread();  
            workThrads[i].start();// 开启线程池中的线程  
        }  
    }  
  
    // 单态模式，获得一个默认线程个数的线程池  
    public static ThreadPool getThreadPool() {  
        return getThreadPool(ThreadPool.worker_num);  
    }  
  
    // 单态模式，获得一个指定线程个数的线程池,worker_num(>0)为线程池中工作线程的个数  
    // worker_num<=0创建默认的工作线程个数  
    public static ThreadPool getThreadPool(int worker_num1) {  
        if (worker_num1 <= 0)  
            worker_num1 = ThreadPool.worker_num;  
        if (threadPool == null)  
            threadPool = new ThreadPool(worker_num1);  
        return threadPool;  
    }  
  
    // 执行任务,其实只是把任务加入任务队列，什么时候执行有线程池管理器觉定  
    public void execute(Runnable task) {  
        synchronized (taskQueue) {  
            taskQueue.add(task);  
            taskQueue.notify();  
        }  
    }  
  
    // 批量执行任务,其实只是把任务加入任务队列，什么时候执行有线程池管理器觉定  
    public void execute(Runnable[] task) {  
        synchronized (taskQueue) {  
            for (Runnable t : task)  
                taskQueue.add(t);  
            taskQueue.notify();  
        }  
    }  
  
    // 批量执行任务,其实只是把任务加入任务队列，什么时候执行有线程池管理器觉定  
    public void execute(List<Runnable> task) {  
        synchronized (taskQueue) {  
            for (Runnable t : task)  
                taskQueue.add(t);  
            taskQueue.notify();  
        }  
    }  
  
    // 销毁线程池,该方法保证在所有任务都完成的情况下才销毁所有线程，否则等待任务完成才销毁  
    public void destroy() {  
        while (!taskQueue.isEmpty()) {// 如果还有任务没执行完成，就先睡会吧  
            try {  
                Thread.sleep(10);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
        // 工作线程停止工作，且置为null  
        for (int i = 0; i < worker_num; i++) {  
            workThrads[i].stopWorker();  
            workThrads[i] = null;  
        }  
        threadPool=null;  
        taskQueue.clear();// 清空任务队列  
    }  
  
    // 返回工作线程的个数  
    public int getWorkThreadNumber() {  
        return worker_num;  
    }  
  
    // 返回已完成任务的个数,这里的已完成是只出了任务队列的任务个数，可能该任务并没有实际执行完成  
    public int getFinishedTasknumber() {  
        return finished_task;  
    }  
  
    // 返回任务队列的长度，即还没处理的任务个数  
    public int getWaitTasknumber() {  
        return taskQueue.size();  
    }  
  
    // 覆盖toString方法，返回线程池信息：工作线程个数和已完成任务个数  
    @Override  
    public String toString() {  
        return "WorkThread number:" + worker_num + "  finished task number:"  
                + finished_task + "  wait task number:" + getWaitTasknumber();  
    }  
  
    /** 
     * 内部类，工作线程 
     */  
    private class WorkThread extends Thread {  
        // 该工作线程是否有效，用于结束该工作线程  
        private boolean isRunning = true;  
  
        /* 
         * 关键所在啊，如果任务队列不空，则取出任务执行，若任务队列空，则等待 
         */  
        @Override  
        public void run() {  
            Runnable r = null;  
            while (isRunning) {// 注意，若线程无效则自然结束run方法，该线程就没用了  
                synchronized (taskQueue) {  
                    while (isRunning && taskQueue.isEmpty()) {// 队列为空  
                        try {  
                            taskQueue.wait(20);  
                        } catch (InterruptedException e) {  
                            e.printStackTrace();  
                        }  
                    }  
                    if (!taskQueue.isEmpty())  
                        r = taskQueue.remove(0);// 取出任务  
                }  
                if (r != null) {  
                    r.run();// 执行任务  
                }  
                finished_task++;  
                r = null;  
            }  
        }  
  
        // 停止工作，让该线程自然执行完run方法，自然结束  
        public void stopWorker() {  
            isRunning = false;  
        }  
    }  
}
```
### 开启一条线程的方法？线程可以取消吗？

- NSThread
- pThread
- GCD
- NSOperation

一旦提交运行即不可取消，尚未提交执行的可以。

### runloop和线程的关系？各个mode是做什么的？如何实现一个runloop

[参考](http://www.cnblogs.com/superYou/p/4645168.html)

防止线程退出



```
do {

} while()
```

### 用过NSOperationQueue么？如果用过或者了解的话，为什么要使用 NSOperationQueue，实现了什么？跟 GCD 之间的区别和类似得地方

### Dispatch_semaphore

### 怎么对一个数组进行线程安全的操作？
封装成一个模型对象，然后各种操作都通过该模型来完成。；

另一种如下：

```objc
synchronized(mutableArray) {
  // 各种数组操作
}
// 出来就不要有针对数组的操作了。
```
尽量在一次 `synchronized` 内完成所有操作，否则可能导致脏读脏写：

```objc
synchronized(mapMark) {
  // 读取了一个元素
}
// 计算了下
synchronized(mapMark) {
  // 写入了一个元素
}
```

### 主线程，子线程，多个位置的 sleep 之后的执行顺序？