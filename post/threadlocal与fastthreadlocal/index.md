# ThreadLocal与FastThreadLocal


ThreadLocal与FastThreadLocal

<!--more-->

### ThreadLocal与FastThreadLocal

FastThreadLocal是Netty中常用的一个工具类，他的基本功能与JDK自带的[ThreadLocal](https://codingroam.github.io/post/threadlocal/)一样，但是性能优于ThreadLocal。在讲解FastThreadLocal之前，先大致讲一下ThreadLocal的原理。

#### 1、ThreadLocal

如果想要在线程中保存一个变量，这个变量是该线程所独有的，其他线程不能对该变量进行访问和修改，那么我们可以使用ThreadLocal实现这一功能.

ThreadLocal的功能主要是通过ThreadLocalMap来实现的，每个线程都会有一个ThreadLocalMap类型的成员变量，ThreadLocalMap本质上就是一个哈希表，key为ThreadLocal，value为该ThreadLocal为该线程保存的变量，如下图所示：
![image-20221214165714815](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221214165714815.png)

ThreadLocalMap中使用一个Entry类型的数组来作为哈希表，并用线性探测法解决哈希冲突，数组的大小永远是2的n次方。Entry是一个静态内部类，且继承了WeakReference<ThreadLocal<?>>，表示其保存的ThreadLocal是一个弱引用，这是为了防止内存泄漏，因为当所有指向ThreadLocal的强引用都为null时，该Entry也就没有必要再指向这个ThreadLocal了，这样下一次gc时该ThreadLoca就会被回收掉。Entry的成员变量Object value就是T和read Local为线程保存的独有的变量。

set方法将ThreadLocal和相应的value保存为一个Entry，再根据ThreadLocal的哈希值将Entry放入哈希表中的相应的位置，并用线性探测法解决哈希冲突。setEntry方法中，如果在查找哈希表的过程中发现某个Entry所保存的ThreadLocal被垃圾回收了，那么会将该Entry清除掉

```java
private ThreadLocal.ThreadLocalMap.Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    ThreadLocal.ThreadLocalMap.Entry e = table[i];
    //找到了相应的entry，直接返回
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
 
 
private ThreadLocal.ThreadLocalMap.Entry getEntryAfterMiss(ThreadLocal<?> key, int i, ThreadLocal.ThreadLocalMap.Entry e) {
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
 
    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        //该threadLocal已经为null，那么entry也就没有存在的必要了，因此需要被清除
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
 
 
private void set(ThreadLocal<?> key, Object value) {
 
    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;
    //将threadlocal的哈希值与哈希表的长度取余，放入哈希表中相应的位置
    int i = key.threadLocalHashCode & (len-1);
    
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        //哈希表中保存了这个threadLocal，因此直接修改value值
        if (k == key) {
            e.value = value;
            return;
        }
        //entry不为null，但entry保存threadLocal为null，说明该threadLocal已经被垃圾回收了，需要替换掉这个entry
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
 
    tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);
    int sz = ++size;
    //如果哈希表的装载因子超过阈值，则需要对哈希表扩容，并对所有的entry重新做哈希
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

#### 2、FastThreadLocal

从 InternalThreadLocalMap 内部实现来看，与 ThreadLocalMap 一样都是采用数组的存储方式，但是 InternalThreadLocalMap 并没有使用线性探测法来解决 Hash 冲突，而是在 FastThreadLocal 初始化的时候分配一个数组索引 index，index 的值采用原子类 AtomicInteger 保证顺序递增，通过调用 InternalThreadLocalMap.nextVariableIndex() 方法获得。然后在读写数据的时候通过数组下标 index 直接定位到 FastThreadLocal 的位置，时间复杂度为 O(1)。如果数组下标递增到非常大，那么数组也会比较大，所以 FastThreadLocal 是通过空间换时间的思想提升读写性能

```java
public FastThreadLocal() {
    index = InternalThreadLocalMap.nextVariableIndex();
}
 
public static int nextVariableIndex() {
    //nextIndex是一个静态变量，每次调用nextVariableIndex()都会自增1，让后赋给FastThreadLocal的index属性
    int index = nextIndex.getAndIncrement();
    if (index < 0) {
        nextIndex.decrementAndGet();
        throw new IllegalStateException("too many thread-local indexed variables");
    }
    return index;
}
 
static final AtomicInteger nextIndex = new AtomicInteger();
```

![image-20221214212559533](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221214212559533.png)

通过上面 FastThreadLocal 的内部结构图可知，FastThreadLocal 使用 Object 数组替代了 Entry 数组，Object[0] 存储的是一个Set<FastThreadLocal<?>> 集合，从数组下标 1 开始都是直接存储的 value 数据，不再采用 ThreadLocal 的键值对形式进行存储。

假设现在我们有一批数据需要添加到数组中，分别为 value1、value2、value3、value4，对应的 FastThreadLocal 在初始化的时候生成的数组索引分别为 1、2、3、4。如下图所示。


![image-20221214212704846](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221214212704846.png)

##### 使用代码示例

```java
public class FastThreadLocalTest {
    private static final FastThreadLocal<;String> THREAD_NAME_LOCAL = new FastThreadLocal<;>();
    private static final FastThreadLocal<;TradeOrder> TRADE_THREAD_LOCAL = new FastThreadLocal<;>();
    public static void main(String[] args) {
        for (int i = 0; i <; 2; i++) {
            int tradeId = i;
            String threadName = "thread-" + i;
            new FastThreadLocalThread(() -> {
                THREAD_NAME_LOCAL.set(threadName);
                TradeOrder tradeOrder = new TradeOrder(tradeId, tradeId % 2 == 0 ? "已支付" : "未支付");
                TRADE_THREAD_LOCAL.set(tradeOrder);
                System.out.println("threadName: " + THREAD_NAME_LOCAL.get());
                System.out.println("tradeOrder info：" + TRADE_THREAD_LOCAL.get());
            }, threadName).start();
        }
    }
}

```

FastThreadLocal 的使用方法几乎和 ThreadLocal 保持一致，只需要把代码中 Thread、ThreadLocal 替换为 FastThreadLocalThread 和 FastThreadLocal 即可。

##### FastThreadLocal源码分析

+ FastThreadLocal.set()

  ```java
  public final void set(V value) {
      if (value != InternalThreadLocalMap.UNSET) { // 1. value 是否为缺省值
          InternalThreadLocalMap threadLocalMap = InternalThreadLocalMap.get(); // 2. 获取当前线程的 InternalThreadLocalMap
          setKnownNotUnset(threadLocalMap, value); // 3. 将 InternalThreadLocalMap 中数据替换为新的 value
      } else {
          remove();
      }
  }
  ```

  set() 的过程主要分为三步：

  1. 判断 value 是否为缺省值(UNSET)，如果等于缺省值，那么直接调用 remove() 方法。这里我们还不知道缺省值和 remove() 之间的联系是什么，我们暂且把 remove() 放在最后分析。
  1. 如果 value 不等于缺省值，接下来会获取当前线程的 InternalThreadLocalMap。
  1. 然后将 InternalThreadLocalMap 中对应数据替换为新的 value。
     

##### FastThreadLocal使用字节填充解决伪共享

FastThreadLocal也是用在多线程场景，所以FastThreadLocal需要解决伪共享问题，FastThreadLocal使用字节填充解决伪共享，详情请移步[java伪共享](https://codingroam.github.io/post/java%E4%BC%AA%E5%85%B1%E4%BA%AB/)

![image-20221214213916504](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221214213916504.png)

> 缓存行
> Cache是由很多个cache line组成的。每个cache line通常是64字节，并且它有效地引用主内存中的一块儿地址。一个Java的long类型变量是8字节，因此在一个缓存行中可以存8个long类型的变量。
> CPU每次从主存中拉取数据时，会把相邻的数据也存入同一个cache line。
> 在访问一个long数组的时候，如果数组中的一个值被加载到缓存中，它会自动加载另外7个。因此你能非常快的遍历这个数组。事实上，你可以非常快速的遍历在连续内存块中分配的任意数据结构。
>
> 伪共享
> 由于多个线程同时操作同一缓存行的不同变量，但是这些变量之间却没有啥关联，但是每次修改，都会导致缓存的数据变成无效，从而明明没有任何修改的内容，还是需要去主存中读（CPU读取主存中的数据会比从L1中读取慢了近2个数量级）但是其实这块内容并没有任何变化，由于缓存的最小单位是一个缓存行，这就是伪共享。
>
> 如果让多线程频繁操作的并且没有关系的变量在不同的缓存行中，那么就不会因为缓存行的问题导致没有关系的变量的修改去影响另外没有修改的变量去读主存了（那么从L1中取是从主存取快2个数量级的）那么性能就会好很多很多。



#####  FastThreadLocal 的一些思考

+ FastThreadLocal 真的一定比 ThreadLocal 快吗？答案是不一定的，只有使用FastThreadLocalThread 类型的线程才会更快，如果是普通线程反而会更慢。

+ FastThreadLocal 会浪费很大的空间吗？虽然 FastThreadLocal 采用的空间换时间的思路，但是在 FastThreadLocal 设计之初就认为不会存在特别多的 FastThreadLocal 对象，而且在数据中没有使用的元素只是存放了同一个缺省对象的引用，并不会占用太多内存空间。

  

##### FastThreadLocal vs ThreadLocal

1. 高效查找。FastThreadLocal 在定位数据的时候可以直接根据数组下标 index 获取，时间复杂度 O(1)。而 JDK 原生的 ThreadLocal 在数据较多时哈希表很容易发生 Hash 冲突，线性探测法在解决 Hash 冲突时需要不停地向下寻找，效率较低。此外，FastThreadLocal 相比 ThreadLocal 数据扩容更加简单高效，FastThreadLocal 以 index 为基准向上取整到 2 的次幂作为扩容后容量，然后把原数据拷贝到新数组。而 ThreadLocal 由于采用的哈希表，所以在扩容后需要再做一轮 rehash。
1. 安全性更高。JDK 原生的 ThreadLocal 使用不当可能造成内存泄漏，只能等待线程销毁。在使用线程池的场景下，ThreadLocal 只能通过主动检测的方式防止内存泄漏，从而造成了一定的开销。然而 FastThreadLocal 不仅提供了 remove() 主动清除对象的方法，而且在线程池场景中 Netty 还封装了 FastThreadLocalRunnable，FastThreadLocalRunnable 最后会执行 FastThreadLocal.removeAll() 将 Set 集合中所有 FastThreadLocal 对象都清理掉
   


---

> 作者: wang  
> URL: https://codingroam.github.io/post/threadlocal%E4%B8%8Efastthreadlocal/  

