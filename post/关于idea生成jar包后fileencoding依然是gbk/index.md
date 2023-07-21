# 关于IDEA生成jar包后FileEncoding依然是GBK

关于IDEA生成jar包后FileEncoding依然是GBK
<!--more-->


## 在启动类增加 Properties类

```java
@SpringBootApplication
public class TokenApplication {
    public static void main(String[] args) {
        Properties properties = System.getProperties();
        properties.forEach((key,value)->{
            System.out.println("key = " + key);
            System.out.println("value = " + value);
        });
        SpringApplication.run(TokenApplication.class,args);
    }
}
```

### 打印在控制台我们可以看到

### file.encoding 的值 为UTF-8 ，说明我们的idea的file.encoding 是为UTF-8

## 如若不是可修改：


![img](https://img-blog.csdnimg.cn/fcbb7addfd264ddfbb9cc67490d25783.png)



### 然后我我们使用maven 将其项目打成jar包，使用java-jar 运行jar包


![img](https://img-blog.csdnimg.cn/c2fcefd8ac724f369075ad668c42c8c9.png)

### 我们看到 其值变为了GBK，在普通的运行当中我认为是不会出错，因今天我们的项目使用jar包运行，在调用python算法时，报utf-8 codec can't decode byte 此错误，是因为他转为GBK（在idea运行，调用算法没有报错）


## 3.1 （第一种方法，较麻烦）换用运行命令

### 我们之前使用的是java -jar 我们在他后边加上 -Dfile.encoding=utf-8

### 新命令： java -jar -Dfile.encoding=utf-8 jar包 运行


![img](https://img-blog.csdnimg.cn/e2f3f2c1185d443e9520edcefd51fcbf.png)

### 我们看到打印的结果为


![img](https://img-blog.csdnimg.cn/096bc14cbc4349219247a0a81277964f.png)

### 其已经修改， 这时候他就已经是UTF-8了

### 这样我们就解决了其file.encoding 为GBK 的问题，但是我们不能总是java -jar 在加上他

### 我们可以写一个cmd脚本来运行此命令，但同样我们可以使用第二种更为简单的方法

## 3.2（第二种方法，简单）配置环境变量

### 同样我们可以再电脑的环境变量中添加


![img](https://img-blog.csdnimg.cn/84efff3acce34cc2a1c1202c5f5aa29e.png)



### key：JAVA_TOOL_OPTIONS value： -Dfile.encoding=utf-8


![img](https://img-blog.csdnimg.cn/6c5d6632d7d4465a9b12b3a91fc3947c.png)

### 添加之后我们保存 ，这时候我们再泳普通的 java -jar 来运行这个jar包


![img](https://img-blog.csdnimg.cn/f8115f5449e5469199ef881ffb5ec834.png)


![img](https://img-blog.csdnimg.cn/aeb66b6039044a479730abcf4a4bbad6.png)&nbsp;

### 我们看到他已经变成了 utf-8 ，这时候我们的项目在调用python算法时就不会报错了，到这里也就解决了 无论如何在idea中设置jar包的file.encoding 为GBK的问题。

### 作者：如果问题，请评论，因是第一次这么解决，不知会不会造成其他影响，如有其他更好建议，欢迎评论。多多指教。



---

> 作者: wang  
> URL: https://codingroam.github.io/post/%E5%85%B3%E4%BA%8Eidea%E7%94%9F%E6%88%90jar%E5%8C%85%E5%90%8Efileencoding%E4%BE%9D%E7%84%B6%E6%98%AFgbk/  

