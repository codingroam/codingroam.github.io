# ThreadLocal详解


ThreadLocal详解

<!--more-->

### ThreadLocal详解

```
1 ThreadLocal有什么缺陷？为什么会导致内存泄漏？
2 Entry的key为什么要用弱引用? 四种引用有什么区别说一下？为什么使用弱引用就可以解决内存泄漏问题? 
3 ThreadLocalMap的Key是什么？ 
4 ThreadLocalMap如何解决value冲突问题?跟HashMap有什么区别？
5 进阶：ThreadLocal能否做到子线程中共享父线程中的数据?InheritableThreadLocal了解过吗？有什么局限性？
6 进阶：FastThreadLocal有了解过吗？
```

#### 1、ThreadLocal简介

> 该类提供线程局部变量。这些变量与普通变量的不同之处在于，每个访问一个变量(通过其get或set方法)的线程都有自己的、独立初始化的变量副本。ThreadLocal实例通常是类中的私有静态字段，希望将状态与线程关联(例如，用户ID或事务ID)。

​		ThreadLocal叫做***线程变量***，意思是ThreadLocal中填充的变量属于当前线程，该变量对其他线程而言是隔离的，也就是说该变量是当前线程独有的变量。ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。

通俗的描述一下threadlocal的存取set()/get()方法大体逻辑如下：

1. set方法：threadlocal在set数据时，首先调用Thread.currentThread()获取当前操作线程，拿到线程之后获取线程的ThreadLocalMap变量(细节还包括首次获取时为null，为当前线程初始化ThreadLocalMap并返回)，然后向ThreadLocalMap中put数据，key是this(threadlocal)，value即为当前线程要保存的值。

2. get方法：逻辑与set类似，先获取当前线程，拿到线程的ThreadLocalMap，调用ThreadLocalMap的get方法，key即为this(当前threadlocal实例对象)，返回当前线程保存的值。

总的来说，ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景

图示：

![image-20221213005720901](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221213005720901.png)

#### 2、ThreadLocal简单使用

```java
public class ThreadLocaDemo {
 
    private static ThreadLocal<String> localVar = new ThreadLocal<String>();
 
    static void print(String str) {
        //打印当前线程中本地内存中本地变量的值
        System.out.println(str + " :" + localVar.get());
        //清除本地内存中的本地变量
        localVar.remove();
    }
    public static void main(String[] args) throws InterruptedException {
 
        new Thread(new Runnable() {
            public void run() {
                ThreadLocaDemo.localVar.set("local_A");
                print("A");
                //打印本地变量
                System.out.println("after remove : " + localVar.get());
               
            }
        },"A").start();
 
        Thread.sleep(1000);
 
        new Thread(new Runnable() {
            public void run() {
                ThreadLocaDemo.localVar.set("local_B");
                print("B");
                System.out.println("after remove : " + localVar.get());
              
            }
        },"B").start();
    }
}
 
A :local_A
after remove : null
B :local_B
after remove : null
 
```

#### 3、ThreadLocal源码原理

1.  **ThreadLocal的set()方法：**

   ```java
    public void set(T value) {
           //1、获取当前线程
           Thread t = Thread.currentThread();
           //2、获取线程中的属性 threadLocalMap ,如果threadLocalMap 不为空，
           //则直接更新要保存的变量值，否则创建threadLocalMap，并赋值
           ThreadLocalMap map = getMap(t);
           if (map != null)
               map.set(this, value);
           else
               // 初始化thradLocalMap 并赋值
               createMap(t, value);
       }
   ```

    从上面的代码可以看出，ThreadLocal  set赋值的时候首先会获取当前线程thread,并获取thread线程中的ThreadLocalMap属性。如果map属性不为空，则直接更新value值，如果map为空，则实例化threadLocalMap,并将value值初始化。

   那么ThreadLocalMap又是什么呢，还有createMap又是怎么做的

   ```java
     static class ThreadLocalMap {
    
           /**
            * The entries in this hash map extend WeakReference, using
            * its main ref field as the key (which is always a
            * ThreadLocal object).  Note that null keys (i.e. entry.get()
            * == null) mean that the key is no longer referenced, so the
            * entry can be expunged from table.  Such entries are referred to
            * as "stale entries" in the code that follows.
            */
           static class Entry extends WeakReference<ThreadLocal<?>> {
               /** The value associated with this ThreadLocal. */
               Object value;
    
               Entry(ThreadLocal<?> k, Object v) {
                   super(k);
                   value = v;
               }
           }
    
           
       }
   ```

   可看出ThreadLocalMap是ThreadLocal的内部静态类，而它的构成主要是用Entry来保存数据 ，而且还是继承的弱引用。在Entry内部使用ThreadLocal作为key，使用我们设置的value作为value。

   ```java
   //这个是threadlocal 的内部方法
   void createMap(Thread t, T firstValue) {
           t.threadLocals = new ThreadLocalMap(this, firstValue);
       }
   
       //ThreadLocalMap 构造方法
   ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
               table = new Entry[INITIAL_CAPACITY];
               int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
               table[i] = new Entry(firstKey, firstValue);
               size = 1;
               setThreshold(INITIAL_CAPACITY);
           }
   ```

   

2.  **ThreadLocal的get方法**

   ```java
       public T get() {
           //1、获取当前线程
           Thread t = Thread.currentThread();
           //2、获取当前线程的ThreadLocalMap
           ThreadLocalMap map = getMap(t);
           //3、如果map数据不为空，
           if (map != null) {
               //3.1、获取threalLocalMap中存储的值
               ThreadLocalMap.Entry e = map.getEntry(this);
               if (e != null) {
                   @SuppressWarnings("unchecked")
                   T result = (T)e.value;
                   return result;
               }
           }
           //如果是数据为null，则初始化，初始化的结果，TheralLocalMap中存放key值为threadLocal，值为null
           return setInitialValue();
       }
    
    
   private T setInitialValue() {
           T value = initialValue();
           Thread t = Thread.currentThread();
           ThreadLocalMap map = getMap(t);
           if (map != null)
               map.set(this, value);
           else
               createMap(t, value);
           return value;
       }
   ```

   

3. **ThreadLocal的remove方法**

   ```java
    public void remove() {
            ThreadLocalMap m = getMap(Thread.currentThread());
            if (m != null)
                m.remove(this);
        }
   ```

    remove方法，直接将ThrealLocal 对应的值从当前相差Thread中的ThreadLocalMap中删除。为什么要删除，这涉及到内存泄露的问题。

   实际上 ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

   所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

   ThreadLocal其实是与线程绑定的一个变量，如此就会出现一个问题：如果没有将ThreadLocal内的变量删除（remove）或替换，它的生命周期将会与线程共存。通常线程池中对线程管理都是采用线程复用的方法，在线程池中线程很难结束甚至于永远不会结束，这将意味着线程持续的时间将不可预测，甚至与JVM的生命周期一致。举个例字，如果ThreadLocal中直接或间接包装了集合类或复杂对象，每次在同一个ThreadLocal中取出对象后，再对内容做操作，那么内部的集合类和复杂对象所占用的空间可能会开始持续膨胀。
   
4. **ThreadLocal与Thread，ThreadLocalMap之间的关系**

   ![image-20221213214510563](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221213214510563.png)



#### 4、ThreadLocal 常见使用场景

ThreadLocal 适用于如下两种场景：

1. **每个线程需要有自己单独的实例**

2. **实例需要在多个方法中共享，但不希望被多线程共享**

对于第一点，每个线程拥有自己实例，实现它的方式很多。例如可以在线程内部构建一个单独的实例。ThreadLoca 可以以非常方便的形式满足该需求。

对于第二点，可以在满足第一点（每个线程有自己的实例）的条件下，通过方法间引用传递的形式实现。ThreadLocal 使得代码耦合度更低，且实现更优雅。

场景

1）存储用户Session

一个简单的用ThreadLocal来存储Session的例子：

```java
private static final ThreadLocal threadSession = new ThreadLocal();
 
    public static Session getSession() throws InfrastructureException {
        Session s = (Session) threadSession.get();
        try {
            if (s == null) {
                s = getSessionFactory().openSession();
                threadSession.set(s);
            }
        } catch (HibernateException ex) {
            throw new InfrastructureException(ex);
        }
        return s;
    }
```

场景二、数据库连接，处理数据库事务

场景三、数据跨层传递（controller,service, dao）

      每个线程内需要保存类似于全局变量的信息（例如在拦截器中获取的用户信息），可以让不同方法直接使用，避免参数传递的麻烦却不想被多线程共享（因为不同线程获取到的用户信息不一样）。

例如，用 ThreadLocal 保存一些业务内容（用户权限信息、从用户系统获取到的用户名、用户ID 等），这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的。

在线程生命周期内，都通过这个静态 ThreadLocal 实例的 get() 方法取得自己 set 过的那个对象，避免了将这个对象（如 user 对象）作为参数传递的麻烦。

比如说我们是一个用户系统，那么当一个请求进来的时候，一个线程会负责执行这个请求，然后这个请求就会依次调用service-1()、service-2()、service-3()、service-4()，这4个方法可能是分布在不同的类中的。这个例子和存储session有些像。

```java
package com.kong.threadlocal;
 
 
public class ThreadLocalDemo05 {
    public static void main(String[] args) {
        User user = new User("jack");
        new Service1().service1(user);
    }
 
}
 
class Service1 {
    public void service1(User user){
        //给ThreadLocal赋值，后续的服务直接通过ThreadLocal获取就行了。
        UserContextHolder.holder.set(user);
        new Service2().service2();
    }
}
 
class Service2 {
    public void service2(){
        User user = UserContextHolder.holder.get();
        System.out.println("service2拿到的用户:"+user.name);
        new Service3().service3();
    }
}
 
class Service3 {
    public void service3(){
        User user = UserContextHolder.holder.get();
        System.out.println("service3拿到的用户:"+user.name);
        //在整个流程执行完毕后，一定要执行remove
        UserContextHolder.holder.remove();
    }
}
 
class UserContextHolder {
    //创建ThreadLocal保存User对象
    public static ThreadLocal<User> holder = new ThreadLocal<>();
}
 
class User {
    String name;
    public User(String name){
        this.name = name;
    }
}
 
执行的结果：
 
service2拿到的用户:jack
service3拿到的用户:jack
```

场景四、Spring使用ThreadLocal解决线程安全问题 

我们知道在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。就是因为Spring对一些Bean（如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等）中非线程安全的“状态性对象”采用ThreadLocal进行封装，让它们也成为线程安全的“状态性对象”，因此有状态的Bean就能够以singleton的方式在多线程中正常工作了。 

一般的Web应用划分为展现层、服务层和持久层三个层次，在不同的层中编写对应的逻辑，下层通过接口向上层开放功能调用。在一般情况下，从接收请求到返回响应所经过的所有程序调用都同属于一个线程，如图9-2所示。
![image-20221213215038034](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221213215038034.png)

这样用户就可以根据需要，将一些非线程安全的变量以ThreadLocal存放，在同一次请求响应的调用线程中，所有对象所访问的同一ThreadLocal变量都是当前线程所绑定的。

#### 5、ThreadLocal内存泄漏

ThreadLocalMap实际上是以Entry作为数据存储载体，Entry将ThreadLocal作为Key，值作为value保存，它继承自WeakReference，注意构造函数里的第一行代码super(k)，这意味着它的Key(ThreadLocal对象)是一个「弱引用」

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

主要两个原因
1 . 没有手动删除这个 Entry
2 . CurrentThread 当前线程依然运行

```
第一点很好理解，只要在使用完下 ThreadLocal ，调用其 remove 方法删除对应的 Entry(找到对应的entry并将entry=null) ，就能避免内存泄漏。
第二点稍微复杂一点，由于ThreadLocalMap 是 Thread 的一个属性，被当前线程所引用，所以ThreadLocalMap的生命周期跟 Thread 一样长。如果threadlocal变量被回收，那么当前线程的threadlocal 变量副本指向的就是key=null, 也即entry(null,value),那这个entry对应的value永远无法访问到。实际私用ThreadLocal场景都是采用线程池，而线程池中的线程都是复用的，这样就可能导致非常多的entry(null,value)出现，从而导致内存泄露。
```
综上， ThreadLocal 内存泄漏的根源是：
  	 	由于ThreadLocalMap 的生命周期跟 Thread 一样长，对于重复利用的线程来说，如果没有手动删除（remove()方法）对应 key 就会导致entry(null，value)的对象越来越多，从而导致内存泄漏。

 		**key为弱引用的好处**：**ThreadLocal 没有被外部强引用的情况下，key就会被回收变成null，在 ThreadLocalMap 中的set/getEntry 方法中，会对 key 为 null（也即是 ThreadLocal 为 null ）进行判断，如果为 null 的话，那么会把 value 置为 null 的．这就意味着使用threadLocal , CurrentThread 依然运行的前提下．就算忘记调用 remove 方法，弱引用比强引用可以多一层保障：弱引用的 ThreadLocal 会被回收．对应value在下一次 ThreadLocaI 调用 get()/set()/remove() 中的任一方法的时候会被清除，从而避免内存泄漏．**


#### 6、ThreadLocalMap 和 HashMap 区别

**ThreadLocalMap 和 HashMap 最大的区别在于解决 hash 冲突。**

HashMap 使用的**链地址法**，而 ThreadLocalMap 使用的**线性探测法/开放地址法**。

**当我们通过 int i = key.threadLocalHashCode & (len-1) 计算出 hash 值，如果出现冲突，顺序查看表中下一单元，直到找出一个空单元或查遍全表。**



---

> 作者: wang  
> URL: https://codingroam.github.io/post/threadlocal/  

