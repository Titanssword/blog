---
title: 服务端面试场景问题 
data: 2024/9/10　
tags:
- interview

categories:
- interview
summary: "服务端面试场景问题&解答"
---
## 假设有一个1个G大的hashmap，此时用户请求过来刚好触发它的扩容，会怎么样，如何优化？
在面对一个假设场景，即一个1GB大小的HashMap在运行时因用户请求触发扩容操作，我们需要从多个维度来探讨这一问题的影响、可能的后果以及如何进行有效的优化。作为高级程序员，我们深知HashMap在Java等编程语言中的重要性，它提供了快速的键值对查找功能，但扩容操作（rehashing）是一个资源密集型的过程，可能严重影响应用的性能和响应时间。

影响与后果
性能下降：HashMap扩容时，需要重新计算所有现有元素的哈希值，并根据新的容量重新定位这些元素。这个过程会占用大量CPU资源，导致系统响应变慢，尤其是在高并发环境下，这种影响更为显著。

内存使用波动：扩容过程中，HashMap会暂时占用更多的内存来存储旧数据结构和新数据结构，直到所有元素都被重新定位到新的表中。这可能导致内存使用量急剧上升，如果系统内存不足，还可能引发垃圾回收（GC），进一步影响性能。

线程安全问题：如果HashMap被多个线程同时访问且未进行适当的同步或使用了线程安全的变体如ConcurrentHashMap，扩容过程中可能会出现数据不一致的问题。

优化策略
合理预估初始容量和负载因子：

在创建HashMap时，尽可能根据预期数据量预估一个合适的初始容量（initialCapacity）和负载因子（loadFactor）。初始容量过大会浪费内存，过小则会导致频繁扩容；负载因子决定了HashMap何时开始扩容，较小的负载因子可以减少扩容次数，但会增加空间占用。
int initialCapacity = 1000000; // 预估的初始键值对数量
float loadFactor = 0.75f; // 负载因子，默认也是0.75
HashMap<KeyType, ValueType> map = new HashMap<>(initialCapacity, loadFactor);
使用更高效的并发集合：

如果HashMap被多个线程并发访问，考虑使用ConcurrentHashMap。它不仅支持更高的并发级别，还通过分段锁等技术减少了扩容时的锁竞争，从而提高了性能。
软引用和弱引用：

对于非必须立即加载的数据，可以考虑使用SoftReference或WeakReference来存储HashMap的引用。这样，在JVM内存不足时，这些引用指向的对象可以被垃圾回收器回收，减少内存压力。
扩容策略优化：

如果HashMap的扩容行为对应用性能影响过大，且无法通过调整初始容量和负载因子来有效缓解，可以考虑自定义HashMap实现，优化其扩容逻辑。例如，可以设计一种更加渐进的扩容策略，避免一次性将所有数据重新哈希。
监控与预警：

引入监控系统，实时监控HashMap的使用情况，包括当前大小、扩容频率等。当发现扩容过于频繁时，及时发出预警，以便开发人员介入分析和优化。

## golang channel 的底层实现

[https://i6448038.github.io/2019/04/11/go-channel/](https://i6448038.github.io/2019/04/11/go-channel/)

## GMP机制
[https://go.cyub.vip/gmp/gmp-model/](https://go.cyub.vip/gmp/gmp-model/)