# 装饰者模式


装饰者模式

<!--more-->

### 装饰者模式

​		**装饰者模式的核心思想是通过创建一个装饰对象（即装饰者），动态扩展目标对象的功能，并且不会改变目标对象的结构，提供了一种比继承更灵活的替代方案。**

​		我们在进行软件开发时要想实现可维护、可扩展，就需要尽量复用代码，并且降低代码的耦合度，而设计模式就是一种可以提高代码可复用性、可维护性、可扩展性以及可读性的解决方案。

大家熟知的23种设计模式，可以分为创建型模式、结构型模式和行为型模式三大类。其中，结构型模式用于设计类或对象的组合方式，以便实现更加灵活的结构。结构型模式又可划分为类结构型模式和对象结构型模式，前者通过继承来组合接口或类，后者通过组合或聚合来组合对象

![设计模式](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.jpeg)

装饰者模式的核心思想是通过创建一个装饰对象（即装饰者），动态扩展目标对象的功能，并且不会改变目标对象的结构，提供了一种比继承更灵活的替代方案。需要注意的是，装饰对象要与目标对象实现相同的接口，或继承相同的抽象类；另外装饰对象需要持有目标对象的引用作为成员变量，而具体的赋能任务往往通过带参构造方法来完成。

#### **▐** 结构

装饰者模式包含四种类，分别是抽象构件类、具体构件类、抽象装饰者类、具体装饰者类，它们各自负责完成特定任务，并且相互之间存在紧密联系。

![image-20221226223328253](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221226223328253.png)

![image-20221226223407555](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221226223407555.png)

▐  使用
有了上述的基本概念，我们将装饰者模式的使用步骤概括为：

step1：创建抽象构件类，定义目标对象的抽象类、将要扩展的功能定义成抽象方法；

step2：创建具体构件类，定义目标对象的实现类，实现抽象构件中声明的抽象方法；

step3：创建抽象装饰者类，维护一个指向抽象构件的引用，并传入构造函数以调用具体构件的实现方法，给具体构件增加功能；

step4：创建具体装饰者类，可以调用抽象装饰者类中定义的方法，并定义若干个新的方法，扩展目标对象的功能。



我们在淘宝上购物时，经常会遇到很多平台和商家的优惠活动：满减、聚划算站内的百亿补贴券、店铺折扣等等。那么在商品自身原价的基础上，叠加了多种优惠活动后，后台应该怎样计算最终的下单结算金额呢？下面就以这种优惠叠加结算的场景为例，简单分析装饰者模式如何使用。

```java
// 定义抽象构件：抽象商品
public interface ItemComponent {
    // 商品价格
    public double checkoutPrice();
}
 
 
// 定义具体构件：具体商品
public class ConcreteItemCompoment implements ItemComponent {
    // 原价
    @Override
    public double checkoutPrice() {
        return 200.0;
    }
}
 
 
// 定义抽象装饰者：创建传参(抽象构件)构造方法，以便给具体构件增加功能
public abstract class ItemAbsatractDecorator implements ItemComponent {
    protected ItemComponent itemComponent;
    
    public ItemAbsatractDecorator(ItemComponent myItem) {
        this.itemComponent = myItem;
    }
    
    @Overrid
    public double checkoutPrice() {
        return this.itemComponent.checkoutPrice();
    }
}
 
 
// 定义具体装饰者A：增加店铺折扣八折
public class ShopDiscountDecorator extends ItemAbsatractDecorator {
    public ShopDiscountDecorator(ItemComponent myItem) {
        super(myItem);
    }
  
    @Override
    public double checkoutPrice() {
        return 0.8 * super.checkoutPrice();
    }
} 
 
 
// 定义具体装饰者B：增加满200减20功能，此处忽略判断逻辑
public class FullReductionDecorator extends ItemAbsatractDecorator {
    public FullReductionDecorator(ItemComponent myItem) {
        super(myItem);
    }
    
    @Override  
    public double checkoutPrice() {
        return super.checkoutPrice() - 20;  
    }
}
 
 
// 定义具体装饰者C：增加百亿补贴券50
public class BybtCouponDecorator extends ItemAbsatractDecorator { 
    public BybtCouponDecorator(ItemComponent myItem) {
        super(myItem);
    }
    
    @Override
    public double checkoutPrice() {
        return super.checkoutPrice() - 50;
    }
}
    
 
 
//客户端调用
public class userPayForItem() {
    public static void main(String[] args) {
        ItemCompoment item = new ConcreteItemCompoment();
        System.out.println("宝贝原价：" + item.checkoutPrice() + " 元"）; 
        item = new ShopDiscountDecorator(item);
        System.out.println("使用店铺折扣后需支付：" + item.checkoutPrice() + " 元"）;
        item = new FullReductionDecorator(item);
        System.out.println("使用满200减20后需支付：" + item.checkoutPrice() + " 元"）;                
        item = new BybtCouponDecorator(item);
        System.out.println("使用百亿补贴券后需支付：" + item.checkoutPrice() + " 元"）;      
    }
}
```

#### **▐** **结果输出**

```css
宝贝原价：200.0 元
使用店铺折扣后需支付：160.0 元
使用满200减20后需支付：140.0 元
使用百亿补贴券后需支付：90.0 元
```



#### **▐** **UML图**![image-20221226223613999](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221226223613999.png)

▐  比较分析

VS 继承
装饰者模式和继承关系都是要对目标类进行功能扩展，但装饰模式可以提供比继承更多的灵活性：继承是静态添加功能，在系统运行前就会确定下来；装饰者模式是动态添加、删除功能。

比如，一个对象需要具备 10 种功能，但客户端可能要求分阶段使用对象功能：在第一阶段只执行第 1-8 项功能，第二阶段执行第 3-10 项功能，这种场景下只需先定义好第 3-8 项功能方法。在程序运行的第一个阶段，使用具体装饰者 A 添加 1、2 功能；在第二个运行阶段，使用具体装饰者 B 添加 9、10 功能。而继承关系难以实现这种需求，它必须在编译期就定义好要使用的功能。

VS 代理模式
装饰者模式常常被拿来和代理模式比较，两者都要实现目标类的相同接口、声明一个目标对象，并且都可以在不修改目标类的前提下进行方法扩展，整体设计思路非常相似。那么两者的区别是什么呢？

首先，装饰者模式的重点在于增强目标对象功能，而代理模式的重点在于保护和隐藏目标对象。其中，装饰者模式需要客户端明确知道目标类，才能对其功能进行增强；代理模式要求客户端对目标类进行透明访问，借助代理类来完成相关控制功能（如日志记录、缓存设置等），隐藏目标类的具体信息。可见，代理类与目标类的关系往往在编译时就确定下来，而装饰者类在运行时动态构造而成。

其次，两者获取目标类的方式不同。装饰者模式是将目标对象作为参数传给构造方法，而代理模式是通过在代理类中创建目标对象的一个实例。

最后，通过上述示例可发现，装饰者模式会使用一系列具体装饰者类来增强目标对象的功能，产生了一种连续、叠加的效应；而代理模式是在代理类中一次性为目标对象添加功能。

﻿

VS 适配器模式
两者都属于包装式行为，即当一个类不能满足需求时，创建辅助类进行包装以满足变化的需求。但是装饰者模式的装饰者类和被装饰类都要实现相同接口，或者装饰类是被装饰类的子类；而适配器模式中，适配器和被适配的类可以有不同接口，并且可能会有部分接口重合。

#### **▐** **JDK源码赏析**

Java I/O标准库是装饰者模式在Java语言中非常经典的应用实例。

如下图所示，InputStream 相当于抽象构件，FilterInputStream 类似于抽象装饰者，它的四个子类等同于具体装饰者。其中，FilterInputStream 中含有被装饰类 InputStream 的引用，其具体装饰者及各自功能为：PushbackInputStream 能弹出一个字节的缓冲区，可将输入流放到回退流中；DataInputStream 与 DataOutputStream搭配使用，用来装饰其它输入流，允许应用程序以一种与机器无关的方式从底层输入流中读取基本 Java 数据类型；BufferedInputStream 使用缓冲数组提供缓冲输入流功能，在每次调用 read() 方法时优先从缓冲区读取数据，比直接从物理数据源读取数据的速度更快；LineNumberInputStream 提供输入流过滤功能，可以跟踪输入流中的行号（以回车符、换行符标记换行）。


![image-20221226223808495](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20221226223808495.png)

FilterInputStream 是所有装饰器类的抽象类，提供特殊的输入流控制。下面源码省略了 skip、available、mark、reset、markSupported 方法，这些方法也都委托给了 InputStream 类。其中， InputStream 提供装饰器类的接口，因而此类并没有对 InputStream 的功能做任何扩展，其扩展主要交给其子类来实现。

```java
public class FilterInputStream extends InputStream {
    //维护一个 InputStream 对象
    protected volatile InputStream in;  
    
    //构造方法参数需要一个 inputStream
    protected FilterInputStream(InputStream in) {     
        this.in = in;
    }
 
    //委托给 InputStream
    public int read() throws IOException {
        return in.read();                            
    }
    
    //委托给 InputStream
    public void close() throws IOException {         
        in.close();
    }
    
    .......
   
}
```

由于源码太长，这里先以 PushbackInputStream 为例，展示 FilterInputStream 的具体装饰者的底层实现，大家感兴趣的话可以自行查阅其它源码哦。PushbackInputStream 内部维护了一个 pushback buf 缓冲区，可以帮助我们试探性地读取数据流，对于不想要的数据也可以返还回去。

```java
public class PushbackInputStream extends FilterInputStream {
    //缓冲区
    protected byte[] buf;       
 
    protected int pos;
 
    private void ensureOpen() throws IOException {
        if (in == null)
            throw new IOException("Stream closed");
    }
    //构造函数可以指定返回的字节个数
    public PushbackInputStream(InputStream in, int size) {      
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("size <= 0");
        }
        //初始化缓冲区的大小
        this.buf = new byte[size];  
        //设置读取的位置
        this.pos = size;                                        
    }
    //默认回退一个
    public PushbackInputStream(InputStream in) {                
        this(in, 1);
    }
 
    public int read() throws IOException {
        //确保流存在
        ensureOpen(); 
        //如果要读取的位置在缓冲区里面
        if (pos < buf.length) { 
            //返回缓冲区中的内容
            return buf[pos++] & 0xff;                           
        }
        //否则调用超类的读函数
        return super.read();                                    
    }
 
    //读取指定的长度
    public int read(byte[] b, int off, int len) throws IOException {    
         ensureOpen();
        if (b == null) {
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        } 
        //缓冲区长度减去读取位置
        int avail = buf.length - pos;  
        //如果大于0，表明部分数据可以从缓冲区读取
        if (avail > 0) {    
            //如果要读取的长度小于可从缓冲区读取的字符
            if (len < avail) { 
                //修改可读取值为实际要读的长度
                avail = len;                    
            }
            //将buf中的数据复制到b中
            System.arraycopy(buf, pos, b, off, avail); 
            //修改pos的值
            pos += avail;
            //修改off偏移量的值
            off += avail;     
            //修改len的值 
            len -= avail;                               
        }
        //如果从缓冲区读取的数据不够
        if (len > 0) {  
            //从流中读取
            len = super.read(b, off, len);              
            if (len == -1) {
                return avail == 0 ? -1 : avail;
            }
            return avail + len;
        }
        return avail;
    }
 
    //不读字符b
    public void unread(int b) throws IOException {      
        ensureOpen();
        if (pos == 0) {
            throw new IOException("Push back buffer is full");
        }
        //实际就是修改缓冲区中的值，同时pos后退
        buf[--pos] = (byte)b;                           
    }
 
    public void unread(byte[] b, int off, int len) throws IOException {
        ensureOpen();
        if (len > pos) {
            throw new IOException("Push back buffer is full");
        }
        //修改缓冲区中的值，pos后退多个
        pos -= len;         
        System.arraycopy(b, off, buf, pos, len);
    }
 
    public void unread(byte[] b) throws IOException {
        unread(b, 0, b.length);
    }
}
```

▐  优点

提供比继承更加灵活的扩展功能，通过叠加不同的具体装饰者的方法，动态地增强目标类的功能。

装饰者和被装饰者可以独立发展，不会相互耦合，比如说我们想再加一个炒河粉只需创建一个炒河粉类继承FastFood即可，而想要增加火腿肠配料就增加一个类去继承 Garnish 抽象装饰者。

▐  缺点

使用装饰模式，可以比使用继承关系创建更少的类，使设计比较易于进行。然而，多层装饰会产生比继承更多的对象，使查错更加困难，尤其是这些对象都很相似。而且，当目标类被多次动态装饰后，程序的复杂性也会大大提升，难以维护。

▐  适用场景

﻿

继承关系不利于系统维护，甚至不能使用继承关系的场景。比如，当继承导致类爆炸时、目标类被 final 修饰时，都不宜通过创建目标类的子类来扩展功能。

要求不影响其他对象，为特定目标对象添加功能。

要求动态添加、撤销对象的功能。



▐  总结

装饰者模式也是一种比较容易理解和上手的设计模式，它可以对多个装饰者类进行花式排列组合，适应多变的用户需求。同时，装饰者模式也是符合开闭原则的，被装饰的对象和装饰者类互相独立、互不干扰。

在介绍装饰者模式的适用场景时，我们可以发现上述场景在实际工程中也比较常见，因此装饰者模式同样应用广泛。除了本文提到的 Java I/O，装饰者模式的典型应用实例还有：Spring cache 中的 TransactionAwareCacheDecorator 类、 Spring session 中的 ServletRequestWrapper 类、Mybatis 缓存中的 decorators 包等等。


---

> 作者: wang  
> URL: https://codingroam.github.io/zh-cn/post/%E8%A3%85%E9%A5%B0%E8%80%85%E6%A8%A1%E5%BC%8F/  

