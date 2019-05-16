# Java 阿里面试题
TreeSet/HashSet 区别
HashMap 如何解决冲突,扩容机制
ConcurrentHashMap 如何做到高并发的
线程池平常怎么用
多个线程等待到某一节点然后统一放行有几种实现方式?
数据库索引结构
select * from t where a=? and b>? order by c limit 0,100 如何加索引
什么是聚簇索引和非聚簇索引
介绍下 Fescar/了解 CAP 吗?redis 里的 CAP 是怎样的?
如何理解幂等?项目中接口的幂等是如何做的?
算法题:两个有序的 list,求交集
蚂蚁金服财富 二面(两次电话面试)

第一次电面

解释下乐观锁悲观锁
JVM 判断对象是否回收?
反射能获得类里面方法的名称吗?参数名称呢?参数类型呢?
动态代理的实现方式?CgLib 和 jdk 的代理有什么区别?
分布式锁有哪些主流实现方式?redis 和 zk 锁有什么区别?
ThreadLocal 作用是什么?说下用法
设计秒杀系统要考虑哪些点? 数据预热,CDN 缓存,超卖问题,流量削峰
从性能角度上如何保证秒杀系统稳定? 觉得应该是预热,削峰,库存扣减的操作放到缓存,减少数据库的访问
A,B 系统转账如何保证分布式数据一致性?
你有什么想问的?
第二次电面

简单介绍下自己的近况
对 kafka 了解吗?Rocketmq 事务性消息怎么实现的?
假设事务提交的消息丢了,没有发到 broker 上面,会怎么处理?
分布式事务一致性如何保证?
二阶段提交解释下,如果本地事务超时会怎样处理?一阶段都可以 commit,二阶段的时候协调器发送全局提交的消息,这个时候消息发送失败了会怎么处理?
TCC 对异常流是如何操作的?
为什么要看开源代码?
最终一致性如何实现的?
有没有遇到过死锁?
A 往 B 转钱,B 往 A 转钱,同时的会死锁吗?如何解决死锁?
设计一个全局唯一流水号? ​ https://tech.meituan.com/2019/03/07/open-source-project-leaf.html
设计幂等方案防止重复提交? ​ 进页面的时候生成一个 token,请求带 token 过来,校验 token
大数相加
工厂方法模式一般如何实现?
单例模式
其他的设计模式
再答一下上次的秒杀系统
写金融类的系统有什么需要关注的地方?
非功能性的设计关注那些? ​ 其实想问你日志规范,代码规范之类的
你有什么想问的?
蚂蚁三面

如何提前发现你的数据有问题,而不是等到用户反馈才知道?

如何防止超卖?

为什么要用 Redis?为什么没有用 db? 数据量不小,查一次耗时长

有没有 QPS?(没有统计过指标)

如何部署?(阿里云采购)

发生过丢消息的情况吗?为什么会丢失? rocket 保证消息不丢失

项目的过程中哪个点比较难?

项目中为什么要用 ThreadLocal 去做租户的隔离?

项目有什么缺点?

100 亿行数据,每个数字 32 位,取最小的的数字

有没有碰到特别难的事情,如何解决的?

业界中间件有什么了解吗?讲一个你深度理解原理的.

高并发的问题有遇到过吗?分布式锁是排他的,如何提升存储效率? ​ 遇到的不多,但如果请求比较多的话会用分布式锁来解决,这么说也不对.应该是递进关系的,后面就给自己挖坑了 ​ 分布式锁是排他的,如何提升存储效率,应该是吞吐量的意思吧

有遇到过很大的流量吗? ​ 没有

描述产生一次 fullGC 的整个过程 ​ fullGC 解析

平时通过什么来提升自己 ​ 通过博客 /技术书籍

你有什么问题? (您觉得我的短板在于?) ​ 没有互联网的思维,不会估算系统容量就往上发,这怎么能行,数据双写如何保证一致 ​

蚂蚁金服国际事业群一面

头一天约了时间,第二天准时打电话过来了,整体下来偏基础,发现自己 JVM 相关菜的一笔,要好好补补,重新看一遍深入理解 java 虚拟机了

简单介绍下自己和项目 blablabla....

你觉得项目里最大的挑战是什么? 答得项目的设计和推动

为什么选择用 rocketmq? 公司推的呗

对 rocketmq 的了解?基本原理 发布订阅,服务注册,消息丢失的情况(几种情况:producer down,broker down,consumer down)

消息如何顺序? 单一队列,多队列分区顺序

rocketmq 事务消息,producer 如果 down 掉,事务无法提交这种情况 https://github.com/apache/rocketmq/blob/master/docs/cn/design.md

rocketmq 是强一致性还是弱一致性

消息重复如何解决?可以在中间件层解决吗?MQ 体系协议了解吗? 业务层幂等,中间件层 kafka 有做过这个处理,不了解

ThreadLocal 是怎样的概念?如何实现线程隔离的?基于这个原理有没有更加优化的方式? ​ 这个优化让我懵逼了

线程池用 ThreadLocal 有什么问题?有什么思路来让业务方不去关注 ThreadLocal 的 set ​ ThreadLocal 在线程复用的时候值可能不是最新的,需要每次都 set

你说你用过 dubbo,那看过 netty 源码吗?
​ 不好意思没有....(那你回去看一下 netty 里的 FastThreadLocal,对 threadLocal 有优化) ​ https://blog.csdn.net/TheLudlows/article/details/85267651

讲讲 dubbo 的基本原理

JVM 调优?
​ 没做过,说了下公司的一些 JVM 参数

频繁 YGC 如何排查?
​ 支支吾吾半天....

换个问题,为什么会发生 YGC?

如果知道哪些对象需要被回收?
​ GCROOT

GCROOT 有哪些?

栈和堆有什么区别?

什么时候会压栈?

程序都是线程在执行,线程和栈和堆的关系? ​ 懵逼了,后来发现是想问,栈线程私有,堆公共,一时没想起来

HashMap 如何实现?为什么会变红黑树呢?如何扩容?为什么是两倍?什么时候会用到位运算?
​ hashmap 到处都是位运算,居然只答出来一个扩容的,脑残 ​ indexFor 取位置的时候也是用的位运算 h&length-1 ​ 有提到扩容的问题.扩容两倍的原因,也是为了取模可以通过上面这个&操作来做提升性能,%据说慢十倍,咱也不知道,咱也不敢问 ​ https://blog.csdn.net/u014532901/article/details/78936283

你有什么想问的?