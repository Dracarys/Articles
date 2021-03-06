#【笔记】多核处理器上的锁

在传统单处理器环境下，虽然系统可以通过时间片轮询（Round-Robin）与优先级抢占（Priority Preemption）方式快速地在不同进程间进行切换，从而造成一种多进程并发执行的假象，但是实际上一次只能运行一个进程。

因此，我们看到经典的操作系统同步原语（Synchronization Primitive），诸如互斥体（Mutex）、信号量（Semaphore）、消息队列（Message Queue）等等均是基于抑制软硬件中断或上下文切换来做到多线程同步的。

那么多核处理器上的锁与传统单核处理器上的锁有什么不同呢？这些锁是如何实现的？要进行同步操作时又该如何选择呢？

## 原子性
“原子性（atomic）”是指，一组操作在执行时作为一个整体进行，不会被打断。之所以能保证这些操作在执行时不会被中断，是因为来自多个处理器核心对同一个存储空间的访问，存储器控制器会去仲裁当前哪个原子操作先进行访存操作，哪个后进行，这些访存操作都会被串行化。

![原子性](https://upload-images.jianshu.io/upload_images/8136508-af1ca825337da627.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

上图中，假定线程 A 与线程 B 在不同的核心上并行执行。第一段采用的是普通读写方式对共享存储单元进行修改的，而第二种则使用了原子的“读-修改-写”操作。第一段所采用的普通读写方式会出什么问题呢？从时序图上我们可以看到，假定线程 A 与线程 B 几乎同时先对共享存储单元进行加载（读取），然后再几乎同时进行计算。由于线程 A 计算速度比较快，它先将计算好的结果存储（写入）到了共享存储单元中；而此后，线程 B 才计算好，再将它计算好的结果存储到该共享单元中，这就导致了数据不一致！因为线程 B 最终所写回的结果是基于一开始线程 A 对该共享存储单元修改之前的，它这么一改就把线程A对共享存储单元的操作给抹掉了。因此这里正确的做法应该是线程 B 必须基于线程 A 所修改完的结果再对共享存储单元进行操作。

第二段采用原子操作则不会有数据不一致问题，无论是线程 A 先对共享存储单元修改完还是线程 B 先修改完，最终结果都是经过这两个线程的计算操作的。CPU 系统总线仲裁器会去裁决哪个先执行，哪个后执行，而后执行的那个会被阻塞（等待）直到先前的操作完成。

>图片及说明引自 [C11标准的原子操作详解](https://www.jianshu.com/p/ae1f912a7607)。

因此，在多核心多线程并行计算的环境下，原子操作是唯一的数据同步手段。另外在此环境下，像互斥体（mutex）、信号量（semaphore）等同步原语的实现也都基于原子操作。

> 原语——在计算机科学中，原语指编程语言中最简单的可用元素。它是目标机器上最小的“可编程单元”。[维基](https://en.wikipedia.org/wiki/Language_primitive)

## 信号量

信号量是由 Edsger Dijkstra 最先提出的，通过一种叫做信号量（semaphore）的特殊类型的变量，实现同步不同执行线程。信号量 s 是具有非负整数值的全局变量，只能由两个特殊的操作来处理，P 和 V：

- P(s)：如果 s 是非零的，那么 P 将 s 减 1, 并且立即返回。如果 s 为零，那么就挂起这个线程，直到 s 变为非零，而一个 V 操作会重启这个线程。在重启之后，P 操作将 s 减 1， 并将控制返回给调用者。
- V(s)：V 操作将 s 加 1.如果有任何线程阻塞在 P 操作等待 s 变成非零，那么 V 操作会重启这鞋线程中的一个，然后该线程将 s 减 1，完成它的 P 操作。

P 中的测试和减 1 操作时不可分割的，也就是说，一旦预测信号量 s 变为非零，就会将 s 减 1，不能有中断。同理 V 中的加 1 操作也是不可分割的，也就是加载、加 1 和存储信号量的过程中没有中断。

> P 和 V 名字的起源：Edsger Dijkstra(1930-2002)出生于荷兰。名字 P 和 V 来源于荷兰语单词 Proberen（测试）和 Verhogen（增加）。

下面是 Posix 标准中定义的信号量函数：

```c
#include <semaphore.h>

int sem_init(sem_t *sem, 0, unsigned int value);
int sem_wait(sem_t *s);    /* P(s) */
int sem_post(sem_t *s);    /* V(s) */
```

## 锁的实现
了解完锁的基础，我来看看一些常见锁的实现：

### 互斥锁

### 自选锁锁

### 读写锁


## 锁的分类

在讲分类之前，先来明确一个概念。互斥锁（Mutex, mutual exlusion lock)，它是一个通用术语，用来表示具有强制排它访问语义的原语。

>原语([Language primitive](https://en.wikipedia.org/wiki/Language_primitive))


平时接触到的锁有很多名字：

- 阻塞锁：
- 忙等锁：
- 复杂锁：

## 锁的选择
### 避免死锁

大多属复杂的锁都可以被实现为阻塞锁或者复杂的自旋锁，而不会影响懂啊他们的功能或者接口。


## 后记
介绍锁的文章恐怕多的恐怕牛毛都要望其项背了，但是放眼望去，大多都是简单的罗列一下各个锁，然后在介绍一些用法，无外乎如此。好一点的在对比一下性能。曾几何时，在自己拜读完多篇关于锁的文章后，产生一种错觉，“原来锁并不复杂”。现在回头看来实在是肤浅幼稚。

随着对计算机知识的学习，深知自己还处在领域的底层，所以越来越不敢冠上原创二字，因为遇到的问题都有现成的解决方案。


## 参考

- 《深入理解UNIX系统内核》 Uresh Vahalia 著，李雨、薛磊、黄庆新 译
- 《C语言编程魔法书：基于 C11 标准》 陈轶 著
- [C11标准的原子操作详解](https://www.jianshu.com/p/ae1f912a7607)
