# 面试题系列之多线程

### Objective-C 中的锁，各自原理如何？适用场景？

- @synchronized 关键字锁
- NSLock对象锁
- NSCondition
- NSConditionLock 条件锁
- NSRecursiveLock 递归锁
- pthread_mutex 互斥锁 (通用互斥锁,类UNix都有的)
- pthread_rwlock
- dispatch_semaphore 信号量实现枷锁（GCD）
- OSSpinLock
- POSIX conditions
- os_unfair_lock

### 互斥锁、自旋锁优缺点？

- 互斥锁的代价更高，因为线程有休眠，要唤醒，但是安全性更好，
- 自旋锁代价小，效率高，但是有优先级反转的可能，且比较耗资源，因为它会不断尝试读取锁状态

### 分别用 C/C++ 和 Objective-C 实现互斥锁、自旋锁。


### 死锁档四个条件、优先级翻转

1. 条件互斥；
2. 占有和等待条件；
3. 不可抢占；
4. 环路等待；

优先级反转是指：如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock
### 什么时候处理多线程，几种方式，优缺点？

当有耗时的任务需要处理，在保证流畅的情况下主线程不能满足处理需求时，就需要开辟一个或多个线程处理该任务。

- NSThread
- NSOperation、NSOperationQueue
- GCD

[参考](http://www.cnblogs.com/andy-zhou/p/5321842.html)

#### 系统有哪些在后台运行的 Thread

### 队列和线程的关系
队列以运行方式来分有串行、并行，从功能上分有主队列和全局队列，队列由分为不同的优先级。可以通过队列来管理线程，线程也可以通过队列来管理多个任务。

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

防止线程退出，

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


### `int a = 0` 一万个线程并发访问，最终 a 的值？

可能正确，也可能不正确，因为线程之间会发生竞争，

