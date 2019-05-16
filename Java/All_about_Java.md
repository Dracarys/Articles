# Java 知识点

## 一、基础篇

JVM

JVM 内存结构

堆、栈、方法区、直接内存、堆和栈区别

Java 内存模型

内存可见性、重排序、顺序一致性、volatile、锁、final

垃圾回收

内存分配策略、垃圾收集器（ G1 ）、GC 算法、GC 参数、对象存活的判定

JVM 参数及调优

Java 对象模型

oop-klass、对象头

HotSpot

即时编译器、编译优化

类加载机制

classLoader、类加载过程、双亲委派（破坏双亲委派）、模块化（ jboss modules、osgi、jigsaw ）

虚拟机性能监控与故障处理工具

jps, jstack, jmap、jstat, jconsole, jinfo, jhat, javap, btrace、TProfiler

编译与反编译

javac、javap、jad、CRF

Java 基础知识

阅读源代码

String、Integer、Long、Enum、BigDecimal、ThreadLocal、ClassLoader & URLClassLoader、ArrayList & LinkedList、HashMap & LinkedHashMap & TreeMap & CouncurrentHashMap、HashSet & LinkedHashSet & TreeSet

Java 中各种变量类型

熟悉 Java String 的使用，熟悉 String 的各种函数

JDK 6 和 JDK 7 中 substring 的原理及区别、

replaceFirst、replaceAll、replace 区别、

String 对“+”的重载、

String.valueOf 和 Integer.toString 的区别、

字符串的不可变性

自动拆装箱

Integer 的缓存机制

熟悉 Java 中各种关键字

transient、instanceof、volatile、synchronized、final、static、const 原理及用法。

集合类

常用集合类的使用、ArrayList 和 LinkedList 和 Vector 的区别 、SynchronizedList 和 Vector 的区别、HashMap、HashTable、ConcurrentHashMap 区别、Java 8 中 stream 相关用法、apache 集合处理工具类的使用、不同版本的 JDK 中 HashMap 的实现的区别以及原因

枚举

枚举的用法、枚举与单例、Enum 类

Java IO&Java NIO，并学会使用

bio、nio 和 aio 的区别、三种 IO 的用法与原理、netty

Java 反射与 javassist

反射与工厂模式、 java.lang.reflect.*

Java 序列化

什么是序列化与反序列化、为什么序列化、序列化底层原理、序列化与单例模式、protobuf、为什么说序列化并不安全

注解

元注解、自定义注解、Java 中常用注解使用、注解与反射的结合

JMS

什么是 Java 消息服务、JMS 消息传送模型

JMX

java.lang.management.*、 javax.management.*

泛型

泛型与继承、类型擦除、泛型中 K T V E ？ object 等的含义、泛型各种用法

单元测试

junit、mock、mockito、内存数据库（ h2 ）

正则表达式

java.lang.util.regex.*

常用的 Java 工具库

commons.lang, commons.*... guava-libraries netty

什么是 API&SPI

异常

异常类型、正确处理异常、自定义异常

时间处理

时区、时令、Java 中时间 API

编码方式

解决乱码问题、常用编码方式

语法糖

Java 中语法糖原理、解语法糖

Java 并发编程

什么是线程，与进程的区别

阅读源代码，并学会使用

Thread、Runnable、Callable、ReentrantLock、ReentrantReadWriteLock、Atomic*、Semaphore、CountDownLatch、、ConcurrentHashMap、Executors

线程池

自己设计线程池、submit() 和 execute()

线程安全

死锁、死锁如何排查、Java 线程调度、线程安全和内存模型的关系

锁

CAS、乐观锁与悲观锁、数据库相关锁机制、分布式锁、偏向锁、轻量级锁、重量级锁、monitor、锁优化、锁消除、锁粗化、自旋锁、可重入锁、阻塞锁、死锁

死锁

volatile

happens-before、编译器指令重排和 CPU 指令重

synchronized

synchronized 是如何实现的？ synchronized 和 lock 之间关系、不使用 synchronized 如何实现一个线程安全的单例

sleep 和 wait

wait 和 notify

notify 和 notifyAll

ThreadLocal

写一个死锁的程序

写代码来解决生产者消费者问题

守护线程

守护线程和非守护线程的区别以及用法

## 二、 进阶篇

Java 底层知识

字节码、class 文件格式

CPU 缓存，L1，L2，L3 和伪共享

尾递归

位运算

用位运算实现加、减、乘、除、取余

设计模式

了解 23 种设计模式

会使用常用设计模式

单例、策略、工厂、适配器、责任链。

实现 AOP

实现 IOC

不用 synchronized 和 lock，实现线程安全的单例模式

nio 和 reactor 设计模式

网络编程知识

tcp、udp、http、https 等常用协议

三次握手与四次关闭、流量控制和拥塞控制、OSI 七层模型、tcp 粘包与拆包

http/1.0 http/1.1 http/2 之前的区别

Java RMI，Socket，HttpClient

cookie 与 session

cookie 被禁用，如何实现 session

用 Java 写一个简单的静态文件的 HTTP 服务器

实现客户端缓存功能，支持返回 304 实现可并发下载一个文件 使用线程池处理客户端请求 使用 nio 处理客户端请求 支持简单的 rewrite 规则 上述功能在实现的时候需要满足“开闭原则”

了解 nginx 和 apache 服务器的特性并搭建一个对应的服务器

用 Java 实现 FTP、SMTP 协议

进程间通讯的方式

什么是 CDN ？如果实现？

什么是 DNS ？

反向代理

框架知识

Servlet 线程安全问题

Servlet 中的 filter 和 listener

Hibernate 的缓存机制

Hiberate 的懒加载

Spring Bean 的初始化

Spring 的 AOP 原理

自己实现 Spring 的 IOC

Spring MVC

Spring Boot2.0

Spring Boot 的 starter 原理，自己实现一个 starter

Spring Security

应用服务器知识

JBoss

tomcat

jetty

Weblogic

工具

git & svn

maven & gradle

## 三、 高级篇

新技术

Java 8

lambda 表达式、Stream API、

Java 9

Jigsaw、Jshell、Reactive Streams

Java 10

局部变量类型推断、G1 的并行 Full GC、ThreadLocal 握手机制

Spring 5

响应式编程

Spring Boot 2.0

性能优化

使用单例、使用 Future 模式、使用线程池、选择就绪、减少上下文切换、减少锁粒度、数据压缩、结果缓存

线上问题分析

dump 获取

线程 Dump、内存 Dump、gc 情况

dump 分析

分析死锁、分析内存泄露

自己编写各种 outofmemory，stackoverflow 程序

HeapOutOfMemory、Young OutOfMemory、MethodArea OutOfMemory、ConstantPool OutOfMemory、DirectMemory OutOfMemory、Stack OutOfMemory Stack OverFlow

常见问题解决思路

内存溢出、线程死锁、类加载冲突

使用工具尝试解决以下问题，并写下总结

当一个 Java 程序响应很慢时如何查找问题、

当一个 Java 程序频繁 FullGC 时如何解决问题、

如何查看垃圾回收日志、

当一个 Java 应用发生 OutOfMemory 时该如何解决、

如何判断是否出现死锁、

如何判断是否存在内存泄露

编译原理知识

编译与反编译

Java 代码的编译与反编译

Java 的反编译工具

词法分析，语法分析（ LL 算法，递归下降算法，LR 算法），语义分析，运行时环境，中间代码，代码生成，代码优化

操作系统知识

Linux 的常用命令

进程同步

缓冲区溢出

分段和分页

虚拟内存与主存

数据库知识

MySql 执行引擎

MySQL 执行计划

如何查看执行计划，如何根据执行计划进行 SQL 优化

SQL 优化

事务

事务的隔离级别、事务能不能实现锁的功能

数据库锁

行锁、表锁、使用数据库锁实现乐观锁、

数据库主备搭建

binlog

内存数据库

h2

常用的 nosql 数据库

redis、memcached

分别使用数据库锁、NoSql 实现分布式锁

性能调优

数据结构与算法知识

简单的数据结构

栈、队列、链表、数组、哈希表、

树

二叉树、字典树、平衡树、排序树、B 树、B+树、R 树、多路树、红黑树

排序算法

各种排序算法和时间复杂度 深度优先和广度优先搜索 全排列、贪心算法、KMP 算法、hash 算法、海量数据处理

大数据知识

Zookeeper

基本概念、常见用法

Solr，Lucene，ElasticSearch

在 linux 上部署 solr，solrcloud，，新增、删除、查询索引

Storm，流式计算，了解 Spark，S4

在 linux 上部署 storm，用 zookeeper 做协调，运行 storm hello world，local 和 remote 模式运行调试 storm topology。

Hadoop，离线计算

HDFS、MapReduce

分布式日志收集 flume，kafka，logstash

数据挖掘，mahout

网络安全知识

什么是 XSS

XSS 的防御

什么是 CSRF

什么是注入攻击

SQL 注入、XML 注入、CRLF 注入

什么是文件上传漏洞

加密与解密

MD5，SHA1、DES、AES、RSA、DSA

什么是 DOS 攻击和 DDOS 攻击

memcached 为什么可以导致 DDos 攻击、什么是反射型 DDoS

SSL、TLS，HTTPS

如何通过 Hash 碰撞进行 DOS 攻击

用 openssl 签一个证书部署到 apache 或 nginx

## 四、架构篇

分布式

数据一致性、服务治理、服务降级

分布式事务

2PC、3PC、CAP、BASE、 可靠消息最终一致性、最大努力通知、TCC

Dubbo

服务注册、服务发现，服务治理

分布式数据库

怎样打造一个分布式数据库、什么时候需要分布式数据库、mycat、otter、HBase

分布式文件系统

mfs、fastdfs

分布式缓存

缓存一致性、缓存命中率、缓存冗余

微服务

SOA、康威定律

ServiceMesh

Docker & Kubernets

Spring Boot

Spring Cloud

高并发

分库分表

CDN 技术

消息队列

ActiveMQ

监控

监控什么

CPU、内存、磁盘 I/O、网络 I/O 等

监控手段

进程监控、语义监控、机器资源监控、数据波动

监控数据采集

日志、埋点

Dapper

负载均衡

tomcat 负载均衡、Nginx 负载均衡

DNS

DNS 原理、DNS 的设计

CDN

数据一致性

## 五、 扩展篇

云计算

IaaS、SaaS、PaaS、虚拟化技术、openstack、Serverlsess

搜索引擎

Solr、Lucene、Nutch、Elasticsearch

权限管理

Shiro

区块链

哈希算法、Merkle 树、公钥密码算法、共识算法、Raft 协议、Paxos 算法与 Raft 算法、拜占庭问题与算法、消息认证码与数字签名

比特币

挖矿、共识机制、闪电网络、侧链、热点问题、分叉

以太坊

超级账本

人工智能

数学基础、机器学习、人工神经网络、深度学习、应用场景。

常用框架

TensorFlow、DeepLearning4J

其他语言

Groovy、Python、Go、NodeJs、Swift、Rust

## 六、 推荐书籍

《深入理解 Java 虚拟机》 《 Effective Java 》 《深入分析 Java Web 技术内幕》 《大型网站技术架构》 《代码整洁之道》 《 Head First 设计模式》 《 maven 实战》 《区块链原理、设计与应用》 《 Java 并发编程实战》 《鸟哥的 Linux 私房菜》 《从 Paxos 到 Zookeeper 》 《架构即未来》

## 最后

这是很久以前摘录的一片有关 Java 开发的知识点，非本人创作，但是因为当时忘了记录出处，所以暂时不详。