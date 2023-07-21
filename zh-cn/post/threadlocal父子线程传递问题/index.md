# ThreadLocal父子线程传递问题


ThreadLocal父子线程传递问题

<!--more-->

### ThreadLocal父子线程传递问题



#### 1、父子线程传递问题

ThreadLocal是线程上下文，如果主线程开启子线程，还可以顺利获得本地变量吗？答案是否定的，以下是实验过程

```java
 public class TTLTest{
    public static void main(String[] args) {
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();

        threadLocal.set("threadLocal-value");
        inheritableThreadLocal.set("inheritableThreadLocal-value");

        // 验证父子线程传递
        new Thread(() -> {
            System.out.println(threadLocal.get()); // null
            System.out.println(inheritableThreadLocal.get()); //inheritableThreadLocal-value
        }).start();

        threadLocal.remove();
        inheritableThreadLocal.remove();
    }
}      

```

可以看到，开启的子线程是无法获得父线程的本地变量的，所以java引入了`InheritableThreadLocal`,在子线程初始化时，会将父线程的本地变量传递到子线程

#### 2、线程池环境的本地变量传递问题

在大多数实际项目中，为了节省线程开启关闭的开销，常常使用线程池来提高线程的复用，线程复用的同时，对本地变量的传递带来了新的影响，上文提到InheritableThreadLocal实现父子线程变量传递是在子线程初始化过程中，而池化的线程是不会重新初始化的，所以InheritableThreadLocal不能在线程池开辟的子线程中传递，或者说，只会在子线程初始化时传递一次，而后在线程池中未被销毁之前，无法再次接受父线程的变量传递。

```java
public class TTLTest{
    public static void main(String[] args) {
        InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();
        // 验证线程池传递
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        for (int i = 0; i < 10; i++) {
            inheritableThreadLocal.set("inheritableThreadLocal-value->" + i);
            executorService.submit(() -> {
                System.out.println(inheritableThreadLocal.get());
            });
        }
        executorService.shutdown();
        inheritableThreadLocal.remove();
    }
}
结果：
inheritableThreadLocal-value->0
inheritableThreadLocal-value->0
inheritableThreadLocal-value->0
inheritableThreadLocal-value->0
...

```

可以看出子线程只保留了最初获得的本地变量
解决方案是阿里巴巴开源框架TTL: transmittable-thread-local

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>transmittable-thread-local</artifactId>
</dependency>

```

```java
public class TTLTest {
    public static void main(String[] args) {
        InheritableThreadLocal<String> inheritableThreadLocal = new InheritableThreadLocal<>();
        TransmittableThreadLocal<String> transmittableThreadLocal = new TransmittableThreadLocal<>();
        // 验证线程池传递2
        ExecutorService executorService2 = TtlExecutors.getTtlExecutorService(Executors.newFixedThreadPool(1));
        for (int i = 0; i < 10; i++) {
            inheritableThreadLocal.set("inheritableThreadLocal-value->" + i);
            transmittableThreadLocal.set("transmittableThreadLocal-value->" + i);
            executorService2.submit(() -> {
                System.out.println(inheritableThreadLocal.get());
                System.out.println(transmittableThreadLocal.get());
            });
        }
        executorService.shutdown();
        inheritableThreadLocal.remove();
        transmittableThreadLocal.remove();
    }
}

结果：
inheritableThreadLocal-value->0
transmittableThreadLocal-value->1
inheritableThreadLocal-value->0
transmittableThreadLocal-value->2
inheritableThreadLocal-value->0
transmittableThreadLocal-value->3
inheritableThreadLocal-value->0
transmittableThreadLocal-value->4
...

```

注意这句代码`TtlExecutors.getTtlExecutorService` 通过对线程池进行包装，从而实现TTL所具备的功能，使用阿里开源的TransmittableThreadLocal 优雅的实现父子线程的数据传递。

---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/threadlocal%E7%88%B6%E5%AD%90%E7%BA%BF%E7%A8%8B%E4%BC%A0%E9%80%92%E9%97%AE%E9%A2%98/  

