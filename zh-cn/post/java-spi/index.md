# Java SPI

SPI之ServiceLoader应用与源码分析
<!--more-->


  在SPI的应用中，Java支持对外开放了一些服务接口（或者抽象类，也可以使用普通类，但是不建议），但是不提供具体的实现，这些实现可以由第三方提供，支持用户以扩展的方式集成到系统中。那么这就需要一种**服务发现机制**。也就是根据一定的规则到指定的位置找到指定服务的配置文件，然后找到实现类。ServiceLoader便提供了这么一个手段，能够在系统中"指定"位置（**META-INF/services**）的"指定"文件（文件名是**服务全限定名称**）中寻找指定服务的三方实现。


  比如在servlet容器中，需要支持servlet规范中对于ServletContainerInitializer的使用定义，会在容器启动的时候寻找ServletContainerInitializer的实现类，调用其onStartup方法。那么可以这样寻找其实现类：

```java
ServiceLoader<ServletContainerInitializer> loadedInitializers = ServiceLoader.load(ServletContainerInitializer.class);
for (ServletContainerInitializer sci:loadedInitializers){
    
	//do something......
}
```


  ServiceLoader的使用方式一般就是先通过load方法得到ServiceLoader实例，然后迭代获取每个实现。我们知道对于增强的for()循环，在编译之后会转变成对应迭代器遍历的字节码实现，所以ServiceLoader需要实现迭代器Iterable接口，事实上也是这样：

```java
public final class ServiceLoader<S> implements Iterable<S>{
   
	......
}
```

  接下来我们看看load方法是如何实现的：

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
   
 		//使用TCCL类加载器
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
```

  SPI使用的是TCCL类加载器，关于TCCL，在 [深入OpenJDK源码全面理解Java类加载器](https://blog.csdn.net/huangzhilin2015/article/details/115013668)进行过介绍，这里不再多聊。之所以使用TCCL是因为ServiceLoader定义在核心包中，会被BootStrapClassLoader加载，但是服务实现者却定义在应用jar包中，BootStrapClassLoader无法加载，那根据Java类加载的双亲委派机制，拿这些实现类还没有办法，所以需要通过TCCL的手段实现父类加载器调用子类加载器加载类的需求。   获取了TCCL之后，我们进入ServiceLoader.load(service, cl)方法调用逻辑：

```java
public static <S> ServiceLoader<S> load(Class<S> service, ClassLoader loader){
   
        return new ServiceLoader<>(service, loader);
    }
```

  直接创建的ServiceLoader实例，那么转到ServiceLoader的构造函数：

```java
private ServiceLoader(Class<S> svc, ClassLoader cl) {
   
		//判空
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        //如果cl为空，那么从getSystemClassLoader获取，这个方法默认返回AppClassLoader
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        //安全检查
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }
```

  经过简单的处理之后，会调用reload方法完成服务实现者的发现与加载动作(关于ClassLoader.getSystemClassLoader，在 [深入OpenJDK源码全面理解Java类加载器](https://blog.csdn.net/huangzhilin2015/article/details/114991932)中有详细分析)，reload方法实现如下：

```java
public void reload() {
   
		//清理providers，以便于重新加载服务提供者
        providers.clear();
        //创建一个LazyIterator
        lookupIterator = new LazyIterator(service, loader);
    }
```

  主要根据service（服务）和loader（类加载器）创建了一个**LazyIterator**，这个LazyIterator看名字就知道是一个懒加载的迭代器，是ServiceLoader的一个私有内部类。前文提到了ServiceLoader实现了Iterable接口，那么我们看看它的iterator();方法是如何实现的：

```java
public Iterator<S> iterator() {
   
        return new Iterator<S>() {
   
			//providers在reload的时候会清空
            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
   
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
   
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
   
                throw new UnsupportedOperationException();
            }

        };
    }
```

  可以看到，reload之后调用hasNext和next方法实际调用的是lookupIterator，也就是reload方法创建的LazyIterator。所以我们转到LazyIterator对应方法的实现（其中的next方法会调用nextService方法，hasNext方法会调用hasNextService方法，所以这里关注nextService和hasNextService方法就可以了）：

```java
private class LazyIterator implements Iterator<S>{
   
	Class<S> service;
	ClassLoader loader;
	Enumeration<URL> configs = null;
	Iterator<String> pending = null;
	String nextName = null;
	
	private boolean hasNextService() {
   
            if (nextName != null) {
   
                return true;
            }
            if (configs == null) {
   
                try {
   
                	//PREFIX是"META-INF/services/"
                    String fullName = PREFIX + service.getName();
                    //调用的ClassLoader.getResources方法寻找资源
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
   
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
   
                if (!configs.hasMoreElements()) {
   
                    return false;
                }
                //解析找到的资源（URL）
                pending = parse(service, configs.nextElement());
            }
            //保存到nextName，在nextService方法中加载与实例化
            nextName = pending.next();
            return true;
        }

        private S nextService() {
   
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
   
            	//加载找到的服务实现
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
   
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
   
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
   
            	//实例化
                S p = service.cast(c.newInstance());
                //存入providers中
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
   
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }
}
```

  代码看起来不少，不过核心逻辑就几点：首先在hasNextService方法中根据PREFIX 和serviceName组装一个fullName，对于ServletContainerInitializer的例子，fullName就是：META-INF/services/javax.servlet.ServletContainerInitializer。然后通过ClassLoader提供的getResources方法寻找实现类，最后在nextService方法中完成实现类的加载与实例化操作，还会存入providers中。   注意ClassLoader的getResources方法返回的是一个URL的枚举类（Enumeration，现在基本都使用迭代器了），会在parse方法中解析该URL：

```java
private Iterator<String> parse(Class<?> service, URL u)
        throws ServiceConfigurationError
    {
   
        InputStream in = null;
        BufferedReader r = null;
        ArrayList<String> names = new ArrayList<>();
        try {
   
            in = u.openStream();
            r = new BufferedReader(new InputStreamReader(in, "utf-8"));
            int lc = 1;
            while ((lc = parseLine(service, u, r, lc, names)) >= 0);
        } catch (IOException x) {
   
            fail(service, "Error reading configuration file", x);
        } finally {
   
            try {
   
                if (r != null) r.close();
                if (in != null) in.close();
            } catch (IOException y) {
   
                fail(service, "Error closing configuration file", y);
            }
        }
        return names.iterator();
    }
```

  这个方法逻辑很简单，就不进行说明了，需要注意的是，一个META-INF/services/{serviceName}文件中可以有多个实现类，parse方法会将解析到的全限定类名都存入names集合中，然后返回其迭代器，交给后面的逻辑迭代加载和实例化。   这里还需要提一点的就是ClassLoader的getResources方法，该方法里会调用findResources方法，这个方法在ClassLoader中返回的是一个空集合：

```java
protected Enumeration<URL> findResources(String name) throws IOException {
   
        return java.util.Collections.emptyEnumeration();
    }
```

  所以需要子类去实现，ExtClassLoader和AppClassLoader都继承了URLClassLoader，在URLClassLoader中有该方法的具体实现，最终是通过URLClassPath完成的资源查找（支持网络字节码和本地文件字节码），最终返回URL枚举类。


  ServiceLoader主要依赖一个懒加载迭代器LazyIterator，LazyIterator生成后会在对ServiceLoader进行迭代的时候才进行资源的搜索和解析工作，工作流程总体说来如下：

1. 根据传入的服务名生成该服务按照规范应该所在的文件名，比如：META-INF/services/javax.servlet.ServletContainerInitializer 
2. 使用指定的TCCL类加载器(如果为空则是AppClassLoader)寻找对应的资源列表，以枚举(Enumeration)的方式返回 
3. 然后遍历找到的资源文件，解析内容得到所有配置的服务实现类 
4. 对这些实现类完成加载和实例化的工作，返回实现类实例对象，同时存入ServiceLoader的providers属性中（一个LinkedHashMap，key为name，value为对应的实例对象）



---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/java-spi/  

