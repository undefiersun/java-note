Synchronized 相关问题
1.Sychronized用过吗？其原理是什么？
2.你刚才提到获取对象的锁，这个“锁”到底是什么？如何确定对象的锁？

3.什么是可重入性，为什么说 Synchronized 是可重入锁？

4.JVM 对 Java 的原生锁做了哪些优化？

5.为什么说 Synchronized 是非公平锁？

6.什么是锁消除和锁粗化？

7.为什么说 Synchronized 是一个悲观锁？乐观锁的实现原理又是什么？什么是

8.乐观锁一定就是好的吗？
可重入锁 ReentrantLock 及其他显式锁相关问题
跟 Synchronized 相比，可重入锁 ReentrantLock 其实现原理有什么不同？

那么请谈谈 AQS 框架是怎么回事儿？

请尽可能详尽地对比下 Synchronized 和 ReentrantLock 的异同。

ReentrantLock 是如何实现可重入性的？

除了 ReetrantLock，你还接触过 JUC 中的哪些并发工具？

请谈谈 ReadWriteLock 和 StampedLock。

如何让 Java 的线程彼此同步？你了解过哪些同步器？请分别介绍下。

CyclicBarrier 和 CountDownLatch 看起来很相似，请对比下呢？
Java 线程池相关问题
Java 中的线程池是如何实现的？

创建线程池的几个核心构造参数？

线程池中的线程是怎么创建的？是一开始就随着线程池的启动创建好的吗？

既然提到可以通过配置不同参数创建出不同的线程池，那么 Java 中默认实现好的线程池又有哪些呢？请比较它们的异同。

如何在 Java 线程池中提交线程？
Java 内存模型相关问题
什么是 Java 的内存模型，Java 中各个线程是怎么彼此看到对方的变量的？

请谈谈 volatile 有什么特点，为什么它能保证变量对所有线程的可见性？

既然 volatile 能够保证线程间的变量可见性，是不是就意味着基于 volatile 变量的运算就是并发安全的？

请对比下 volatile 对比 Synchronized 的异同。

请谈谈 ThreadLocal 是怎么解决并发安全的？

很多人都说要慎用 ThreadLocal，谈谈你的理解，使用 ThreadLocal 需要注意些什么？
并发队列相关问题
谈下对基于链表的非阻塞无界队列 ConcurrentLinkedQueue 原理的理解？

ConcurrentLinkedQueue 内部是如何使用 CAS 非阻塞算法来保证多线程下入队出队操作的线程安全？

基于链表的阻塞队列 LinkedBlockingQueue 原理。

阻塞队列LinkedBlockingQueue 内部是如何使用两个独占锁 ReentrantLock 以及对应的条件变量保证多线程先入队出队操作的线程安全？

为什么不使用一把锁，使用两把为何能提高并发度？

基于数组的阻塞队列 ArrayBlockingQueue 原理。

ArrayBlockingQueue 内部如何基于一把独占锁以及对应的两个条件变量实现出入队操作的线程安全？

谈谈对无界优先级队列 PriorityBlockingQueue 原理？

PriorityBlockingQueue 内部使用堆算法保证每次出队都是优先级最高的元素，元素入队时候是如何建堆的，元素出队后如何调整堆的平衡的？
