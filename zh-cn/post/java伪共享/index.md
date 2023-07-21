# Java伪共享


Java伪共享

<!--more-->



### Java伪共享

维基百科对伪共享的定义如下：

> In computer science, false sharing is a performance-degrading usage pattern that can arise in systems with distributed, coherent caches at the size of the smallest resource block managed by the caching mechanism. When a system participant attempts to periodically access data that will never be altered by another party, but those data shares a cache block with data that are altered, the caching protocol may force the first participant to reload the whole unit despite a lack of logical necessity. The caching system is unaware of activity within this block and forces the first participant to bear the caching system overhead required by true shared access of a resource

其大致意思是：
CPU的缓存是以缓存行(cache line)为单位进行缓存的，当多个线程修改不同变量，而这些变量又处于同一个缓存行时就会影响彼此的性能。例如：线程1和线程2共享一个缓存行，线程1只读取缓存行中的变量1，线程2修改缓存行中的变量2，虽然线程1和线程2操作的是不同的变量，由于变量1和变量2同处于一个缓存行中，当变量2被修改后，缓存行失效，线程1要重新从主存(或者低级cache)中读取，因此导致缓存失效，从而产生性能问题。为了更深入一步理解伪共享，我们先看一下CPU缓存

#### 1、Cpu三级缓存

CPU的速度要远远大于内存的速度，为了解决这个问题，CPU引入了三级缓存：L1，L2和L3三个级别，L1最靠近CPU，L2次之，L3离CPU最远，L3之后才是主存。速度是L1>L2>L3>主存。越靠近CPU的容量越小。CPU获取数据会依次从三级缓存中查找，如果找不到再从主存中加载。

当CPU要读取一个数据时，首先从一级缓存中查找，如果没有找到再从二级缓存中查找，如果还是没有就从三级缓存或内存中查找。一般来说，每级缓存的命中率大概都在80%左右，也就是说全部数据量的80%都可以在一级缓存中找到，只剩下20%的总数据量才需要从二级缓存、三级缓存或内存中读取，由此可见一级缓存是整个CPU缓存架构中最为重要的部分

#### 2、MESI协议（高速缓存一致性协议）

##### **缓存行状态**

CPU的缓存是以缓存行(cache line)为单位的，MESI协议描述了多核处理器中一个缓存行的状态。在MESI协议中，每个缓存行有4个状态，分别是：

- M（修改，Modified）：本地处理器已经修改缓存行，即是脏行，它的内容与内存中的内容不一样，并且此 cache 只有本地一个拷贝(专有)；
- E（专有，Exclusive）：缓存行内容和内存中的一样，而且其它处理器都没有这行数据；
- S（共享，Shared）：缓存行内容和内存中的一样, 有可能其它处理器也存在此缓存行的拷贝；
- I（无效，Invalid）：缓存行失效, 不能使用

##### **缓存行状态转换**

在MESI协议中，每个Cache的Cache控制器不仅知道自己的读写操作，而且也监听(snoop)其它Cache的读写操作。每个Cache line所处的状态根据本核和其它核的读写操作在4个状态间进行迁移。MESI协议状态迁移图如下：

![image-20221214175723524](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221214175723524.png)

MESI协议-状态转换

- 初始：一开始时，缓存行没有加载任何数据，所以它处于 I 状态。
- 本地写（Local Write）：如果本地处理器写数据至处于 I 状态的缓存行，则缓存行的状态变成 M。
- 本地读（Local Read）：如果本地处理器读取处于 I 状态的缓存行，很明显此缓存没有数据给它。此时分两种情况：(1)其它处理器的缓存里也没有此行数据，则从内存加载数据到此缓存行后，再将它设成 E 状态，表示只有我一家有这条数据，其它处理器都没有；(2)其它处理器的缓存有此行数据，则将此缓存行的状态设为 S 状态。（备注：如果处于M状态的缓存行，再由本地处理器写入/读出，状态是不会改变的）
- 远程读（Remote Read）：假设我们有两个处理器 c1 和 c2，如果 c2 需要读另外一个处理器 c1 的缓存行内容，c1 需要把它缓存行的内容通过内存控制器 (Memory Controller) 发送给 c2，c2 接到后将相应的缓存行状态设为 S。在设置之前，内存也得从总线上得到这份数据并保存。
- 远程写（Remote Write）：其实确切地说不是远程写，而是 c2 得到 c1 的数据后，不是为了读，而是为了写。也算是本地写，只是 c1 也拥有这份数据的拷贝，这该怎么办呢？c2 将发出一个 RFO (Request For Owner) 请求，它需要拥有这行数据的权限，其它处理器的相应缓存行设为 I，除了它自已，谁不能动这行数据。这保证了数据的安全，同时处理 RFO 请求以及设置I的过程将给写操作带来很大的性能消耗。

##### 缓存行

CPU缓存是以缓存行（cache line）为单位存储的。缓存行通常是 64 字节，并且它有效地引用主内存中的一块地址。一个 Java 的 long 类型是 8 字节，因此在一个缓存行中可以存 8 个 long 类型的变量。所以，如果你访问一个 long 数组，当数组中的一个值被加载到缓存中，它会额外加载另外 7 个，以致你能非常快地遍历这个数组。事实上，你可以非常快速的遍历在连续的内存块中分配的任意数据结构。而如果你在数据结构中的项在内存中不是彼此相邻的（如链表），你将得不到免费缓存加载所带来的优势，并且在这些数据结构中的每一个项都可能会出现缓存未命中。下图是一个CPU缓存行的示意图：

![image-20221214175906551](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221214175906551.png)

上图中，一个运行在处理器 core1上的线程想要更新变量 X 的值，同时另外一个运行在处理器 core2 上的线程想要更新变量 Y 的值。但是，这两个频繁改动的变量都处于同一条缓存行。两个线程就会轮番发送 RFO 消息，占得此缓存行的拥有权。当 core1 取得了拥有权开始更新 X，则 core2 对应的缓存行需要设为 I 状态。当 core2 取得了拥有权开始更新 Y，则 core1 对应的缓存行需要设为 I 状态(失效态)。轮番夺取拥有权不但带来大量的 RFO 消息，而且如果某个线程需要读此行数据时，L1 和 L2 缓存上都是失效数据，只有 L3 缓存上是同步好的数据。从前一篇我们知道，读 L3 的数据非常影响性能。更坏的情况是跨槽读取，L3 都要 miss，只能从内存上加载。

**表面上 X 和 Y 都是被独立线程操作的，而且两操作之间也没有任何关系。只不过它们共享了一个缓存行，但所有竞争冲突都是来源于共享。**

#### 3、java中的伪共享

解决伪共享最直接的方法就是填充（padding），例如下面的VolatileLong，一个long占8个字节，Java的对象头占用8个字节（32位系统）或者12字节（64位系统，默认开启对象头压缩，不开启占16字节）。一个缓存行64字节，那么我们可以填充6个long（6 * 8 = 48 个字节）。这样就能避免多个VolatileLong共享缓存行。

```java
public class VolatileLong {
    private volatile long v;
    // private long v0, v1, v2, v3, v4, v5  // 去掉注释，开启填充，避免缓存行共享
}
```

这是最简单直接的方法，Java 8中引入了一个更加简单的解决方案：`@Contended`注解：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.TYPE})
public @interface Contended {
    String value() default "";
}
```

Contended注解可以用于类型上和属性上，加上这个注解之后虚拟机会自动进行填充，从而避免伪共享。这个注解在Java8 ConcurrentHashMap、ForkJoinPool和Thread等类中都有应用。我们来看一下Java8中ConcurrentHashMap中如何运用Contended这个注解来解决伪共享问题。以下说的ConcurrentHashMap都是Java8版本。

##### ConcurrentHashMap中伪共享解决方案

ConcurrentHashMap的size操作通过CounterCell来计算，哈希表中的每个节点都对用了一个CounterCell，每个CounterCell记录了对应Node的键值对数目。这样每次计算size时累加各个CounterCell就可以了。ConcurrentHashMap中CounterCell以数组形式保存，而数组在内存中是连续存储的，CounterCell中只有一个long类型的value属性，这样CPU会缓存CounterCell临近的CounterCell，于是就形成了伪共享。

ConcurrentHashMap中用Contended注解自动对CounterCell来进行填充：

```java
/**
 * Table of counter cells. When non-null, size is a power of 2.
 */
private transient volatile CounterCell[] counterCells; // CounterCell数组，CounterCell在内存中连续

public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}

// 计算size时直接对各个CounterCell的value进行累加
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}

// 使用Contended注解自动进行填充避免伪共享
@sun.misc.Contended static final class CounterCell {
    volatile long value;
    CounterCell(long x) { value = x; }
}
```

需要注意的是`@sun.misc.Contended`注解在user classpath中是不起作用的，需要通过一个虚拟机参数来开启：-XX:-RestrictContended

#### 4、总结

1. **CPU缓存是以缓存行为单位进行操作的。产生伪共享问题的根源在于不同的核同时操作同一个缓存行。**
1. **可以通过填充来解决伪共享问题，Java8 中引入了`@sun.misc.Contended`注解来自动填充。**
1. **并不是所有的场景都需要解决伪共享问题，因为CPU缓存是有限的，填充会牺牲掉一部分缓存**

---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/java%E4%BC%AA%E5%85%B1%E4%BA%AB/  

